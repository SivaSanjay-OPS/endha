# LoanFlow — Single-file Frontend (README)

**Short problem statement**

LoanFlow is a single-file, client-side prototype for managing loan applications and approvals. It simulates the end-to-end front-end experience for three roles: **Admin**, **Lender**, and **Borrower**. Borrowers upload KYC and supporting documents and submit requests; those documents reach the Admin review queue for approval or rejection (with CAPTCHA checks). Lenders and Admins can create loans from uploaded files. The app stores data in `localStorage` so you can run it locally without a backend.

---

## What this file contains

* A single HTML file that includes all HTML, CSS, and JavaScript required to run the front-end demo.
* Tailored UI panels for Dashboard, Loans, Documents, Transactions, and Reports.
* Drop zone for file uploads (client-side metadata only) and file forwarding to Admin or Lender.
* Simple CAPTCHA prompts on critical actions (demo placeholder for integration with real CAPTCHA service).
* Local persistence using `localStorage` under the key `loanflow:v7`.

---

## Roles and responsibilities (in the app)

* **Borrower**

  * Uploads files (KYC / income proof / others) via the shared drop zone.
  * Posts a request to Admin ("Post to Admin") or sends to a Lender ("Send to Lender").
  * Must provide requested amount (≥ 1,000) and at least one file for the request to be valid.
  * Actions trigger a simple CAPTCHA prompt (demo).

* **Admin**

  * Receives borrower-submitted documents in an Admin review queue.
  * Approves or rejects applications (CAPTCHA required). Approving creates a loan record.
  * Can create loans directly for borrowers using the Create Loan panel.
  * Sees consolidated Documents table and an Admin-specific view of inbox.

* **Lender**

  * Receives forwarded borrower documents in their Inbox.
  * Can review file metadata and (in the demo) mark as accepted or request further action.
  * Can create loans for borrowers (Create Loan panel available to Lender and Admin).

---

## How to run (quick)

1. Save the single-file HTML content (the `LoanFlow — Single File (Admin / Lender / Borrower) v7`) as `index.html` on your computer.
2. Open `index.html` in a modern browser (Chrome, Edge, Firefox). No server required.
3. Use the **Role** buttons in the sidebar to switch between Admin / Lender / Borrower and exercise the flows.
4. To reset the demo data: open the browser Console and run `localStorage.removeItem('loanflow:v7')` and reload.

---

## Code structure (single-file sections)

The single file is organized into these logical blocks (comments in the file point to the same):

1. **Static UI & CSS** — top of file. Styles create the dark, minimal aesthetic and include responsive adjustments.
2. **HTML layout** — main app container with `nav.sidebar` and `main` views for Dashboard, Loans, Documents, Transactions, Reports. Also includes a modal backdrop and a toast area.
3. **State & persistence** — `STORAGE_KEY = 'loanflow:v7'`, `loadState()` and `saveState()` functions, plus a seeded `DEFAULT` state used on first run.
4. **Runtime object** — `runtime` keeps the active role, user id, and the current upload buffer (`uploadedFiles`).
5. **Drop zone & file handlers** — `handleFiles()` validates file types and sizes (max 10 files, 5MB each) and stores file **metadata** in the upload buffer.
6. **Document posting flows**

   * `postToAdmin` — borrower posts files: creates a `docs` record and an `inbox` entry for admin review. Validates numeric amount and files and uses a demo CAPTCHA prompt.
   * `sendToLender` — borrower sends files to lender similarly.
7. **Create loan flow** — `createLoanBtn` allows Lender/Admin to create a loan from the uploaded files. This generates a `loans` entry and a `docs` record linking files to the loan.
8. **Admin review flows** — `adminApproveDoc`, `adminRejectDoc`, and UI helpers that convert a document/inbox item into an approved loan record.
9. **Rendering functions** — `renderAll()` orchestrates UI updates; sub-renderers update Dashboard, Loans table, Documents, Inbox, Lender Inbox, and Create-Borrower select.
10. **Charts** — a bar chart (Chart.js) visualizes loan amounts in Reports. It is optional and degrades gracefully if Chart.js fails.

---

## Primary data model (stored in `localStorage`)

* `users`: list of user objects `{id, name, role, credit}`
* `loans`: list of loans `{id, borrowerId, amount, term, files[], status, remaining}`
* `docs`: document submissions `{id, from, to, files[], amount, purpose, status, date}`
* `inbox`: task items for admin/lender derived from posted docs `{id, type, docId, fromApplicant, files[], amount}`
* `txns`: transaction records (present but minimal in demo)
* `approvalLog`: audit entries for approvals/rejections

---

## Key functions and what they do (walkthrough)

* `handleFiles(list)` — receives a FileList or array, validates type and size, pushes simple metadata (`{name,size,type}`) to `runtime.uploadedFiles`.

* `renderFileList()` — shows the current upload buffer in the Upload panel.

