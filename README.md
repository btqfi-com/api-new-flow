# Merchant API Integration Guide

This guide helps developers integrate payment system into their applications.

## Quick Start

### 1. Authentication

All API requests require a merchant token. Contact us to get your unique token.

```javascript
// Include your merchant token in headers
const headers = {
    Authorization: "Bearer YOUR_MERCHANT_TOKEN",
    "Content-Type": "application/json",
};
```

### 2. Base URL

```
https://pay.btqfi.com
```

### 3. Payment Limits

-   **Minimum amount:** $15.00
-   **Maximum amount:** $2,000.00
-   **Supported payment methods:** VISA, MASTERCARD

**Note:** Each country may have different minimum and maximum amounts. Check the country details endpoint for specific limits.

### 4. Payment Method Requirements

**Important:** Payment method requirements depend on the country:

-   **If a country has payment methods available:** `paymentMethod` is **REQUIRED**
-   **If a country has no payment methods (`paymentMethods: null`):** `paymentMethod` is **OPTIONAL** (can be omitted)

Always check the country details endpoint first to see available payment methods.

## API Endpoints

### Get Available Countries

**Purpose:** Get list of countries where payments are supported.

```javascript
// JavaScript example
const response = await fetch("/merchant/allowed-countries", {
    method: "GET",
    headers: headers,
});

const countries = await response.json();
console.log(countries.data);
```

**Response:**

```json
{
    "success": true,
    "data": [
        {
            "country": "Poland",
            "code": "POL",
            "maxAmount": 2000,
            "minAmount": 15,
            "paymentMethods": [
                {
                    "code": "VISA",
                    "name": "Visa"
                },
                {
                    "code": "MASTERCARD",
                    "name": "MasterCard"
                }
            ]
        },
        {
            "country": "Albania",
            "code": "ALB",
            "maxAmount": 2000,
            "minAmount": 15,
            "paymentMethods": null
        }
    ]
}
```

### Get Country Details

**Purpose:** Get detailed information about a specific country including available payment methods.

```javascript
// JavaScript example
const countryCode = "POL";
const response = await fetch(`/merchant/allowed-countries/${countryCode}`, {
    method: "GET",
    headers: headers,
});

const countryDetails = await response.json();
console.log(countryDetails.data.paymentMethods);
```

**Response:**

```json
{
    "success": true,
    "data": {
        "country": "Poland",
        "code": "POL",
        "maxAmount": 2000,
        "minAmount": 15,
        "paymentMethods": [
            {
                "code": "VISA",
                "name": "Visa"
            },
            {
                "code": "MASTERCARD",
                "name": "MasterCard"
            }
        ]
    }
}
```

**Note:** If `paymentMethods` is `null`, payment method is optional for this country.

**Example for country without payment methods:**

```javascript
// JavaScript example for Albania
const countryCode = "ALB";
const response = await fetch(`/merchant/allowed-countries/${countryCode}`, {
    method: "GET",
    headers: headers,
});

const countryDetails = await response.json();
console.log(countryDetails.data.paymentMethods); // null
```

**Response for country without payment methods:**

```json
{
    "success": true,
    "data": {
        "country": "Albania",
        "code": "ALB",
        "maxAmount": 2000,
        "minAmount": 15,
        "paymentMethods": null
    }
}
```

### Get Payment Methods

**Purpose:** Get all available payment methods.

```javascript
// JavaScript example
const response = await fetch("/merchant/methods", {
    method: "GET",
    headers: headers,
});

const methods = await response.json();
console.log(methods.data);
```

**Response:**

```json
{
    "success": true,
    "data": [
        {
            "code": "VISA",
            "name": "Visa"
        },
        {
            "code": "MASTERCARD",
            "name": "MasterCard"
        }
    ]
}
```

### Create Payment

**Purpose:** Create a new payment transaction and get payment URL.

```javascript
// JavaScript example
const paymentData = {
    amount: 100.0,
    countryCode: "POL",
    successCallback: "https://yoursite.com/success",
    failureCallback: "https://yoursite.com/failure",
    postbackUrl: "https://yoursite.com/webhook",
    customerEmail: "customer@example.com",
    customerName: "John Doe",
    customerIp: "192.168.1.1",
    // paymentMethod is REQUIRED if country has payment methods available
    paymentMethod: "VISA",
};

const response = await fetch("/merchant/payment", {
    method: "POST",
    headers: headers,
    body: JSON.stringify(paymentData),
});

const result = await response.json();

if (result.success) {
    // Redirect user to payment page
    window.location.href = result.data.paymentUrl;
} else {
    console.error("Payment creation failed:", result.message);
}
```

