# Merchant API Integration Guide

This guide helps developers integrate the Telestore Proxy payment system into their applications.

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
https://your-domain.com/merchant
```

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
            "currncies": ["USD", "EUR"]
        },
        {
            "country": "Ukraine",
            "code": "UKR",
            "currncies": ["USD", "EUR"]
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
        "currncies": ["USD", "EUR"],
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
    currency: "USD",
    paymentMethod: "VISA",
    countryCode: "POL",
    successCallback: "https://yoursite.com/success",
    failureCallback: "https://yoursite.com/failure",
    postbackUrl: "https://yoursite.com/webhook",
    customerEmail: "customer@example.com",
    customerName: "John Doe",
    customerIp: "192.168.1.1",
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

-   `amount` (number, required): Payment amount (minimum $15)
-   `currency` (string, required): Currency code (USD, EUR)
-   `paymentMethod` (string, required): Payment method code (VISA, MASTERCARD)
-   `countryCode` (string, required): Country code (POL, UKR, etc.)
-   `successCallback` (string, required): URL to redirect after successful payment
-   `failureCallback` (string, required): URL to redirect after failed payment
-   `postbackUrl` (string, required): Webhook URL for payment notifications
-   `customerEmail` (string, required): Customer email address
-   `customerName` (string, required): Customer full name
-   `customerIp` (string, required): Customer IP address

**Response:**

```json
{
    "success": true,
    "data": {
        "paymentId": "payment_123456",
        "paymentUrl": "https://payment-gateway.com/pay/123456",
        "status": "pending"
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
        "status": "status of payment"
        // lots of different fields
    }
}
```

## Complete Integration Example

Here's a complete example of how to integrate payments into your website:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Payment Integration Example</title>
    </head>
    <body>
        <h1>Make a Payment</h1>

        <form id="paymentForm">
            <div>
                <label>Amount (USD):</label>
                <input type="number" id="amount" min="15" step="0.01" value="15" required />
            </div>

            <div>
                <label>Country:</label>
                <select id="country" required>
                    <option value="">Select country</option>
                </select>
            </div>

            <div>
                <label>Payment Method:</label>
                <select id="paymentMethod" required>
                    <option value="">Select payment method</option>
                </select>
            </div>

            <button type="submit">Pay Now</button>
        </form>

        <script>
            const MERCHANT_TOKEN = "YOUR_MERCHANT_TOKEN";
            const BASE_URL = "https://your-domain.com/merchant";

            const headers = {
                Authorization: `Bearer ${MERCHANT_TOKEN}`,
                "Content-Type": "application/json",
            };

            // Load countries on page load
            async function loadCountries() {
                try {
                    const response = await fetch(`${BASE_URL}/allowed-countries`, {
                        headers: headers,
                    });
                    const result = await response.json();

                    if (result.success) {
                        const countrySelect = document.getElementById("country");
                        result.data.forEach((country) => {
                            const option = document.createElement("option");
                            option.value = country.code;
                            option.textContent = country.country;
                            countrySelect.appendChild(option);
                        });
                    }
                } catch (error) {
                    console.error("Failed to load countries:", error);
                }
            }

            // Load payment methods
            async function loadPaymentMethods() {
                try {
                    const response = await fetch(`${BASE_URL}/methods`, {
                        headers: headers,
                    });
                    const result = await response.json();

                    if (result.success) {
                        const methodSelect = document.getElementById("paymentMethod");
                        result.data.forEach((method) => {
                            const option = document.createElement("option");
                            option.value = method.code;
                            option.textContent = method.name;
                            methodSelect.appendChild(option);
                        });
                    }
                } catch (error) {
                    console.error("Failed to load payment methods:", error);
                }
            }

            // Handle form submission
            document.getElementById("paymentForm").addEventListener("submit", async function (e) {
                e.preventDefault();

                const amount = parseFloat(document.getElementById("amount").value);
                const countryCode = document.getElementById("country").value;
                const paymentMethod = document.getElementById("paymentMethod").value;

                if (amount < 15) {
                    alert("Minimum amount is $15");
                    return;
                }

                const paymentData = {
                    amount: amount,
                    currency: "USD",
                    paymentMethod: paymentMethod,
                    countryCode: countryCode,
                    successCallback: window.location.origin + "/success",
                    failureCallback: window.location.origin + "/failure",
                    postbackUrl: window.location.origin + "/webhook",
                    customerEmail: "customer@example.com",
                    customerName: "John Doe",
                    customerIp: "127.0.0.1",
                };

                try {
                    const response = await fetch(`${BASE_URL}/payment`, {
                        method: "POST",
                        headers: headers,
                        body: JSON.stringify(paymentData),
                    });

                    const result = await response.json();

                    if (result.success) {
                        window.location.href = result.data.paymentUrl;
                    } else {
                        alert("Payment creation failed: " + result.message);
                    }
                } catch (error) {
                    console.error("Error creating payment:", error);
                    alert("Failed to create payment. Please try again.");
                }
            });

            // Initialize page
            loadCountries();
            loadPaymentMethods();
        </script>
    </body>
</html>
```

