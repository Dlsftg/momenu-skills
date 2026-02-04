---
name: mom-factura-webhooks
description: Implement webhook-based payment confirmation for Mom Factura API Reference payments and status polling for E-kwanza. Use when building payment confirmation flows, receiving webhook notifications for Bank Reference, polling E-kwanza status, or handling order state transitions from OPEN to PAID.
license: MIT
metadata:
  author: mom-factura
  version: "2.1"
  language: pt
---

# Mom Factura Webhooks & Status

Receive payment confirmations for deferred payments: webhook for Bank Reference, status polling for E-kwanza.

**Base URL:** `https://api.momenu.online`
**Auth:** `x-api-key` header required on all requests.

## How It Works

### Bank Reference (Webhook)

1. Login at [momenu.toquemedia.net](https://momenu.toquemedia.net)
2. Go to the **Desenvolvedores** menu
3. Add your webhook URL
4. Save the configuration

### E-kwanza (Status Polling)

1. Create an E-kwanza payment — receive a `code` in the response
2. Poll `GET /api/payment/ekwanza/status/:code` at regular intervals (minimum 5s)
3. When `status` is `"paid"`, the payment is confirmed and `invoiceUrl` is returned

## Webhook Payload (POST to your URL)

```json
{
  "merchantTransactionId": "abc123...",
  "ekwanzaTransactionId": "EKZ456...",
  "operationStatus": "1",
  "operationData": { ... }
}
```

**operationStatus values:** `"1"` Paid · `"3"` Cancelled/Expired · `"4"` Failed/Refused · `"5"` Error

## Webhook Server Example - Node.js / Express

```javascript
const express = require("express");
const app = express();

app.use(express.json());

app.post("/webhook/meu-webhook", (req, res) => {
  const { merchantTransactionId, operationStatus } = req.body;

  if (operationStatus === "1") {
    // Payment confirmed — update your database, notify the customer, etc.
    console.log("Payment confirmed:", merchantTransactionId);
  } else if (["3", "4", "5"].includes(operationStatus)) {
    console.log("Payment failed:", operationStatus);
  }

  res.status(200).json({ received: true });
});

app.listen(3000);
```

## Webhook Server Example - Python / Flask

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/webhook/meu-webhook", methods=["POST"])
def momenu_webhook():
    data = request.json
    transaction_id = data.get("merchantTransactionId")
    status = data.get("operationStatus")

    if status == "1":
        # Payment confirmed
        print(f"Payment confirmed: {transaction_id}")
    elif status in ["3", "4", "5"]:
        print(f"Payment failed: {status}")

    return jsonify({"received": True}), 200
```

## E-kwanza Status Polling Example

```javascript
async function verificarStatusEkwanza(code) {
  const response = await fetch(
    `https://api.momenu.online/api/payment/ekwanza/status/${code}`,
    { headers: { "x-api-key": "SUA_API_KEY" } }
  );

  const data = await response.json();

  if (data.status === "paid") {
    console.log("Pago! Fatura:", data.invoiceUrl);
  }

  return data;
}
```

## Fallback: Status Polling for Reference

If webhook delivery fails, use the status endpoint as fallback:

**Reference:** GET `/api/payment/reference/status/:operationId`

When paid, returns `invoiceUrl`.

## Notes

- Webhook delivery is fire-and-forget (no retries)
- Webhook is currently available for Bank Reference only
- E-kwanza uses status polling as the primary confirmation method
- Your webhook endpoint must return HTTP 2xx to acknowledge receipt
- MCX payments are immediate and do not use webhooks or polling
- Rate limiting: 100 req/min general, minimum 5s interval for E-kwanza polling, 30s for Reference polling