**Request Parameters:**

-   `amount` (number, required): Payment amount (check country limits for min/max)
-   `countryCode` (string, required): Country code (POL, etc.)
-   `successCallback` (string, required): URL to redirect after successful payment
-   `failureCallback` (string, required): URL to redirect after failed payment
-   `postbackUrl` (string, required): Webhook URL for payment notifications
-   `customerEmail` (string, required): Customer email address
-   `customerName` (string, required): Customer full name
-   `customerIp` (string, required): Customer IP address
-   `paymentMethod` (string, conditional): Payment method code (VISA, MASTERCARD) - **REQUIRED if country has payment methods, OPTIONAL if country has no payment methods**
-   `externalClientId` (string, optional): External client identifier
-   `paymentTimeMaxTimestamp` (number, optional): Maximum payment time timestamp
-   `paymentTimeRealtiveTimestamp` (number, optional): Relative payment time timestamp

**Response:**

```json
{
    "success": true,
    "data": {
        "paymentId": "payment_123456",
        "paymentUrl": "https://payment-gateway.com/pay/123456",
        "id": "1"
    }
}
```

### Check Payment Status

**Purpose:** Check the current status of a payment.

```javascript
// JavaScript example
const paymentId = "payment_123456";
const response = await fetch(`/merchant/payment/${paymentId}`, {
    method: "GET",
    headers: headers,
});

const status = await response.json();
console.log("Payment status:", status.data.status);
```

**Response:**

```json
{
    "success": true,
    "data": {
        "paymentId": "payment_123456",
        "status": "success",// 'created', 'invoice_created', 'pending', 'success', 'failed', 'cancelled', 'expired'
        "amount": 100.0,
        "createdAt": "2024-01-01T12:00:00Z",
        "updatedAt": "2024-01-01T12:05:00Z"
    }
}
```

## Error Handling

### Common Error Codes

| Code                           | Message                                         | Description                              | Solution                                       | Common Causes                                 |
| ------------------------------ | ----------------------------------------------- | ---------------------------------------- | ---------------------------------------------- | --------------------------------------------- |
| `COUNTRY_CODE_NOT_SPECIFIED`   | Country code not specified                      | Missing country code parameter           | Provide a valid country code                   | Empty or missing countryCode field            |
| `COUNTRY_NOT_FOUND`            | Country not found                               | Invalid country code                     | Use a country code from allowed countries list | Using unsupported country (e.g., "XXX")       |
| `PAYMENT_ID_REQUIRED`          | Payment ID is required                          | Missing payment ID parameter             | Provide a valid payment ID                     | Empty payment ID in URL parameter             |
| `MISSING_REQUIRED_FIELDS`      | Missing required fields                         | Required parameter not provided          | Check all required fields                      | Missing amount or customer data               |
| `AMOUNT_LESS_THAN_MINIMUM`     | Amount is less than the minimum amount          | Amount below country minimum             | Use amount >= country's minimum amount         | Amount below country-specific minimum         |
| `AMOUNT_GREATER_THAN_MAXIMUM`  | Amount is greater than the maximum amount       | Amount above country maximum             | Use amount <= country's maximum amount         | Amount above country-specific maximum         |
| `INVALID_PAYMENT_METHOD`       | Invalid payment method                          | Unsupported payment method               | Use VISA or MASTERCARD                         | Using "PAYPAL" or other unsupported methods   |
| `INVALID_COUNTRY_CODE`         | Country code is invalid                         | Invalid country code format              | Use valid country codes (POL, UKR, etc.)       | Using lowercase or wrong format (e.g., "pol") |
| `PAYMENT_METHOD_NOT_SUPPORTED` | Payment method is not supported in this country | Payment method not available for country | Choose different payment method or country     | VISA not available in specific country        |
| `PAYMENT_METHOD_REQUIRED`      | Payment method is required for this country     | Country requires payment method          | Provide payment method for this country        | Country has payment methods but none provided |
| `SOMETHING_WENT_WRONG`         | Something went wrong                            | Internal server error                    | Contact support                                | Database connection issues, server errors     |
| `TELEWORLD_PROVIDER_ERROR`     | TeleWorld provider error                        | Payment provider error                   | Try again later or contact support             | External payment system down                  |
| `PROVIDER_ERROR`               | Provider error                                  | External payment system error            | Try again later or contact support             | Payment gateway maintenance or errors         |

### Error Response Format

```json
{
    "success": false,
    "message": "Country code not specified",
    "code": "COUNTRY_CODE_NOT_SPECIFIED"
}
```

### Handling Errors in Code

