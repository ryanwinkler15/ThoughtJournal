# Application Flow Document

## Overview

The ThoughtJournal application has three main phases of user engagement:

1. **Authentication Flow**: Establishing user identity via Supabase Auth.
2. **Subscription Flow**: Granting paid access to journaling features via Stripe.
3. **Journaling Flow**: Creating, viewing, and persisting user thoughts using Supabase as the backend.

This document describes each step in detail, from the initial visit to the site through managing thoughts.

## Key Components & Pages

- **Frontend**: React/Next.js pages and components styled with ShadCN UI.
  - Pages:
    - `/` (Home page): Displays different states depending on user auth and subscription.
    - `/api/create-checkout-session`: Serverless function to create Stripe checkout sessions.
    - `/api/webhook`: Serverless function for Stripe webhook events.
    - `/api/insert-thought`: Serverless function to insert a new thought.
  - UI Components:
    - Sign-In/Sign-Up Forms
    - Subscribe Button
    - “What are you thinking today?” prompt
    - Thought Input Field
    - Thought Display List

- **Backend (Supabase)**:
  - `auth` for user accounts
  - `profiles` table for subscription status
  - `thoughts` table for user thoughts
  - RLS policies to secure data

- **Stripe**:
  - Checkout sessions for subscription purchase.
  - Webhook events to update user subscription status.

---

## Initial Visit (Unauthenticated User)

1. **User Action**: The user visits `https://your-app-domain.com/`.
2. **System Behavior** (Client):
   - The Next.js page checks for a current user session using the Supabase client-side auth.
   - **Condition**: No session found → user is considered unauthenticated.
3. **UI**:
   - The home page displays a sign-in/sign-up form using ShadCN UI components.
   - No journaling or subscription features are shown.
   
**End State**: The user is prompted to sign in or sign up.

---

## Sign-Up Flow (Unauthenticated → Authenticated, Unpaid)

1. **User Action**: User clicks "Sign Up" on the home page and enters email + password.
2. **System Behavior**:
   - Frontend calls `supabase.auth.signUp({ email, password })`.
   - If successful, Supabase returns a user session.
   - The frontend stores this session and updates the state to authenticated.
3. **UI**:
   - After successful sign-up, the page re-renders.
   - Since the user is now authenticated, the code checks `subscription_active`.
   - Initially, `subscription_active` in `profiles` defaults to false for new users.
4. **Displayed State**:
   - Because the user is authenticated but unpaid:
     - The home page displays a “Subscribe” button instead of the journaling interface.

**End State**: The user is authenticated but not yet subscribed. They see a “Subscribe” button.

---

## Sign-In Flow (Existing User, Possibly Unpaid)

1. **User Action**: Returning user enters email + password in the sign-in form.
2. **System Behavior**:
   - Frontend calls `supabase.auth.signIn({ email, password })`.
   - On success, a session is returned.
   - The frontend checks `profiles.subscription_active` for this user.
3. **UI**:
   - If `subscription_active` is false, user sees “Subscribe” button.
   - If `subscription_active` is true (previously subscribed), user sees the journaling interface.

**End State**: Authenticated user sees either “Subscribe” button (if unpaid) or journaling features (if paid).

---

## Subscription Flow (Unpaid → Paid)

1. **User Action**: Authenticated but unpaid user clicks “Subscribe” button.
2. **System Behavior**:
   - The frontend calls `/api/create-checkout-session`.
   - **API Route (/api/create-checkout-session)**:
     - Server-side code uses `stripe.checkout.sessions.create()` with the monthly subscription product/price ID.
     - Retrieves the currently authenticated user’s ID from the Supabase session or JWT.
     - Returns the `session.url` to the frontend.
3. **UI**:
   - Frontend redirects the user’s browser to the Stripe Checkout page (hosted by Stripe).
   
**User at Stripe Checkout**:
4. **User Action**: User enters credit card info and completes payment on Stripe’s checkout page.
5. **Stripe Behavior**:
   - After successful payment, Stripe triggers a `checkout.session.completed` event and sends it to the `/api/webhook` endpoint in the backend.