## Error Handling

### Common Error Codes

| Code                         | Message                    | Description                            | Solution                                       |
| ---------------------------- | -------------------------- | -------------------------------------- | ---------------------------------------------- |
| `COUNTRY_CODE_NOT_SPECIFIED` | Country code not specified | Missing country code parameter         | Provide a valid country code                   |
| `COUNTRY_NOT_FOUND`          | Country not found          | Invalid country code                   | Use a country code from allowed countries list |
| `PAYMENT_ID_REQUIRED`        | Payment ID required        | Missing payment ID parameter           | Provide a valid payment ID                     |
| `PAYMENT_NOT_FOUND`          | Payment not found          | Invalid payment ID                     | Check payment ID format and existence          |
| `INVALID_AMOUNT`             | Invalid payment amount     | Amount below minimum or invalid format | Use amount >= $15                              |
| `INVALID_CURRENCY`           | Invalid currency           | Unsupported currency code              | Use USD or EUR                                 |
| `INVALID_PAYMENT_METHOD`     | Invalid payment method     | Unsupported payment method             | Use VISA or MASTERCARD                         |
| `MISSING_REQUIRED_FIELD`     | Missing required field     | Required parameter not provided        | Check all required fields                      |
| `INVALID_CALLBACK_URL`       | Invalid callback URL       | Invalid success/failure URL format     | Use valid HTTPS URLs                           |

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
                case "INVALID_AMOUNT":
                    alert("Amount must be at least $15");
                    break;
                case "INVALID_PAYMENT_METHOD":
                    alert("Please select a valid payment method");
                    break;
                case "COUNTRY_NOT_FOUND":
                    alert("Please select a valid country");
                    break;
                default:
                    alert("Payment error: " + result.message);
            }
            return null;
        }

        return result.data;
    } catch (error) {
        console.error("Network error:", error);
        alert("Network error. Please check your connection.");
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

    // Verify webhook signature (implement security)
    // Process payment status update
    // Update your database
    // Send confirmation emails, etc.

    res.status(200).json({ success: true });
});
```

### Webhook Data Format

```json
{
	"status": "success", // The status of the payment could be "success"/"failure"/"pending",
	"paymentId": "<Payment id>",
    "externalClientId": <external client id>
}
```

## Best Practices

### 1. Always Validate Input

```javascript
function validatePaymentData(data) {
    if (data.amount < 15) {
        throw new Error("Amount must be at least $15");
    }

    if (!data.customerEmail || !data.customerEmail.includes("@")) {
        throw new Error("Valid email is required");
    }

    // Add more validation as needed
}
```

### 2. Handle Network Errors

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

### 3. Store Payment IDs

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

### 4. Implement Retry Logic

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

## Testing

### Test Environment

-   Use test merchant tokens for development
-   Test with small amounts ($15 minimum)
-   Verify webhook endpoints are accessible

### Test Payment Flow

1. Create payment with test data
2. Redirect to payment URL
3. Complete payment on test gateway
4. Verify webhook received
5. Check payment status via API

## Support

For integration support:

-   Email: support@telestore.com
-   Documentation: https://docs.telestore.com
-   API Status: https://status.telestore.com