```javascript
async function createPayment(paymentData) {
    try {
        const response = await fetch("/merchant/payment", {
            method: "POST",
            headers: headers,
            body: JSON.stringify(paymentData),
        });

        const result = await response.json();

        if (!result.success) {
            // Handle specific errors
            switch (result.code) {
                case "AMOUNT_LESS_THAN_MINIMUM":
                    alert(
                        "Amount is below the minimum for this country. Please check country limits."
                    );
                    break;
                case "AMOUNT_GREATER_THAN_MAXIMUM":
                    alert(
                        "Amount exceeds the maximum for this country. Please check country limits."
                    );
                    break;
                case "INVALID_PAYMENT_METHOD":
                    alert("Please select a valid payment method (VISA or MasterCard)");
                    break;
                case "COUNTRY_NOT_FOUND":
                    alert("Please select a valid country from the list");
                    break;
                case "PAYMENT_METHOD_NOT_SUPPORTED":
                    alert(
                        "This payment method is not supported in the selected country. Please choose a different method or country."
                    );
                    break;
                case "PAYMENT_METHOD_REQUIRED":
                    alert(
                        "Payment method is required for this country. Please select a payment method."
                    );
                    break;
                case "MISSING_REQUIRED_FIELDS":
                    alert(
                        "Please fill in all required fields: amount, country, customer email, and customer name"
                    );
                    break;
                case "INVALID_COUNTRY_CODE":
                    alert("Please use a valid 3-letter country code (e.g., POL, UKR)");
                    break;
                case "SOMETHING_WENT_WRONG":
                case "TELEWORLD_PROVIDER_ERROR":
                case "PROVIDER_ERROR":
                    alert(
                        "Payment service temporarily unavailable. Please try again in a few minutes or contact support if the problem persists."
                    );
                    break;
                default:
                    alert("Payment error: " + result.message);
            }
            return null;
        }

        return result.data;
    } catch (error) {
        console.error("Network error:", error);
        alert("Network error. Please check your internet connection and try again.");
        return null;
    }
}
```

## Webhook Integration

### Setting Up Webhooks

Your `postbackUrl` will receive notifications about payment status changes:

```javascript
// Example webhook handler (Node.js/Express)
app.post("/webhook", (req, res) => {
    const paymentData = req.body;

    console.log("Payment webhook received:", paymentData);

    // Process payment status update
    // Update your database
    // Send confirmation emails, etc.

    res.status(200).json({ success: true });
});
```

### Webhook Data Format

```json
{
    "status": "success", // The status of the payment could be "success"/"failure"/"pending"
    "paymentId": "<Payment id>",
    "externalClientId": "<external client id>"
}
```

## Best Practices

### 1. Always Check Country Details First

```javascript
async function createPaymentWithValidation(paymentData) {
    // First, get country details to check payment method requirements
    const countryResponse = await fetch(`/merchant/allowed-countries/${paymentData.countryCode}`, {
        headers: headers,
    });

    const countryData = await countryResponse.json();

    if (!countryData.success) {
        throw new Error("Invalid country code");
    }

    const country = countryData.data;

    // Check if payment method is required
    if (country.paymentMethods && country.paymentMethods.length > 0) {
        if (!paymentData.paymentMethod) {
            throw new Error("Payment method is required for this country");
        }

        // Validate payment method is supported
        const supportedMethods = country.paymentMethods.map((m) => m.code);
        if (!supportedMethods.includes(paymentData.paymentMethod)) {
            throw new Error("Payment method not supported in this country");
        }
    } else if (country.paymentMethods === null) {
        // Country has no payment methods, paymentMethod is optional
        console.log("Payment method is optional for this country");
    }

    // Validate amount limits
    if (paymentData.amount < country.minAmount) {
        throw new Error(`Amount must be at least ${country.minAmount}`);
    }

    if (paymentData.amount > country.maxAmount) {
        throw new Error(`Amount cannot exceed ${country.maxAmount}`);
    }

    // Create payment
    return await createPayment(paymentData);
}
```

### 2. Always Validate Input

```javascript
function validatePaymentData(data) {
    // Validate amount (check against country limits)
    if (data.amount < 15) {
        throw new Error("Amount must be at least $15");
    }

    if (data.amount > 2000) {
        throw new Error("Amount cannot exceed $2,000");
    }

    // Validate email
    if (!data.customerEmail || !data.customerEmail.includes("@")) {
        throw new Error("Valid email is required");
    }

    // Validate country code
    if (!data.countryCode || data.countryCode.length !== 3) {
        throw new Error("Valid 3-letter country code is required (e.g., POL, UKR)");
    }

    // Validate payment method (if provided)
    if (data.paymentMethod && !["VISA", "MASTERCARD"].includes(data.paymentMethod)) {
        throw new Error("Only VISA and MasterCard payment methods are supported");
    }

    // Validate URLs
    if (!data.successCallback.startsWith("https://")) {
        throw new Error("Success callback URL must use HTTPS");
    }

    if (!data.failureCallback.startsWith("https://")) {
        throw new Error("Failure callback URL must use HTTPS");
    }

    if (!data.postbackUrl.startsWith("https://")) {
        throw new Error("Postback URL must use HTTPS");
    }
}
```

