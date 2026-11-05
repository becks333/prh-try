# Supabase Wallet Setup

The Supabase project is the source of truth for sign-in, GHS balances, and deposit/withdrawal requests. New accounts start with a GHS 0.00 balance. Requests remain pending until an allowlisted admin approves or rejects them.

## Configure the project

1. Create a Supabase project and open its SQL Editor.
2. Run [`supabase/schema.sql`](supabase/schema.sql).
3. In Supabase project settings, enable Email authentication. Configure the site URL and allowed redirect URLs for the URL where you will host this app. Email confirmation can remain enabled.
4. Put the project URL and the public anon/publishable key in [`supabase-config.js`](supabase-config.js). Never put a `service_role` key in this file or in browser code.
5. Host this folder on an HTTP(S) origin and open `index.html`. Supabase Auth is not intended to be deployed from `file://`.

## Add the administrator

1. Create your own account through the wallet page and confirm its email if confirmation is enabled.
2. In Supabase Authentication, find that user and copy the user UUID.
3. In the SQL Editor, run the following with your UUID:

```sql
insert into public.wallet_admins (user_id)
values ('YOUR_AUTH_USER_UUID')
on conflict (user_id) do nothing;
```

The SQL schema provides an admin allowlist and row-level security for a production Supabase admin interface. The current [`admin.html`](admin.html) is still a browser-local demo page; it does not sign in an admin or connect to the Supabase allowlist. Do not use it to manage real accounts or requests.

## Request flow

Users submit a GHS amount and optional note. The request is visible to the user and allowlisted admins. Approval credits a deposit or deducts a withdrawal in the virtual GHS ledger; rejection leaves the balance unchanged. Withdrawal requests are checked against available balance, including other pending withdrawals. Admins should independently verify any deposit evidence before approving it.

This setup changes virtual balances only; it does not initiate bank or mobile-money payments. The browser uses only the public anon key, while balance writes are restricted to the database functions in the schema.

## Demo login and password reset requests

The current PRH login and admin approval pages also include a browser-local demo flow. A password reset request from that login is stored in the browser's `localStorage` request queue, so the admin page can see it only when opened in the same browser profile. After an admin marks the request handled, the user can enter and confirm a new password on the login page; the demo updates that browser's saved account. This does not verify the user's identity or send an email, and must not be used for real accounts. For production, use Supabase Auth's password recovery flow and a server-backed request/review workflow instead of the demo login.

The browser-demo admin page includes a searchable registered-account list and a delete option. Deleting a demo account requires confirmation and permanently removes that account plus its browser-local requests and activity; it also signs out that account if it is currently logged in. The page reads the same browser-local `prhAccounts` data as the demo login, so it only shows demo accounts created in that browser profile; it is not a list of Supabase users and is not protected as a production admin directory. Do not use it to manage real accounts.
