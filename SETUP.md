# 300 Essential Business English Expressions — Sales Funnel

Two self-contained pages:

| File | Role |
|---|---|
| `Sales Page.dc.html` | The sales page. Every CTA points at `paymentUrl`. |
| `Thank You.dc.html` | Post-payment download page. Set this as your gateway's success-redirect URL. |

Both are single HTML files — no build step, no npm install, no framework. Open either in a browser or drop it on any static host.

## 1. Where to change things

All values live in one `siteConfig` object at the top of each file's logic script.

**`Sales Page.dc.html` → `siteConfig`**

| Placeholder | What to put there |
|---|---|
| `PAYMENT_URL_HERE` | Your payment/checkout link. Used by all 7 CTAs. |
| `SALES_PAGE_URL` | Public URL of this page (used for canonical + OG tags — also edit the `<link rel="canonical">` and `og:url` in the head). |
| `EBOOK_COVER_IMAGE` | Cover image URL for social sharing (`og:image`, `twitter:image` in the head). |
| `CURRENCY`, `SALE_PRICE`, `ORIGINAL_PRICE` | e.g. `₹`, `299`, `999`. Set `showOriginalPrice: false` (Tweaks) to hide the strike-through. |
| `AUTHOR_NAME`, `AUTHOR_PHOTO`, `AUTHOR_BIO` | Author card + footer copyright. |
| `CONTACT_EMAIL`, `INSTAGRAM_URL`, `LINKEDIN_URL` | Footer + FAQ. |
| `PRIVACY_POLICY_URL`, `TERMS_URL`, `REFUND_POLICY_URL` | Footer legal links. |
| `GA_MEASUREMENT_ID`, `META_PIXEL_ID` | Leave `""` and no events are sent anywhere. |

**`Thank You.dc.html` → `siteConfig`**: `EBOOK_DOWNLOAD_URL`, `SALES_PAGE_URL`, `AUTHOR_NAME`, `CONTACT_EMAIL`.

### Book cover image
The 3D book on both pages is drawn in CSS (no image file needed). To use your own transparent PNG instead, replace the innermost cover `<div>` in the hero with:

```html
<img src="EBOOK_COVER_IMAGE" alt="300 Essential Business English Expressions eBook cover" loading="lazy" style="width:min(72vw,286px);height:auto" />
```

### Author photo
Replace the `AUTHOR_PHOTO` placeholder block in the author card with an `<img src="…" alt="AUTHOR_NAME" loading="lazy" style="width:112px;height:112px;object-fit:cover" />`.

## 2. Payment gateway wiring

```
Sales Page → CTA (PAYMENT_URL_HERE) → gateway → success redirect → https://yourdomain.com/thank-you → download
```

1. Create the product in your gateway (Razorpay / Stripe Payment Link / Lemon Squeezy / Gumroad / Instamojo…).
2. Copy the hosted checkout link into `paymentUrl`.
3. In the gateway settings, set the **success / after-payment redirect URL** to your thank-you page URL, e.g. `https://yourdomain.com/thank-you`.
4. Set the failure/cancel redirect back to the sales page.

## 3. Deploy

**Vercel (recommended)** — rename the files as a tiny static site:

```
/index.html          ← Sales Page.dc.html
/thank-you/index.html ← Thank You.dc.html
/support.js          ← keep alongside (both pages load it)
```

Then `npx vercel deploy --prod` in that folder, or drag the folder into the Vercel dashboard. `/thank-you` then works as a clean URL.

**Netlify** — same folder structure; drag-and-drop into Netlify Drop, or `netlify deploy --prod --dir .`

**Any other host** (S3 + CloudFront, Cloudflare Pages, GitHub Pages, Hostinger, cPanel) — upload the same three files. No server runtime required.

## 4. Analytics

Paste your GA4 / Meta Pixel / Clarity snippets into the marked `ANALYTICS PLACEHOLDER` comment in each file's `<helmet>`, then fill `gaMeasurementId` / `metaPixelId`. Events already wired:

`page_view`, `header_cta_click`, `hero_cta_click`, `mid_page_cta_click`, `value_cta_click`, `late_cta_click`, `final_cta_click`, `sticky_cta_click`, `checkout_click`, `thank_you_page_view`, `ebook_download_click`

While the IDs are empty nothing is transmitted — events only go to `window.dataLayer` if a tag manager has created it.

## 5. Download security — read this

`EBOOK_DOWNLOAD_URL` is embedded in the page source, so **this is not a protected download**: anyone who obtains that URL can fetch the PDF without paying. It's fine to launch with, and acceptable for most low-ticket digital products, but be aware of it.

`getDownloadUrl()` in `Thank You.dc.html` is the single seam for hardening it later — swap its body for a call to a backend endpoint that verifies the order and returns an expiring signed URL (S3 presigned URL, Cloudflare signed URL, or a tokenised link tied to the gateway's payment ID). Nothing else on the page changes.

The page deliberately says "Thank you for your purchase" rather than "payment verified", because no server-side verification exists yet.

## 6. Testing checklist

- [ ] All 7 sales-page CTAs open the correct payment URL (header, hero, product, value stack, mid CTA block, final CTA, mobile sticky)
- [ ] Nav anchors jump to What's Inside / Who It's For / FAQ
- [ ] Gateway success redirect lands on `/thank-you`
- [ ] Download button saves the correct, complete PDF
- [ ] `Back to Home` returns to the sales page; no auto-redirect off the thank-you page
- [ ] Mobile: no horizontal scroll at 320px, 375px, 414px; sticky CTA appears after the hero and is dismissible by scrolling back up
- [ ] Tablet (768px) and desktop (1280px, 1920px) layouts
- [ ] FAQ accordion opens by click and by keyboard (Tab + Enter); focus rings visible
- [ ] `prefers-reduced-motion` disables the float/reveal animations
- [ ] All footer links resolve (no `#` placeholders left)
- [ ] Page title, meta description, OG/Twitter image render correctly in a link-preview debugger
- [ ] Analytics events fire in GA4 DebugView / Meta Pixel Helper
- [ ] Browsers: Chrome, Safari (iOS), Firefox, Edge
- [ ] Lighthouse mobile ≥ 90 performance (only two web fonts are loaded; no JS libraries)
