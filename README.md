# Elite TV Repair

A complete English-language TV repair enquiry website for Dubai, Sharjah and Ajman. Built with Next.js App Router, React, TypeScript, Tailwind CSS, shadcn/ui, Lucide, Motion, React Hook Form and Zod. All 45 content routes share one typography and component system, plus a custom 404, enquiry API, robots.txt and sitemap.xml.

## Run locally with standard Next.js

Requirements: Node.js 22.18+ and pnpm 11+. Node 22.18+ runs the TypeScript-backed tests without a separate test compiler.

```bash
pnpm install --frozen-lockfile
cp .env.example .env.local
pnpm dev:next
```

Open the URL printed by Next.js (normally http://localhost:3000). For a production Node/Vercel deployment:

```bash
pnpm typecheck
pnpm lint
pnpm test
pnpm build:next
pnpm start:next
```

On Vercel select the Next.js framework and set the build command to `pnpm build:next`. The output directory is `.next-standard`, declared in next.config.ts; do not treat it as a static export. The booking page and enquiry API require a server runtime. Configure environment variables before building; public values are embedded in client assets.

## Hosted runtime

The included Sites-compatible starter uses Vinext to build the same Next.js App Router source for Cloudflare Workers. `pnpm dev`, `pnpm build` and `pnpm start` use that hosting runtime. Standard Next.js outputs are isolated in `.next-standard` to prevent collision with Vinext's generated files. Preserve the Sites build plugin and `.openai/hosting.json` when using this hosting workflow. Run the included Sites build helper when preparing a deployment archive. Publish only the archive built from the exact pushed source commit.

## Business configuration still needed

Set these values in `.env.local` locally or the deployment environment:

| Variable | Required value |
| --- | --- |
| `NEXT_PUBLIC_BUSINESS_PHONE` | Business phone in international E.164 form, e.g. a real `+971…` number |
| `NEXT_PUBLIC_WHATSAPP_NUMBER` | Real international WhatsApp number, digits only including country code |
| `NEXT_PUBLIC_BUSINESS_EMAIL` | Business email address |
| `NEXT_PUBLIC_BUSINESS_ADDRESS` | Verified business address |
| `NEXT_PUBLIC_WEBSITE_URL` | Canonical deployed HTTPS origin without a trailing slash |
| `ENQUIRY_WEBHOOK_URL` | Secure HTTPS endpoint that actually records or delivers enquiries |
| `ENQUIRY_WEBHOOK_TOKEN` | Server-only bearer token accepted by that endpoint |

`TODO` or an empty value leaves contact channels unconfigured. Phone and WhatsApp actions display an explanatory dialog, with no fake destination. A missing enquiry integration returns HTTP 503 and explicitly states that the request has not been sent. Business configuration is centralised in `config/business.ts`. Never put the webhook token in a `NEXT_PUBLIC_` variable or source control.

Also confirm individual neighbourhood coverage, assessment charges, appointment policies, any repair warranty conditions, enquiry provider identity and retention period. Review and complete the legal pages before accepting live enquiries. Supply genuine customer reviews with permission before replacing the review placeholder. No ratings, years of experience, certificates, arrival guarantees or official brand affiliations are invented.

## Real enquiry endpoint contract

The site posts JSON to `ENQUIRY_WEBHOOK_URL` after validating the request on the server. It sets:

- `Authorization: Bearer <ENQUIRY_WEBHOOK_TOKEN>`
- `Content-Type: application/json`
- `Idempotency-Key: <enquiry UUID>`

Example payload shape (illustrative field names; no real customer record):

```json
{
  "id": "a-client-generated-uuid",
  "receivedAt": "an-ISO-timestamp",
  "source": "Elite TV Repair",
  "kind": "booking",
  "name": "Customer name",
  "phone": "International number",
  "email": "",
  "area": "Dubai",
  "district": "Customer district",
  "address": "Building or villa details",
  "brand": "Samsung",
  "model": "Full model code",
  "screenSize": "44–55 inches",
  "problem": "Black Screen",
  "message": "Customer description",
  "preferredDate": "YYYY-MM-DD",
  "window": "Morning (9am–12pm)",
  "consent": true
}
```

Quick enquiries use `kind: "quick"` and include phone, area, brand and problem. Contact enquiries use `kind: "contact"` and include name, phone, optional email, area, brand, message and consent. The quick form presents contact consent through its visible submission notice.

The endpoint must authenticate the token, validate its own input, durably accept the enquiry, deduplicate the idempotency key, and return a **2xx JSON response containing `{"accepted":true}` only after acceptance**. It may return a provider reference, but the site shows the stable client enquiry UUID. A plain HTTP 200 without that acknowledgement is treated as unconfirmed delivery. The website never claims that submitting confirms an appointment.

Delivery uses a 12-second timeout and rejects redirects. If the provider accepts the request but its response is lost, a retry retains the same reference while the form data remains unchanged. The provider must enforce idempotency to avoid duplicate records. The site validates size, content type, origin, phone, city, brand, required fields, dates and honeypot input. Add provider/edge rate limiting before enabling the endpoint for a public campaign; in-memory limits are not used because they are unreliable across Worker isolates.

## Architecture

- `app/`: explicit static pages, dynamic service/brand/problem/area routes, metadata routes and enquiry API.
- `components/layout/`: sticky header, dropdown menus, accessible mobile drawer, footer and contextual contact actions.
- `components/forms/`: quick form, contact form and six-step booking wizard.
- `components/home/`: accessible FAQs, interactive diagnostic selector and selective motion.
- `components/common.tsx`: shared typography-led sections, cards, process and CTAs.
- `components/detail-page.tsx`: distinct layouts for services, brands, problems and areas.
- `data/content.ts`: typed original services, brand guidance, fault guides, areas, process and FAQs.
- `config/business.ts`: centralised contact and canonical-origin configuration.
- `lib/`: shared client/server Zod schemas, delivery handling and metadata helpers.
- `tests/`: content integrity and request/delivery behaviour tests.
- `public/fonts/`: self-hosted DM Sans (400/500/600/700), with SIL OFL licence.
- `public/images/`: optimised original illustrative technician asset.

## Design and reference inspection

Reference content was inspected on 10 October 2026. A direct read-only computed-style inspection of YalaRide at a 1363px browser viewport verified:

| Element | Observed desktop value |
| --- | --- |
| Font | DM Sans |
| Hero H1 | 48px, 700, 56px line-height, -2.88px tracking |
| Sample section H2 | 48px, 700, 56–60px line-height |
| Sample category H3 | 16px, 600, 24px line-height |
| Hero paragraph | 16px, 400, 24px line-height |
| Header height | 81px |
| Navigation gap | 24px |
| Sample content width | 1216px |
| Sample section padding | 80px vertically (one early section used 40px) |
| Search control | 600 weight, 8px radius; sampled hidden responsive controls had zero measured dimensions |

These are selected observed desktop elements, not universal values for every component. Reference mobile behaviour and complete card metrics were not verified. No reference source code, text, branding or photography was copied. The new website uses the verified, legally reusable DM Sans family with the user's requested 56px/38px/22px desktop heading targets, fluid mobile sizes, a 1240px content area, navy/teal palette and original section layouts. Source font licensing: https://github.com/google/fonts/tree/main/ofl/dmsans.

Dubai TV Repair was inspected only to understand service categories and symptom types: LED/OLED/LCD/Smart/Plasma, display, mainboard, backlight, power, audio, HDMI and network/software faults. Its contact information, service guarantees, ratings and certifications were not adopted. District-level coverage is deliberately not presented as confirmed.

## Photography and assets

The technician photograph is original AI-generated illustrative imagery created with the built-in image generation tool, not a picture of actual Elite staff. The selected image is optimised into `public/images/technician.webp` and rendered with Next.js Image. The brand and service visuals are original CSS illustrations and typographic names; they do not imply official authorisation. `ASSETS.md` records provenance and the image-generation brief. Replace the illustrative technician image with licensed real business photography if available.

## SEO

All pages have individual titles and descriptions, canonical URLs and Open Graph metadata. The sitemap covers all 45 content routes. Service detail pages include Service schema; detail pages include BreadcrumbList schema; the FAQ page includes FAQPage schema matching visible text. No AggregateRating or LocalBusiness schema is emitted because verified business address/contact details and reviews are not supplied. Add LocalBusiness data only after verifying the business identity, physical address and contact details. Brand names describe independent repair enquiries.

## Verification

See `QA.md` for the actual check results and outstanding visual verification. Forms were tested with a stubbed webhook, not a real business integration. The published site has no configured submission endpoint or real phone/WhatsApp number yet.