* `postToAdmin` handler — triggered by the "Post to Admin" button. Steps:

  1. Confirm the current role is Borrower.
  2. Validate there are files and a numeric requested amount ≥ 1,000.
  3. Present a demo CAPTCHA prompt and validate the answer.
  4. Create a `docs` object and an `inbox` entry (type `to_admin`).
  5. Clear upload buffer and persist state.

* `sendToLender` — same pattern as Post to Admin but targets Lender.

* `createLoanBtn` — lender/admin flow to create a loan from the buffer:

  * Validates borrower, amount, and term, then creates the `loan` and a `docs` record linking the attached files to the loan.

* `adminApproveDoc(inboxId)` — used when Admin approves an `inbox` item. It creates a loan record derived from the inbox data, marks docs as approved, logs the approval, removes item from `inbox`, and persists state.

* `adminRejectDoc(inboxId)` — prompts for a reason and marks the doc/inbox item as rejected.

* `renderAll()` and helper renderers update tables and charts on every change.

---

## Outputs you will observe when running the file

* When a borrower posts files to Admin: a new row appears in **Documents** and **Admin Review Queue**; after Admin approval a loan appears in **All Loans** and in **Recent Loans** on Dashboard.
* When the Lender receives a forwarded file: it appears in the **Lender Inbox** table.
* Creating a loan directly as Lender/Admin will add a loan row and a doc entry linking files and the loan.
* Toast notifications appear for all major actions and errors.
* Local changes persist across reloads until you reset `localStorage`.

---

## Security notes and integration points (how to turn this demo into a real app)

This is intentionally a front-end-only demo. To move to production:

1. **File uploads**

   * Replace client-only metadata with real file uploads using signed S3 (or similar) URLs. The client should request a signed URL from the backend, then PUT the file directly to storage.
   * Perform virus scanning (ClamAV or cloud provider) before marking files as verified.

2. **CAPTCHA**

   * Replace the demo `prompt()` CAPTCHA with a real CAPTCHA provider (Google reCAPTCHA v3, hCaptcha). Send the token to your backend and verify server-side with the provider's API before performing the sensitive action.

3. **Authentication & Authorization**

   * Implement an auth backend (JWT or session cookies) and replace the runtime role-switch with real login & role-checks.
   * Server must enforce role-based access control — never trust client-side checks.

4. **Persistence & API**

   * Build a REST API: endpoints for `POST /docs`, `GET /inbox`, `POST /inbox/:id/approve`, `POST /loans`, `GET /loans`, `POST /upload/sign`, etc.
   * Move local data into a relational DB (Postgres recommended) and add server-side validations.

5. **Payments & Transactions**

   * Integrate a payment gateway (Stripe, Razorpay) for recording actual repayments and generating ledger entries.

6. **Audit & Logging**

   * Keep immutable audit logs for admin approvals and file access. Avoid logging sensitive PII in clear text.

---

## Testing checklist (manual)

1. Borrower: upload files, enter amount ≥ 1000, click **Post to Admin** — confirm a new document is created and an inbox item appears for Admin. Verify the uploaded file names are visible in the Documents table.
2. Admin: approve the inbox item — confirm a loan record is created and appears on Dashboard/Loans.
3. Lender: accept forwarded documents in the Lender Inbox (demo action) and create a loan via the Create Loan panel. Verify doc linking.
4. Error flows: attempt to post without files, post with invalid amount, or fail CAPTCHA — verify clear error messages and no records created.
5. Mobile: shrink browser width and confirm panels stack and remain usable.

---

## Known limitations in this single-file demo

* Files are not actually uploaded to a server; only metadata is stored.
* CAPTCHA is a `prompt()` placeholder and must be replaced with a real provider.
* No real authentication — role switching is a UI convenience for demo only.
* Charting relies on Chart.js (bundled via CDN); if blocked the Reports view will be empty.

---

## Where to edit / key variables

* Change the local storage key: `STORAGE_KEY` (near the top of the script) when building multiple versions.
* Adjust file size or count limits in `handleFiles()`.
* Replace CAPTCHA prompts in `postToAdmin`, `sendToLender`, and `adminApproveDoc` with `grecaptcha.execute(...)` and send tokens to your server.

---

## Suggested next steps I can do for you

* Replace the demo CAPTCHA with real reCAPTCHA/hCaptcha integration (front-end + server verification code example).
* Convert this single-file React-free / Chart-free variant if you want zero external libraries.
* Generate a short presentation/demo script and screenshots you can show to a teacher to satisfy rubric points.

---

## License

You may use and modify this single-file demo freely for non-commercial educational purposes. If you publish or distribute it, please credit the author.

---

If you want, I can now create a downloadable `README.md` file inside the same canvas (so you can copy or export it), or I can inject this README as a comment block directly inside the single-file HTML so the whole package is self-documenting. Which do you prefer?
