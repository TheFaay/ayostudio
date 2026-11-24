# CLAUDE.md: Ayo Hair Studio website (build phase)

Read this file first. Then read `PHASE.md` for the checklist. Source design files from Stitch live in `/design/`: `DESIGN.md` (design system), `code.html` (reference build), `screen.png` (full-page mock).

## 1. Purpose

A one-page salon website whose main job is to turn visitors into **booking requests**. The booking form sends data to our **n8n appointment-reminder workflow** (Group 5 Capstone, Idea 10). Everything else on the page supports that goal.

Do not let design work break the integration. If a design choice conflicts with the contract in section 3, the contract wins.

## 2. Design system ("Warm Atelier")

Follow `/design/DESIGN.md`. Summary:

| Token | Value |
|---|---|
| Primary (cinnamon terracotta) | `#9B422B`, hover `#843723` |
| Espresso (hero, footer, dark cards) | `#1C1917` |
| Canvas (alabaster) | `#FAF7F2` / surface `#FCF9F4` |
| Linen container | `#F6F3EE`, `#F3ECE1`, `#E9DFCE` |
| Muted taupe (metadata only) | `#786F66` |
| Headings | Space Grotesk, 600 to 700, negative tracking |
| Body | Inter 400 to 500 |
| Labels | Space Grotesk uppercase, tracking 0.1 to 0.12em |
| Radius | 0.5rem cards and inputs, pill (9999px) for buttons and chips |
| Grid | 12 col desktop (1024px+), 8 tablet (768 to 1023), 4 mobile |
| Margins / gutters | 4rem / 2.5rem desktop, 2rem / 1.5rem tablet, 1.25rem / 1rem mobile |

**Liquid glass** is used only on floating elements: sticky header, sticky section nav, mobile "Book now" bar. Recipe: `backdrop-filter: blur(18px)` over `rgba(250,247,242,.75)` with a `1px solid rgba(255,255,255,.6)` rim.
Always add a fallback: `@supports not (backdrop-filter: blur(1px))` uses a solid surface, and `@media (prefers-reduced-transparency: reduce)` removes blur.

**Never put glass behind body text, form fields, prices or FAQ answers.** Those stay on solid surfaces.

Page order: top bar, hero with booking card, menu, signature looks, about band, process, reviews, FAQ, footer.

## 3. n8n integration contract (do not change without telling the user)

The booking form replaces the n8n Form Trigger with a **Webhook trigger**. The rest of Workflow A (Sheets, Gmail) and all of Workflow B stay as in the build guide.

### Request

`POST` JSON to the Webhook node **Production URL** (never the Test URL), stored in one constant `WEBHOOK_URL` at the top of the page script.

```json
{
  "Name": "Elena Vance",
  "Email": "elena@example.com",
  "Business": "Ayo Hair Studio",
  "Service": "Haircut and style",
  "Appointment Date": "2026-12-03",
  "Preferred Time": "11:30 AM"
}
```

### Field rules

| Key (exact, case-sensitive) | Rule |
|---|---|
| `Name` | Required, trimmed |
| `Email` | Required, valid format |
| `Business` | Hidden input, constant `Ayo Hair Studio`. Must match the Business dropdown value used by the dashboard filter |
| `Service` | Send the **label text**, never the internal code. Stitch's `<option value="cut">` must become `value="Haircut and style"` |
| `Appointment Date` | **`yyyy-MM-dd`** only. Native `<input type="date">` already returns this. Never reformat it. A mismatch is the #1 reason Workflow B sends no reminder |
| `Preferred Time` | Optional. Only sent if the Sheet has the matching column (see decision D1 in `PHASE.md`) |

Do not send `Created At` or `Status`. n8n sets those (`{{ $now }}` and `Pending`).

### Response

The workflow ends the request with a **Respond to Webhook** node returning HTTP 200 and JSON `{ "ok": true }`.
The page shows the success state only when `response.ok` is true. On any network or non-2xx error it shows the error box and keeps the user's input.

### n8n side (for reference)

1. Webhook: POST, path `hair-booking`, Respond = "Using 'Respond to Webhook' Node", Options, Allowed Origins (CORS) = the live site origin.
2. Google Sheets, Append row: map from `{{ $json.body.Name }}`, `{{ $json.body.Email }}`, `{{ $json.body.Business }}`, `{{ $json.body.Service }}`, `{{ $json.body['Appointment Date'] }}`. Created At `{{ $now }}`, Status `Pending`.
3. Respond to Webhook: `{ "ok": true }`.
4. Gmail: To `{{ $('Webhook').item.json.body.Email }}`. Subject: Booking request received. Body must never promise an unconfirmed slot.

## 4. Fixes required to the Stitch export (`code.html`)

The Stitch file is a design prototype. Before it is production-ready:

- [ ] Remove the `Toggle demo state` button and the `toggleSuccessDemo()` function.
- [ ] Remove the viewport switcher simulator (`setPreviewViewport`, `viewport-stage`, `stage-mobile-bar`) and any preview-only wrapper.
- [ ] `handleBookingSubmit` currently shows success **without sending anything**. Replace it with a real `fetch` POST to `WEBHOOK_URL`.
- [ ] Add `name` attributes to every field (`Name`, `Email`, `Service`, `Appointment Date`, `Preferred Time`).
- [ ] Change service `<option>` values from codes (`cut`, `blowout`, `trim`, `color`, `gloss`, `braids`, `takedown`, `consult`) to the exact label text.
- [ ] Add `min` (today) to the date input, and block Sunday and Monday (studio closed).
- [ ] Add the hidden `Business` input and a hidden honeypot field.
- [ ] Disable the submit button while sending, and restore it on error.
- [ ] Replace the 8 Stitch-hosted images (`lh3.googleusercontent.com`) with images we own, saved in `/assets/` and optimised. Hot-linked Stitch URLs can expire.
- [ ] Replace Tailwind **CDN** (`cdn.tailwindcss.com`) with a compiled build for production, or accept the CDN only for a prototype.
- [ ] Keep the Stitch prices and times as the source of truth (for example Haircut 75 min from $110). They differ from earlier drafts.

## 5. Rules for building

1. Plain, accessible HTML, CSS and JS. No frameworks unless the user asks.
2. Mobile first. Test at 390px, 768px and 1280px.
3. Minimum tap target 48px. Visible focus rings on everything interactive. Text contrast at least WCAG AA.
4. Typographic apostrophes and quotes in all visible text.
5. No secrets in the page. The webhook URL is public by nature, so protect it with CORS and the honeypot, and never put Google, Gmail or Telegram credentials in front-end code.
6. Keep the "Never promise an unconfirmed slot" wording everywhere (form note, success state, emails).
7. Do not invent business facts. Address, phone, email, hours, reviews and photos are placeholders until the owner supplies real ones.
8. After any change to the form, re-run the verification tests in `PHASE.md`.

## 6. Definition of done

A visitor on a phone can book in under a minute, a new row appears in the Google Sheet with `Pending`, they get the "request received" email, and a booking for tomorrow triggers the 8:00 reminder and the Telegram summary. Both n8n workflows are Active and the page uses the Production URL.
