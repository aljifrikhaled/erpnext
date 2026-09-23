# Technical Assessment: ERPNext Custom Frappe App

**Environment Verified:**
- ERPNext Repository: https://github.com/frappe/erpnext
- ERPNext Version: 16.35.0 (verified via `erpnext/__init__.py`)
- Commit: `12cd563fb9a79731f75ae2a45b1446a0a2dd9e74`

### 1. Recommended Implementation for Workflows and UI/UX Changes
To implement workflows and UI changes while keeping the ERPNext repository completely unmodified, the custom app should rely on standard Frappe extension mechanisms declared in the custom app's `hooks.py`:
- **Workflows & Server-Side Logic:** Use `doc_events` in `hooks.py` to intercept document lifecycle events (e.g., `validate`, `on_submit`) instead of modifying ERPNext controllers. To completely replace a standard ERPNext method, use `override_whitelisted_methods`. For extending DocType classes, use `extend_doctype_class` (as seen implemented in ERPNext for `Address`).
- **UI/UX Changes (Client-Side):** Use `doctype_js` and `page_js` in the custom app's `hooks.py` to inject custom JavaScript into specific standard ERPNext DocTypes or Pages. To inject global scripts or styles, use `app_include_js`, `app_include_css`, or `web_include_css`.
- **Workspaces:** Define new Workspaces as custom JSON configurations within the custom app (not modifying ERPNext's standard workspaces) and export them.

### 2. Dependencies on ERPNext Code
The custom app will depend on ERPNext's Python and JS code.
- **Python Imports:** The custom app will import standard models or utilities, e.g., extending standard controllers located in `erpnext/controllers/` (such as `accounts_controller.py` or `stock_controller.py`). It may also import utility functions like `get_default_company()` from `erpnext/__init__.py`.
- **Overrides:** The custom app may use `override_whitelisted_methods` to redefine ERPNext methods at runtime. For example, ERPNext overrides `frappe.www.contact.send_message` to `erpnext.templates.utils.send_message`. The custom app would do exactly the same in its `hooks.py`.
- **Internal APIs:** No ERPNext files should be directly edited or copied.

### 3. Runtime Boundaries (Server-Side vs. Client-Side)
- **Server-Side (Python):** Any code executing via `doc_events`, `override_whitelisted_methods`, `extend_doctype_class`, or Frappe API endpoints (`@frappe.whitelist()`) runs in the server environment executing as Python.
- **Client-Side (Browser):** Any code specified in `doctype_js`, `page_js`, `app_include_js`, `app_include_css`, and assets in the custom app's `public/` directory (like `public/js/*.js`, `public/scss/*.scss`) executes in the customer's browser.
- **Templates:** Server-side rendering (Jinja templates) occurs on the server, but the resulting HTML, along with defined CSS/JS, is sent to the client browser.

### 4. Browser-Delivered Assets
- **Sources:** Assets delivered to the browser originate from the `public/` directory of the installed apps. In ERPNext, this includes `erpnext/public/js/` and `erpnext/public/scss/`. The custom app will serve assets from its own `custom_app/public/` directory.
- **Newly Written vs Copied:** The custom app assets will be newly written custom code. They should not be copied from ERPNext.
- **Build Output:** Assets are typically bundled. For example, ERPNext bundles files into `erpnext.bundle.js` and `erpnext.bundle.css` via build configurations (using build tools like esbuild underlying Frappe). The custom app will have its own bundles.
- **Transmission:** Yes, custom-app code (specifically JavaScript, CSS, icons, images, and HTML templates located in the app's `public` directory and included via `hooks.py`) is directly transmitted to subscribers' browsers.

### 5. Hosted-SaaS Design Delivery
In a standard Frappe/ERPNext SaaS deployment (where the customer is a subscriber, not an infrastructure administrator):
- The deployment responds with standard HTTP responses (HTML, JSON, JS, CSS, images).
- **No runnable copies:** The design **does not** send customers any app package (like a `.zip` or `.tar.gz` of the app), database backup, Virtual Machine image, Docker container, or server-side executable. The only executable code the customer receives is client-side JavaScript meant for browser execution.

### 6. Verification
Since the app does not yet exist, the assumptions can be verified with a minimal prototype:
- **Prototype:** Create a custom app with a `hooks.py` that includes a `doctype_js` for `Sales Invoice` linking to `public/js/custom_invoice.js`, and an `override_whitelisted_methods` overriding a basic API.
- **Verification:** Inspect the network tab in the browser developer tools when loading a Sales Invoice. The capture will show `custom_invoice.js` being downloaded by the browser. A review of the server file system and Frappe site configuration will confirm no VM/container images or backend source code archives are exposed via standard web routes.

### Summary
- **Recommended Implementation:** Extend via standard `hooks.py` extensions (`doc_events`, `doctype_js`, `override_whitelisted_methods`, etc.) to keep the ERPNext repository strictly unmodified.
- **Server-Side:** Python modules handling data logic, validation, API endpoints, and template rendering remain strictly on the server.
- **Client-Side (Browser):** Custom JavaScript (`public/js/`), CSS (`public/scss/`), and media files registered in the custom app are delivered to and executed within the subscriber's browser. No backend runnable image, container, or package is transmitted.
