# Top N Shop: rebuild the public site from the design + add "Manage Collection" to the CRM admin

You are working in the topnshop.com codebase (public wholesale site + CRM admin). Do everything below in one run.

Do not stop to ask me questions. When something is unclear, take the most sensible option, keep going, and list it under "Assumptions" in your final report. Do every setup step yourself, in code or through the tools and APIs you can reach: database migrations, storage buckets or folders, environment variables, image processing setup, seed data, service worker registration, build and deploy config. Leave a step for me only if it needs a password or an account you truly cannot reach, and then tell me exactly what to click.

## 0. Before you write code

1. Read the whole repo first: framework, routing, database, auth and roles, file storage, how the CRM admin is built, the existing Hindi/English toggle, and how the site is deployed. Read `docs/CRM_AUDIT.md` if it exists.
2. Keep the current stack. No new framework. Add a library only when it clearly earns its place.
3. Work on a new git branch and commit in small, logical steps.
4. Do not change existing CRM behaviour: bulk CRM logic, staff roles and dashboards, Staff Bank incentives, tasks, inventory, and the BaseLinker code behind its feature flag all stay exactly as they are. You are adding one new module and replacing the public pages.
5. If the public site and the CRM live in separate folders or projects, do both and connect them through one API.

## 1. The design files

The folder `topnshop-handoff/design/` holds the approved mobile designs:

- `Main.dc.html`: home page
- `Menu.dc.html`: side menu, open
- `Collection.dc.html`: collection page, View 1 (one photo per screen, like Reels)
- `CollectionView2.dc.html`: collection page, View 2 (2 designs in one row)
- `Reviews.dc.html`: reviews page
- `Help.dc.html`: help page (contact numbers, policy, about us)

These are exports from a design tool. Read the HTML and the inline styles as the spec for layout, spacing, sizes, colours, fonts and the exact wording. Ignore and never ship `support.js`, `<x-dc>`, `<helmet>`, the `{{...}}` holes and the `text/x-dc` script. Rebuild every page properly with this project's own components.

Rules for all pages:

- Match the design closely on phones 360px to 430px wide. On wider screens keep the same single column (max-width 480px, centred). Mobile is the product; desktop only has to look tidy.
- Colours: ink `#1C1710`, white `#FFFFFF`, soft band `#F4F2EC`, lines `#D9D4C8` and `#E4DFD3`, muted text `#5E574B`, gold on dark `#D9B45C`, gold text on white `#8A5F0F`, WhatsApp green `#0F7A40`, photo placeholder `#E9E6DF`.
- Fonts: Anek Latin (width 75, weight 800) for headings and big numbers, Mukta for everything else. Self-host both with `font-display: swap`.
- The buyers are older Indian shop owners. Keep the simple wording exactly as written in the design. Every tap target is at least 44px. No tiny grey text.
- Text inside [SQUARE BRACKETS] is a placeholder. Never invent it. Make each one an editable field in admin Site settings (section 3) and hide that row or section on the public site while it is empty.
- Never invent reviews, numbers, prices or policies.
- Keep the हिंदी toggle working on every page. Put all new text through the existing translation system. Write simple, everyday Hindi for the new strings and list them in your report so I can check them.
- The new design is light only. Remove the old dark mode toggle from the public site.

## 2. Public pages

**Home (`Main.dc.html`)**
- Build every section in the order shown.
- "WhatsApp us" opens a WhatsApp chat with the sales number. "Call now" dials it. "Get samples" opens WhatsApp with a ready message asking for samples. "Open in Google Maps" uses the map link from settings.
- The Call now / WhatsApp us bar stays fixed at the bottom while scrolling.
- The offer strip, the counters (shop owners, Instagram, YouTube, years) and all contact numbers come from Site settings.

**Side menu (`Menu.dc.html`)**
- Slides in from the right over a dark layer. Items: Collection, Reviews, Need help. Below: WhatsApp us, Call now, Instagram, YouTube, हिंदी.
- Closes on the X, on a tap outside, on Esc and on the phone's back button. Lock page scroll while open and keep keyboard focus inside it.

