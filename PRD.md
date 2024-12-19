# Project Requirements Document (PRD)

## Project Title
**ThoughtJournal**

## Overview
**Summary:**  
ThoughtJournal is a simple web application that allows authenticated, paying users to record and view their private thoughts. Users who are not authenticated or have not paid cannot access the journaling features. The frontend uses React (with Next.js for easy routing and deployment), ShadCN UI components for styling, Supabase for authentication and database management, and Stripe for payment processing.

## Objectives
1. **User Experience**:  
   - Provide a minimalistic user interface using ShadCN UI.  
   - Prompt the user to sign in if not authenticated.  
   - If authenticated but unpaid, prompt the user to purchase a subscription.  
   - If authenticated and paid, show a “What are you thinking today?” prompt. On clicking, an input field appears for the user to submit a thought. The submitted thought immediately appears in the list below.

2. **Technical Demonstration**:  
   - Show integration with Supabase for:
     - Authentication (email/password).
     - Secure data storage of user thoughts.
   - Show integration with Stripe for:
     - Subscription-based paywall (e.g., a monthly subscription).
   - Use ShadCN UI components to provide a clean and consistent frontend.

3. **Reliability and Security**:  
   - Use Supabase’s Row-Level Security (RLS) to ensure only authenticated users see their own thoughts.  
   - Ensure all sensitive credentials (Stripe and Supabase keys) are handled through environment variables and server-side logic.

## Scope
### In-Scope Features
- **Authentication (Supabase)**:
  - User sign-up/sign-in using email and password.
  - Session management via Supabase client library.
  - Redirecting unauthenticated users to a sign-in page.
  
- **Subscription Paywall (Stripe)**:
  - A single subscription product (monthly subscription) offered via Stripe Checkout.
  - Stripe integration includes:
    - A server-side endpoint to create a Checkout session.
    - A webhook endpoint to handle successful subscription events and update the user’s subscription status in Supabase.

- **Journaling Feature**:
  - “What are you thinking today?” prompt visible only to authenticated and subscribed users.
  - Click prompt to reveal a text input field (e.g., a modal or inline input).
  - Submit a thought via a POST request to an API route that inserts it into Supabase.
  - Refresh the list of thoughts after insertion to show the new thought immediately.
  - Fetch and display all existing thoughts for the authenticated user on page load.

- **UI Components (ShadCN)**:
  - A sign-in/sign-up form.
  - A “Subscribe” button for unpaid users.
  - A prompt and input field for journaling.
  - A styled list or card layout for displaying thoughts.

### Out-of-Scope Features
- Multiple subscription tiers (only one subscription tier will be offered).
- Complex styling beyond basic ShadCN default components.
- Offline capabilities.
- Automated testing (beyond manual testing).

## User Stories
1. **Unauthenticated User**:
   - Visits the home page.
   - Sees a sign-in/sign-up form (no journaling features).
   - Must sign in or sign up to proceed.

2. **Authenticated but Unpaid User**:
   - After signing in, sees a page with a “Subscribe to Unlock” button.
   - Clicking this button redirects them to Stripe Checkout.
   - Cannot see or create thoughts until subscription is active.

3. **Authenticated and Paid User**:
   - Sees “What are you thinking today?” prompt.
   - Clicks it, a text input appears.
   - User enters a thought and presses enter.
   - The thought is inserted into Supabase and displayed instantly.
   - Refreshing the page shows all previously submitted thoughts.

## Functional Requirements

1. **Authentication (Supabase)**:
   - **Sign-Up**:
     - Fields: email, password.
     - On success, the user is automatically authenticated.
   - **Sign-In**:
     - Fields: email, password.
     - On success, load the user’s subscription status from Supabase.
   - **Session Management**:
     - Use Supabase’s auth state to determine if user is logged in.
     - If not logged in, redirect to sign-in page.

2. **Payment (Stripe)**:
   - **Subscription Product**:  
     - A single monthly subscription plan created in Stripe’s dashboard (manually set up beforehand).
   - **Checkout Flow**:
     - Authenticated, unpaid user clicks “Subscribe” button.
     - Server-side Next.js API route creates a Stripe Checkout session using `stripe.checkout.sessions.create`.
     - User is redirected to Stripe’s hosted checkout page.
   - **Webhook Handling**:
     - A server-side endpoint to receive `checkout.session.completed` event.
     - On success, update `subscription_active` field in the user’s profile table in Supabase.
   - **Post-Payment State**:
     - After Stripe Checkout success, user is redirected back to the site.
     - Frontend checks user’s `subscription_active` field and displays journaling features.

3. **Journaling Feature (Supabase)**:
   - **Data Model**:
     - `profiles` table: 
       - `id` (matches `auth.users` id),
       - `subscription_active` (boolean, default false).
     - `thoughts` table:
       - `id` (primary key),
       - `user_id` (foreign key referencing `auth.users.id`),
       - `content` (text field),
       - `created_at` (timestamp).
   - **Insert Thought**:
     - Frontend calls an API route (Next.js API endpoint) that uses the Supabase service role key to insert a new record.
     - On success, UI updates immediately.
   - **Fetch Thoughts**:
     - On page load, if `subscription_active` is true, fetch all thoughts for the authenticated user.
     - Display them in chronological order.

4. **UI/UX (ShadCN UI)**:
   - Use ShadCN React components for:
     - Buttons (Sign-In, Sign-Up, Subscribe).
     - Input fields for email, password, and thought content.
     - Modal or inline expansion for the “What are you thinking today?” prompt.
   - Keep the UI minimalistic and consistent.

