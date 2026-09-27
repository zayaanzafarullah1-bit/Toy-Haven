# Toy Haven

A front-end e-commerce website assignment for **Collectible Figurines, Toys, Board Games and Diecast Model Cars** — built with plain HTML5, CSS3 and vanilla JavaScript only (no frameworks, no back end).

## Folder structure

```
Toy-Haven/
├── index.html
├── products.html
├── cart.html
├── checkout.html
├── wishlist.html
├── feedback.html
│
├── css/
│   └── style.css
│
├── js/
│   ├── data.js          (product catalogue — 20 products)
│   ├── main.js           (nav, storage helpers, cart/wishlist counts, slider, newsletter)
│   ├── products.js       (cards, modal, search/filter/sort)
│   ├── cart.js
│   ├── checkout.js
│   ├── wishlist.js
│   └── feedback.js
│
├── images/
│   ├── logo.png
│   ├── banner1.jpg … banner4.jpg
│   └── products/         (referenced by path; JS shows a generated placeholder if a file is missing)
│
├── manifest.json
├── service-worker.js
├── favicon.ico
└── README.md
```

## Running it locally

No build step or server-side language is required, but service workers need `http://` (not `file://`) to register, so serve the folder with any static server:

```bash
cd Toy-Haven
python3 -m http.server 8080
# then open http://localhost:8080
```

or with Node's `npx serve`, or the VS Code "Live Server" extension.

## Deploying to GitHub Pages

1. Create a new GitHub repository and push the `Toy-Haven` folder contents to the `main` branch (the files, not the parent folder, should sit at the repo root — or in a `/docs` folder if you prefer).
2. In the repository, go to **Settings → Pages**.
3. Under **Build and deployment → Source**, choose **Deploy from a branch**.
4. Choose branch `main` and folder `/ (root)` (or `/docs`), then **Save**.
5. GitHub publishes the site at `https://<your-username>.github.io/<repo-name>/` within a minute or two.
6. Because all paths in this project are relative (`css/style.css`, `js/main.js`, `images/...`), no path changes are needed for the site to work under a repo subpath.

## Where the service worker is registered

`js/main.js` registers it inside the shared `DOMContentLoaded` listener that runs on every page:

```js
if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("service-worker.js").catch((err) => {
    console.warn("Service worker registration failed:", err);
  });
}
```

Because every HTML page loads `js/main.js`, the service worker is registered no matter which page a visitor lands on first. `service-worker.js` then pre-caches the app shell (HTML/CSS/JS) on install so the site keeps working offline.

## localStorage keys

| Key                 | Shape                                             | Used by                          |
|----------------------|----------------------------------------------------|-----------------------------------|
| `toyHavenCart`       | `[{ id, name, price, image, qty }]`                | main.js, products.js, cart.js, checkout.js |
| `toyHavenWishlist`   | `[{ id, name, price, image, status }]`             | main.js, products.js, wishlist.js |
| `toyHavenOrders`     | `[{ orderNumber, date, name, email, address, payment, items, subtotal, delivery, total }]` | checkout.js |
| `toyHavenFeedback`   | `[{ name, email, rating, message, date }]`         | feedback.js |
| `toyHavenNewsletter` | `["email@example.com", ...]`                       | main.js |

All reads/writes go through `safeGet()` / `safeSet()` in `main.js`, which wrap `JSON.parse` / `JSON.stringify` in `try/catch` so a corrupted value can never crash the page.

## Testing guide

Manual test checklist to run through in a browser (desktop + mobile width):

1. **Navigation** — resize below 900px, confirm the hamburger opens/closes and animates; confirm the active page is highlighted.
2. **Home** — hero slider auto-advances every 5s and via the dots; category cards link to filtered product views; Product of the Day changes date-to-date; newsletter form rejects invalid emails and accepts valid ones (check `toyHavenNewsletter` in DevTools → Application → Local Storage).
3. **Products** — type in search, click category chips, change the sort dropdown, and confirm the result count and grid update together; open a product card to check the modal (focus moves in, `Esc` and the overlay both close it, quantity stepper works); add to cart and to wishlist from both the card and the modal.
4. **Cart** — increase/decrease quantity, edit the quantity input directly, remove an item, clear the whole cart, and confirm subtotal/delivery/grand total recalculate each time; confirm free delivery kicks in over $75.
5. **Checkout** — submit with empty fields and an invalid email to see inline errors; submit a valid order and confirm the animated success panel, a random order number, and that the cart is now empty; check `toyHavenOrders` in Local Storage for the saved order.
6. **Wishlist** — change an item's status, filter by status, move an item to the cart, remove an item, and confirm the empty state appears once the list is emptied.
7. **Feedback** — submit with a short message to see validation, then submit a valid entry and confirm the success message; open/close FAQ items and confirm only one stays open at a time.
8. **Offline** — after visiting the site once, use DevTools → Network → "Offline" and reload; the app shell should still render courtesy of the service worker.
9. **Accessibility** — tab through every page with a keyboard only; confirm visible focus outlines, that the modal traps sensible focus, and that all images have meaningful `alt` text.
10. **Broken images** — rename or delete a file inside `images/products/` and reload; the JS fallback should replace it with a generated initials placeholder instead of a broken-image icon.

