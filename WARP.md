# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

Project overview
- This is a static single-page site driven by index.html, with CSS in css/ and JavaScript in js/.
- Frontend behavior is jQuery-based (js/jquery-3.2.1.min.js) with a bundled plugins file (js/plugins.js) providing: Waypoints, Parallax, Masonry + ImagesLoaded, Slick slider, jQuery Validation, and Mailchimp AjaxChimp. App-specific logic lives in js/main.js.
- The contact form submits via AJAX to inc/sendEmail.php, which relays messages using PHP’s mail(). Newsletter signup uses Mailchimp via the cfg.mailChimpURL set in js/main.js.

Commands you’ll use
- Serve the site with PHP (enables the contact form backend):
  - php -S 127.0.0.1:8000 -t .
  - Then open http://127.0.0.1:8000
- Static-only preview (no PHP, contact form will fail):
  - python3 -m http.server 8000
  - Then open http://127.0.0.1:8000
- Exercise the contact form endpoint locally (ensures PHP handler wiring works):
  - curl -X POST \
    -d "contactName=Test User" \
    -d "contactEmail=test@example.com" \
    -d "contactSubject=Hello" \
    -d "contactMessage=This is a test message with more than 15 characters." \
    http://127.0.0.1:8000/inc/sendEmail.php
- Build: none (assets are already committed; there is no bundler or package manager config).
- Lint/format: none configured in-repo.
- Tests: none present; there is no test runner configured.

High-level architecture and data flow
- Entry point: index.html defines sections (home, about, services, works, clients, contact) and links all assets. It also embeds a Google Maps iframe and renders the newsletter form (id="mc-form").
- Styling: css/base.css (reset/utilities), css/vendor.css (third‑party styles), css/main.css (site-specific styles). Font assets are under css/font-awesome, css/micons, and fonts/.
- Behavior:
  - js/plugins.js: vendor bundle that provides the site’s UI primitives (e.g., smooth scroll, parallax, masonry grid, slick carousels), jQuery Validation, and AjaxChimp.
  - js/main.js: initializes effects (preloader, off‑canvas menu, AOS, counters, masonry, sliders), wires smooth scrolling, and contains two integrations:
    - Newsletter: $('#mc-form').ajaxChimp({ url: cfg.mailChimpURL }). Update cfg.mailChimpURL to your Mailchimp list URL.
    - Contact form: $('#contactForm').validate(...). On submit it POSTs serialized form data to inc/sendEmail.php and expects the literal string "OK" for success.
- Backend hook: inc/sendEmail.php validates inputs and calls mail() using $siteOwnersEmail. Set $siteOwnersEmail to a real inbox before deploying. For local dev, note that PHP mail() may be a no‑op without an MTA.

Repo conventions and gotchas
- There is no Node/Poetry/etc. Use a simple static or PHP server for local work.
- Contact form success is string-matched on "OK"; any other output is treated as an error message and shown in .message-warning.
- If you only use a static server, AJAX to inc/sendEmail.php will fail (PHP won’t execute). Use the PHP built‑in server when testing forms.
- Mailchimp signup uses JSONP; ensure cfg.mailChimpURL in js/main.js points to your list’s "post?" URL (AjaxChimp converts it to "post-json?").