**Webhook Handling**:
6. **System Behavior** (Server-side `/api/webhook`):
   - Verifies the event with the `STRIPE_WEBHOOK_SECRET`.
   - Extracts the user’s ID from metadata (attached when creating the session).
   - Updates the `profiles` table in Supabase:
     - `subscription_active = true` for that user.

**Redirection from Stripe**:
7. **Stripe Behavior**:
   - Stripe redirects the user back to the app’s success URL (e.g., `https://your-app-domain.com/`).
8. **System Behavior (Frontend)**:
   - On reload, the user is authenticated (session persists).
   - Checks Supabase `subscription_active`.
   - Now `subscription_active = true`.
9. **UI**:
   - Instead of “Subscribe” button, the user now sees the “What are you thinking today?” prompt.

**End State**: The user is authenticated and subscribed, ready to journal.

---

## Journaling Flow (Authenticated & Paid User)

1. **Initial Page Load (Paid User)**:
   - User visits `/`.
   - Auth state is checked; user is authenticated.
   - `subscription_active` is true, so journaling UI loads.
2. **System Behavior**:
   - The frontend calls `supabase.from('thoughts').select('*')` filtered by the user’s `auth.uid()`.
   - RLS ensures the user only gets their own thoughts.
   - Returns a list of thoughts (empty if none created yet).
3. **UI**:
   - Displays “What are you thinking today?” prompt.
   - Displays a list of previously entered thoughts (if any).

**Creating a New Thought**:
4. **User Action**: User clicks on “What are you thinking today?”.
   - A text input field appears (either a modal or inline input).
5. **User Action**: User types a new thought and presses enter.
6. **System Behavior (Inserting a Thought)**:
   - Frontend calls `/api/insert-thought` with the thought text.
   - `/api/insert-thought` serverless function:
     - Verifies the user’s auth (using JWT or session).
     - Uses `SUPABASE_SERVICE_ROLE_KEY` (server-side) to insert a new row in `thoughts`:
       - `user_id` = current user’s ID
       - `content` = user’s entered text
       - `created_at` = now()
7. **System Behavior (Response)**:
   - On success, `/api/insert-thought` returns the inserted thought.
8. **UI Update**:
   - Frontend receives the newly inserted thought and adds it to the local state.
   - The page now shows the new thought at the top (or bottom, depending on chosen display order).
   
**End State**: The user can repeatedly add new thoughts. Each time, the UI updates with the new thought instantly. Upon page refresh, the user’s full list of thoughts is fetched again from Supabase.

---

## Logging Out

1. **User Action**: User chooses to log out (if a logout button is provided).
2. **System Behavior**:
   - Frontend calls `supabase.auth.signOut()`.
3. **UI**:
   - State updates: no session detected.
   - User is redirected to the sign-in/sign-up page.

**End State**: User is unauthenticated and must sign in again to access thoughts or subscription features.

---

## Error Handling Scenarios

1. **Failed Sign-In**:
   - Supabase returns an error if email/password are incorrect.
   - UI displays an error message.
   - User can retry with correct credentials.

2. **Failed Subscription Payment**:
   - If user cancels at Stripe Checkout or payment fails:
     - Stripe redirects back to the app’s cancel URL.
     - `subscription_active` remains false.
     - UI still shows “Subscribe” button.

3. **Failed Thought Insertion**:
   - If `/api/insert-thought` returns an error (e.g., database error):
     - UI shows an error message.
     - User can retry submitting the thought.

---

## Summary of Flow

- **Unauthenticated Users**: See sign-in/sign-up form only.  
- **Authenticated, Unpaid Users**: See “Subscribe” button. On clicking, they start the Stripe subscription process.  
- **Authenticated, Paid Users**: Gain access to journaling features. They can add thoughts and view their thought history.

Each transition is triggered by a user action (sign-up, sign-in, subscribe, create thought) and system checks (auth state, subscription status, data insertion) and results in the UI updating to reflect the new state.

---

