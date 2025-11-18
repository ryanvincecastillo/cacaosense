# CacaoSense API Reference

## Overview

The CacaoSense API is a RESTful API built with FastAPI. It provides endpoints for managing farmers, harvest reports, products, orders, and accessing AI-powered predictions.

**Base URL:** `https://api.cacaosense.com/v1`

**Authentication:** Bearer token (JWT from Supabase Auth)

---

## Authentication

All API requests require authentication via Bearer token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

### Get Token

Tokens are obtained through Supabase Auth. Use the Supabase client libraries or direct API calls.

---

## Response Format

All responses follow this structure:

```json
{
    "success": true,
    "data": { ... },
    "message": "Operation successful",
    "meta": {
        "page": 1,
        "per_page": 20,
        "total": 100
    }
}
```

### Error Response

```json
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Invalid input data",
        "details": [
            {
                "field": "email",
                "message": "Invalid email format"
            }
        ]
    }
}
```

---

## Endpoints

### Farmers

#### List Farmers

```
GET /farmers
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| page | integer | Page number (default: 1) |
| per_page | integer | Items per page (default: 20, max: 100) |
| status | string | Filter by status: active, inactive |
| search | string | Search by name or phone |
| sort_by | string | Sort field: name, overall_score, last_delivery_at |
| sort_order | string | asc or desc |

**Response:**
```json
{
    "success": true,
    "data": [
        {
            "id": "uuid",
            "name": "Juan dela Cruz",
            "phone": "+639123456789",
            "barangay": "Calinan",
            "municipality": "Davao City",
            "farm_size_hectares": 2.5,
            "tree_count": 500,
            "overall_score": 85.5,
            "reliability_score": 88.0,
            "quality_score": 82.0,
            "engagement_score": 90.0,
            "status": "active",
            "last_report_at": "2024-01-15T08:30:00Z",
            "last_delivery_at": "2024-01-10T14:00:00Z"
        }
    ],
    "meta": {
        "page": 1,
        "per_page": 20,
        "total": 45
    }
}
```

---

#### Get Farmer

```
GET /farmers/{farmer_id}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "id": "uuid",
        "name": "Juan dela Cruz",
        "phone": "+639123456789",
        "messenger_id": "psid_12345",
        "barangay": "Calinan",
        "municipality": "Davao City",
        "province": "Davao",
        "farm_size_hectares": 2.5,
        "tree_count": 500,
        "tree_age_years": 5,
        "cacao_variety": "Trinitario",
        "overall_score": 85.5,
        "reliability_score": 88.0,
        "quality_score": 82.0,
        "engagement_score": 90.0,
        "status": "active",
        "registration_date": "2023-06-15",
        "created_at": "2023-06-15T10:00:00Z",
        "updated_at": "2024-01-15T08:30:00Z",
        "stats": {
            "total_reports": 24,
            "total_delivered_kg": 1250.5,
            "avg_delivery_kg": 52.1,
            "delivery_accuracy_pct": 92.5
        }
    }
}
```

---

#### Create Farmer

```
POST /farmers
```

**Request Body:**
```json
{
    "name": "Juan dela Cruz",
    "phone": "+639123456789",
    "barangay": "Calinan",
    "municipality": "Davao City",
    "farm_size_hectares": 2.5,
    "tree_count": 500,
    "tree_age_years": 5,
    "cacao_variety": "Trinitario"
}
```

**Response:** `201 Created`

---

#### Update Farmer

```
PUT /farmers/{farmer_id}
```

**Request Body:** Same as Create (partial updates allowed)

**Response:** `200 OK`

---

#### Delete Farmer

```
DELETE /farmers/{farmer_id}
```

**Response:** `204 No Content`

---

### Harvest Reports

#### List Harvest Reports

```
GET /harvest-reports
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| page | integer | Page number |
| per_page | integer | Items per page |
| farmer_id | uuid | Filter by farmer |
| status | string | pending, confirmed, delivered, cancelled |
| date_from | date | Filter from date |
| date_to | date | Filter to date |

**Response:**
```json
{
    "success": true,
    "data": [
        {
            "id": "uuid",
            "farmer_id": "uuid",
            "farmer_name": "Juan dela Cruz",
            "reported_at": "2024-01-15T08:30:00Z",
            "estimated_volume_kg": 50.0,
            "estimated_pickup_date": "2024-01-20",
            "quality_rating": "good",
            "status": "pending",
            "actual_volume_kg": null,
            "delivered_at": null
        }
    ]
}
```

