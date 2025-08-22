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
