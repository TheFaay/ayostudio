# PHASE.md: Build checklist (Ayo Hair Studio website + n8n)

Work top to bottom. Tick each box only when it is verified, not when it is written. Rules and the data contract are in `CLAUDE.md`.

## Decisions to make first

- [ ] **D1. Preferred Time field.** Stitch's form has a "Preferred Slot" dropdown, but the Google Sheet has no column for it.
  - Option A (recommended): add a `Preferred Time` column to the Sheet (Plain text) and map it in Workflow A.
  - Option B: remove the slot field from the form.
- [ ] **D2. Hosting.** Where will the page live (the group's website builder, GitHub Pages, Netlify, or similar)? The site origin is needed for CORS.
- [ ] **D3. Real business details.** Address, phone, email, hours, prices, stylist bio, real reviews and photos.
- [ ] **D4. Business name.** Confirm the exact spelling `Ayo Hair Studio`. It must match the dashboard filter value.

## Phase 0: Project setup

- [ ] Create the project folder with `/design/` (DESIGN.md, code.html, screen.png), `/assets/`, `index.html`, `CLAUDE.md`, `PHASE.md`.
- [ ] Open `screen.png` and `code.html` side by side to confirm they match.
- [ ] Copy the design tokens from `DESIGN.md` into CSS variables (or the Tailwind config).

## Phase 1: n8n and Google Sheet (do this before the front end)

- [ ] One group Google account is used for Sheets, Gmail and Telegram.
- [ ] Sheet headers in Row 1: Name, Email, Business, Service, Appointment Date, Created At, Status (plus Preferred Time if D1 = A).
- [ ] Appointment Date column is formatted **Plain text**.
- [ ] Workflow A trigger replaced: Form Trigger removed, **Webhook** added (POST, path `hair-booking`, Respond = Respond to Webhook node).
- [ ] CORS allowed origins set to the site origin (use `*` only while testing).
- [ ] Google Sheets Append row maps `$json.body.*` to each column, Created At = `{{ $now }}`, Status = `Pending`.
- [ ] **Respond to Webhook** node returns `{ "ok": true }` with status 200.
- [ ] Gmail confirmation reads the email from the Webhook node and does not promise an unconfirmed slot.
- [ ] Workflow B unchanged: schedule 8:00, Get rows, Filter `yyyy-MM-dd` = tomorrow, Gmail reminder, Code summary, Telegram.
- [ ] Telegram bot is in the group and the Chat ID is set.
- [ ] Test the Webhook **Test URL** with a manual POST (curl, Postman or the browser console) and confirm a row and an email.

## Phase 2: Front-end build from the Stitch design

- [ ] Header: glass top bar with logo, links (Services, Looks, About, Process, FAQ) and the Book now button.
- [ ] Hero: espresso background, headline "Your hair, your chair, your time.", facts row, booking card.
- [ ] Booking card (solid surface, not glass), fields in this order: Name, Email, Service, live time and price line, Preferred Date, (Preferred Time if D1 = A).
- [ ] Menu section with four groups and the Stitch prices and durations.
- [ ] Signature looks: six cards with "Book this look" buttons that pre-select the service and scroll to the booking card.
- [ ] About band, process timeline (four steps), reviews, FAQ accordion (five questions), footer.
- [ ] Mobile: floating glass "Book now" bar fixed to the bottom, vertical timeline, one-column looks.
- [ ] Remove demo-only code listed in `CLAUDE.md` section 4.
- [ ] Replace Stitch-hosted images with our own files in `/assets/`.
- [ ] Add glass fallbacks (`@supports` and `prefers-reduced-transparency`).

## Phase 3: Wire the form to n8n

- [ ] `WEBHOOK_URL` constant set to the **Production URL**.
- [ ] Every input has the exact `name` from the contract.
- [ ] Service `<option>` values are the label text (no codes).
- [ ] `Appointment Date` is sent as `yyyy-MM-dd`.
- [ ] Hidden `Business` input = `Ayo Hair Studio`. Hidden honeypot field present.
- [ ] Client validation: required name, valid email, service chosen, date today or later, not Sunday or Monday.
- [ ] `fetch` POST with `Content-Type: application/json`.
- [ ] Success state appears **only** when the response is OK. It shows name, email, service and date.
- [ ] Error state keeps the user's input and re-enables the button.
- [ ] Submit button is disabled while sending.

## Phase 4: Quality pass

- [ ] Responsive check at 390px, 768px, 1280px. No sideways scroll.
- [ ] Keyboard-only walkthrough: every control reachable, focus visible, FAQ opens with Enter or Space.
- [ ] Contrast check on terracotta buttons, taupe labels and anything over glass.
- [ ] Dark mode (if kept): form fields and prices remain readable.
- [ ] Images have alt text. Page has a title and meta description.
- [ ] Lighthouse: Performance, Accessibility and Best Practices all green.
- [ ] No console errors. No Stitch URLs left in the code.

## Phase 5: Verification tests (before Demo Day)

| # | Test | Expected |
|---|---|---|
| 1 | Book for **tomorrow** from the live site | New Sheet row with `Pending` and an immediate confirmation email |
| 2 | Run Workflow B manually | Customer gets a reminder email. Telegram gets tomorrow's list |
| 3 | Book for **next month**, run Workflow B | No reminder is sent |
| 4 | Submit with a bad email | Page shows an error, nothing is sent |
| 5 | Turn the webhook off, then submit | Page shows an error and keeps the typed values |
| 6 | Pick Sunday or Monday | Page blocks the date |
| 7 | Click "Book this look" | Form scrolls into view with the right service selected |
| 8 | Check the Sheet row | Business spelling matches the dashboard filter. Date is `yyyy-MM-dd` |

- [ ] All eight tests pass.
- [ ] Dashboard shows the new rows (allow a few minutes of CSV lag).

## Phase 6: Launch and Demo Day

- [ ] Both workflows are **Active**.
- [ ] Page deployed. CORS origin updated from `*` to the real site origin.
- [ ] Production URL is in the live page. No Test URL anywhere.
- [ ] Production URL button/link is on every group member's site, if the group shares one booking entry.
- [ ] Test bookings were made early to allow for dashboard lag.
- [ ] Telegram bot is in the group.
- [ ] Placeholder text (address, phone, sample reviews) replaced or clearly approved.

## Known failure points

- **No reminder sent:** date not `yyyy-MM-dd`, or the Service/Business text differs from what the filter or dashboard expects.
- **Form says success but no row:** the page showed success without checking `response.ok`, or the workflow isn't Active.
- **CORS error in the browser console:** the Webhook's Allowed Origins doesn't match the site's origin.
- **Duplicate reminders:** add a Reminder Sent column and update it after sending.