## Requirement checklist

- [x] HTML5, CSS3, vanilla JavaScript only — no frameworks or back end
- [x] Semantic HTML (`header`, `nav`, `main`, `section`, `article`, `aside`, `figure`, `figcaption`, `footer`, `form`)
- [x] Required file structure matched exactly
- [x] Poppins font, specified colour palette, Grid + Flexbox, radius/shadows/transitions
- [x] Mobile-first responsive layout with an animated hamburger menu
- [x] Home: header, hero, auto slider, category cards, featured products, Product of the Day, Why Choose Us, newsletter, footer
- [x] Products: 20 items in JS objects, search, category filter, sorting, modal, add to cart/wishlist
- [x] Cart: quantity controls, remove, clear, subtotal, delivery fee, grand total, localStorage
- [x] Checkout: custom validation, order summary, random order number, order history, cart clearing, animated success
- [x] Wishlist: Interested/Owned/Not Interested, status filter, move to cart, remove, empty state, localStorage
- [x] Feedback: validated form with storage, FAQ accordion with smooth animation
- [x] Reusable functions: `getCart`, `saveCart`, `addToCart`, `updateCartCount`, `getWishlist`, `saveWishlist`, `renderProducts`-equivalent (`renderProductCard` + `attachProductCardEvents`), `validateEmail`
- [x] Specified localStorage keys, always JSON-safe
- [x] Accessibility: alt text, labels, keyboard navigation, focus states, ARIA attributes, accessible modal, semantic headings
- [x] SEO: unique titles, meta descriptions, viewport, favicon, manifest on every page
- [x] PWA: `manifest.json`, `service-worker.js`, registration explained above
- [x] Relative image paths with a JavaScript `onerror` fallback

## Simple viva questions and answers

**Q1. Why does every page load `main.js` before its own page-specific script?**
`main.js` defines the shared/reusable functions (cart, wishlist, validation, navigation) that the page-specific scripts (e.g. `cart.js`) depend on. Loading order matters in plain `<script>` tags because there's no module bundler resolving dependencies automatically.

**Q2. How is the cart total kept in sync with the badge in the header?**
Every function that changes the cart (`addToCart`, `removeFromCart`, quantity edits) calls `saveCart()`, which writes to `localStorage` and then calls `updateCartCount()`. That function re-reads the cart and updates every element with a `data-cart-count` attribute, so the header badge is always correct regardless of which page triggered the change.

**Q3. How does the product modal handle accessibility?**
It's built with `role="dialog"` and `aria-modal="true"`, moves focus to the close button when opened, restores focus to the previously focused element when closed, and can be dismissed with the `Escape` key or by clicking the overlay — in addition to the visible close button.

**Q4. What happens if `localStorage` contains corrupted JSON?**
`safeGet()` wraps `JSON.parse` in a `try/catch`. If parsing fails, it logs the error and returns the caller-supplied fallback (usually an empty array), so the page renders an empty cart/wishlist instead of crashing.

**Q5. How is the "Product of the Day" chosen, and why is it deterministic?**
It's calculated as `Math.floor(Date.now() / 86400000) % PRODUCTS.length` — dividing the current timestamp by the number of milliseconds in a day gives a whole number that only changes once every 24 hours, so every visitor sees the same featured product on a given day without needing a server.

**Q6. How does the image fallback work?**
Every product `<img>` has an `onerror="handleImageError(this)"` attribute. If the browser can't load the real file, `handleImageError` swaps the `src` for a small inline SVG (built as a base64 data URI) showing the product's initials on a brand-coloured background, so the layout never shows a broken-image icon.

**Q7. Where is the service worker registered, and what does it cache?**
It's registered in `main.js`'s `DOMContentLoaded` listener on every page. On install, `service-worker.js` pre-caches the "app shell" — every HTML page, the stylesheet, and every JS file — using the Cache API, and serves those files from cache first so the site still loads with no network connection.

**Q8. Why use `IntersectionObserver` instead of a scroll event listener for reveal animations?**
`IntersectionObserver` only fires callbacks when an element's visibility actually changes relative to the viewport, which is far cheaper than a `scroll` listener that runs constantly and would need manual position math on every event.

**Q9. How is checkout validation implemented without any library?**
`validateCheckoutForm()` checks each field with small helper functions (`validateNotEmpty`, `validateEmail`) and a check for a selected radio button, adding an `is-invalid` class to the relevant `.field` wrapper (which reveals its error message via CSS) whenever a check fails, and blocks form submission until every check passes.

**Q10. How would you extend this project to use a real backend?**
The `localStorage` calls are already isolated inside a handful of helper functions (`getCart`/`saveCart`, `getWishlist`/`saveWishlist`, the order/feedback storage in `checkout.js`/`feedback.js`). Swapping those internals for `fetch()` calls to a real API would let the rest of the codebase — rendering, validation, UI state — carry on unchanged.
