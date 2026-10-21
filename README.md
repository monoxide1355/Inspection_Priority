[README.md](https://github.com/user-attachments/files/33782727/README.md)
# Inspection Queue: setup

Two files:
- `schema.sql` creates the table, the audit log, the rules, and live updates.
- `index.html` is the whole app (one page, no build step).

## 1. Database (5 min)
1. Open your Supabase project (existing or new) and go to **SQL Editor > New query**.
2. Paste all of `schema.sql` and click **Run**. It's safe to re-run, and it only touches its own two tables.

## 2. Logins
Only people you add can sign in.
1. **Authentication > Sign In / Providers**: turn **Allow new users to sign up** off. Keep **Email** enabled.
2. **Authentication > Users > Add user > Send invitation**: add each person (both leads, the inspector, yourself).
3. **Authentication > URL Configuration**: set **Site URL** to where the page will live (step 4), and add the same URL under **Redirect URLs**.
4. Email limits: Supabase's built-in email sender only allows a few emails per hour. If people get "rate limit" errors signing in, set up custom SMTP under **Authentication > Emails > SMTP Settings**, using the same mail account as your gage email if that works.

People stay signed in on each device, so they only use the link once per browser.

## 3. Connect the page
Open `index.html` in a text editor and fill in these two lines near the bottom:
```js
const SUPABASE_URL = "YOUR_SUPABASE_URL";
const SUPABASE_ANON_KEY = "YOUR_SUPABASE_ANON_KEY";
```
Both are in **Project Settings > API** (Project URL, and the `anon` / publishable key). The anon key is meant to be public. The database rules are what protect the data.

## 4. Host it
Any static host works. The easiest options:
- **Netlify Drop** (app.netlify.com/drop): drag the folder in and you get a URL right away.
- **Cloudflare Pages** or **GitHub Pages**
- An internal web server

It needs a real `https://` URL for the sign-in links. Opening the file straight from a shared drive won't work.

## How it behaves
- **Queue order**: expedited items by due date, then first pieces in the order submitted, then finals by due date and submission time.
- First pieces are Machine Shop only, and the database forces their due date to today (Eastern time).
- Expediting and holding both require a reason.
- The database stamps who started, completed, held, or expedited each item, so it can't be faked from the browser.
- Only the person who added a request can remove it. Everyone else uses Hold.
- Every add, status change, expedite and removal is written to `inspection_request_log`. View it in **Table Editor**.
- Screens update live, with a once-a-minute refresh as a backup.
- Completed items show for 30 days. Older ones stay in the database.

## Useful queries (SQL Editor)
```sql
-- Full history for one part
select * from inspection_request_log where part = '1187-01' order by at;

-- What got expedited this month, and by whom
select part, actor, detail, at from inspection_request_log
where action = 'expedited' and at > date_trunc('month', now()) order by at;
```