**Collection (`Collection.dc.html` and `CollectionView2.dc.html`)**
- Top: offer strip, then the cart bar: "N pieces in cart", under it "Add X more for minimum order" (once the minimum is reached: "Minimum order done"), and the black cart button with the total in ₹.
- Under the cart bar: View 1 and View 2 buttons. Remember the last choice on that device.
- View 1 (default): one design fills the photo area. Swipe up or down for the next or previous design, one design per swipe (scroll-snap). Only the photo area scrolls. The bottom panel stays fixed and always shows the design that is on screen: Price ₹X "for one piece", Fabric, size buttons (only the sizes that design has), "Select all sizes", "Add to cart". The small thumbnails on the photo switch between that design's photos; swiping left or right does the same. The hint "Swipe up for next design" shows until the first swipe, then never again.
- View 2: 2 designs in one row with photo, price and fabric, loading more as you scroll. Tapping a design opens that same design in View 1.
- "Download" opens a small sheet with two choices: "Photos of this design" and "All photos" (section 4).
- Show only live designs, newest first. Sold out designs disappear from the page.
- Every design has its own link (for example `?design=TNS-0123`) so a link shared on WhatsApp opens that design.

**Cart and order (no design file: build it in the same style)**
- Each selected size adds 1 piece of that design. In the cart each line has + and − buttons. Show photo, design code, size, quantity, price, total pieces and total ₹.
- Minimum order is 30 pieces. Sizes, colours and designs can be mixed. Below the minimum the main button is disabled and says "Add X more pieces".
- Sample order: 5 to 29 pieces is allowed as a "Sample order", charged at each design's sample price. Say clearly in the cart that the sample rate applies.
- New buyer offer (text and on/off in settings): 5% off, up to ₹500, on the first order of a phone number.
- "Order on WhatsApp": ask once for name, shop name, city and phone (remember them), save the order in a new `website_orders` table with status "new", then open WhatsApp to the sales number with a ready message listing design codes, sizes, quantities, total pieces and total ₹.
- Do NOT create bulk CRM orders automatically and do NOT credit any Staff Bank incentive from website orders.
- No online payment in this version.
- The cart survives refresh and closing the browser.

**Reviews (`Reviews.dc.html`)**
- Three sections filled from admin: video reviews, WhatsApp chat screenshots, written reviews. Hide a section while it is empty. Videos load only when tapped (poster image first).

**Help (`Help.dc.html`)**
- Contact cards from settings with Call (`tel:`) and WhatsApp (`wa.me`) buttons exactly as in the design, then the policy rows, then About us.

## 3. Admin: "Manage Collection" inside the existing CRM admin

Add a new menu item "Manage Collection" in the CRM admin, using the CRM's existing layout and components. Give it to the owner role by default and keep the allowed roles in one config so I can add staff later.

**Design list**
- Columns: cover photo, design code, price, sample price, fabric, sizes, status, date added.
- Search by code or fabric, filter by status, sort by date or price.
- Edit price and fabric right in the list. Save when I leave the field and show a small "Saved" tick.
- Select many designs at once to change price, change fabric, hide or show, mark sold out, or delete.

**Add or edit a design**
- Pick or drop many photos at once. Multi-select from a phone gallery must work.
- Fields: price, sample price, fabric (a list that remembers past values, plus free text), sizes (S, M, L, XL, XXL on/off), status (Live, Hidden, Sold out), optional note.
- Design code is made automatically (TNS-0001, TNS-0002 ...) and can be edited.
- Photos: add more, delete one, replace one, drag to reorder. The first photo is the cover.
- Deleting a design is a soft delete with a confirm box and 30 days to restore.

**Fast bulk add**
- I can drop 50 photos, choose "each photo is one design" or group photos into designs, set one price, fabric and sizes for all of them, and fix single designs later.

**Upload behaviour**
- Shrink photos in the browser before upload (long side max 2000px), upload 3 at a time, show real progress % per photo and in total, retry failed photos, and never lose what I typed if something fails.
- On the server keep the original and make WebP copies at about 240, 480, 960 and 1600px wide (AVIF too if it is cheap). Save width, height and a tiny blur placeholder. Use content-hashed file names so an image URL never changes.
- Use the existing file storage. If photos currently sit on a disk that the host can wipe, move them to permanent object storage and set that up yourself.
- Changes show on the public site straight away (refresh the data cache, not the image cache).

