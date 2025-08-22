# Postman Import Instructions

This guide explains how to import the BtqFinance Merchant API collection into Postman.

## Files Included

1. **`BtqFinance_Merchant_API.postman_collection.json`** - Main API collection
2. **`BtqFinance_Merchant_API.postman_environment.json`** - Environment variables
3. **`POSTMAN_IMPORT_README.md`** - This instruction file

## Import Steps

### 1. Import Collection

1. Open Postman
2. Click **Import** button (top left)
3. Choose **File** tab
4. Select `BtqFinance_Merchant_API.postman_collection.json`
5. Click **Import**

### 2. Import Environment

1. Click **Import** button again
2. Choose **File** tab
3. Select `BtqFinance_Merchant_API.postman_environment.json`
4. Click **Import**

### 3. Configure Environment Variables

1. Click the **Environments** dropdown (top right)
2. Select **"BtqFinance Merchant API Environment"**
3. Update the following variables:

#### Required Variables:

-   `base_url` - Your API base URL (e.g., `https://pay.btqfi.com`)
-   `merchant_token` - Your merchant authentication token

#### Optional Variables (can be left as default):

-   `country_code` - Default country code with payment methods (default: `POL`)
-   `country_code_no_methods` - Country code without payment methods (default: `ALB`)
-   `test_amount` - Test payment amount (default: `100.0`)
-   `test_customer_email` - Test customer email
-   `test_customer_name` - Test customer name
-   `test_customer_ip` - Test customer IP
-   `success_callback` - Success callback URL
-   `failure_callback` - Failure callback URL
-   `postback_url` - Webhook URL

## Available Requests

### 1. Get Allowed Countries

-   **Method:** GET
-   **URL:** `{{base_url}}/merchant/allowed-countries`
-   **Description:** Get list of supported countries

### 2. Get Country Details

-   **Method:** GET
-   **URL:** `{{base_url}}/merchant/allowed-countries/{{country_code}}`
-   **Description:** Get detailed country information with payment methods

### 3. Get Country Details (No Payment Methods)

-   **Method:** GET
-   **URL:** `{{base_url}}/merchant/allowed-countries/ALB`
-   **Description:** Get detailed information about Albania (country without payment methods)

### 4. Get Payment Methods

-   **Method:** GET
-   **URL:** `{{base_url}}/merchant/methods`
-   **Description:** Get all available payment methods

### 5. Create Payment (With Payment Method)

-   **Method:** POST
-   **URL:** `{{base_url}}/merchant/payment`
-   **Description:** Create a new payment with VISA method (for countries that require payment method)

### 6. Create Payment (Without Payment Method)

-   **Method:** POST
-   **URL:** `{{base_url}}/merchant/payment`
-   **Description:** Create a payment without payment method (for countries that don't require it)

### 7. Create Payment (With Optional Fields)

-   **Method:** POST
-   **URL:** `{{base_url}}/merchant/payment`
-   **Description:** Create payment with all optional fields

### 8. Check Payment Status

-   **Method:** GET
-   **URL:** `{{base_url}}/merchant/payment/{{payment_id}}`
-   **Description:** Check payment status by ID

## Features

### Automatic Tests

Each request includes automatic tests that:

-   Verify status code is 200
-   Check response is valid JSON
-   Validate response has `success` field
-   Automatically save payment ID for status checks
-   **NEW:** Validate payment method codes are strings (VISA, MASTERCARD)

### Environment Variables

All requests use environment variables for easy configuration:

-   No need to manually update URLs or tokens
-   Easy switching between environments (dev/staging/prod)
-   Secure token storage

### Multiple Payment Examples

Three different payment creation examples:

1. **With Payment Method** - For countries that require payment method (e.g., POL)
2. **Without Payment Method** - For countries that don't require payment method (e.g., ALB)
3. **Full** - All optional fields included

### Payment Method Requirements

**Important:** Payment method requirements depend on the country:

-   **If a country has payment methods available:** `paymentMethod` is **REQUIRED**
-   **If a country has no payment methods:** `paymentMethod` is **OPTIONAL** (can be omitted)

Always check the country details endpoint first to see available payment methods.

## Testing Workflow

1. **Start with "Get Allowed Countries"** to see available countries
2. **Use "Get Country Details"** to see payment methods for specific country
3. **Use "Get Country Details (No Payment Methods)"** to see example without payment methods
4. **Create a payment** using appropriate example based on country requirements
5. **Check payment status** using the returned payment ID

## Troubleshooting

### Common Issues:

1. **401 Unauthorized**

    - Check that `merchant_token` is set correctly
    - Verify token format: `Bearer YOUR_TOKEN`

2. **404 Not Found**

    - Verify `base_url` is correct
    - Check that the API endpoint exists

3. **400 Bad Request**

    - Review request body format
    - Check required fields are present
    - Verify data types (amount should be number, not string)
    - **NEW:** Check if payment method is required for the selected country

4. **Environment Variables Not Working**
    - Make sure environment is selected in dropdown
    - Check variable names match exactly (case-sensitive)
    - Verify variables are enabled

### Getting Help

If you encounter issues:

1. Check the API documentation
2. Verify your merchant token is valid
3. Test with minimal request first
4. Check Postman console for detailed error messages
