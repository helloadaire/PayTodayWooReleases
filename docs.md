# PayToday WooCommerce Plugin: Developer Documentation

**Plugin:** PayToday Payment Gateway for WooCommerce  
**Current version:** 1.1.2  
**License:** GNU General Public License v3.0  

This document describes the development, update, and rollback processes so the next developer can maintain and extend the plugin.

---

## Table of Contents

1. [Overview & Architecture](#1-overview--architecture)
2. [Development Process](#2-development-process)
3. [Update Process](#3-update-process)
4. [Rollback Process](#4-rollback-process)
5. [Reference & Resources](#5-reference--resources)

---

## 1. Overview & Architecture

### 1.1 Purpose

The plugin integrates PayToday as a WooCommerce payment gateway. It supports:

- **Classic checkout** and **WooCommerce Blocks** (cart/checkout)
- **Sandbox** and **live** environments (separate credentials and API base URLs)
- **Popup-based payment flow**: configuration intent → payment intent → PayToday hosted page in popup → status polling (AJAX + optional cron) and return URL handling

### 1.2 Technology Stack

| Layer | Technology |
|-------|------------|
| **Backend** | PHP 7.0+, WordPress 5.0+, WooCommerce 3.0+ |
| **Updates** | [Plugin Update Checker](https://github.com/YahnisElsts/plugin-update-checker) (PUC) v5 — custom update server |
| **Frontend (Blocks)** | `wp-element`, `wc-blocks-registry`, `wc-settings`; build output in `build/` |
| **APIs** | PayToday REST: JWT tokens; endpoints differ by environment |

### 1.3 Directory Structure

```
paytoday_woo_v1.1.2/   (or paytoday-woocommerce/)
├── paytoday-woocommerce.php   # Main plugin file: bootstrap, update checker, rollback, gateway class
├── includes/
│   └── class-wc-PayToday-gateway-blocks-support.php   # WooCommerce Blocks integration
├── build/
│   ├── index.js
│   └── index.asset.php
├── plugin-update-checker/     # Third-party PUC library (do not modify)
├── README.md                  # User-facing readme
├── readme.txt                 # WordPress.org-style readme (if used)
└── DEVELOPER.md               # This file
```

### 1.4 Key Components

| Component | Location | Responsibility |
|-----------|----------|----------------|
| **Gateway registration** | `paytoday-woocommerce.php` | `payToday_add_gateway_class`, `payToday_init_gateway_class` |
| **Gateway class** | `WC_PayToday_Gateway` (same file) | Settings, `process_payment`, configuration/payment intent APIs, JWT decode, status polling |
| **Configuration intent** | `create_configuration_intent()` | Gets auth token from PayToday (`/web/configuration/intent/`) |
| **Payment intent** | `create_payment_intent()` | Creates payment and gets `payment_url` + `payment_token` |
| **Status checks** | `query_payment_intent()`, `check_payment_status()`, AJAX/cron | Poll PayToday for transaction status |
| **Return handler** | `handle_paytoday_return()` | `woocommerce_api_paytoday_return` — front-channel return with `status`, `reference`, `invoice_number` |
| **Blocks support** | `includes/class-wc-PayToday-gateway-blocks-support.php` | Registers PayToday for Blocks; script in `build/index.js` |
| **Update checker** | Top of main file + `plugin-update-checker/` | PUC reads `update-info.json` for version and package URL |
| **Rollback** | Main file: filters + `admin_post_my_plugin_rollback` | Rollback link visibility + install of previous version from fixed ZIP URL |

### 1.5 External Dependencies

- **Update manifest:** `https://raw.githubusercontent.com/helloadaire/PayToday-Update-JSON/master/update-info.json`
- **Release ZIPs:** GitHub Releases — e.g. `https://github.com/helloadaire/PayTodayWooReleases/releases/download/vX.Y.Z/...`
- **PayToday APIs:**  
  - Sandbox: `https://admin.today-ww.net`  
  - Live: `https://admin.today.com.na`

---

## 2. Development Process

### 2.1 Local Setup

1. **Requirements:** PHP 7.0+, WordPress 5.0+, WooCommerce 3.0+.
2. **Install:** Place plugin in `wp-content/plugins/` (e.g. `paytoday-woocommerce/`).
3. **Activate:** WooCommerce must be active first; then activate PayToday.
4. **Config:** WooCommerce → Settings → Payments → PayToday: set Live (and Sandbox) Shop Key, Shop Handle, and optional keys. For sandbox, fill Sandbox Shop Key and Sandbox Shop Handle.

### 2.2 Code Layout (Main File)

- **Lines ~1–185:** Bootstrap: plugin header, Plugin Update Checker, rollback cache/links/handler, admin notices.
- **Lines ~188–275:** Gateway registration and `WC_PayToday_Gateway` constructor (settings load, service URL, hooks).
- **Lines ~278–356:** `init_form_fields()` (all gateway options).
- **Lines ~358–443:** `process_payment()` — configuration intent → payment intent → redirect to payment URL.
- **Lines ~345–453:** `create_configuration_intent()`.
- **Lines ~455–649:** `create_payment_intent()`.
- **Lines ~451–457, 655–457:** `log()`, `decode_jwt()`.
- **Lines ~459–723:** `query_payment_intent()`, `schedule_payment_status_check()`, `check_payment_status()`, `manual_check_payment_status()`.
- **Lines ~525–616:** `add_payment_popup_script()` (inline JS for popup + AJAX polling).
- **Lines ~618–658:** `validate_fields()` (billing validation).
- **Lines ~662–683:** `handle_paytoday_return()`.
- **Lines ~686–753:** Cron/AJAX handlers and Blocks registration.

### 2.3 Coding Standards

- **PHP:** Follow [WordPress PHP Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/).
- **Naming:** Classes `WC_PayToday_*`; hooks/options prefixed (`paytoday_`, `payToday_` for legacy functions). Be consistent with existing style.
- **Security:** Use `wp_create_nonce()`, `wp_verify_nonce()` for rollback; `current_user_can('update_plugins')`; sanitize/escape all inputs and outputs (`sanitize_text_field`, `esc_url`, `esc_js`, etc.).
- **Logging:**  
  - `error_log('PayToday Debug - ...')` for low-level debugging.  
  - `$this->log('...')` when WooCommerce debug logging is enabled (gateway option).

### 2.4 Making Code Changes

1. **Version:** Bump the `Version` in the plugin header of `paytoday-woocommerce.php` for any release.
2. **Avoid editing:** `plugin-update-checker/` — treat as vendored; upgrade only when necessary and test thoroughly.
3. **Blocks:** If you change the Blocks integration, rebuild frontend assets that output to `build/` and ensure `index.asset.php` is updated if dependencies change.
4. **APIs:** Any change to PayToday request/response (fields, URLs, JWT structure) must be validated against both sandbox and live; document expected payloads.

### 2.5 Testing Checklist

- [ ] Enable **Sandbox**; set Sandbox Shop Key and Sandbox Shop Handle; run a test payment end-to-end.
- [ ] Enable **Debug** in gateway settings; confirm logs in WooCommerce log and `wp-content/debug.log` (if `WP_DEBUG_LOG` is on).
- [ ] Test **return URL**: complete payment and ensure `handle_paytoday_return` runs and order status/notes update.
- [ ] Test **status polling**: AJAX and, if used, cron; confirm order moves to completed/failed as expected.
- [ ] Test **Blocks checkout** if your store uses it.
- [ ] Test with **Sandbox off** (live credentials) in a staging environment before production.

### 2.6 Debugging

- **PayToday Debug:** Search logs for `PayToday Debug -` to trace configuration intent, payment intent, and status calls.
- **Rollback:** Search for `[PayToday Rollback]` to trace version checks and rollback actions.
- **WooCommerce log:** Source `paytoday` when debug mode is enabled for the gateway.

---

## 3. Update Process

### 3.1 How Updates Work

The plugin uses **Plugin Update Checker (PUC)** so updates are delivered **outside** the WordPress.org repository:

1. On WordPress admin load, PUC requests:  
   `https://raw.githubusercontent.com/helloadaire/PayToday-Update-JSON/master/update-info.json`
2. That JSON contains the **latest version** and **download URL** (ZIP).
3. If the installed version is lower, WordPress shows an update available and allows “Update now”.
4. WordPress then downloads the ZIP and runs the standard plugin upgrader.

No WordPress.org account or SVN is involved.

### 3.2 Update Manifest: `update-info.json`

**Repository:** [PayToday-Update-JSON](https://github.com/helloadaire/PayToday-Update-JSON) (or equivalent), file: `update-info.json` in the default branch (e.g. `master`).

**Expected structure (conceptual):**

```json
{
  "version": "1.1.2",
  "download_url": "https://github.com/helloadaire/PayTodayWooReleases/releases/download/v1.1.2/paytoday_woo_v1.1.2.zip",
  "requires": "5.0",
  "tested": "6.4",
  "requires_php": "7.0"
}
```

- **version** — Must be a valid version string (e.g. semver). PUC compares this to the plugin header `Version`.
- **download_url** — Direct link to the plugin ZIP. Must be the **exact** ZIP you want sites to install.
- Other fields may be used by PUC for compatibility info; keep them in sync with the plugin’s `readme.txt`/README if you use them.

**To release a new version:**

1. Bump `Version` in `paytoday-woocommerce.php`.
2. Build/create the plugin ZIP (e.g. from Git tag or release artifact).
3. Upload the ZIP to GitHub Releases (or the host used in `download_url`).
4. Update `update-info.json`: set `version` and `download_url` to the new ZIP URL.
5. Commit and push so the raw URL returns the new JSON.

### 3.3 Releasing a New Version (Step-by-Step)

1. **Code:** Implement changes; bump plugin header `Version` (e.g. to `1.1.3`).
2. **Test:** Run through the [Testing Checklist](#25-testing-checklist).
3. **Package:** Create a ZIP of the plugin directory (same structure as in repo; no `.git` or dev-only files).
4. **Release:** Create a new release on the **releases** repo (e.g. `PayTodayWooReleases`), tag e.g. `v1.1.3`, attach the ZIP.
5. **Manifest:** In the **update JSON** repo, set `version` to `1.1.3` and `download_url` to the new ZIP’s URL (e.g. GitHub release asset URL).
6. **Verify:** On a test site with an older version, open **Plugins** or **Updates** and confirm the update appears and installs correctly.

### 3.4 Caching and Update Visibility

- PUC may cache update info; sites might see the new version after a short delay.
- After an upgrade, the plugin clears the rollback transient `paytoday_latest_version` on `upgrader_process_complete` and `activated_plugin` so rollback logic sees the new “current” version.

---

## 4. Rollback Process

### 4.1 What Rollback Does

Rollback **reinstalls the previous known-good version** of the plugin:

1. User must have `update_plugins` capability.
2. User clicks the **Rollback** link in the plugin row (Plugins list).
3. Plugin verifies nonce and capability, then:
   - Deactivates the current plugin.
   - Deletes the current plugin directory.
   - Installs the plugin from a **hardcoded previous-version ZIP URL**.
   - Activates the newly installed plugin.
   - Redirects to Plugins page with `?rollback=success` or `?rollback=failed`.

So rollback is a **one-step revert** to a fixed previous version (e.g. 1.1.1 when current is 1.1.2).

### 4.2 When the Rollback Link Appears

The Rollback link is shown **only if the installed version is considered the “latest”**:

1. **Latest version source:** The plugin fetches `update-info.json` (same URL as PUC) and reads `version` from it. Result is cached in transient `paytoday_latest_version` for 1 hour.
2. **Comparison:** If **installed version ≥ manifest version**, the plugin is treated as “latest” and the Rollback link is shown. If a **newer** version exists in the manifest, the link is **hidden** (to avoid rolling back when an update is available).
3. **Cache:** If the manifest can’t be fetched or is invalid, `paytoday_latest_version` may be set to `'error'`; in that case the link is still shown so admins can roll back if needed.

So: **Rollback link = “this site is on the latest version (or we couldn’t determine otherwise), so offer rollback to the previous release.”**

### 4.3 Rollback Link Implementation (Code Reference)

- **Filter:** `plugin_action_links_{plugin_basename}` — adds the Rollback link.
- **Logic:** Read plugin header `Version`; get `version` from `update-info.json` (with transient cache); if current ≥ that version (or cache error), add link.
- **Link target:** `admin-post.php?action=my_plugin_rollback&_wpnonce=...`
- **Handler:** `admin_post_my_plugin_rollback` — capability + nonce check, then deactivate, delete, install from ZIP, activate, redirect.

### 4.4 Changing the Rollback Target Version

The URL of the **previous** version is **hardcoded** in the rollback handler:

```php
$previous_version_zip = 'https://github.com/helloadaire/PayTodayWooReleases/releases/download/v1.1.1/paytoday_woo_v1.1.1.zip';
```

**When you release a new version (e.g. 1.1.3):**

1. In the same release cycle, update this line so that “previous” points to the version **before** the one you just released.  
   - Example: after releasing 1.1.3, set rollback to install **1.1.2** (so rollback from 1.1.3 → 1.1.2).
2. Ensure that ZIP exists at the given URL (e.g. on GitHub Releases) and is the exact package you want to restore.

So each new release should ship with the rollback URL pointing to the **immediately previous** version.

### 4.5 Manual Rollback (Without the Link)

If the plugin is broken and the Rollback link can’t be used:

1. Deactivate the plugin from the Plugins screen (or via WP-CLI: `wp plugin deactivate paytoday-woocommerce`).
2. Delete the plugin folder under `wp-content/plugins/` (e.g. `paytoday-woocommerce/`).
3. Download the desired previous release ZIP from the releases repo.
4. In WordPress: **Plugins → Add New → Upload Plugin**, upload the ZIP, install, activate.

Alternatively, replace the plugin folder via FTP/SFTP with the contents of the previous version ZIP, then activate.

### 4.6 Rollback Security

- **Nonce:** `my_plugin_rollback` — must be present and valid in the request.
- **Capability:** `update_plugins` — only users who can update plugins can roll back.
- **Admin-only:** Handler runs in `admin_post_`; no public-facing endpoint.

### 4.7 After Rollback

- Success: redirect to `plugins.php?rollback=success`; admin notice “Plugin rolled back successfully and activated.”
- Failure: redirect to `plugins.php?rollback=failed`; notice “Rollback failed. Check debug.log for details.”
- Logs: All rollback steps log with prefix `[PayToday Rollback]`; check `debug.log` if something fails (e.g. delete folder, install, or activate).

---

## 5. Reference & Resources

### 5.1 Important URLs

| Purpose | URL |
|--------|-----|
| Update manifest (JSON) | `https://raw.githubusercontent.com/helloadaire/PayToday-Update-JSON/master/update-info.json` |
| Release ZIPs (example) | `https://github.com/helloadaire/PayTodayWooReleases/releases/download/v1.1.1/paytoday_woo_v1.1.1.zip` |
| PayToday sandbox API | `https://admin.today-ww.net` |
| PayToday live API | `https://admin.today.com.na` |

### 5.2 Key Hooks & Options

- **Actions:** `plugins_loaded` (gateway init), `woocommerce_api_paytoday_return` (return handler), `paytoday_check_payment_status` (cron), `wp_ajax_paytoday_check_status` / `wp_ajax_nopriv_paytoday_check_status` (AJAX status).
- **Filters:** `woocommerce_payment_gateways`, `plugin_action_links_...` (rollback link).
- **Options:** `woocommerce_paytoday_settings` (gateway settings). Transient: `paytoday_latest_version` (rollback version cache).

### 5.3 Order Meta (PayToday)

- `_paytoday_authorization_token` — Bearer token for PayToday API.
- `_paytoday_payment_token` — Payment/intent identifier for status lookup.
- `_paytoday_payment_url` — URL opened in popup.
- `_paytoday_status_check_started`, `_paytoday_polling_active` — used for status polling flow.

### 5.4 Document History

| Date | Change |
|------|--------|
| 2025-01-30 | Initial developer documentation (development, update, rollback). |

---

*For end-user setup and troubleshooting, see **README.md**. For PayToday API details, refer to PayToday’s official documentation.*
