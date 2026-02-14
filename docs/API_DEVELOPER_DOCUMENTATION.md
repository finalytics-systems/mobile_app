# Mobile App API – Developer Documentation

This document describes the REST APIs exposed by the **mobile_app** app for items, customers, sales orders, and loyalty points. All endpoints require **authenticated** access (no guest access).

---

## Base URL and authentication

- **Base URL:** `https://<your-site>/api/method/`
- **Method:** `POST` for all endpoints below.
- **Headers:**
  - `Content-Type: application/json`
  - `Authorization: token <api_key>:<api_secret>`  
    (Create API keys in **User** → **API Access** in Frappe/ERPNext.)

**Important:** The correct module path is **`mobile_app.api`** (one “mobile_app”), not `mobile_app.mobile_app.api`. Use the URLs as shown in each section.

---

## 1. Item stock and prices

**Purpose:** Returns items with warehouse-wise stock and pricing (current sales price, base price, web retail price, minimum sale price).

**Endpoint:** `mobile_app.api.get_item_stock_and_prices`

**Full URL example:**  
`https://<your-site>/api/method/mobile_app.api.get_item_stock_and_prices`

### Parameters (inside `filters`)

| Parameter            | Type   | Required | Description |
|----------------------|--------|----------|-------------|
| `item_code`          | string | No       | Filter by single item code. |
| `item_group`         | string | No       | Filter by item group. |
| `warehouse`          | string | No       | Filter by warehouse name. |
| `company`            | string | No       | Filter by company (via warehouse). |
| `price_list`         | string | No       | Price list for prices (default: `"Sales Price List"`). |
| `include_zero_stock` | bool   | No       | Include rows with 0 stock (default: `true`). Set `false` to exclude zero stock. |

### Request body example

```json
{
  "filters": {}
}
```

With filters:

```json
{
  "filters": {
    "item_code": "ITEM-001",
    "warehouse": "Stores - FH",
    "price_list": "Sales Price List",
    "include_zero_stock": false
  }
}
```

### Response

Array of objects, one per item–warehouse combination:

| Field                   | Type   | Description |
|-------------------------|--------|-------------|
| `item`                  | string | Item code. |
| `item_name`             | string | Item name. |
| `item_group`             | string | Item group. |
| `warehouse`              | string | Warehouse name. |
| `available_stock`       | float  | Available quantity (from Bin). |
| `current_sales_price_wp`| float  | Price list rate (current sales price). |
| `base_price`            | float  | Custom base price. |
| `web_retail_price`      | float  | Web retail price (WRP). |
| `minimum_sale_price`    | float  | Minimum sales price. |

Only **non-disabled** items are returned.

---

## 2. Customers with loyalty balance

**Purpose:** Returns customers (optionally a single customer) with their loyalty points balance from Loyalty Point Entry.

**Endpoint:** `mobile_app.api.get_customers_with_loyalty_balance`

**Full URL example:**  
`https://<your-site>/api/method/mobile_app.api.get_customers_with_loyalty_balance`

### Parameters (inside `filters`)

| Parameter  | Type   | Required | Description |
|------------|--------|----------|-------------|
| `customer` | string | No       | If provided, only this customer is returned; otherwise all non-disabled customers. |

### Request body example

All customers:

```json
{
  "filters": {}
}
```

Single customer:

```json
{
  "filters": {
    "customer": "CUST-00001"
  }
}
```

### Response

Array of customer objects:

| Field                  | Type   | Description |
|------------------------|--------|-------------|
| `id`                   | string | Customer ID (name). |
| `customer_name`        | string | Customer name. |
| `email`                | string | Email ID. |
| `mobile`               | string | Mobile number. |
| `custom_is_bff_member` | value  | BFF member flag. |
| `customer_group`       | string | Customer group. |
| `territory`            | string | Territory. |
| `disabled`             | value  | Disabled flag. |
| `loyalty_points_balance` | float | Sum of loyalty points (from Loyalty Point Entry). |

Only **non-disabled** customers are returned when no `customer` filter is used.

---

## 3. Sales orders

**Purpose:** Returns sales orders with their line items and tax rows. Can filter to a single order or return multiple.

**Endpoint:** `mobile_app.api.get_sales_orders`

**Full URL example:**  
`https://<your-site>/api/method/mobile_app.api.get_sales_orders`

### Parameters (inside `filters`)

