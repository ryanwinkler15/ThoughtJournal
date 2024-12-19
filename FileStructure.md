

# File Structure

```
/
├─ app/
│  ├─ layout.tsx
│  ├─ globals.css              // main global styles, including Tailwind base imports
│  ├─ page.tsx                 // main page logic that checks auth & subscription state
│  ├─ auth/
│  │  ├─ page.tsx              // sign-in/sign-up page
│  ├─ api/
│  │  ├─ create-checkout-session/
│  │  │  └─ route.ts           // POST route to create Stripe Checkout session
│  │  ├─ webhook/
│  │  │  └─ route.ts           // POST route for Stripe webhook events
│  │  └─ insert-thought/
│  │     └─ route.ts           // POST route to insert a new thought into Supabase
│  ├─ (authenticated)/
│  │  ├─ journal/
│  │  │  └─ page.tsx           // page that shows the journaling UI if user is paid & authenticated
│  │  // Additional authenticated routes can be added here if needed
│  
├─ components/
│  ├─ SignInForm.tsx           // UI for signing in
│  ├─ SignUpForm.tsx           // UI for signing up
│  ├─ SubscribeButton.tsx      // Button that triggers the subscription flow
│  ├─ WhatAreYouThinkingPrompt.tsx  // The initial prompt to create a new thought
│  ├─ ThoughtInput.tsx         // Input field/modal for entering a new thought
│  ├─ ThoughtList.tsx          // Displays the user's submitted thoughts
│  
├─ lib/
│  ├─ supabaseClient.ts        // Setup Supabase client instance
│  ├─ stripe.ts                // Setup Stripe server-side instance
│  ├─ auth.ts                  // Functions to check auth session, subscription status
│
├─ styles/
│  └─ shadcn/                  // Directory for ShadCN UI generated components & styles
│     // ShadCN UI components may also reside in /components as recommended by ShadCN setup
│  
├─ public/                      // Public assets, if needed
│
├─ .env.local                   // Environment variables (not committed)
├─ .gitignore
├─ package.json
├─ tsconfig.json
├─ postcss.config.js
├─ tailwind.config.js
└─ README.md
```

---

# Notes on Structure

1. **`app/layout.tsx`**:  
   Defines the root layout for all pages, includes `<html>`, `<body>`, and any global providers like Supabase auth provider if necessary.

2. **`app/page.tsx`**:  
   The main landing page. On initial load:
   - If no user session: Show a link or redirect to `/auth` page for sign-in/sign-up.
   - If authenticated but unpaid: Show the `SubscribeButton`.
   - If authenticated and paid: Redirect or link to `/journal` page to start journaling.

3. **`app/auth/page.tsx`**:  
   Displays sign-in/sign-up forms using `SignInForm.tsx` and `SignUpForm.tsx`.

4. **`app/api/create-checkout-session/route.ts`**:  
   Serverless route to create a Stripe Checkout session. Uses `stripe.ts` from `lib/` and reads user session info to associate the purchase with the user.

5. **`app/api/webhook/route.ts`**:  
   Receives Stripe webhook events. On `checkout.session.completed`, updates `subscription_active` in Supabase. Uses `supabaseClient` with a service role key server-side.

6. **`app/api/insert-thought/route.ts`**:  
   Inserts a new thought into Supabase for the authenticated user. Uses service role key server-side and checks the request for a valid session token.

7. **`app/(authenticated)/journal/page.tsx`**:  
   Accessible only if user is authenticated and `subscription_active = true`. Displays:
   - `WhatAreYouThinkingPrompt.tsx` -> shows `ThoughtInput.tsx` on click.
   - `ThoughtList.tsx` -> lists current user’s thoughts from Supabase.

8. **`components/`**:  
   Houses reusable UI components built with ShadCN UI and React.  
   - `SignInForm.tsx` / `SignUpForm.tsx`: Simple forms that call Supabase auth methods.
   - `SubscribeButton.tsx`: Triggers `fetch('/api/create-checkout-session')` and redirects to Stripe Checkout.
   - `WhatAreYouThinkingPrompt.tsx`: The clickable prompt displayed to paid users.
   - `ThoughtInput.tsx`: Text input for new thoughts, sends POST request to `/api/insert-thought`.
   - `ThoughtList.tsx`: Displays fetched thoughts, styled with ShadCN components.

9. **`lib/`**:
   - `supabaseClient.ts`: Instantiates a Supabase client using `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`.
   - `stripe.ts`: Instantiates a Stripe server-side client using `STRIPE_SECRET_KEY`.
   - `auth.ts`: Functions to get current user session, check if `subscription_active`, etc. Used in server components and routes.

10. **`styles/` and `shadcn/`**:
    - Contains Tailwind configuration and ShadCN UI-generated files.  
    - `globals.css` in `app/` imports Tailwind base, components, and utilities. ShadCN styling is integrated as per its setup instructions.

11. **`public/`**:
    - Static assets (icons, images) if needed.

12. **`package.json`, `tsconfig.json`, `postcss.config.js`, `tailwind.config.js`**:
    - Standard configuration files for dependencies, TypeScript, PostCSS, and Tailwind setup.

---