---

#### Create Harvest Report (Manual)

```
POST /harvest-reports
```

**Request Body:**
```json
{
    "farmer_id": "uuid",
    "estimated_volume_kg": 50.0,
    "readiness_days": 5,
    "quality_rating": "good",
    "notes": "Beans look healthy"
}
```

---

#### Update Harvest Report

```
PUT /harvest-reports/{report_id}
```

Used to update status, add actual delivery data:

```json
{
    "status": "delivered",
    "actual_volume_kg": 48.5,
    "actual_quality": "good",
    "delivered_at": "2024-01-20T14:00:00Z"
}
```

---

### Products

#### List Products

```
GET /products
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| category | string | tablea, chocolate_bar, powder, nibs, butter |
| is_active | boolean | Filter active/inactive |

**Response:**
```json
{
    "success": true,
    "data": [
        {
            "id": "uuid",
            "name": "Premium Tablea 500g",
            "sku": "TAB-500",
            "category": "tablea",
            "beans_per_unit_kg": 0.6,
            "unit_price": 350.00,
            "production_cost": 180.00,
            "margin": 170.00,
            "margin_percentage": 48.57,
            "is_active": true
        }
    ]
}
```

---

#### Create Product

```
POST /products
```

**Request Body:**
```json
{
    "name": "Premium Tablea 500g",
    "sku": "TAB-500",
    "category": "tablea",
    "beans_per_unit_kg": 0.6,
    "unit_price": 350.00,
    "production_cost": 180.00,
    "shelf_life_days": 180
}
```

---

### Orders

#### List Orders

```
GET /orders
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| status | string | pending, confirmed, in_production, ready, delivered, cancelled |
| date_from | date | Filter from delivery date |
| date_to | date | Filter to delivery date |

---

#### Create Order

```
POST /orders
```

**Request Body:**
```json
{
    "customer_name": "ABC Store",
    "customer_contact": "+639123456789",
    "delivery_address": "123 Main St, Davao City",
    "delivery_date": "2024-01-25",
    "items": [
        {
            "product_id": "uuid",
            "quantity": 100
        },
        {
            "product_id": "uuid",
            "quantity": 50
        }
    ],
    "notes": "Please pack in boxes of 10"
}
```

---

#### Update Order Status

```
PATCH /orders/{order_id}/status
```

**Request Body:**
```json
{
    "status": "confirmed"
}
```

---

### Supply Predictions

#### Get Supply Forecast

```
GET /predictions/supply
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| days | integer | Forecast days ahead (7, 14, 30) |

**Response:**
```json
{
    "success": true,
    "data": {
        "forecast_period": {
            "start": "2024-01-16",
            "end": "2024-01-22"
        },
        "total_predicted_kg": 450.0,
        "confidence_level": 0.87,
        "confidence_category": "high",
        "daily_forecast": [
            {
                "date": "2024-01-16",
                "predicted_kg": 65.0,
                "confidence": 0.92
            },
            {
                "date": "2024-01-17",
                "predicted_kg": 70.0,
                "confidence": 0.89
            }
        ],
        "by_farmer": [
            {
                "farmer_id": "uuid",
                "farmer_name": "Juan dela Cruz",
                "predicted_kg": 50.0
            }
        ],
        "comparison": {
            "previous_period_kg": 420.0,
            "change_percentage": 7.14
        }
    }
}
```

---

#### Generate New Prediction

```
POST /predictions/supply/generate
```

Triggers AI model to generate new predictions.

**Response:**
```json
{
    "success": true,
    "data": {
        "prediction_id": "uuid",
        "generated_at": "2024-01-15T10:00:00Z",
        "model_version": "prophet-v1.2"
    }
}
```

---

### Revenue Optimization

#### Get Recommendations

```
GET /optimization/revenue
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| period_days | integer | Planning period (default: 7) |