**Also add in admin**
- Reviews manager: videos, WhatsApp screenshots, written reviews.
- Site settings: offer strip, counters, contact numbers and their labels, policy texts, shop address and timings, GST number, social links, minimum order, sample minimum, payment text, map link.
- Website orders: a simple list of orders placed from the site, with status.
- A change log: who changed what (for example price old → new) and when.

## 4. Photo downloads for buyers

- "Photos of this design" saves all photos of the current design in full quality.
- "All photos" saves the whole live collection with one tap.
- It must work on Android Chrome and iPhone Safari. Where the browser can share files, use that so photos go to the gallery. Otherwise give a zip.
- For the whole collection, build the zips on the server in the background and rebuild them when the collection changes. Split into parts a phone can handle (about 150 MB each), show the size before download, show real % while downloading, and allow cancel and resume. Never build a huge zip in the phone's memory.
- Downloads are JPG (WhatsApp friendly), named with the design code, for example `TNS-0123_1.jpg`.

## 5. Speed: these are hard requirements

- Real loading bar: on first visit a slim bar at the top shows the REAL loading %, meaning bytes received out of total bytes for what the first screen needs. No fake timer. It goes away when the first photo is visible. It never blocks the page: text and buttons work immediately.
- Load once: a photo that was loaded once must come from the device afterwards, including after refresh, after reopening the browser, and on a weak network. Use long-lived immutable cache headers on the hashed image URLs plus a service worker: cache-first for images, fonts and static files, stale-while-revalidate for collection data. Cap the cache (about 300 MB) and remove the least recently used photos first. Show the last saved collection instantly, then update quietly.
- Right size every time: `srcset` and `sizes`, WebP/AVIF, width and height set, blur placeholder first. Nothing jumps when a photo arrives.
- View 1 keeps only the current design and 2 on each side in the page, and preloads the next 2 photos when idle. View 2 uses a virtual list with lazy photos.
- The collection API is paginated, small and compressed.
- No freezing anywhere: no heavy work on scroll (IntersectionObserver, passive listeners, transform and opacity animations only), no task longer than 50ms on the main thread, async image decoding, zip and compression work in Web Workers.
- Targets on a mid-range Android phone on Slow 4G: LCP under 2.5s, CLS under 0.05, INP under 200ms, Lighthouse mobile performance 90 or more on Home and Collection. Measure, fix, and give me the before and after numbers.
- Offline: designs already seen can still be browsed. Otherwise show a clear "No internet" message.

## 6. Cookie notice

- A small bar at the bottom on the first visit only, in plain words: "We use cookies to keep your cart and make the site fast." Buttons: "OK" and "Only needed". Remember the choice for 12 months.
- It sits above the fixed Call/WhatsApp bar and above the collection bottom panel. It never covers them, never blocks scrolling or taps on the rest of the page, and is never a full-screen layer.
- Analytics and marketing scripts load only after "OK". The cart, the language choice and the saved view always work.
- Add a short "Cookies and privacy" row to the policy list on the Help page, with its text in Site settings.

## 7. Checks before you finish

- Test at 360px, 390px and 430px. Handle iPhone Safari and Android Chrome properly: `100dvh`, safe-area space under the fixed bottom bars, scroll-snap.
- Use real buttons and links with labels, as in the design markup.
- Add tests for the cart maths (minimum order, sample rate, new buyer offer), for admin permissions and for the image pipeline. Run the existing tests. Nothing in the CRM may break.
- Seed 12 sample designs with placeholder photos for local development only, never in production.
- Page titles, descriptions, a share image and a sitemap. Old URLs must keep working through redirects.

## 8. Final report

Short and in plain language:
1. What you built.
2. How to open Manage Collection and add the first design.
3. Assumptions you made.
4. Placeholders I still have to fill.
5. Speed numbers, before and after.
6. Anything you could not do, and exactly why.
