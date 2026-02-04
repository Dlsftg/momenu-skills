# Webhook & Status Reference

## Webhook (Primary)

Payment confirmation for Reference is delivered via webhook. E-kwanza uses status polling as the primary confirmation method.

**Your webhook receives a POST with:**
```json
{
  "merchantTransactionId": "abc123...",
  "ekwanzaTransactionId": "EKZ456...",
  "operationStatus": "1",
  "operationData": { ... }
}
```

**operationStatus values:** `"1"` Paid · `"3"` Cancelled/Expired · `"4"` Failed/Refused · `"5"` Error

**Configuration:** Login at [momenu.toquemedia.net](https://momenu.toquemedia.net), go to **Desenvolvedores** menu, and add your webhook URL.

## Status Endpoints (Fallback)

Use these if webhook delivery fails or as a manual check.

### E-kwanza Status

**GET** `/api/payment/ekwanza/status/:code`

Pending:
```json
{ "success": true, "status": "pending", "operationCode": "OP123..." }
```

Paid:
```json
{
  "success": true,
  "status": "paid",
  "operationCode": "OP123...",
  "invoiceUrl": "https://invoice-momenu.toquemedia.net/invoices/..."
}
```

### Bank Reference Status

**GET** `/api/payment/reference/status/:operationId`

Pending:
```json
{ "success": true, "payment": { "status": "pending", "message": "Aguardando pagamento" } }
```

Paid:
```json
{
  "success": true,
  "payment": { "status": "paid", "message": "Pagamento confirmado" },
  "invoiceUrl": "https://invoice-momenu.toquemedia.net/invoices/..."
}
```

## Rate Limiting

- General: 100 req/min per IP
- Payments: 20 req/min per IP (POST only)
- Status endpoints (GET) count toward general limit
- Minimum polling interval: 5 seconds (E-kwanza), 30 seconds (Reference)
