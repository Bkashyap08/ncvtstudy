# ITI Exam Guru — ITI CBT Preparation Portal

A mobile-first ITI study portal starter with Hindi UI, trade categories, PDF study material, admin login/upload, MCQ question entry, CBT timer, answer palette, result/review screen, and Supabase backend.

> **Important:** This is a runnable starter project. You must connect your own Supabase project and run the SQL setup before real PDF uploads or cloud data work. Before using it for a high-stakes exam, harden CBT scoring as noted below.

## Features included

- Responsive React + TypeScript frontend
- ITI trade category cards
- Study-material search/filter
- Admin login through Supabase Auth
- Admin-only PDF upload to Supabase Storage
- Metadata saved in PostgreSQL
- PDF opens through a 5-minute signed URL
- Add MCQs from Admin Panel
- Create mock test records from Admin Panel
- CBT question palette, timer, answer selection, submit and result review
- Demo UI and sample local test questions if Supabase isn't configured

## 1. Requirements

- Node.js 20+ (18+ may also work)
- A Supabase account
- Terminal/command prompt

## 2. Run locally

1. Extract the ZIP.
2. Open a terminal inside the `iti-exam-guru` folder.
3. Install packages:

   ```bash
   npm install
   ```

4. Copy `.env.example` to `.env` and fill in the Supabase Project URL and anon/publishable key:

   ```env
   VITE_SUPABASE_URL=https://YOUR_PROJECT_ID.supabase.co
   VITE_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_OR_PUBLISHABLE_KEY
   ```

5. Start the local development server:

   ```bash
   npm run dev
   ```

6. Open the localhost URL printed by Vite.

Do not put a Supabase service-role/secret key in `.env` for this frontend. Only the public anon/publishable key belongs in client-side environment variables.

## 3. Supabase setup

1. Create a new project at https://supabase.com/dashboard.
2. Open **SQL Editor**.
3. Run the entire `supabase/schema.sql` file.
4. In **Authentication → Users**, create an admin user with email/password.
5. Copy that user's UUID.
6. Run this SQL, replacing the UUID:

   ```sql
   insert into public.user_roles (user_id, role)
   values ('REPLACE-WITH-AUTH-USER-UUID', 'admin')
   on conflict (user_id) do update set role = 'admin';
   ```

7. Add your Supabase Project URL and anon/publishable key to `.env`.
8. Restart the Vite server after changing `.env`.

The schema creates a private `iti-pdfs` bucket and RLS policies. The website lists published PDF metadata publicly, but the PDF object itself is opened through a signed URL created by the app.

### PDF upload flow

Admin Login → Admin → PDF अपलोड → enter title/trade/subject → choose a PDF → अपलोड और प्रकाशित करें.

The browser checks PDF extension/MIME and size for usability, but client checks are not a security boundary. Storage bucket policies and RLS policies are the real access control.

## 4. Add questions and tests

- Admin Login → Admin → प्रश्न बैंक to add MCQs.
- Admin Login → Admin → टेस्ट to create a test.
- Add questions to a test by running SQL in the Supabase SQL Editor. Example:

  ```sql
  -- Replace IDs with the UUID of your test and question.
  insert into public.test_questions (test_id, question_id, position)
  values
    ('TEST-UUID-HERE', 'QUESTION-UUID-HERE', 1);
  ```

- The starter CBT screen uses all published questions as a fallback if no question links are set for a test. Add more MCQs before public use.

## 5. Free-tier deployment

### Vercel (frontend)
1. Push the project to a private/public GitHub repository.
2. Import it into https://vercel.com/.
3. Set `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` under Project Settings → Environment Variables.
4. Deploy.

### Supabase (backend)
Keep your database and storage in the Supabase project. Free-tier limits and policies can change, so review current plan limits and backups before launching.

## 6. Before public launch — important CBT security work

This is a starter/demo and not yet a tamper-resistant exam system:
- The current frontend reads `correct_option` for result review. A determined user can inspect browser requests and see correct answers before submission.
- Result scoring currently happens in the browser and is not authoritative.
- Attempt insert uses a null user id in this starter. Student accounts and per-user attempt history should be implemented before production.
- The test form creates a test but doesn't yet include a full visual question assignment editor; link question IDs through SQL as above.
- For a real competitive CBT portal, implement a Supabase Edge Function or backend endpoint that serves questions without `correct_option`, receives answers, calculates scores server-side, saves attempt + answers, and returns only the final result.
- Add rate limiting, email verification, password recovery, monitoring, backup policy, terms/privacy, and a content moderation/ownership process.
- Use only PDFs you own, have permission to distribute, or that are licensed for sharing.

## Project structure

```text
iti-exam-guru/
  src/
    lib/supabase.ts
    App.tsx
    main.tsx
    styles.css
    types.ts
  supabase/
    schema.sql
  .env.example
  package.json
  vite.config.ts
  README.md
```
