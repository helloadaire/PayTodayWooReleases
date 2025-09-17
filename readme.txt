=== PayToday WooCommerce Plugin ===
Contributors: PayToday
Donate link: https://paytoday.com/donate
Tags: payment gateway, woocommerce, paytoday, payments, e-commerce
Requires at least: 5.0
Tested up to: 6.0
Requires PHP: 7.0
Stable tag: 1.1.0
License: GPL-2.0+
License URI: http://www.gnu.org/licenses/gpl-2.0.html

== Description ==

Seamless WooCommerce integration for PayToday payments.

The PayToday WooCommerce Plugin enables merchants to easily accept PayToday payments on their WooCommerce stores. With just a few settings, you can integrate PayToday as a payment gateway and start processing transactions securely.

== Installation ==

1. Upload the `paytoday_woo` folder to the `/wp-content/plugins/` directory.
2. Activate the plugin through the 'Plugins' menu in WordPress.
3. Go to `WooCommerce > Settings > Payments` and enable the PayToday payment gateway.
4. Configure your `Shop Key`, `Shop Handle`, and `Private Key` in the settings to start accepting PayToday payments.

== Frequently Asked Questions ==

= What is PayToday? =
PayToday is a payment gateway that allows e-commerce merchants to accept payments securely via their WooCommerce store. It supports payments through PayToday using a Shop Key and Private Key for authentication.

= How do I get my Shop Key and Private Key? =
You can obtain these keys by signing up on [PayToday](https://paytoday.com) and accessing the 'API Keys' section in your account.

= How do I configure PayToday in WooCommerce? =
- Navigate to `WooCommerce > Settings > Payments`.
- Find PayToday in the list of available payment gateways and click 'Manage'.
- Enter your Shop Key, Shop Handle, Private Key, and Return URL.
- Save your settings.

= What happens when a payment is processed? =
When a customer completes a payment, PayToday redirects them to the `Return URL` you specified in the plugin settings. If successful, the order is marked as complete in WooCommerce.

== Changelog ==

= 1.1.0 =
* Updated plugin structure and documentation.
* Improved PayToday payment gateway integration.

= 1.0.0 =
* Initial release with PayToday plugin integration.
* Supports PayToday payments for WooCommerce.

== Upgrade Notice ==

= 1.1.0 =
Updated version with improved plugin structure and documentation.

= 1.0.0 =
First release. No upgrades necessary.

== Screenshots ==

1. PayToday payment gateway settings in WooCommerce.
2. WooCommerce checkout page with PayToday as a payment method.
3. Successful payment redirect after processing.

== Acknowledgements ==

- Thanks to PayToday for providing the API for integrating payments.

== Support ==

For support, please contact us at [support@paytoday.com](mailto:support@paytoday.com).