### 3. Handle Network Errors

```javascript
async function safeApiCall(url, options) {
    try {
        const response = await fetch(url, options);

        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }

        return await response.json();
    } catch (error) {
        console.error("API call failed:", error);
        throw error;
    }
}
```

### 4. Store Payment IDs

Always store the `paymentId` returned from payment creation to check status later:

```javascript
// Store payment ID in your database
const payment = await createPayment(paymentData);
if (payment) {
    await savePaymentToDatabase({
        paymentId: payment.paymentId,
        amount: paymentData.amount,
        status: "pending",
        customerEmail: paymentData.customerEmail,
    });
}
```

### 5. Implement Retry Logic

```javascript
async function checkPaymentStatus(paymentId, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            const status = await fetch(`/merchant/payment/${paymentId}`, {
                headers: headers,
            });

            const result = await status.json();
            if (result.success) {
                return result.data;
            }
        } catch (error) {
            console.error(`Attempt ${i + 1} failed:`, error);
            if (i === maxRetries - 1) throw error;

            // Wait before retry
            await new Promise((resolve) => setTimeout(resolve, 1000 * (i + 1)));
        }
    }
}
```

### 6. Handle Payment Status Changes

```javascript
// Check payment status periodically
async function monitorPayment(paymentId) {
    const checkInterval = setInterval(async () => {
        try {
            const status = await checkPaymentStatus(paymentId);

            if (status.status === "completed") {
                clearInterval(checkInterval);
                console.log("Payment completed successfully");
                // Update UI, send confirmation email, etc.
            } else if (status.status === "failed") {
                clearInterval(checkInterval);
                console.log("Payment failed");
                // Handle failed payment
            }
        } catch (error) {
            console.error("Error checking payment status:", error);
        }
    }, 5000); // Check every 5 seconds

    // Stop checking after 10 minutes
    setTimeout(() => {
        clearInterval(checkInterval);
    }, 600000);
}
```

## Test Data Examples

### Example 1: Country with Payment Methods (Payment Method Required)

```javascript
// Poland has payment methods, so paymentMethod is REQUIRED
const paymentData = {
    amount: 100.0,
    countryCode: "POL",
    successCallback: "https://yoursite.com/success",
    failureCallback: "https://yoursite.com/failure",
    postbackUrl: "https://yoursite.com/webhook",
    customerEmail: "customer@example.com",
    customerName: "John Doe",
    customerIp: "192.168.1.1",
    paymentMethod: "VISA", // REQUIRED for Poland
};
```

### Example 2: Country without Payment Methods (Payment Method Optional)

```javascript
// Some countries might not have payment methods, so paymentMethod is OPTIONAL
const paymentData = {
    amount: 100.0,
    countryCode: "ALB",
    successCallback: "https://yoursite.com/success",
    failureCallback: "https://yoursite.com/failure",
    postbackUrl: "https://yoursite.com/webhook",
    customerEmail: "customer@example.com",
    customerName: "John Doe",
    customerIp: "192.168.1.1",
    // paymentMethod omitted - OPTIONAL for this country
};
```

### Example 3: Minimal Payment (Check Country First)

```javascript
async function createMinimalPayment(countryCode, amount, customerData) {
    // Always check country details first
    const countryResponse = await fetch(`/merchant/allowed-countries/${countryCode}`, {
        headers: headers,
    });

    const countryData = await countryResponse.json();
    const country = countryData.data;

    const paymentData = {
        amount: amount,
        countryCode: countryCode,
        successCallback: "https://yoursite.com/success",
        failureCallback: "https://yoursite.com/failure",
        postbackUrl: "https://yoursite.com/webhook",
        customerEmail: customerData.email,
        customerName: customerData.name,
        customerIp: customerData.ip,
    };

    // Add payment method only if country requires it
    if (country.paymentMethods && country.paymentMethods.length > 0) {
        paymentData.paymentMethod = country.paymentMethods[0].code; // Use first available method
    } else if (country.paymentMethods === null) {
        // Country has no payment methods, paymentMethod is optional
        console.log("Payment method is optional for this country");
    }

    return await createPayment(paymentData);
}
```
