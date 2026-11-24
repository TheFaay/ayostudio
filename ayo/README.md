# Ayo Hair Studio

One-page, plain HTML/CSS/JavaScript salon site built from the supplied Stitch design and build documents.

## Before publishing

1. Set `WEBHOOK_URL` near the top of the script in `index.html` to the n8n Webhook node's **Production URL**.
2. Set the Webhook's allowed CORS origin to the deployed site origin.
3. Confirm the business name and service labels against the Google Sheet/dashboard filters.
4. Replace owner-dependent placeholders with approved address, contact details, hours, stylist bio and client reviews.
5. If the Sheet gets a `Preferred Time` column, add that field to the form and request payload together.

The page is static and can be served by any static host. No build step or framework is required. The booking form remains in an unconnected state until the production webhook URL is configured.

The lookbook photos are examples only and are not represented as Ayo Hair Studio client work. Before launch, replace them with owner-approved salon photos if available; image loading currently depends on Unsplash.

## Project files

- `index.html` — finished single-page website and booking form.
- `design/` — original Stitch reference, screenshot and design system.
- `assets/` — reserved for studio-approved images and brand assets. The current lookbook uses credited Unsplash inspiration photos loaded from Unsplash.
- `CLAUDE.md`, `PHASE.md` — supplied build instructions and checklist.
