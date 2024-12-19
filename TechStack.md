
# Tech Stack & Packages

## Overview

ThoughtJournal is a web application built using modern, popular technologies that streamline development, authentication, database operations, payment processing, and UI styling. The following sections detail the chosen stack and each package’s role in the project.

## Core Technologies

1. **Frontend Framework:** **Next.js (React)**  
   - **Why Next.js?**  
     - Server-side rendering for better performance and SEO.  
     - Simple integration with APIs via `app` or `pages/api` routes.  
     - Seamless integration with React and an established ecosystem of libraries.
   - **Version:** Latest stable (e.g., `^13.x` if using the App Router or `12.x` if not).
   
2. **Frontend Library:** **React**  
   - **Why React?**  
     - A component-based library for building interactive UIs.  
     - Large ecosystem, community support, and compatibility with ShadCN UI.
   - **Version:** Latest stable (e.g., `^18.x`).

3. **UI Components & Styling:** **ShadCN UI** (with Tailwind CSS)  
   - **Why ShadCN UI?**  
     - A collection of components built on top of Tailwind CSS and Radix UI primitives.  
     - Ensures a consistent, modern, and accessible UI without heavy customization.
   - **Packages:**  
     - `@shadcn/ui` (or instructions as per ShadCN UI setup)
     - `tailwindcss` for utility classes
     - `postcss` & `autoprefixer` for Tailwind integration
   - **Version:** Follow ShadCN UI and Tailwind CSS latest stable.

4. **Backend (as a Service):** **Supabase**  
   - **Why Supabase?**  
     - Open-source Firebase alternative providing Auth, Database (Postgres), and Storage.  
     - Quick setup, managed Postgres database, built-in Auth, and SQL capabilities.
   - **Packages:**
     - `@supabase/supabase-js` for client-side and server-side communication with the Supabase backend.
   - **Version:** Latest stable (e.g., `^2.x`).

5. **Payment Processing:** **Stripe**  
   - **Why Stripe?**  
     - Industry-standard for handling online payments, secure, easy to integrate.  
     - Supports subscriptions, webhooks, and robust APIs for a seamless user payment experience.
   - **Packages:**
     - `stripe` (Node SDK) for server-side integration.
     - `@stripe/stripe-js` for client-side actions, if needed.
   - **Version:** Latest stable (e.g., `^11.x` for `stripe` node SDK).

6. **Language:** **TypeScript**  
   - **Why TypeScript?**  
     - Static typing improves developer experience and reduces runtime errors.  
     - Well-integrated with Next.js, Supabase, and Stripe SDKs.
   - **Packages:**
     - `typescript`  
     - `@types/node` and `@types/react` for type definitions.
   - **Version:** Latest stable (e.g., `^5.x`).

## Supporting Packages

1. **Environment Variable Management:**
   - `dotenv` (if needed) to load `.env` files locally.
   - Next.js automatically loads `.env.local` files, so `dotenv` might be optional.

2. **API Routes and Middleware:**
   - No additional packages needed for Next.js API routes.  
   - Webhook verification done via the Stripe SDK’s built-in methods.

3. **Authentication & State Handling:**
   - Supabase handles auth sessions client-side using `@supabase/supabase-js`.
   - No additional complex state management library (like Redux) is required.  
   - Use React’s `useState`, `useEffect`, and potentially `useContext` for session handling.

4. **Data Fetching:**
   - Supabase client for direct queries.  
   - No need for a separate library like SWR or React Query unless desired. (Optional)

5. **RLS & Database Policies:**
   - Configured directly in the Supabase dashboard or via SQL migration scripts. No extra npm packages needed.

6. **Deployment & Hosting:**
   - **Vercel** for hosting (no package required; just the CLI if desired).
   - **Supabase** for backend (hosted service, no package needed beyond `@supabase/supabase-js`).

7. **Linting & Formatting:**
   - `eslint` and `eslint-config-next` for linting.
   - `prettier` for code formatting.
   - **Why?**  
     - Ensures code quality, consistency, and adherence to best practices.

8. **Testing (Optional):**
   - `jest` and `@testing-library/react` if tests are desired. This is out-of-scope per the PRD, but can be added.

## Example `package.json` Dependencies

```json
{
  "dependencies": {
    "@shadcn/ui": "latest",
    "@supabase/supabase-js": "^2.0.0",
    "@stripe/stripe-js": "^1.x",
    "next": "latest",
    "react": "latest",
    "react-dom": "latest",
    "stripe": "^11.x",
    "tailwindcss": "^3.x"
  },
  "devDependencies": {
    "@types/node": "^20.x",
    "@types/react": "^18.x",
    "@types/react-dom": "^18.x",
    "autoprefixer": "^10.x",
    "eslint": "^8.x",
    "eslint-config-next": "latest",
    "postcss": "^8.x",
    "prettier": "^2.x",
    "typescript": "^5.x"
  }
}
```

*(Versions are indicative. Always check for the latest stable releases.)*

## Integration Details

- **Supabase Integration**:
  - Initialize a Supabase client in a shared `lib/supabaseClient.ts` file.
  - Use the `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` for client-side operations.
  - Use the `SUPABASE_SERVICE_ROLE_KEY` in server-side Next.js API routes for privileged operations like inserting thoughts securely.

- **Stripe Integration**:
  - Use `stripe` package (Node SDK) in `/api/create-checkout-session` and `/api/webhook`.
  - Store Stripe secret keys in environment variables (`STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`).
  - Use `@stripe/stripe-js` if you need client-side functions (optional). For this flow, typically redirects to Stripe Checkout happen directly from the backend.

- **ShadCN UI & Tailwind**:
  - Follow ShadCN UI setup instructions to integrate with Tailwind.
  - Configure `tailwind.config.js` to support required ShadCN components and dark mode if needed.
  - Build UI using the provided components (Button, Input, Modal, etc.).