| Parameter     | Type   | Required | Description |
|---------------|--------|----------|-------------|
| `sales_order` | string | No       | If provided, only this Sales Order name is returned; otherwise multiple orders (by date/name). |

### Request body example

All sales orders (subject to permissions):

```json
{
  "filters": {}
}
```

Single sales order:

```json
{
  "filters": {
    "sales_order": "SAL-ORD-2024-00001"
  }
}
```

### Response

Array of sales order objects, each containing header fields plus `items` and `taxes`:

**Header fields:**

| Field               | Type   | Description |
|---------------------|--------|-------------|
| `sales_order`       | string | Sales Order name. |
| `customer`          | string | Customer link. |
| `customer_name`     | string | Customer name. |
| `transaction_date`  | string | Transaction date. |
| `delivery_date`     | string | Delivery date. |
| `status`            | string | Status. |
| `grand_total`       | float  | Grand total. |
| `rounded_total`     | float  | Rounded total. |
| `company`           | string | Company. |
| `currency`          | string | Currency. |
| `territory`         | string | Territory. |
| `docstatus`         | int    | Document status. |
| `items`             | array  | Line items (see below). |
| `taxes`             | array  | Tax rows (see below). |

**Each element in `items`:**

| Field               | Type   | Description |
|---------------------|--------|-------------|
| `item_name`        | string | Row name. |
| `item_code`        | string | Item code. |
| `item_description` | string | Item description. |
| `qty`              | float  | Quantity. |
| `rate`             | float  | Rate. |
| `amount`           | float  | Amount. |
| `delivery_date`    | string | Delivery date. |
| `warehouse`        | string | Warehouse. |
| `uom`              | string | UOM. |
| `stock_uom`        | string | Stock UOM. |
| `conversion_factor`| float  | Conversion factor. |

**Each element in `taxes`:**

| Field         | Type   | Description |
|---------------|--------|-------------|
| `tax_name`    | string | Row name. |
| `charge_type` | string | Charge type. |
| `account_head`| string | Account head. |
| `description` | string | Description. |
| `rate`        | float  | Rate. |
| `tax_amount`  | float  | Tax amount. |
| `total`       | float  | Total. |
| `cost_center` | string | Cost center. |

---

## 4. Loyalty points entries

**Purpose:** Returns Loyalty Point Entry records, optionally filtered by customer.

**Endpoint:** `mobile_app.api.get_loyalty_points_entries`

**Full URL example:**  
`https://<your-site>/api/method/mobile_app.api.get_loyalty_points_entries`

### Parameters (inside `filters`)

| Parameter  | Type   | Required | Description |
|------------|--------|----------|-------------|
| `customer` | string | No       | If provided, only entries for this customer; otherwise all entries. |

### Request body example

All entries:

```json
{
  "filters": {}
}
```

Single customer:

```json
{
  "filters": {
    "customer": "CUST-00001"
  }
}
```

### Response

Array of loyalty point entry objects:

| Field                | Type   | Description |
|----------------------|--------|-------------|
| `name`               | string | Entry name. |
| `customer`           | string | Customer. |
| `loyalty_points`     | float  | Loyalty points. |
| `loyalty_program`    | string | Loyalty program. |
| `loyalty_program_tier` | string | Tier. |
| `posting_date`       | string | Posting date. |
| `expiry_date`        | string | Expiry date. |
| `invoice_type`       | string | Invoice type. |
| `invoice`            | string | Linked invoice. |
| `company`            | string | Company. |
| `docstatus`          | int    | Document status. |
| `creation`           | string | Creation timestamp. |
| `modified`            | string | Last modified timestamp. |

Entries are ordered by `posting_date` and `creation` descending.

---

## Quick reference – URLs and filters

| API description        | Method path                                      | Main optional filters |
|------------------------|---------------------------------------------------|------------------------|
| Item stock and prices  | `mobile_app.api.get_item_stock_and_prices`        | `item_code`, `item_group`, `warehouse`, `company`, `price_list`, `include_zero_stock` |
| Customers + loyalty    | `mobile_app.api.get_customers_with_loyalty_balance` | `customer` |
| Sales orders           | `mobile_app.api.get_sales_orders`                | `sales_order` |
| Loyalty points entries | `mobile_app.api.get_loyalty_points_entries`      | `customer` |

Always send a JSON body with a `filters` object (can be empty `{}`). Use **POST** and **Authorization: token &lt;api_key&gt;:&lt;api_secret&gt;** for all calls.