**Response:**
```json
{
    "success": true,
    "data": {
        "period": {
            "start": "2024-01-16",
            "end": "2024-01-22"
        },
        "available_supply_kg": 450.0,
        "recommendations": [
            {
                "product_id": "uuid",
                "product_name": "Premium Tablea 500g",
                "recommended_units": 200,
                "beans_required_kg": 120.0,
                "expected_revenue": 70000.00,
                "expected_profit": 34000.00,
                "priority": 1,
                "reason": "Highest margin product with strong demand"
            },
            {
                "product_id": "uuid",
                "product_name": "Dark Chocolate Bar 100g",
                "recommended_units": 150,
                "beans_required_kg": 90.0,
                "expected_revenue": 52500.00,
                "expected_profit": 22500.00,
                "priority": 2,
                "reason": "Good margin, pending orders"
            }
        ],
        "totals": {
            "beans_allocated_kg": 420.0,
            "beans_remaining_kg": 30.0,
            "total_revenue": 122500.00,
            "total_profit": 56500.00
        },
        "safe_order_threshold": {
            "can_accept_up_to_kg": 400.0,
            "confidence": 0.85,
            "message": "You can safely accept orders requiring up to 400kg of beans"
        }
    }
}
```

---

### Alerts

#### List Alerts

```
GET /alerts
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| is_read | boolean | Filter read/unread |
| alert_type | string | supply_dip, farmer_inactive, etc. |
| severity | string | info, warning, critical |

**Response:**
```json
{
    "success": true,
    "data": [
        {
            "id": "uuid",
            "alert_type": "supply_dip",
            "severity": "warning",
            "title": "Supply Dip Expected",
            "message": "Predicted supply for next week is 30% below average",
            "is_read": false,
            "created_at": "2024-01-15T08:00:00Z",
            "action_url": "/dashboard/supply",
            "action_label": "View Forecast"
        }
    ]
}
```

---

#### Mark Alert as Read

```
PATCH /alerts/{alert_id}/read
```

---

#### Dismiss Alert

```
DELETE /alerts/{alert_id}
```

---

### Dashboard

#### Get Dashboard Summary

```
GET /dashboard/summary
```

**Response:**
```json
{
    "success": true,
    "data": {
        "supply": {
            "next_7_days_kg": 450.0,
            "confidence": "high",
            "trend": "up",
            "trend_percentage": 12.5
        },
        "farmers": {
            "total": 45,
            "active": 42,
            "pending_reports": 8
        },
        "orders": {
            "pending": 5,
            "in_production": 3,
            "due_this_week": 4,
            "total_value": 125000.00
        },
        "inventory": {
            "raw_beans_kg": 200.0,
            "days_of_supply": 3.2
        },
        "alerts": {
            "unread_count": 3,
            "critical_count": 1
        },
        "revenue": {
            "this_month": 450000.00,
            "last_month": 380000.00,
            "growth_percentage": 18.4
        }
    }
}
```

---

### Webhook (Messenger Bot)

#### Messenger Webhook Verification

```
GET /webhook/messenger
```

**Query Parameters:**
| Parameter | Description |
|-----------|-------------|
| hub.mode | Should be "subscribe" |
| hub.verify_token | Your verify token |
| hub.challenge | Challenge to return |

---

#### Receive Messenger Messages

```
POST /webhook/messenger
```

Receives messages from Facebook Messenger API.

**Request Body:** Facebook Messenger webhook payload

---

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| VALIDATION_ERROR | 400 | Invalid input data |
| UNAUTHORIZED | 401 | Missing or invalid token |
| FORBIDDEN | 403 | Insufficient permissions |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | Resource already exists |
| RATE_LIMITED | 429 | Too many requests |
| SERVER_ERROR | 500 | Internal server error |

---

## Rate Limiting

- **Standard:** 100 requests per minute
- **Burst:** 20 requests per second
- **Predictions:** 10 requests per minute

Rate limit headers are included in responses:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1705312800
```

---

## Pagination

All list endpoints support pagination:

```
GET /farmers?page=2&per_page=50
```

Response includes meta:
```json
{
    "meta": {
        "page": 2,
        "per_page": 50,
        "total": 145,
        "total_pages": 3
    }
}
```

---

## Filtering & Sorting

Most list endpoints support:

- **Filtering:** `?status=active&category=tablea`
- **Sorting:** `?sort_by=created_at&sort_order=desc`
- **Searching:** `?search=juan`

---

## Webhooks (Outbound)

Configure webhooks to receive real-time updates:

```
POST /webhooks
```

**Request Body:**
```json
{
    "url": "https://your-server.com/webhook",
    "events": ["harvest.created", "order.status_changed", "alert.created"],
    "secret": "your-webhook-secret"
}
```

**Events:**
- `harvest.created`
- `harvest.delivered`
- `order.created`
- `order.status_changed`
- `alert.created`
- `prediction.generated`

---

## SDK & Libraries

- **JavaScript/TypeScript:** `npm install @cacaosense/sdk`
- **Python:** `pip install cacaosense`

Coming soon.