5. **Routing (Next.js)**:
   - `/` (home): Check auth state.  
     - If unauthenticated: show sign-in/sign-up forms.  
     - If authenticated but unpaid: show “Subscribe” button.  
     - If authenticated and paid: show journaling interface.
   - `/api/create-checkout-session`: For creating Stripe Checkout sessions.
   - `/api/webhook`: For Stripe webhook events.
   - Use Next.js page routing or App Router (decided: Next.js App Router for modern convention).

6. **Security and Access Control**:
   - Use environment variables for keys:
     - `NEXT_PUBLIC_SUPABASE_URL`
     - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
     - `SUPABASE_SERVICE_ROLE_KEY` (server-side only)
     - `STRIPE_PUBLIC_KEY`
     - `STRIPE_SECRET_KEY`
     - `STRIPE_WEBHOOK_SECRET`
   - RLS Policies on `thoughts` table:
     - Only allow `select` and `insert` where `user_id = auth.uid()`.
   - Check `subscription_active` in the frontend before showing the journaling components.
   - Handle all paywall checks server-side to avoid bypasses.

## Non-Functional Requirements

1. **Performance**:
   - The page should load quickly and the UI should be responsive.
   - Fetching user thoughts should happen efficiently (use Supabase’s native APIs).

2. **Security**:
   - No sensitive keys in client code (except the public anon key and public Stripe key).
   - Properly configured webhooks with `STRIPE_WEBHOOK_SECRET`.
   - HTTPS deployment (handled by hosting platform).

3. **Usability**:
   - Clear instructions for sign-in/sign-up.
   - Straightforward “Subscribe” call-to-action for unpaid users.
   - Intuitive journaling flow.

4. **Maintainability**:
   - Organize code in a standard Next.js project structure:
     - `/app` or `/pages` for routes
     - `/components` for ShadCN UI components
     - `/lib` for Supabase and Stripe helpers
     - `/api` for API endpoints
   - Comment code where needed.
   - Store environment variables in `.env.local`.

## Data Model Details

- **`profiles` table**:
  - `id`: UUID, references `auth.users.id`
  - `subscription_active`: boolean, default to false

- **`thoughts` table**:
  - `id`: serial or UUID primary key
  - `user_id`: UUID, references `auth.users.id`
  - `content`: text
  - `created_at`: timestamp, default now()

Row-Level Security (RLS) on `thoughts`:
```sql
-- Enable RLS
ALTER TABLE thoughts ENABLE ROW LEVEL SECURITY;

-- Policy to allow a user to insert their own thoughts
CREATE POLICY "Insert own thoughts" ON thoughts
FOR INSERT
WITH CHECK (auth.uid() = user_id);

-- Policy to allow a user to select only their own thoughts
CREATE POLICY "Select own thoughts" ON thoughts
FOR SELECT
USING (auth.uid() = user_id);
```

## Deployment and Environment

- **Hosting**:
  - Host frontend and backend (Next.js) on Vercel.
  - Supabase project hosted on Supabase’s managed platform.
  - Stripe keys from Stripe Dashboard (Test mode for development).

- **Environment Variables Setup (in `.env.local`)**:
  ```
  NEXT_PUBLIC_SUPABASE_URL=https://<your-supabase-project-ref>.supabase.co
  NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-anon-key>
  SUPABASE_SERVICE_ROLE_KEY=<your-service-role-key>
  STRIPE_PUBLIC_KEY=pk_test_...
  STRIPE_SECRET_KEY=sk_test_...
  STRIPE_WEBHOOK_SECRET=whsec_...
  ```

- **Stripe Configuration**:
  - Create a product and monthly price in Stripe Dashboard.
  - Use the price ID in the `/api/create-checkout-session` endpoint.

## User Flows

1. **Sign-Up Flow**:
   1. User visits `/`.
   2. If not authenticated, shows sign-in/sign-up form.
   3. User signs up with email and password.
   4. On success, user is logged in but `subscription_active = false`.
   5. Page shows a “Subscribe” button.

2. **Subscription Flow**:
   1. Authenticated, unpaid user clicks “Subscribe”.
   2. User is redirected to Stripe Checkout.
   3. After payment, Stripe webhook updates `subscription_active = true` for the user.
   4. User returns to `/` automatically (Stripe redirect URL).
   5. Now sees the journaling prompt.

3. **Journaling Flow**:
   1. Authenticated and paid user sees “What are you thinking today?” prompt.
   2. Clicks it, a text input appears.
   3. User types a thought and presses enter.
   4. A request is sent to `/api/insert-thought`.
   5. On success, the new thought is displayed immediately.
   6. Refreshing the page shows all previously submitted thoughts.

## Validation and Acceptance Criteria

- **Authentication**:
  - Unauthenticated user cannot see journaling prompt.
  - Authenticated user sees correct UI based on subscription status.

- **Payment**:
  - Unpaid user sees “Subscribe” button.
  - After payment, `subscription_active` is true.
  - Paid user sees journaling interface.

- **Journaling**:
  - Inserted thought appears immediately.
  - Refresh shows persisted thoughts.

- **Security**:
  - Users cannot see others’ thoughts.
  - Stripe secrets and Supabase service keys are never exposed client-side.

## Conclusion

This PRD outlines all choices and steps to build a fully integrated ThoughtJournal application using React, Next.js, Supabase, Stripe, and ShadCN UI. All major decisions have been made, leaving no ambiguity for implementation.