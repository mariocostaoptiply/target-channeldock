# ChannelDock Target - Field Mappings

**Target System**: ChannelDock
**Base URL**: https://channeldock.com/portal/api/v2
**Date**: 2026-01-28
**Version**: 1.0

## Overview

This document describes the field mappings for the ChannelDock target integration. This target sends buy orders from Optiply to ChannelDock as deliveries/inbounds.

## Authentication

**Method**: API Key + API Secret (HTTP Headers)

**Required Credentials**:

| Credential | Description |
|------------|-------------|
| `api_key` | ChannelDock API Key |
| `api_secret` | ChannelDock API Secret |

## BuyOrders Endpoint

**Endpoint**: `/seller/delivery`
**Method**: POST

### Request Headers

| Header | Value |
|--------|-------|
| Content-Type | application/json |
| api_key | (from config) |
| api_secret | (from config) |

### Field Mappings

| Target Field | Transformation | Required | Notes |
|-------------|----------------|----------|-------|
| `ref` | Direct | Yes | Reference ID from Optiply (optiplyBuyOrderId) |
| `supplier_id` | Direct | Yes | Supplier identifier |
| `delivery_date` | Date (YYYY-MM-DD) | No | Expected delivery date |
| `delivery_type` | Default: "inbound" | Yes | Always set to "inbound" for buy orders |

### Default Values

| Target Field | Default Value | Required |
|-------------|---------------|----------|
| `delivery_type` | "inbound" | Yes |

### Date Transformations

| Target Field | Format | Source |
|-------------|--------|--------|
| `delivery_date` | YYYY-MM-DD | Optiply `expectedDeliveryTime` field |

## BuyOrderLines (Nested Items)

**Nested Path**: `items[]`

Line items are sent as a nested array within the delivery payload.

### Field Mappings

| Target Field | Transformation | Required | Notes |
|-------------|----------------|----------|-------|
| `items[].ean` | Direct | Yes | Product EAN (optiplyWebshopProductRemoteID) |
| `items[].amount` | Direct | Yes | Quantity ordered |

## Example Payload

```json
{
  "ref": "OPT-ORDER-123",
  "supplier_id": 13,
  "delivery_date": "2026-01-30",
  "delivery_type": "inbound",
  "items": [
    {
      "ean": "8712345678901",
      "amount": 100
    },
    {
      "ean": "8712345678902",
      "amount": 50
    }
  ]
}
```

## Response

**Success Response**:
```json
{
  "response": "success",
  "message": "Inserted delivery: 218",
  "delivery_id": 218
}
```

## Business Rules

- **Nested Items**: Line items must be provided in the `items[]` array
- **Default Type**: All deliveries are created as "inbound" type
- **Required Fields**: `ref`, `supplier_id`, and at least one item with `ean` and `amount` are required

## Troubleshooting

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| 500 "Endpoint not found" | Wrong endpoint path or user type | Verify user is a Seller in ChannelDock |
| 401 Unauthorized | Invalid API credentials | Verify api_key and api_secret |
| 400 Bad Request | Missing required fields | Check payload has ref, supplier_id, and items |
| 500 Internal Server Error | Invalid supplier_id or EAN | Verify supplier and product exist in ChannelDock |

### Debug Logging

The target logs:
- Request URL and method
- Request payload (JSON)
- Response status and body (first 500 chars)

Enable DEBUG level logging for detailed diagnostics.

## Configuration

Create a config file (e.g., `config.json`):

```json
{
  "api_key": "YOUR_API_KEY",
  "api_secret": "YOUR_API_SECRET",
  "url_base": "https://channeldock.com/portal/api/v2"
}
```

**Important**: Never commit credentials to version control. Use `.secrets/` directory and add to `.gitignore`.

## Related Documentation

- ChannelDock API Documentation: https://documenter.getpostman.com/view/16435285/2s9XxsUbPy
- Hotglue Target SDK: https://docs.hotglue.com/custom-connectors/targets
