# PayToday WooCommerce Plugin

A secure and easy-to-use payment gateway plugin that integrates PayToday payments with WooCommerce stores.

## Features

- **Seamless Integration**: Easy integration with WooCommerce checkout process
- **Secure Payments**: Process payments securely through PayToday's payment gateway
- **Sandbox Mode**: Test payments in sandbox environment before going live
- **Order Management**: Automatic order status updates based on payment results
- **Custom Return URLs**: Configurable return URLs for successful payments
- **WordPress Admin Integration**: Full integration with WordPress admin dashboard

## Requirements

- **WordPress**: 5.0 or higher
- **WooCommerce**: 3.0 or higher
- **PHP**: 7.0 or higher
- **PayToday Account**: Active PayToday merchant account with API credentials

## Installation

### Method 1: WordPress Admin Dashboard

1. Download the latest release from the [Releases](../../releases) page
2. Log in to your WordPress admin dashboard
3. Navigate to `Plugins > Add New > Upload Plugin`
4. Choose the downloaded zip file and click `Install Now`
5. Activate the plugin after installation

### Method 2: Manual Installation

1. Download and extract the plugin files
2. Upload the `paytoday_woo` folder to `/wp-content/plugins/`
3. Activate the plugin through the WordPress admin `Plugins` menu

### Method 3: FTP Upload

1. Download and extract the plugin files
2. Upload the entire `paytoday_woo` folder to your `/wp-content/plugins/` directory via FTP
3. Activate the plugin in your WordPress admin panel

## Configuration

1. Navigate to `WooCommerce > Settings > Payments`
2. Find "PayToday" in the payment methods list and click `Manage`
3. Configure the following settings:

### Required Settings

| Setting | Description |
|---------|-------------|
| **Enable/Disable** | Enable PayToday payment gateway |
| **Title** | Payment method title shown to customers |
| **Description** | Payment method description shown at checkout |
| **Shop Key** | Your PayToday shop key (from PayToday dashboard) |
| **Shop Handle** | Your PayToday shop handle |
| **Private Key** | Your PayToday private key |
| **Return URL** | URL where customers return after payment |

### Optional Settings

| Setting | Description | Default |
|---------|-------------|---------|
| **Sandbox Mode** | Enable for testing payments | Disabled |
| **Service URL** | PayToday service endpoint | Auto-configured |

## Getting PayToday API Credentials

1. Sign up for a PayToday merchant account at [PayToday](https://paytoday.com)
2. Log in to your PayToday dashboard
3. Navigate to the 'API Keys' or 'Developer' section
4. Generate or copy your:
   - Shop Key
   - Shop Handle  
   - Private Key

## Payment Flow

1. Customer selects PayToday as payment method at checkout
2. Customer is redirected to PayToday's secure payment page
3. Customer completes payment on PayToday platform
4. Customer is redirected back to your store
5. Order status is automatically updated based on payment result

## Testing

### Sandbox Mode

1. Enable "Sandbox Mode" in the plugin settings
2. Use test credentials provided by PayToday
3. Process test transactions to verify integration
4. Disable sandbox mode when ready for live payments

## Changelog

### Version 1.1.0
- Updated plugin structure and documentation
- Improved PayToday payment gateway integration
- Enhanced error handling and logging
- Better WordPress coding standards compliance

### Version 1.0.0
- Initial release with PayToday plugin integration
- Supports PayToday payments for WooCommerce
- Basic payment processing and order management

## Support

- **Phone**: [+264814578396](tel:+264814578396)
- **Website**: [https://paytoday.com](https://paytoday.com)
- **Documentation**: [PayToday Developer Docs](https://site.paytoday.com.na/help-center/custom-web-integrations/)
