# Building a Complete E-Commerce Store with Superpowers, Next.js, and .NET

A beginner-friendly, step-by-step tutorial for building a production-quality online store using [Superpowers](https://github.com/obra/superpowers), a Next.js frontend, and a C#/.NET modular monolith backend with CQRS and DDD tactical patterns.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [Final Project Overview](#3-final-project-overview)
4. [Module Breakdown](#4-module-breakdown)
5. [Module-by-Module Feature Specification](#5-module-by-module-feature-specification)
6. [Backend Architecture Tutorial](#6-backend-architecture-tutorial)
7. [Frontend Architecture Tutorial](#7-frontend-architecture-tutorial)
8. [Step-by-Step Superpowers Workflow](#8-step-by-step-superpowers-workflow)
9. [Superpowers Prompt Examples](#9-superpowers-prompt-examples)
10. [Database Design](#10-database-design)
11. [API Design](#11-api-design)
12. [Testing Tutorial](#12-testing-tutorial)
13. [Running the Application](#13-running-the-application)
14. [Deployment Notes](#14-deployment-notes)
15. [Common Mistakes and Troubleshooting](#15-common-mistakes-and-troubleshooting)
16. [Final Review Checklist](#16-final-review-checklist)

---

## 1. Introduction

### What This Tutorial Builds

This tutorial guides you through building a fully functional online e-commerce store from an empty folder to a working application. By the end, you will have:

- A **Next.js** frontend with product listings, a shopping cart, checkout flow, order tracking, and customer account pages.
- A **C#/.NET** backend structured as a **modular monolith**, with ten independent modules covering every concern of an online store.
- A clean **CQRS** architecture inside each module so reads and writes are clearly separated.
- **DDD tactical patterns** (aggregates, value objects, domain events, repositories, and more) used to organise each module's internal logic.
- A complete set of REST API endpoints connecting the two sides.
- Practical **unit tests**, **integration tests**, and basic **end-to-end test** examples.

You will use **Superpowers** throughout to accelerate every part of the build — generating boilerplate, scaffolding modules, writing tests, and fixing bugs — so the focus stays on understanding the architecture rather than typing repetitive code.

### What Superpowers Is and Why It Is Useful

[Superpowers](https://github.com/obra/superpowers) is an open-source, spec-driven AI coding assistant. Instead of asking a chat model ad-hoc questions, Superpowers lets you write structured specifications that describe *what* you want to build, then uses those specs as grounding context when it generates code. This gives you two big advantages:

1. **Consistency.** Because every prompt shares the same specification context, the code that Superpowers generates fits together. It will not invent a different naming convention in each file.
2. **Repeatability.** You can re-run any generation step and get a result that still matches the overall design. This is especially valuable when you are learning, because you can experiment and then regenerate cleanly.

In a project of this size — ten backend modules, a full frontend, a database schema, and a test suite — consistency and repeatability are the difference between a project that feels manageable and one that collapses under its own contradictions. Superpowers makes the manageable version accessible to beginners.

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Browser / Client                    │
└────────────────────────┬────────────────────────────────┘
                         │  HTTPS / REST JSON
┌────────────────────────▼────────────────────────────────┐
│              Next.js Frontend  (port 3000)              │
│  Pages · Components · API Client · State Management     │
└────────────────────────┬────────────────────────────────┘
                         │  HTTP REST calls
┌────────────────────────▼────────────────────────────────┐
│           .NET Modular Monolith  (port 5000)            │
│                                                         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │ Catalog  │ │Inventory │ │ Pricing  │ │   Cart   │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │ Ordering │ │ Payment  │ │Shipping  │ │Customer  │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘  │
│  ┌──────────┐ ┌────────────────────┐                   │
│  │ Identity │ │   Notification     │                   │
│  └──────────┘ └────────────────────┘                   │
│                                                         │
│  Shared: MediatR · EF Core · Domain Events Bus         │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│             PostgreSQL Database  (port 5432)            │
│   Each module owns its own schema / table prefix        │
└─────────────────────────────────────────────────────────┘
```

Each backend module is a C# class library project inside the same .NET solution. Modules communicate with each other by publishing and subscribing to **domain events** through an in-process event bus (MediatR notifications). No module calls another module's internal classes directly. The API layer is a thin ASP.NET Core Web API project that routes requests into the correct module via MediatR commands and queries.

---

## 2. Prerequisites

### Required Tools and Versions

| Tool | Minimum Version | Install |
|---|---|---|
| Node.js | 20 LTS | https://nodejs.org |
| npm | 10+ | Bundled with Node.js |
| .NET SDK | 8.0 | https://dotnet.microsoft.com/download |
| PostgreSQL | 15+ | https://www.postgresql.org/download |
| Git | 2.40+ | https://git-scm.com |
| VS Code (or JetBrains Rider) | Latest | https://code.visualstudio.com |
| Superpowers | Latest | See below |

### Required Knowledge

You do not need to be an expert in any of these, but you should be comfortable with:

- Basic C# syntax (classes, interfaces, async/await)
- Basic TypeScript/JavaScript syntax
- What a REST API is and how HTTP requests work
- Running commands in a terminal
- Basic Git (init, add, commit)

If you are brand new to C# or TypeScript, spending a few hours on the official "getting started" guides for each will make this tutorial much smoother.

### How to Install and Configure Superpowers

**Step 1 — Clone and install Superpowers:**

```bash
git clone https://github.com/obra/superpowers.git
cd superpowers
npm install
npm run build
npm link   # makes the `superpowers` command available globally
```

**Step 2 — Verify the installation:**

```bash
superpowers --version
```

**Step 3 — Initialise Superpowers in your project folder:**

```bash
mkdir ecommerce-store
cd ecommerce-store
superpowers init
```

This creates a `.superpowers/` folder containing:

```
.superpowers/
  config.json        # project-level settings
  specs/             # where you store your specification files
  context/           # auto-generated context summaries
```

**Step 4 — Connect your AI provider.** Edit `.superpowers/config.json` and add your API key:

```json
{
  "provider": "anthropic",
  "model": "claude-sonnet-4-6",
  "apiKey": "YOUR_API_KEY_HERE"
}
```

Superpowers supports Anthropic Claude, OpenAI, and other providers. Claude is recommended for this tutorial because of its strength with large, structured codebases.

---

## 3. Final Project Overview

### Top-Level Folder Structure

```
ecommerce-store/
├── .superpowers/          # Superpowers configuration and specs
│   ├── config.json
│   └── specs/
│       ├── architecture.md
│       ├── catalog-module.md
│       ├── inventory-module.md
│       └── ...
├── frontend/              # Next.js application
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── next.config.js
├── backend/               # .NET solution
│   ├── ECommerceStore.sln
│   ├── src/
│   │   ├── Api/           # ASP.NET Core Web API host
│   │   ├── Modules/
│   │   │   ├── Catalog/
│   │   │   ├── Inventory/
│   │   │   ├── Pricing/
│   │   │   ├── Cart/
│   │   │   ├── Ordering/
│   │   │   ├── Payment/
│   │   │   ├── Shipping/
│   │   │   ├── Customer/
│   │   │   ├── Identity/
│   │   │   └── Notification/
│   │   └── Shared/        # Shared kernel (base classes, interfaces)
│   └── tests/
│       ├── Unit/
│       └── Integration/
└── docker-compose.yml     # Local development database
```

### Frontend Structure (Next.js)

```
frontend/src/
├── app/                   # Next.js App Router
│   ├── layout.tsx
│   ├── page.tsx           # Home / product listing
│   ├── products/
│   │   └── [id]/
│   │       └── page.tsx   # Product detail
│   ├── cart/
│   │   └── page.tsx
│   ├── checkout/
│   │   └── page.tsx
│   ├── orders/
│   │   ├── page.tsx       # Order history
│   │   └── [id]/
│   │       └── page.tsx   # Order detail
│   ├── account/
│   │   └── page.tsx
│   ├── login/
│   │   └── page.tsx
│   └── register/
│       └── page.tsx
├── components/
│   ├── ui/                # Generic UI primitives (buttons, inputs, etc.)
│   ├── catalog/           # Product card, product grid, search bar
│   ├── cart/              # Cart drawer, cart item, cart summary
│   ├── checkout/          # Checkout form, payment form, address form
│   └── shared/            # Header, footer, navigation
├── lib/
│   ├── api/               # API client functions (one file per module)
│   │   ├── catalog.ts
│   │   ├── cart.ts
│   │   ├── orders.ts
│   │   └── ...
│   └── hooks/             # React hooks for data fetching
└── types/                 # TypeScript type definitions
```

### Backend Structure (C#/.NET)

Each module follows the same internal folder layout:

```
Modules/Catalog/
├── ECommerceStore.Modules.Catalog.csproj
├── Application/
│   ├── Commands/
│   │   ├── CreateProduct/
│   │   │   ├── CreateProductCommand.cs
│   │   │   └── CreateProductCommandHandler.cs
│   │   └── UpdateProduct/
│   ├── Queries/
│   │   ├── GetProductById/
│   │   │   ├── GetProductByIdQuery.cs
│   │   │   └── GetProductByIdQueryHandler.cs
│   │   └── ListProducts/
│   └── DTOs/
│       └── ProductDto.cs
├── Domain/
│   ├── Aggregates/
│   │   └── Product.cs
│   ├── Entities/
│   │   └── Category.cs
│   ├── ValueObjects/
│   │   ├── ProductName.cs
│   │   └── Money.cs
│   ├── Events/
│   │   └── ProductCreatedEvent.cs
│   ├── Repositories/
│   │   └── IProductRepository.cs
│   └── Services/
│       └── ProductSearchService.cs
├── Infrastructure/
│   ├── Persistence/
│   │   ├── CatalogDbContext.cs
│   │   ├── ProductRepository.cs
│   │   └── Configurations/
│   │       └── ProductConfiguration.cs
│   └── Migrations/
└── Api/
    └── CatalogEndpoints.cs
```

### How the Frontend and Backend Communicate

The Next.js frontend communicates with the .NET backend exclusively through REST HTTP calls. The backend exposes a versioned REST API at `/api/v1/`. CORS is configured on the backend to accept requests from `http://localhost:3000` during development.

All API responses use a consistent JSON envelope:

```json
{
  "success": true,
  "data": { ... },
  "errors": []
}
```

Authentication uses JWT bearer tokens. The frontend stores the token in an HTTP-only cookie (set by the backend on login) and sends it automatically with each request.

---

## 4. Module Breakdown

The backend is divided into ten modules. Each module is a self-contained C# project that owns its own domain logic, data access, and API endpoints. No module reaches into another module's database tables or internal classes.

| Module | Responsibility |
|---|---|
| **Catalog** | Everything customers see when browsing — products, categories, search, and product detail pages. |
| **Inventory** | Tracks how many units of each product are available, reserves stock when a cart item is added, and releases it when an order is cancelled. |
| **Pricing** | Calculates the price a customer actually pays, including base prices, percentage discounts, coupon codes, and tax. |
| **Cart** | Manages the contents of a customer's active shopping cart before checkout begins. |
| **Ordering** | Creates and manages orders after checkout. Orchestrates confirmation, cancellation, and status tracking. |
| **Payment** | Handles payment initiation (talking to a payment gateway), confirms successful payments, handles failures, and processes refunds. |
| **Shipping** | Manages shipping addresses, available shipping methods, shipment creation once an order is paid, and tracking updates. |
| **Customer** | Stores customer profiles, saved addresses, and preferences. |
| **Identity** | Handles user registration, login, JWT issuance, and role-based access control. |
| **Notification** | Listens for domain events from other modules and sends emails (order confirmation, payment receipt, shipping update). |

### Why This Split?

Each module maps to a clearly distinct business concern. The rule of thumb is: if a team of two people could own this area independently, it deserves its own module. This split means:

- You can change how prices are calculated without touching order creation logic.
- You can swap the payment gateway without affecting inventory.
- You can add a new notification type without modifying the ordering module.

---

## 5. Module-by-Module Feature Specification

This section provides the detailed contract for each module. Use these specifications as inputs when writing Superpowers prompts.

---

### 5.1 Catalog Module

**Purpose:** Provides the product catalogue that customers browse. It is the read-heavy public face of the store.

**Main Features:**
- Browse products by category
- Search products by name or description
- View product detail (name, description, images, SKU)
- Manage products and categories (admin)

**Commands:**

| Command | Description |
|---|---|
| `CreateProductCommand` | Creates a new product with name, description, SKU, category, and images |
| `UpdateProductCommand` | Updates product name, description, or images |
| `ArchiveProductCommand` | Soft-deletes a product so it no longer appears in listings |
| `CreateCategoryCommand` | Creates a new category |
| `UpdateCategoryCommand` | Renames a category or changes its parent |

**Queries:**

| Query | Description |
|---|---|
| `GetProductByIdQuery` | Returns full product detail by ID |
| `GetProductBySkuQuery` | Returns product detail by SKU |
| `ListProductsQuery` | Returns a paginated list with optional category filter |
| `SearchProductsQuery` | Full-text search across name and description |
| `ListCategoriesQuery` | Returns the full category tree |

**Domain Models:**

- `Product` *(Aggregate Root)* — owns its identity, name, description, SKU, images, and category reference. Enforces that a SKU is unique and a name is not empty.
- `Category` *(Entity)* — has a name and optional parent category.
- `ProductName` *(Value Object)* — wraps a string, enforces 3–200 characters, trims whitespace.
- `Sku` *(Value Object)* — uppercase alphanumeric string, 4–50 characters.
- `ProductImage` *(Value Object)* — URL and alt text.

**Repository Interfaces:**

```csharp
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(ProductId id, CancellationToken ct);
    Task<Product?> GetBySkuAsync(Sku sku, CancellationToken ct);
    Task<PagedResult<Product>> ListAsync(ListProductsFilter filter, CancellationToken ct);
    Task AddAsync(Product product, CancellationToken ct);
    Task UpdateAsync(Product product, CancellationToken ct);
}

public interface ICategoryRepository
{
    Task<List<Category>> GetAllAsync(CancellationToken ct);
    Task<Category?> GetByIdAsync(CategoryId id, CancellationToken ct);
    Task AddAsync(Category category, CancellationToken ct);
}
```

**Domain Events:**

| Event | Published When |
|---|---|
| `ProductCreatedEvent` | A new product is successfully saved |
| `ProductArchivedEvent` | A product is archived |

**API Endpoints:**

| Method | Path | Description |
|---|---|---|
| GET | `/api/v1/products` | List products (paginated, filterable) |
| GET | `/api/v1/products/{id}` | Get product by ID |
| GET | `/api/v1/products/sku/{sku}` | Get product by SKU |
| GET | `/api/v1/products/search?q=` | Search products |
| POST | `/api/v1/products` | Create product (admin) |
| PUT | `/api/v1/products/{id}` | Update product (admin) |
| DELETE | `/api/v1/products/{id}` | Archive product (admin) |
| GET | `/api/v1/categories` | List categories |
| POST | `/api/v1/categories` | Create category (admin) |

**Database Tables:**

```
catalog.products        (id, sku, name, description, category_id, is_archived, created_at, updated_at)
catalog.product_images  (id, product_id, url, alt_text, sort_order)
catalog.categories      (id, name, parent_id, created_at)
```

**Integration Points:**
- `Inventory` module listens to `ProductCreatedEvent` to initialise a stock record.
- `Pricing` module listens to `ProductCreatedEvent` to initialise a price record.

---

### 5.2 Inventory Module

**Purpose:** Tracks how much stock is available for each product and coordinates reservations so two customers cannot buy the last unit simultaneously.

**Main Features:**
- Track available stock per product
- Reserve stock when a cart item is added
- Release reservation when a cart item is removed or a cart expires
- Deduct stock when an order is confirmed
- Adjust stock (admin)

**Commands:**

| Command | Description |
|---|---|
| `InitialiseStockCommand` | Creates a stock record for a new product with a given quantity |
| `ReserveStockCommand` | Reserves units for a cart item |
| `ReleaseReservationCommand` | Releases a previous reservation |
| `DeductStockCommand` | Permanently deducts stock when an order is confirmed |
| `AdjustStockCommand` | Admin-only manual adjustment (positive or negative) |

**Queries:**

| Query | Description |
|---|---|
| `GetStockLevelQuery` | Returns available and reserved quantities for a product |
| `GetLowStockProductsQuery` | Returns products where available stock is below a threshold |

**Domain Models:**

- `StockItem` *(Aggregate Root)* — owns `productId`, `quantityOnHand`, `quantityReserved`. Computes `quantityAvailable = quantityOnHand - quantityReserved`. Raises a `StockDepletedEvent` when available stock reaches zero.
- `StockReservation` *(Entity)* — records a specific reservation: `cartId`, `quantity`, `expiresAt`.
- `Quantity` *(Value Object)* — a non-negative integer.

**Domain Events:**

| Event | Published When |
|---|---|
| `StockReservedEvent` | Units are successfully reserved |
| `StockReleasedEvent` | A reservation is released |
| `StockDeductedEvent` | Stock is permanently deducted |
| `StockDepletedEvent` | Available quantity reaches zero |

**API Endpoints:**

| Method | Path | Description |
|---|---|---|
| GET | `/api/v1/inventory/{productId}` | Get stock level |
| POST | `/api/v1/inventory/{productId}/reserve` | Reserve stock |
| POST | `/api/v1/inventory/{productId}/release` | Release reservation |
| POST | `/api/v1/inventory/{productId}/adjust` | Manual adjustment (admin) |

**Database Tables:**

```
inventory.stock_items        (id, product_id, quantity_on_hand, quantity_reserved, updated_at)
inventory.stock_reservations (id, stock_item_id, cart_id, quantity, expires_at, created_at)
inventory.stock_adjustments  (id, stock_item_id, delta, reason, adjusted_by, created_at)
```

**Integration Points:**
- Listens to `ProductCreatedEvent` (Catalog) to run `InitialiseStockCommand`.
- Listens to `CartItemAddedEvent` (Cart) to run `ReserveStockCommand`.
- Listens to `CartItemRemovedEvent` (Cart) to run `ReleaseReservationCommand`.
- Listens to `OrderConfirmedEvent` (Ordering) to run `DeductStockCommand`.
- Listens to `OrderCancelledEvent` (Ordering) to run `ReleaseReservationCommand`.

---

### 5.3 Pricing Module

**Purpose:** Calculates the final price a customer pays, including base price, any active discounts, applied coupon codes, and applicable taxes.

**Main Features:**
- Store and update base prices per product
- Apply percentage or fixed-amount discounts to products or categories
- Validate and apply coupon codes
- Calculate tax based on shipping address

**Commands:**

| Command | Description |
|---|---|
| `SetProductPriceCommand` | Sets or updates the base price for a product |
| `CreateDiscountCommand` | Creates a percentage or fixed discount rule |
| `DeactivateDiscountCommand` | Deactivates a discount rule |
| `CreateCouponCommand` | Creates a coupon code with a value and expiry |
| `RedeemCouponCommand` | Marks a coupon as used by a customer |

**Queries:**

| Query | Description |
|---|---|
| `GetProductPriceQuery` | Returns base price for a product |
| `CalculateCartPriceQuery` | Returns itemised pricing for a full cart including discounts and tax |
| `ValidateCouponQuery` | Checks if a coupon code is valid and returns its value |

**Domain Models:**

- `ProductPrice` *(Aggregate Root)* — owns `productId` and `amount` (a `Money` value object).
- `Discount` *(Aggregate Root)* — owns type (percentage/fixed), value, target (product or category), start/end dates.
- `Coupon` *(Aggregate Root)* — owns code, discount value, usage limit, expiry, list of redemptions.
- `Money` *(Value Object)* — amount (decimal) and currency code. Enforces non-negative amounts. Supports addition and percentage reduction.
- `TaxRate` *(Value Object)* — percentage rate tied to a jurisdiction.

**Domain Events:**

| Event | Published When |
|---|---|
| `CouponRedeemedEvent` | A coupon is successfully applied to a cart |

**API Endpoints:**

| Method | Path | Description |
|---|---|---|
| GET | `/api/v1/pricing/products/{productId}` | Get base price |
| POST | `/api/v1/pricing/products/{productId}` | Set base price (admin) |
| POST | `/api/v1/pricing/cart/calculate` | Calculate total cart price |
| POST | `/api/v1/pricing/coupons/validate` | Validate a coupon code |
| POST | `/api/v1/pricing/discounts` | Create a discount (admin) |

**Database Tables:**

```
pricing.product_prices (id, product_id, amount, currency, updated_at)
pricing.discounts      (id, type, value, target_type, target_id, starts_at, ends_at, is_active)
pricing.coupons        (id, code, type, value, usage_limit, used_count, expires_at, created_at)
pricing.coupon_redemptions (id, coupon_id, customer_id, order_id, redeemed_at)
```

**Integration Points:**
- Listens to `ProductCreatedEvent` (Catalog) to run `SetProductPriceCommand` with an initial price of zero.
- The Cart module calls `CalculateCartPriceQuery` when rendering the cart total.
- The Ordering module calls `CalculateCartPriceQuery` when creating an order to lock in prices.

---

### 5.4 Cart Module

**Purpose:** Manages the contents of a customer's active shopping session before checkout begins.

**Main Features:**
- Create a cart (guest or authenticated)
- Add, remove, and update items
- View the cart with live pricing
- Expire abandoned carts

**Commands:**

| Command | Description |
|---|---|
| `CreateCartCommand` | Creates a new empty cart for a customer or guest session |
| `AddItemToCartCommand` | Adds a product to the cart or increments quantity if already present |
| `RemoveItemFromCartCommand` | Removes a line item from the cart |
| `UpdateItemQuantityCommand` | Changes the quantity of a line item |
| `ClearCartCommand` | Removes all items from the cart |
| `ExpireCartCommand` | Marks an abandoned cart as expired |

**Queries:**

| Query | Description |
|---|---|
| `GetCartQuery` | Returns the cart with items, quantities, and calculated totals |

**Domain Models:**

- `Cart` *(Aggregate Root)* — owns `customerId` (nullable for guest), status (`Active`/`CheckedOut`/`Expired`), and a collection of `CartItem` entities. Enforces that a product can only appear once.
- `CartItem` *(Entity)* — owns `productId`, `quantity`, `unitPrice` (snapshotted at add time), `productName`.
- `CartStatus` *(Value Object / Enum)* — Active, CheckedOut, Expired.

**Domain Events:**

| Event | Published When |
|---|---|
| `CartItemAddedEvent` | An item is added to the cart |
| `CartItemRemovedEvent` | An item is removed from the cart |
| `CartCheckedOutEvent` | The cart transitions to CheckedOut |
| `CartExpiredEvent` | The cart is expired |

**API Endpoints:**

| Method | Path | Description |
|---|---|---|
| GET | `/api/v1/cart` | Get current cart |
| POST | `/api/v1/cart` | Create cart |
| POST | `/api/v1/cart/items` | Add item to cart |
| PUT | `/api/v1/cart/items/{productId}` | Update item quantity |
| DELETE | `/api/v1/cart/items/{productId}` | Remove item |
| DELETE | `/api/v1/cart` | Clear cart |

**Database Tables:**

```
cart.carts      (id, customer_id, session_id, status, created_at, updated_at, expires_at)
cart.cart_items (id, cart_id, product_id, product_name, quantity, unit_price, added_at)
```

**Integration Points:**
- Publishes `CartItemAddedEvent` → Inventory module reserves stock.
- Publishes `CartItemRemovedEvent` → Inventory module releases reservation.
- Publishes `CartCheckedOutEvent` → Ordering module uses the cart to create an order.

---

### 5.5 Ordering Module

**Purpose:** Creates and manages orders. An order is the permanent record of a customer's intent to purchase.

**Main Features:**
- Create an order from a checked-out cart
- Confirm an order once payment succeeds
- Cancel an order
- Track order status

**Commands:**

| Command | Description |
|---|---|
| `CreateOrderCommand` | Creates a new order from a cart, locking in items and prices |
| `ConfirmOrderCommand` | Moves an order to Confirmed status after successful payment |
| `CancelOrderCommand` | Cancels a Pending or Confirmed order |
| `UpdateOrderStatusCommand` | Internal use — updates status as fulfilment progresses |

**Queries:**

| Query | Description |
|---|---|
| `GetOrderByIdQuery` | Returns full order detail |
| `ListOrdersForCustomerQuery` | Returns a customer's order history (paginated) |
| `GetOrderStatusQuery` | Returns just the current status of an order |

**Domain Models:**

- `Order` *(Aggregate Root)* — owns `customerId`, `status`, `lineItems`, `shippingAddress`, `pricingSnapshot`, `paymentId`. Status transitions: `Pending → Confirmed → Shipped → Delivered | Cancelled`.
- `OrderLineItem` *(Entity)* — `productId`, `productName`, `quantity`, `unitPrice`.
- `OrderStatus` *(Value Object / Enum)* — Pending, Confirmed, Shipped, Delivered, Cancelled.
- `ShippingAddress` *(Value Object)* — street, city, state, postcode, country.
- `PricingSnapshot` *(Value Object)* — subtotal, discount, tax, total at time of order creation.

**Domain Events:**

| Event | Published When |
|---|---|
| `OrderCreatedEvent` | A new order is saved |
| `OrderConfirmedEvent` | Payment has succeeded and the order is confirmed |
| `OrderCancelledEvent` | An order is cancelled |
| `OrderShippedEvent` | A shipment is created for the order |

**API Endpoints:**

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/orders` | Create order from cart |
| GET | `/api/v1/orders` | List customer's orders |
| GET | `/api/v1/orders/{id}` | Get order detail |
| GET | `/api/v1/orders/{id}/status` | Get order status |
| POST | `/api/v1/orders/{id}/cancel` | Cancel order |

**Database Tables:**

```
ordering.orders            (id, customer_id, status, subtotal, discount, tax, total, currency, shipping_address_json, created_at, updated_at)
ordering.order_line_items  (id, order_id, product_id, product_name, quantity, unit_price)
ordering.order_status_history (id, order_id, from_status, to_status, changed_at, reason)
```

**Integration Points:**
- Listens to `CartCheckedOutEvent` (Cart) to run `CreateOrderCommand`.
- Listens to `PaymentConfirmedEvent` (Payment) to run `ConfirmOrderCommand`.
- Listens to `PaymentFailedEvent` (Payment) to run `CancelOrderCommand`.
- Publishes `OrderConfirmedEvent` → Inventory deducts stock, Notification sends confirmation email.
- Publishes `OrderCancelledEvent` → Inventory releases reservations.
- Publishes `OrderShippedEvent` → Notification sends shipping update.

---

### 5.6 Payment Module

**Purpose:** Handles all money movement — initiating a payment with the gateway, recording the result, and processing refunds.

**Main Features:**
- Initiate a payment for an order
- Record a successful payment confirmation
- Handle and record a failed payment
- Initiate a refund

**Commands:**

| Command | Description |
|---|---|
| `InitiatePaymentCommand` | Calls the payment gateway and creates a pending payment record |
| `ConfirmPaymentCommand` | Records a successful gateway callback |
| `RecordPaymentFailureCommand` | Records a failed payment attempt |
| `InitiateRefundCommand` | Initiates a full or partial refund via the gateway |

**Queries:**

| Query | Description |
|---|---|
| `GetPaymentForOrderQuery` | Returns the payment record for an order |
| `GetRefundStatusQuery` | Returns the status of a refund |

**Domain Models:**

- `Payment` *(Aggregate Root)* — owns `orderId`, `amount`, `currency`, `status`, `gatewayReference`, `gatewayResponse`.
- `Refund` *(Entity)* — `paymentId`, `amount`, `reason`, `status`, `gatewayRefundReference`.
- `PaymentStatus` *(Value Object / Enum)* — Pending, Confirmed, Failed, Refunded.
- `Money` *(Value Object)* — shared with Pricing via the Shared kernel.

**Domain Events:**

| Event | Published When |
|---|---|
| `PaymentInitiatedEvent` | Payment request is sent to gateway |
| `PaymentConfirmedEvent` | Gateway confirms success |
| `PaymentFailedEvent` | Gateway reports failure |
| `RefundInitiatedEvent` | Refund is requested |

**API Endpoints:**

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/payments` | Initiate payment |
| POST | `/api/v1/payments/confirm` | Gateway webhook — confirm payment |
| POST | `/api/v1/payments/fail` | Gateway webhook — record failure |
| POST | `/api/v1/payments/{id}/refund` | Initiate refund |
| GET | `/api/v1/payments/orders/{orderId}` | Get payment for order |

**Database Tables:**

```
payment.payments (id, order_id, amount, currency, status, gateway_reference, gateway_response_json, created_at, updated_at)
payment.refunds  (id, payment_id, amount, reason, status, gateway_refund_reference, created_at, updated_at)
```

**Integration Points:**
- Listens to `OrderCreatedEvent` (Ordering) to begin the payment flow.
- Publishes `PaymentConfirmedEvent` → Ordering confirms the order.
- Publishes `PaymentFailedEvent` → Ordering cancels the order.
- Publishes `RefundInitiatedEvent` → Notification sends refund email.

---

### 5.7 Shipping Module

**Purpose:** Manages everything related to physically delivering an order to the customer.

**Main Features:**
- Store and validate shipping addresses
- Present available shipping methods with rates
- Create a shipment when an order is confirmed
- Record and display tracking updates

**Commands:**

| Command | Description |
|---|---|
| `SaveShippingAddressCommand` | Saves a validated shipping address to a customer's profile |
| `CreateShipmentCommand` | Creates a shipment record for a confirmed order |
| `UpdateTrackingCommand` | Adds a new tracking event to a shipment |

**Queries:**

| Query | Description |
|---|---|
| `GetShippingMethodsQuery` | Returns available shipping methods and rates for an address |
| `GetShipmentForOrderQuery` | Returns shipment and tracking info for an order |

**Domain Models:**

- `Shipment` *(Aggregate Root)* — owns `orderId`, `address`, `shippingMethod`, `trackingNumber`, `carrier`, and a list of `TrackingEvent` entities.
- `TrackingEvent` *(Entity)* — `description`, `location`, `occurredAt`.
- `ShippingAddress` *(Value Object)* — same structure as in Ordering; both use the Shared kernel definition.
- `ShippingMethod` *(Value Object)* — name, carrier, estimated days, rate.

**Domain Events:**

| Event | Published When |
|---|---|
| `ShipmentCreatedEvent` | A shipment record is created |
| `ShipmentDispatchedEvent` | Carrier confirms pickup |

**API Endpoints:**

| Method | Path | Description |
|---|---|---|
| GET | `/api/v1/shipping/methods` | Get available shipping methods |
| GET | `/api/v1/shipping/orders/{orderId}` | Get shipment for order |
| POST | `/api/v1/shipping/shipments` | Create shipment (internal/admin) |
| POST | `/api/v1/shipping/tracking` | Webhook — add tracking update |

**Database Tables:**

```
shipping.shipments      (id, order_id, tracking_number, carrier, shipping_method, address_json, created_at)
shipping.tracking_events (id, shipment_id, description, location, occurred_at)
```

**Integration Points:**
- Listens to `OrderConfirmedEvent` (Ordering) to run `CreateShipmentCommand`.
- Publishes `ShipmentCreatedEvent` → Ordering updates status to Shipped.
- Publishes `ShipmentCreatedEvent` → Notification sends shipping email.

---

### 5.8 Customer Module

**Purpose:** Manages customer profile data, saved addresses, and preferences.

**Main Features:**
- Create a customer profile on registration
- Update profile information
- Manage saved addresses
- Store notification preferences

**Commands:**

| Command | Description |
|---|---|
| `CreateCustomerProfileCommand` | Creates a profile linked to an Identity user |
| `UpdateCustomerProfileCommand` | Updates display name, phone, or avatar |
| `AddAddressCommand` | Saves a new address to the customer's profile |
| `RemoveAddressCommand` | Deletes a saved address |
| `UpdatePreferencesCommand` | Updates notification and display preferences |

**Queries:**

| Query | Description |
|---|---|
| `GetCustomerProfileQuery` | Returns the customer's full profile |
| `GetCustomerAddressesQuery` | Returns all saved addresses |

**Domain Models:**

- `Customer` *(Aggregate Root)* — owns `userId`, `displayName`, `email`, `phone`, `addresses`, `preferences`.
- `Address` *(Entity)* — same fields as ShippingAddress plus a label (e.g. "Home", "Work").
- `CustomerPreferences` *(Value Object)* — email notifications on/off, preferred currency.

**API Endpoints:**

| Method | Path | Description |
|---|---|---|
| GET | `/api/v1/customers/me` | Get own profile |
| PUT | `/api/v1/customers/me` | Update own profile |
| GET | `/api/v1/customers/me/addresses` | List saved addresses |
| POST | `/api/v1/customers/me/addresses` | Add address |
| DELETE | `/api/v1/customers/me/addresses/{id}` | Remove address |
| PUT | `/api/v1/customers/me/preferences` | Update preferences |

**Database Tables:**

```
customer.customers    (id, user_id, display_name, email, phone, created_at, updated_at)
customer.addresses    (id, customer_id, label, street, city, state, postcode, country, is_default)
customer.preferences  (id, customer_id, email_notifications, preferred_currency)
```

**Integration Points:**
- Listens to `UserRegisteredEvent` (Identity) to run `CreateCustomerProfileCommand`.

---

### 5.9 Identity and Access Module

**Purpose:** Handles everything related to who a user is and what they are allowed to do.

**Main Features:**
- Register a new user account
- Log in with email and password
- Issue and validate JWT tokens
- Assign and enforce roles (Customer, Admin)

**Commands:**

| Command | Description |
|---|---|
| `RegisterUserCommand` | Creates a new user account with hashed password |
| `LoginCommand` | Validates credentials and returns a JWT |
| `RefreshTokenCommand` | Issues a new JWT using a refresh token |
| `ChangePasswordCommand` | Updates a user's password |
| `AssignRoleCommand` | Assigns a role to a user (admin only) |

**Queries:**

| Query | Description |
|---|---|
| `GetUserByIdQuery` | Returns a user record |
| `GetUserByEmailQuery` | Looks up a user by email |

**Domain Models:**

- `User` *(Aggregate Root)* — owns `email`, `passwordHash`, `roles`, `isActive`.
- `Role` *(Value Object / Enum)* — Customer, Admin.
- `Email` *(Value Object)* — normalised email string with format validation.
- `RefreshToken` *(Entity)* — `token`, `expiresAt`, `isRevoked`.

**Domain Events:**

| Event | Published When |
|---|---|
| `UserRegisteredEvent` | A new user account is created |
| `UserLoggedInEvent` | A user successfully logs in |

**API Endpoints:**

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/auth/register` | Register |
| POST | `/api/v1/auth/login` | Login |
| POST | `/api/v1/auth/refresh` | Refresh JWT |
| POST | `/api/v1/auth/change-password` | Change password |

**Database Tables:**

```
identity.users          (id, email, password_hash, is_active, created_at, updated_at)
identity.user_roles     (user_id, role)
identity.refresh_tokens (id, user_id, token, expires_at, is_revoked, created_at)
```

**Integration Points:**
- Publishes `UserRegisteredEvent` → Customer module creates a customer profile.

---

### 5.10 Notification Module

**Purpose:** Sends transactional emails triggered by events in other modules. This module owns no domain data of its own — it is purely reactive.

**Main Features:**
- Send order confirmation email when an order is confirmed
- Send payment receipt email when a payment is confirmed
- Send shipping update email when a shipment is created
- Send refund notification email when a refund is initiated

**Handlers (Event Consumers):**

| Event Consumed | Action |
|---|---|
| `OrderConfirmedEvent` | Sends order confirmation email |
| `PaymentConfirmedEvent` | Sends payment receipt email |
| `ShipmentCreatedEvent` | Sends shipping notification email |
| `RefundInitiatedEvent` | Sends refund notification email |

**Domain Models:**

There are no domain aggregates in this module. Each handler reads the event payload, loads any additional data it needs via queries to other modules' read models, renders an email template, and sends it.

**Database Tables:**

```
notification.sent_notifications (id, recipient_email, template_name, sent_at, status, payload_json)
```

**Integration Points:**
- Purely a consumer. Subscribes to events from Ordering, Payment, and Shipping.

---

## 6. Backend Architecture Tutorial

### Step 1 — Create the .NET Solution

```bash
mkdir backend
cd backend

# Create the solution file
dotnet new sln -n ECommerceStore

# Create the API host project
dotnet new webapi -n ECommerceStore.Api -o src/Api
dotnet sln add src/Api/ECommerceStore.Api.csproj

# Create the Shared kernel
dotnet new classlib -n ECommerceStore.Shared -o src/Shared
dotnet sln add src/Shared/ECommerceStore.Shared.csproj

# Create module class libraries (repeat for each module)
dotnet new classlib -n ECommerceStore.Modules.Catalog -o src/Modules/Catalog
dotnet sln add src/Modules/Catalog/ECommerceStore.Modules.Catalog.csproj

dotnet new classlib -n ECommerceStore.Modules.Inventory -o src/Modules/Inventory
dotnet sln add src/Modules/Inventory/ECommerceStore.Modules.Inventory.csproj

dotnet new classlib -n ECommerceStore.Modules.Pricing -o src/Modules/Pricing
dotnet sln add src/Modules/Pricing/ECommerceStore.Modules.Pricing.csproj

dotnet new classlib -n ECommerceStore.Modules.Cart -o src/Modules/Cart
dotnet sln add src/Modules/Cart/ECommerceStore.Modules.Cart.csproj

dotnet new classlib -n ECommerceStore.Modules.Ordering -o src/Modules/Ordering
dotnet sln add src/Modules/Ordering/ECommerceStore.Modules.Ordering.csproj

dotnet new classlib -n ECommerceStore.Modules.Payment -o src/Modules/Payment
dotnet sln add src/Modules/Payment/ECommerceStore.Modules.Payment.csproj

dotnet new classlib -n ECommerceStore.Modules.Shipping -o src/Modules/Shipping
dotnet sln add src/Modules/Shipping/ECommerceStore.Modules.Shipping.csproj

dotnet new classlib -n ECommerceStore.Modules.Customer -o src/Modules/Customer
dotnet sln add src/Modules/Customer/ECommerceStore.Modules.Customer.csproj

dotnet new classlib -n ECommerceStore.Modules.Identity -o src/Modules/Identity
dotnet sln add src/Modules/Identity/ECommerceStore.Modules.Identity.csproj

dotnet new classlib -n ECommerceStore.Modules.Notification -o src/Modules/Notification
dotnet sln add src/Modules/Notification/ECommerceStore.Modules.Notification.csproj
```

### Step 2 — Install Core NuGet Packages

```bash
# MediatR for CQRS and domain events
dotnet add src/Shared package MediatR
dotnet add src/Api package MediatR.Extensions.Microsoft.DependencyInjection

# EF Core for all module projects
dotnet add src/Shared package Microsoft.EntityFrameworkCore
dotnet add src/Modules/Catalog package Npgsql.EntityFrameworkCore.PostgreSQL

# FluentValidation for request validation
dotnet add src/Shared package FluentValidation

# JWT authentication
dotnet add src/Modules/Identity package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add src/Modules/Identity package System.IdentityModel.Tokens.Jwt
```

### Step 3 — Define the Shared Kernel

The Shared kernel contains base classes and interfaces that every module can use. It must not contain any business logic.

```csharp
// src/Shared/Domain/AggregateRoot.cs
public abstract class AggregateRoot<TId>
{
    public TId Id { get; protected set; } = default!;

    private readonly List<INotification> _domainEvents = new();
    public IReadOnlyList<INotification> DomainEvents => _domainEvents.AsReadOnly();

    protected void RaiseDomainEvent(INotification domainEvent)
        => _domainEvents.Add(domainEvent);

    public void ClearDomainEvents()
        => _domainEvents.Clear();
}

// src/Shared/Domain/Entity.cs
public abstract class Entity<TId>
{
    public TId Id { get; protected set; } = default!;
}

// src/Shared/Domain/ValueObject.cs
public abstract class ValueObject
{
    protected abstract IEnumerable<object?> GetEqualityComponents();

    public override bool Equals(object? obj)
    {
        if (obj is null || obj.GetType() != GetType()) return false;
        return ((ValueObject)obj).GetEqualityComponents()
            .SequenceEqual(GetEqualityComponents());
    }

    public override int GetHashCode()
        => GetEqualityComponents()
            .Aggregate(1, (hash, obj) => HashCode.Combine(hash, obj?.GetHashCode() ?? 0));
}

// src/Shared/Application/PagedResult.cs
public record PagedResult<T>(List<T> Items, int TotalCount, int Page, int PageSize);

// src/Shared/Application/ICommand.cs
public interface ICommand<TResult> : IRequest<TResult> { }

// src/Shared/Application/IQuery.cs
public interface IQuery<TResult> : IRequest<TResult> { }
```

### Step 4 — Implement CQRS in a Module

Here is a complete example using the Catalog module's `CreateProductCommand`.

**Command:**

```csharp
// src/Modules/Catalog/Application/Commands/CreateProduct/CreateProductCommand.cs
public record CreateProductCommand(
    string Name,
    string Description,
    string Sku,
    Guid CategoryId,
    decimal Price
) : ICommand<Guid>;
```

**Command Handler:**

```csharp
// src/Modules/Catalog/Application/Commands/CreateProduct/CreateProductCommandHandler.cs
public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, Guid>
{
    private readonly IProductRepository _productRepository;
    private readonly IPublisher _publisher;

    public CreateProductCommandHandler(
        IProductRepository productRepository,
        IPublisher publisher)
    {
        _productRepository = productRepository;
        _publisher = publisher;
    }

    public async Task<Guid> Handle(
        CreateProductCommand request,
        CancellationToken cancellationToken)
    {
        var product = Product.Create(
            ProductName.Create(request.Name),
            Sku.Create(request.Sku),
            request.Description,
            new CategoryId(request.CategoryId));

        await _productRepository.AddAsync(product, cancellationToken);

        foreach (var domainEvent in product.DomainEvents)
            await _publisher.Publish(domainEvent, cancellationToken);

        product.ClearDomainEvents();

        return product.Id.Value;
    }
}
```

**Query:**

```csharp
// src/Modules/Catalog/Application/Queries/GetProductById/GetProductByIdQuery.cs
public record GetProductByIdQuery(Guid ProductId) : IQuery<ProductDto?>;
```

**Query Handler:**

```csharp
// src/Modules/Catalog/Application/Queries/GetProductById/GetProductByIdQueryHandler.cs
public class GetProductByIdQueryHandler : IRequestHandler<GetProductByIdQuery, ProductDto?>
{
    private readonly IProductRepository _productRepository;

    public GetProductByIdQueryHandler(IProductRepository productRepository)
        => _productRepository = productRepository;

    public async Task<ProductDto?> Handle(
        GetProductByIdQuery request,
        CancellationToken cancellationToken)
    {
        var product = await _productRepository
            .GetByIdAsync(new ProductId(request.ProductId), cancellationToken);

        return product is null ? null : ProductDto.From(product);
    }
}
```

### Step 5 — Implement the Domain Aggregate

```csharp
// src/Modules/Catalog/Domain/Aggregates/Product.cs
public class Product : AggregateRoot<ProductId>
{
    public ProductName Name { get; private set; } = null!;
    public string Description { get; private set; } = string.Empty;
    public Sku Sku { get; private set; } = null!;
    public CategoryId CategoryId { get; private set; } = null!;
    public bool IsArchived { get; private set; }
    public DateTime CreatedAt { get; private set; }

    private Product() { }

    public static Product Create(
        ProductName name,
        Sku sku,
        string description,
        CategoryId categoryId)
    {
        var product = new Product
        {
            Id = new ProductId(Guid.NewGuid()),
            Name = name,
            Sku = sku,
            Description = description,
            CategoryId = categoryId,
            IsArchived = false,
            CreatedAt = DateTime.UtcNow
        };

        product.RaiseDomainEvent(new ProductCreatedEvent(product.Id, product.Sku));
        return product;
    }

    public void Archive()
    {
        if (IsArchived) return;
        IsArchived = true;
        RaiseDomainEvent(new ProductArchivedEvent(Id));
    }
}
```

### Step 6 — Implement a Repository

```csharp
// src/Modules/Catalog/Infrastructure/Persistence/ProductRepository.cs
public class ProductRepository : IProductRepository
{
    private readonly CatalogDbContext _context;

    public ProductRepository(CatalogDbContext context) => _context = context;

    public async Task<Product?> GetByIdAsync(ProductId id, CancellationToken ct)
        => await _context.Products.FirstOrDefaultAsync(p => p.Id == id, ct);

    public async Task<PagedResult<Product>> ListAsync(
        ListProductsFilter filter,
        CancellationToken ct)
    {
        var query = _context.Products
            .Where(p => !p.IsArchived);

        if (filter.CategoryId.HasValue)
            query = query.Where(p => p.CategoryId == new CategoryId(filter.CategoryId.Value));

        var total = await query.CountAsync(ct);
        var items = await query
            .Skip((filter.Page - 1) * filter.PageSize)
            .Take(filter.PageSize)
            .ToListAsync(ct);

        return new PagedResult<Product>(items, total, filter.Page, filter.PageSize);
    }

    public async Task AddAsync(Product product, CancellationToken ct)
    {
        await _context.Products.AddAsync(product, ct);
        await _context.SaveChangesAsync(ct);
    }

    public async Task UpdateAsync(Product product, CancellationToken ct)
    {
        _context.Products.Update(product);
        await _context.SaveChangesAsync(ct);
    }
}
```

### Step 7 — Handle Domain Events Across Modules

Domain events are the only way modules communicate. When the Catalog module raises a `ProductCreatedEvent`, the Inventory module handles it to create a stock record. MediatR's `INotificationHandler<T>` interface makes this clean.

```csharp
// src/Modules/Catalog/Domain/Events/ProductCreatedEvent.cs
public record ProductCreatedEvent(ProductId ProductId, Sku Sku) : INotification;

// src/Modules/Inventory/Application/EventHandlers/ProductCreatedEventHandler.cs
public class ProductCreatedEventHandler : INotificationHandler<ProductCreatedEvent>
{
    private readonly ISender _mediator;

    public ProductCreatedEventHandler(ISender mediator) => _mediator = mediator;

    public async Task Handle(ProductCreatedEvent notification, CancellationToken ct)
    {
        await _mediator.Send(new InitialiseStockCommand(
            notification.ProductId.Value,
            quantityOnHand: 0), ct);
    }
}
```

The key rule: a handler in module B may only send commands or queries *within* module B. It must not directly manipulate module A's data.

### Step 8 — Register Modules in the API Host

Each module exposes a static `AddCatalogModule` extension method that registers its own services, DbContext, and MediatR handlers.

```csharp
// src/Modules/Catalog/CatalogModule.cs
public static class CatalogModule
{
    public static IServiceCollection AddCatalogModule(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.AddDbContext<CatalogDbContext>(options =>
            options.UseNpgsql(configuration.GetConnectionString("Default"),
                npgsql => npgsql.MigrationsHistoryTable(
                    "__EFMigrationsHistory", "catalog")));

        services.AddScoped<IProductRepository, ProductRepository>();
        services.AddScoped<ICategoryRepository, CategoryRepository>();

        services.AddMediatR(cfg =>
            cfg.RegisterServicesFromAssembly(typeof(CatalogModule).Assembly));

        return services;
    }
}

// src/Api/Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddCatalogModule(builder.Configuration)
    .AddInventoryModule(builder.Configuration)
    .AddPricingModule(builder.Configuration)
    .AddCartModule(builder.Configuration)
    .AddOrderingModule(builder.Configuration)
    .AddPaymentModule(builder.Configuration)
    .AddShippingModule(builder.Configuration)
    .AddCustomerModule(builder.Configuration)
    .AddIdentityModule(builder.Configuration)
    .AddNotificationModule(builder.Configuration);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
app.MapControllers();
app.Run();
```

### Step 9 — Create API Controllers

Each module contributes its own controller, which lives in `src/Api/Controllers/` and depends only on `ISender` (MediatR).

```csharp
// src/Api/Controllers/CatalogController.cs
[ApiController]
[Route("api/v1/products")]
public class CatalogController : ControllerBase
{
    private readonly ISender _mediator;

    public CatalogController(ISender mediator) => _mediator = mediator;

    [HttpGet]
    public async Task<IActionResult> List([FromQuery] ListProductsQuery query, CancellationToken ct)
    {
        var result = await _mediator.Send(query, ct);
        return Ok(result);
    }

    [HttpGet("{id:guid}")]
    public async Task<IActionResult> GetById(Guid id, CancellationToken ct)
    {
        var result = await _mediator.Send(new GetProductByIdQuery(id), ct);
        return result is null ? NotFound() : Ok(result);
    }

    [HttpPost]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> Create(
        [FromBody] CreateProductCommand command,
        CancellationToken ct)
    {
        var id = await _mediator.Send(command, ct);
        return CreatedAtAction(nameof(GetById), new { id }, new { id });
    }
}
```

---

## 7. Frontend Architecture Tutorial

### Step 1 — Create the Next.js Project

```bash
cd ecommerce-store
npx create-next-app@latest frontend \
  --typescript \
  --tailwind \
  --app \
  --src-dir \
  --import-alias "@/*"
cd frontend
```

Install additional dependencies:

```bash
npm install axios react-query @tanstack/react-query zod react-hook-form @hookform/resolvers
```

### Step 2 — Configure the API Client

Create a base Axios instance that automatically attaches the auth token:

```typescript
// src/lib/api/client.ts
import axios from "axios";

export const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL ?? "http://localhost:5000",
  withCredentials: true,
});

apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem("auth_token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem("auth_token");
      window.location.href = "/login";
    }
    return Promise.reject(error);
  }
);
```

Create per-module API function files:

```typescript
// src/lib/api/catalog.ts
import { apiClient } from "./client";

export interface Product {
  id: string;
  name: string;
  description: string;
  sku: string;
  categoryId: string;
  images: { url: string; altText: string }[];
}

export interface PagedResult<T> {
  items: T[];
  totalCount: number;
  page: number;
  pageSize: number;
}

export async function listProducts(params?: {
  categoryId?: string;
  page?: number;
  pageSize?: number;
}): Promise<PagedResult<Product>> {
  const { data } = await apiClient.get("/api/v1/products", { params });
  return data;
}

export async function getProduct(id: string): Promise<Product> {
  const { data } = await apiClient.get(`/api/v1/products/${id}`);
  return data;
}

export async function searchProducts(q: string): Promise<PagedResult<Product>> {
  const { data } = await apiClient.get("/api/v1/products/search", {
    params: { q },
  });
  return data;
}
```

### Step 3 — Product Listing Page

```typescript
// src/app/page.tsx
"use client";

import { useQuery } from "@tanstack/react-query";
import { listProducts } from "@/lib/api/catalog";
import ProductCard from "@/components/catalog/ProductCard";
import SearchBar from "@/components/catalog/SearchBar";

export default function HomePage() {
  const { data, isLoading, error } = useQuery({
    queryKey: ["products"],
    queryFn: () => listProducts({ page: 1, pageSize: 20 }),
  });

  if (isLoading) return <div className="p-8 text-center">Loading products...</div>;
  if (error) return <div className="p-8 text-center text-red-500">Failed to load products.</div>;

  return (
    <main className="max-w-7xl mx-auto px-4 py-8">
      <SearchBar />
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mt-6">
        {data?.items.map((product) => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
    </main>
  );
}
```

### Step 4 — Product Detail Page

```typescript
// src/app/products/[id]/page.tsx
import { getProduct } from "@/lib/api/catalog";
import AddToCartButton from "@/components/cart/AddToCartButton";

interface Props {
  params: { id: string };
}

export default async function ProductDetailPage({ params }: Props) {
  const product = await getProduct(params.id);

  return (
    <div className="max-w-4xl mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold">{product.name}</h1>
      <p className="text-gray-600 mt-4">{product.description}</p>
      <AddToCartButton productId={product.id} />
    </div>
  );
}
```

### Step 5 — Cart Page

```typescript
// src/app/cart/page.tsx
"use client";

import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { getCart, removeCartItem, updateCartItemQuantity } from "@/lib/api/cart";
import Link from "next/link";

export default function CartPage() {
  const queryClient = useQueryClient();
  const { data: cart, isLoading } = useQuery({
    queryKey: ["cart"],
    queryFn: getCart,
  });

  const removeMutation = useMutation({
    mutationFn: removeCartItem,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ["cart"] }),
  });

  if (isLoading) return <div>Loading cart...</div>;
  if (!cart?.items.length) {
    return (
      <div className="text-center py-16">
        <p>Your cart is empty.</p>
        <Link href="/" className="text-blue-600 underline">
          Continue shopping
        </Link>
      </div>
    );
  }

  return (
    <div className="max-w-2xl mx-auto px-4 py-8">
      <h1 className="text-2xl font-bold mb-6">Your Cart</h1>
      {cart.items.map((item) => (
        <div key={item.productId} className="flex justify-between items-center border-b py-4">
          <span>{item.productName}</span>
          <span>x{item.quantity}</span>
          <span>${(item.unitPrice * item.quantity).toFixed(2)}</span>
          <button
            onClick={() => removeMutation.mutate(item.productId)}
            className="text-red-500 text-sm"
          >
            Remove
          </button>
        </div>
      ))}
      <Link
        href="/checkout"
        className="block mt-6 bg-blue-600 text-white text-center py-3 rounded"
      >
        Proceed to Checkout
      </Link>
    </div>
  );
}
```

### Step 6 — Login Page with Validation

```typescript
// src/app/login/page.tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import { useMutation } from "@tanstack/react-query";
import { login } from "@/lib/api/auth";
import { useRouter } from "next/navigation";

const schema = z.object({
  email: z.string().email("Enter a valid email address"),
  password: z.string().min(8, "Password must be at least 8 characters"),
});

type FormData = z.infer<typeof schema>;

export default function LoginPage() {
  const router = useRouter();
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<FormData>({ resolver: zodResolver(schema) });

  const mutation = useMutation({
    mutationFn: login,
    onSuccess: (data) => {
      localStorage.setItem("auth_token", data.token);
      router.push("/");
    },
  });

  return (
    <div className="max-w-md mx-auto px-4 py-16">
      <h1 className="text-2xl font-bold mb-6">Sign In</h1>
      <form onSubmit={handleSubmit((data) => mutation.mutate(data))}>
        <div className="mb-4">
          <label className="block text-sm font-medium mb-1">Email</label>
          <input
            {...register("email")}
            type="email"
            className="w-full border rounded px-3 py-2"
          />
          {errors.email && (
            <p className="text-red-500 text-sm mt-1">{errors.email.message}</p>
          )}
        </div>
        <div className="mb-6">
          <label className="block text-sm font-medium mb-1">Password</label>
          <input
            {...register("password")}
            type="password"
            className="w-full border rounded px-3 py-2"
          />
          {errors.password && (
            <p className="text-red-500 text-sm mt-1">{errors.password.message}</p>
          )}
        </div>
        {mutation.isError && (
          <p className="text-red-500 text-sm mb-4">
            Login failed. Please check your credentials.
          </p>
        )}
        <button
          type="submit"
          disabled={mutation.isPending}
          className="w-full bg-blue-600 text-white py-2 rounded"
        >
          {mutation.isPending ? "Signing in..." : "Sign In"}
        </button>
      </form>
    </div>
  );
}
```

---

## 8. Step-by-Step Superpowers Workflow

### Phase 1 — Initialise the Project

```bash
cd ecommerce-store
superpowers init
```

Write your first specification file, which grounds every future prompt:

```bash
# .superpowers/specs/architecture.md
```

Paste the following content into that file:

```markdown
# E-Commerce Store Architecture

## Stack
- Frontend: Next.js 14, TypeScript, Tailwind CSS, TanStack Query
- Backend: C# .NET 8, ASP.NET Core Web API
- Database: PostgreSQL 15
- ORM: Entity Framework Core 8 with Npgsql
- CQRS: MediatR
- Validation: FluentValidation
- Authentication: JWT Bearer tokens

## Architecture Pattern
- Modular monolith backend with ten modules
- CQRS inside each module (commands for writes, queries for reads)
- DDD tactical patterns: aggregates, entities, value objects, domain services, repositories, domain events
- Domain events for cross-module communication (via MediatR INotification)
- No module accesses another module's DbContext or internal classes

## Naming Conventions
- Commands: [Verb][Noun]Command (e.g. CreateProductCommand)
- Queries: [Get|List][Noun]Query (e.g. GetProductByIdQuery)
- Handlers: [CommandName]Handler, [QueryName]Handler
- Events: [Noun][PastTense]Event (e.g. ProductCreatedEvent)
- Repositories: I[Aggregate]Repository (e.g. IProductRepository)
- DTOs: [Noun]Dto (e.g. ProductDto)

## Module List
Catalog, Inventory, Pricing, Cart, Ordering, Payment, Shipping, Customer, Identity, Notification
```

Now run your first Superpowers command:

```bash
superpowers run "Read architecture.md. Generate the complete .NET solution structure: the sln file, all module class library project files, and the API host project. Include the correct project references."
```

### Phase 2 — Generate a Backend Module

For each module, write a spec file and generate the code:

```bash
superpowers run "Read architecture.md and catalog-module.md. Generate the complete Catalog module: domain aggregates, value objects, domain events, repository interfaces, commands, command handlers, queries, query handlers, EF Core DbContext, repository implementation, and the ASP.NET Core controller. Follow the naming conventions in architecture.md."
```

### Phase 3 — Add CQRS Handlers

```bash
superpowers run "Read architecture.md. In the Ordering module, generate the CreateOrderCommand, its handler, the OrderCreatedEvent, and the EF Core persistence code. The handler should save the order and publish the domain event."
```

### Phase 4 — Generate API Endpoints

```bash
superpowers run "Read architecture.md. Generate the CartController for the Cart module. It should expose the endpoints defined in the cart-module spec: GET /api/v1/cart, POST /api/v1/cart/items, PUT /api/v1/cart/items/{productId}, DELETE /api/v1/cart/items/{productId}. Use MediatR ISender to dispatch commands and queries."
```

### Phase 5 — Generate Database Migrations

```bash
# After generating the module's DbContext and entity configurations
superpowers run "Read architecture.md. Write the EF Core IEntityTypeConfiguration classes for the Catalog module. Use the catalog schema prefix. Include configurations for Product, Category, and ProductImage."
```

Then run migrations from the terminal:

```bash
dotnet ef migrations add InitialCatalogSchema \
  --project src/Modules/Catalog \
  --startup-project src/Api \
  --context CatalogDbContext \
  --output-dir Infrastructure/Migrations
```

### Phase 6 — Generate Frontend Pages

```bash
superpowers run "Read architecture.md. Generate the Next.js checkout page at src/app/checkout/page.tsx. It should: fetch the current cart, show an address form, show shipping method selection, show a payment form, and submit a create-order request followed by a payment initiation request. Use react-hook-form with zod validation."
```

### Phase 7 — Connect Frontend to Backend

```bash
superpowers run "Read architecture.md. Generate the TypeScript API client file src/lib/api/orders.ts. Include functions: createOrder, listOrders, getOrder, cancelOrder. Each function should call the correct backend endpoint and return typed data."
```

### Phase 8 — Review and Improve Generated Code

```bash
superpowers run "Read architecture.md. Review the Catalog module's ProductRepository. Check that: it uses async/await correctly, handles cancellation tokens, does not leak DbContext, and follows the repository interface contract. Suggest improvements."
```

### Phase 9 — Write Tests

```bash
superpowers run "Read architecture.md. Write unit tests for CreateProductCommandHandler. Use xUnit and Moq. Test: (1) a valid product is saved and the domain event is published, (2) an invalid name throws a validation error."
```

### Phase 10 — Fix Bugs

```bash
superpowers run "I am getting this error: [paste error]. Read architecture.md and the relevant module spec. Identify the cause and generate a fix."
```

---

## 9. Superpowers Prompt Examples

The prompts below are ready to copy and paste. Replace values in `[brackets]` with your specifics.

### Generating the Full Solution Structure

```
Read architecture.md. Generate the complete .NET 8 solution file and all .csproj files for:
- ECommerceStore.sln
- src/Api/ECommerceStore.Api.csproj (ASP.NET Core Web API)
- src/Shared/ECommerceStore.Shared.csproj (Class Library)
- One Class Library .csproj for each of the ten modules

Include the correct project references: each module references Shared, and Api references all modules.
Include the NuGet packages: MediatR, EF Core, Npgsql, FluentValidation, and JWT Bearer.
```

### Creating a Single Backend Module

```
Read architecture.md and [module-name]-module.md.
Generate the complete [ModuleName] module with the following structure:
- Domain/Aggregates/[AggregateName].cs
- Domain/ValueObjects/ (all value objects listed in the spec)
- Domain/Events/ (all domain events listed in the spec)
- Domain/Repositories/I[AggregateName]Repository.cs
- Application/Commands/ (one folder per command, each with Command.cs and CommandHandler.cs)
- Application/Queries/ (one folder per query, each with Query.cs and QueryHandler.cs)
- Application/DTOs/[AggregateName]Dto.cs
- Infrastructure/Persistence/[Module]DbContext.cs
- Infrastructure/Persistence/[AggregateName]Repository.cs
- Infrastructure/Persistence/Configurations/[AggregateName]Configuration.cs
- [Module]Module.cs (service registration extension method)
Follow all naming conventions in architecture.md.
```

### Creating CQRS Commands and Queries

```
Read architecture.md.
In the [ModuleName] module, generate the following:
1. [CommandName]Command.cs — a record with properties: [list properties]
2. [CommandName]CommandHandler.cs — implements IRequestHandler<[CommandName]Command, [ReturnType]>
   It should: load the aggregate, call the domain method, save via repository, publish domain events.
3. [QueryName]Query.cs — a record with properties: [list properties]
4. [QueryName]QueryHandler.cs — implements IRequestHandler<[QueryName]Query, [ReturnType]>
   It should: load from repository and map to DTO.
```

### Creating API Endpoints

```
Read architecture.md.
Generate [ModuleName]Controller.cs in src/Api/Controllers/.
The controller should:
- Use route prefix /api/v1/[resource]
- Inject ISender from MediatR
- Implement these endpoints:
  [list endpoints from the module spec]
- Apply [Authorize] with the correct roles where indicated
- Return 201 Created for POST, 200 OK for GET, 404 NotFound when entity is missing
```

### Creating EF Core Entity Configurations and Migrations

```
Read architecture.md.
Generate the EF Core IEntityTypeConfiguration<[EntityName]> class for [EntityName] in the [ModuleName] module.
The entity has these properties: [list properties and types].
Use the schema "[module_schema_name]".
Map value objects as owned entities or as simple column conversions.
Use snake_case column names.
```

### Creating Frontend Pages in Next.js

```
Read architecture.md.
Generate the Next.js page at src/app/[path]/page.tsx.
This page should:
- Be a client component
- Fetch data using TanStack Query from [API endpoint(s)]
- Show a loading state while fetching
- Show an error message if the request fails
- Render [describe the UI]
- Use Tailwind CSS classes for styling
```

### Creating API Client Functions

```
Read architecture.md.
Generate src/lib/api/[module].ts.
Include TypeScript interfaces for all request and response shapes used by the [ModuleName] module.
Include async functions for: [list operations].
Each function should use the shared apiClient Axios instance.
Export all types and functions.
```

### Creating Validation Rules

```
Read architecture.md.
Generate a FluentValidation validator class for [CommandName]Command in the [ModuleName] module.
Validate:
- [list fields and rules]
Register the validator in [Module]Module.cs.
Add the ValidationBehaviour MediatR pipeline behaviour to src/Shared if it does not already exist.
```

### Creating Unit Tests

```
Read architecture.md.
Generate xUnit unit tests for [HandlerName] in the [ModuleName] module.
Use Moq to mock I[AggregateName]Repository and IPublisher.
Test cases:
1. Happy path — [describe expected outcome]
2. [Failure case 1] — [describe expected exception or result]
3. [Failure case 2] — [describe expected exception or result]
Follow the Arrange / Act / Assert pattern.
```

### Creating Integration Tests

```
Read architecture.md.
Generate an integration test class for the [endpoint description] endpoint.
Use WebApplicationFactory<Program> and a real PostgreSQL test database.
Test cases:
1. [Happy path description] — assert HTTP [status code] and [response body field]
2. [Failure case] — assert HTTP [status code]
Seed the database with [describe seed data] before each test.
```

### Refactoring Code

```
Read architecture.md.
Refactor [file path].
Current problem: [describe the issue — e.g. the handler contains business logic that belongs in the aggregate].
Desired outcome: [describe the target state].
Do not change the public interface of any class.
Do not change the database schema.
```

### Debugging Errors

```
Read architecture.md.
I am getting the following error when I run [command or request]:

[paste the full error message and stack trace]

The relevant file is [file path]. Here is the relevant code:

[paste the relevant code section]

Identify the root cause and generate a corrected version of the file.
```

---

## 10. Database Design

### Approach

Each module owns its own PostgreSQL schema. This means:

- Module A's tables live in schema `catalog.*`
- Module B's tables live in schema `inventory.*`
- No foreign keys cross schema boundaries — modules reference each other only by ID values

This keeps the database organised in a way that mirrors the module boundaries in the code. You can also move a module to its own database later with minimal effort if you ever need to scale out.

### EF Core Schema Configuration

Each module's `DbContext` sets its default schema:

```csharp
// src/Modules/Catalog/Infrastructure/Persistence/CatalogDbContext.cs
public class CatalogDbContext : DbContext
{
    public CatalogDbContext(DbContextOptions<CatalogDbContext> options) : base(options) { }

    public DbSet<Product> Products => Set<Product>();
    public DbSet<Category> Categories => Set<Category>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.HasDefaultSchema("catalog");
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(CatalogDbContext).Assembly);
    }
}
```

### Sample EF Core Entity Configuration

```csharp
// src/Modules/Catalog/Infrastructure/Persistence/Configurations/ProductConfiguration.cs
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.ToTable("products");

        builder.HasKey(p => p.Id);
        builder.Property(p => p.Id)
            .HasConversion(id => id.Value, value => new ProductId(value))
            .HasColumnName("id");

        builder.OwnsOne(p => p.Name, name =>
        {
            name.Property(n => n.Value).HasColumnName("name").HasMaxLength(200).IsRequired();
        });

        builder.OwnsOne(p => p.Sku, sku =>
        {
            sku.Property(s => s.Value).HasColumnName("sku").HasMaxLength(50).IsRequired();
            sku.HasIndex(s => s.Value).IsUnique();
        });

        builder.Property(p => p.Description).HasColumnName("description");
        builder.Property(p => p.IsArchived).HasColumnName("is_archived");
        builder.Property(p => p.CreatedAt).HasColumnName("created_at");

        builder.HasMany(p => p.Images)
            .WithOne()
            .HasForeignKey("product_id")
            .OnDelete(DeleteBehavior.Cascade);
    }
}
```

### All Module Tables

**Catalog schema:**
```sql
catalog.products        (id uuid PK, sku varchar(50) UNIQUE, name varchar(200), description text, category_id uuid, is_archived bool, created_at timestamptz, updated_at timestamptz)
catalog.product_images  (id uuid PK, product_id uuid FK, url text, alt_text varchar(200), sort_order int)
catalog.categories      (id uuid PK, name varchar(100), parent_id uuid nullable FK, created_at timestamptz)
```

**Inventory schema:**
```sql
inventory.stock_items        (id uuid PK, product_id uuid, quantity_on_hand int, quantity_reserved int, updated_at timestamptz)
inventory.stock_reservations (id uuid PK, stock_item_id uuid FK, cart_id uuid, quantity int, expires_at timestamptz, created_at timestamptz)
inventory.stock_adjustments  (id uuid PK, stock_item_id uuid FK, delta int, reason text, adjusted_by uuid, created_at timestamptz)
```

**Pricing schema:**
```sql
pricing.product_prices     (id uuid PK, product_id uuid UNIQUE, amount numeric(18,2), currency char(3), updated_at timestamptz)
pricing.discounts          (id uuid PK, type varchar(20), value numeric(18,2), target_type varchar(20), target_id uuid, starts_at timestamptz, ends_at timestamptz, is_active bool)
pricing.coupons            (id uuid PK, code varchar(50) UNIQUE, type varchar(20), value numeric(18,2), usage_limit int, used_count int, expires_at timestamptz, created_at timestamptz)
pricing.coupon_redemptions (id uuid PK, coupon_id uuid FK, customer_id uuid, order_id uuid, redeemed_at timestamptz)
```

**Cart schema:**
```sql
cart.carts      (id uuid PK, customer_id uuid nullable, session_id varchar(100), status varchar(20), created_at timestamptz, updated_at timestamptz, expires_at timestamptz)
cart.cart_items (id uuid PK, cart_id uuid FK, product_id uuid, product_name varchar(200), quantity int, unit_price numeric(18,2), added_at timestamptz)
```

**Ordering schema:**
```sql
ordering.orders                (id uuid PK, customer_id uuid, status varchar(20), subtotal numeric(18,2), discount numeric(18,2), tax numeric(18,2), total numeric(18,2), currency char(3), shipping_address_json jsonb, created_at timestamptz, updated_at timestamptz)
ordering.order_line_items      (id uuid PK, order_id uuid FK, product_id uuid, product_name varchar(200), quantity int, unit_price numeric(18,2))
ordering.order_status_history  (id uuid PK, order_id uuid FK, from_status varchar(20), to_status varchar(20), changed_at timestamptz, reason text)
```

**Payment schema:**
```sql
payment.payments (id uuid PK, order_id uuid UNIQUE, amount numeric(18,2), currency char(3), status varchar(20), gateway_reference varchar(200), gateway_response_json jsonb, created_at timestamptz, updated_at timestamptz)
payment.refunds  (id uuid PK, payment_id uuid FK, amount numeric(18,2), reason text, status varchar(20), gateway_refund_reference varchar(200), created_at timestamptz, updated_at timestamptz)
```

**Shipping schema:**
```sql
shipping.shipments       (id uuid PK, order_id uuid UNIQUE, tracking_number varchar(100), carrier varchar(100), shipping_method varchar(100), address_json jsonb, created_at timestamptz)
shipping.tracking_events (id uuid PK, shipment_id uuid FK, description text, location varchar(200), occurred_at timestamptz)
```

**Customer schema:**
```sql
customer.customers   (id uuid PK, user_id uuid UNIQUE, display_name varchar(100), email varchar(200), phone varchar(30), created_at timestamptz, updated_at timestamptz)
customer.addresses   (id uuid PK, customer_id uuid FK, label varchar(50), street varchar(200), city varchar(100), state varchar(100), postcode varchar(20), country char(2), is_default bool)
customer.preferences (id uuid PK, customer_id uuid FK UNIQUE, email_notifications bool, preferred_currency char(3))
```

**Identity schema:**
```sql
identity.users          (id uuid PK, email varchar(200) UNIQUE, password_hash text, is_active bool, created_at timestamptz, updated_at timestamptz)
identity.user_roles     (user_id uuid FK, role varchar(50), PRIMARY KEY (user_id, role))
identity.refresh_tokens (id uuid PK, user_id uuid FK, token text UNIQUE, expires_at timestamptz, is_revoked bool, created_at timestamptz)
```

**Notification schema:**
```sql
notification.sent_notifications (id uuid PK, recipient_email varchar(200), template_name varchar(100), sent_at timestamptz, status varchar(20), payload_json jsonb)
```

### Running Migrations

Run each module's migrations separately:

```bash
# Catalog
dotnet ef migrations add InitialSchema \
  --project src/Modules/Catalog \
  --startup-project src/Api \
  --context CatalogDbContext \
  --output-dir Infrastructure/Migrations

# Inventory
dotnet ef migrations add InitialSchema \
  --project src/Modules/Inventory \
  --startup-project src/Api \
  --context InventoryDbContext \
  --output-dir Infrastructure/Migrations

# ... repeat for each module
```

Apply all migrations at startup:

```csharp
// src/Api/Program.cs  (after building the app)
using (var scope = app.Services.CreateScope())
{
    scope.ServiceProvider.GetRequiredService<CatalogDbContext>().Database.Migrate();
    scope.ServiceProvider.GetRequiredService<InventoryDbContext>().Database.Migrate();
    // ... etc
}
```

---

## 11. API Design

### Catalog

| Method | Path | Auth | Request Body | Response |
|---|---|---|---|---|
| GET | `/api/v1/products` | None | — | `PagedResult<ProductDto>` |
| GET | `/api/v1/products/{id}` | None | — | `ProductDto` |
| GET | `/api/v1/products/search?q=boots` | None | — | `PagedResult<ProductDto>` |
| POST | `/api/v1/products` | Admin | `CreateProductRequest` | `{ id }` |
| PUT | `/api/v1/products/{id}` | Admin | `UpdateProductRequest` | `204` |
| DELETE | `/api/v1/products/{id}` | Admin | — | `204` |
| GET | `/api/v1/categories` | None | — | `List<CategoryDto>` |
| POST | `/api/v1/categories` | Admin | `CreateCategoryRequest` | `{ id }` |

**Sample GET /api/v1/products response:**
```json
{
  "items": [
    {
      "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "name": "Running Shoes",
      "description": "Lightweight trail running shoes.",
      "sku": "SHOE-001",
      "categoryId": "abc12345-...",
      "images": [
        { "url": "https://cdn.example.com/shoes.jpg", "altText": "Running Shoes" }
      ]
    }
  ],
  "totalCount": 42,
  "page": 1,
  "pageSize": 20
}
```

**Sample POST /api/v1/products request:**
```json
{
  "name": "Running Shoes",
  "description": "Lightweight trail running shoes.",
  "sku": "SHOE-001",
  "categoryId": "abc12345-...",
  "price": 89.99
}
```

**Validation rules for CreateProductRequest:**
- `name`: required, 3–200 characters
- `description`: optional, max 2000 characters
- `sku`: required, 4–50 characters, alphanumeric and hyphens only
- `categoryId`: required, must be a valid UUID
- `price`: required, greater than or equal to 0

**Sample error response (validation failure):**
```json
{
  "success": false,
  "errors": [
    { "field": "sku", "message": "SKU must be between 4 and 50 characters." },
    { "field": "price", "message": "Price must be a non-negative value." }
  ]
}
```

### Authentication

| Method | Path | Auth | Request Body | Response |
|---|---|---|---|---|
| POST | `/api/v1/auth/register` | None | `RegisterRequest` | `{ userId }` |
| POST | `/api/v1/auth/login` | None | `LoginRequest` | `{ token, refreshToken }` |
| POST | `/api/v1/auth/refresh` | None | `RefreshRequest` | `{ token }` |

**Sample POST /api/v1/auth/login request:**
```json
{
  "email": "customer@example.com",
  "password": "SecurePassword123!"
}
```

**Sample response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "d7c3e2f1...",
  "expiresIn": 3600
}
```

### Cart

| Method | Path | Auth | Request | Response |
|---|---|---|---|---|
| GET | `/api/v1/cart` | Optional | — | `CartDto` |
| POST | `/api/v1/cart/items` | Optional | `{ productId, quantity }` | `204` |
| PUT | `/api/v1/cart/items/{productId}` | Optional | `{ quantity }` | `204` |
| DELETE | `/api/v1/cart/items/{productId}` | Optional | — | `204` |

### Orders

| Method | Path | Auth | Request | Response |
|---|---|---|---|---|
| POST | `/api/v1/orders` | Customer | `CreateOrderRequest` | `{ orderId }` |
| GET | `/api/v1/orders` | Customer | — | `PagedResult<OrderSummaryDto>` |
| GET | `/api/v1/orders/{id}` | Customer | — | `OrderDto` |
| POST | `/api/v1/orders/{id}/cancel` | Customer | `{ reason }` | `204` |

**Sample CreateOrderRequest:**
```json
{
  "cartId": "...",
  "shippingAddress": {
    "street": "123 Main St",
    "city": "Springfield",
    "state": "IL",
    "postcode": "62701",
    "country": "US"
  },
  "shippingMethod": "standard",
  "couponCode": "SUMMER10"
}
```

---

## 12. Testing Tutorial

### Unit Tests — Domain Logic

Test your aggregates and value objects without any dependencies:

```csharp
// tests/Unit/Catalog/Domain/ProductTests.cs
public class ProductTests
{
    [Fact]
    public void Create_WithValidInputs_RaisesProductCreatedEvent()
    {
        var product = Product.Create(
            ProductName.Create("Running Shoes"),
            Sku.Create("SHOE-001"),
            "Great shoes",
            new CategoryId(Guid.NewGuid()));

        Assert.Single(product.DomainEvents);
        Assert.IsType<ProductCreatedEvent>(product.DomainEvents[0]);
    }

    [Fact]
    public void Create_WithEmptyName_ThrowsDomainException()
    {
        Assert.Throws<DomainException>(() =>
            ProductName.Create(string.Empty));
    }

    [Fact]
    public void Archive_AlreadyArchivedProduct_DoesNotRaiseAdditionalEvent()
    {
        var product = Product.Create(
            ProductName.Create("Shoes"),
            Sku.Create("SHOE-002"),
            "desc",
            new CategoryId(Guid.NewGuid()));

        product.ClearDomainEvents();
        product.Archive();
        product.Archive(); // second call

        Assert.Single(product.DomainEvents);
    }
}
```

### Unit Tests — Command Handlers

```csharp
// tests/Unit/Catalog/Application/CreateProductCommandHandlerTests.cs
public class CreateProductCommandHandlerTests
{
    private readonly Mock<IProductRepository> _repositoryMock = new();
    private readonly Mock<IPublisher> _publisherMock = new();

    private CreateProductCommandHandler CreateHandler()
        => new(_repositoryMock.Object, _publisherMock.Object);

    [Fact]
    public async Task Handle_ValidCommand_SavesProductAndPublishesEvent()
    {
        var handler = CreateHandler();
        var command = new CreateProductCommand(
            "Running Shoes", "Great shoes", "SHOE-001", Guid.NewGuid(), 89.99m);

        var id = await handler.Handle(command, CancellationToken.None);

        _repositoryMock.Verify(r => r.AddAsync(
            It.Is<Product>(p => p.Sku.Value == "SHOE-001"),
            It.IsAny<CancellationToken>()), Times.Once);

        _publisherMock.Verify(p => p.Publish(
            It.IsAny<ProductCreatedEvent>(),
            It.IsAny<CancellationToken>()), Times.Once);

        Assert.NotEqual(Guid.Empty, id);
    }
}
```

### Unit Tests — Query Handlers

```csharp
// tests/Unit/Catalog/Application/GetProductByIdQueryHandlerTests.cs
public class GetProductByIdQueryHandlerTests
{
    [Fact]
    public async Task Handle_ExistingProduct_ReturnsMappedDto()
    {
        var product = Product.Create(
            ProductName.Create("Shoes"), Sku.Create("SHOE-001"),
            "desc", new CategoryId(Guid.NewGuid()));

        var repoMock = new Mock<IProductRepository>();
        repoMock.Setup(r => r.GetByIdAsync(product.Id, default))
            .ReturnsAsync(product);

        var handler = new GetProductByIdQueryHandler(repoMock.Object);
        var result = await handler.Handle(
            new GetProductByIdQuery(product.Id.Value), default);

        Assert.NotNull(result);
        Assert.Equal("Shoes", result!.Name);
    }

    [Fact]
    public async Task Handle_NonExistentProduct_ReturnsNull()
    {
        var repoMock = new Mock<IProductRepository>();
        repoMock.Setup(r => r.GetByIdAsync(It.IsAny<ProductId>(), default))
            .ReturnsAsync((Product?)null);

        var handler = new GetProductByIdQueryHandler(repoMock.Object);
        var result = await handler.Handle(
            new GetProductByIdQuery(Guid.NewGuid()), default);

        Assert.Null(result);
    }
}
```

### Integration Tests — API Endpoints

```csharp
// tests/Integration/Catalog/ProductsEndpointTests.cs
public class ProductsEndpointTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public ProductsEndpointTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Replace the real DbContext with a test database
                var descriptor = services.Single(
                    d => d.ServiceType == typeof(DbContextOptions<CatalogDbContext>));
                services.Remove(descriptor);
                services.AddDbContext<CatalogDbContext>(options =>
                    options.UseNpgsql("Host=localhost;Database=ecommerce_test;Username=postgres;Password=postgres"));
            });
        }).CreateClient();
    }

    [Fact]
    public async Task GET_Products_ReturnsOkWithItems()
    {
        var response = await _client.GetAsync("/api/v1/products");

        response.EnsureSuccessStatusCode();
        var body = await response.Content.ReadFromJsonAsync<PagedResult<ProductDto>>();
        Assert.NotNull(body);
    }

    [Fact]
    public async Task POST_Product_WithValidBody_Returns201()
    {
        _client.DefaultRequestHeaders.Authorization =
            new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", GetAdminToken());

        var response = await _client.PostAsJsonAsync("/api/v1/products", new
        {
            name = "Test Product",
            description = "Test",
            sku = "TEST-001",
            categoryId = Guid.NewGuid(),
            price = 10.00
        });

        Assert.Equal(System.Net.HttpStatusCode.Created, response.StatusCode);
    }

    private static string GetAdminToken() => "test-admin-token"; // use a real token generator in production tests
}
```

### Frontend Component Tests

```typescript
// frontend/src/components/catalog/__tests__/ProductCard.test.tsx
import { render, screen } from "@testing-library/react";
import ProductCard from "../ProductCard";

const mockProduct = {
  id: "1",
  name: "Running Shoes",
  description: "Great shoes",
  sku: "SHOE-001",
  categoryId: "cat-1",
  images: [{ url: "https://example.com/img.jpg", altText: "Shoes" }],
};

describe("ProductCard", () => {
  it("renders the product name", () => {
    render(<ProductCard product={mockProduct} />);
    expect(screen.getByText("Running Shoes")).toBeInTheDocument();
  });

  it("links to the product detail page", () => {
    render(<ProductCard product={mockProduct} />);
    expect(screen.getByRole("link")).toHaveAttribute("href", "/products/1");
  });
});
```

### End-to-End Test Example (Playwright)

```typescript
// e2e/purchase-flow.spec.ts
import { test, expect } from "@playwright/test";

test("complete purchase flow", async ({ page }) => {
  // Browse to a product
  await page.goto("http://localhost:3000");
  await page.getByText("Running Shoes").click();
  await expect(page).toHaveURL(/\/products\//);

  // Add to cart
  await page.getByRole("button", { name: "Add to Cart" }).click();

  // Go to cart
  await page.goto("http://localhost:3000/cart");
  await expect(page.getByText("Running Shoes")).toBeVisible();

  // Proceed to checkout
  await page.getByRole("link", { name: "Proceed to Checkout" }).click();
  await expect(page).toHaveURL("/checkout");

  // Fill in address
  await page.fill('[name="street"]', "123 Main St");
  await page.fill('[name="city"]', "Springfield");
  await page.fill('[name="postcode"]', "62701");

  // Place order
  await page.getByRole("button", { name: "Place Order" }).click();
  await expect(page).toHaveURL(/\/orders\//);
  await expect(page.getByText("Order Confirmed")).toBeVisible();
});
```

---

## 13. Running the Application

### Step 1 — Start the Database

Create a `docker-compose.yml` in the root of the project:

```yaml
version: "3.9"
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: ecommerce_dev
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

```bash
docker compose up -d
```

### Step 2 — Configure Environment Variables

Create `backend/src/Api/appsettings.Development.json`:

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Database=ecommerce_dev;Username=postgres;Password=postgres"
  },
  "Jwt": {
    "Secret": "your-super-secret-key-minimum-32-characters-long",
    "Issuer": "ECommerceStore",
    "Audience": "ECommerceStore",
    "ExpiryMinutes": 60
  },
  "Cors": {
    "AllowedOrigins": ["http://localhost:3000"]
  }
}
```

Create `frontend/.env.local`:

```
NEXT_PUBLIC_API_URL=http://localhost:5000
```

### Step 3 — Run Database Migrations

```bash
cd backend
dotnet ef database update \
  --project src/Modules/Catalog \
  --startup-project src/Api \
  --context CatalogDbContext

# Repeat for each module
```

### Step 4 — Run the Backend

```bash
cd backend
dotnet run --project src/Api
```

The API will be available at `http://localhost:5000`. Swagger UI will be at `http://localhost:5000/swagger`.

### Step 5 — Run the Frontend

```bash
cd frontend
npm run dev
```

The frontend will be available at `http://localhost:3000`.

### Step 6 — Seed Sample Data

Create a seed script or use the Superpowers prompt below to generate one:

```
Read architecture.md. Generate a C# data seeder class that inserts:
- 3 categories (Electronics, Clothing, Books)
- 5 products spread across the categories
- Initial stock of 100 units for each product
- Prices for each product
Run this seeder inside Program.cs only when the environment is Development and the products table is empty.
```

### Step 7 — Test the Complete Purchase Flow

1. Register an account: `POST /api/v1/auth/register`
2. Login: `POST /api/v1/auth/login` — copy the JWT token
3. Browse products: `GET /api/v1/products`
4. Add to cart: `POST /api/v1/cart/items`
5. View cart: `GET /api/v1/cart`
6. Create order: `POST /api/v1/orders`
7. Simulate payment confirmation: `POST /api/v1/payments/confirm`
8. Check order status: `GET /api/v1/orders/{id}`

---

## 14. Deployment Notes

### Preparing the Next.js Frontend

```bash
cd frontend
npm run build
```

The build output goes to `frontend/.next/`. For static export (if you are not using server-side rendering features):

```bash
npm run build && npm run export
```

Set these environment variables in your hosting platform:

```
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
```

Next.js deployments work on Vercel (zero-config), Netlify, or any Node.js host. For a Docker deployment:

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/package*.json ./
RUN npm ci --production
EXPOSE 3000
CMD ["npm", "start"]
```

### Preparing the .NET Backend

```bash
cd backend
dotnet publish src/Api -c Release -o ./publish
```

Environment variables for production:

```
ConnectionStrings__Default=Host=your-db-host;Database=ecommerce;Username=...;Password=...
Jwt__Secret=your-production-secret-minimum-32-characters
Jwt__Issuer=ECommerceStore
Jwt__Audience=ECommerceStore
ASPNETCORE_ENVIRONMENT=Production
```

For a Docker deployment:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 8080

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ECommerceStore.sln .
COPY src/ src/
RUN dotnet restore
RUN dotnet publish src/Api -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "ECommerceStore.Api.dll"]
```

### Production-Readiness Checklist

- [ ] All JWT secrets are stored in environment variables or a secrets manager (not in config files)
- [ ] HTTPS is enforced on both frontend and backend
- [ ] CORS is restricted to your production frontend domain only
- [ ] Database connection string uses a non-admin user with least-privilege access
- [ ] Migrations run automatically on startup (or via a separate migration job before deployment)
- [ ] Structured logging is configured (e.g. Serilog to a centralised log store)
- [ ] Health check endpoints are in place (`/health`)
- [ ] Rate limiting is applied to authentication endpoints
- [ ] Payment webhook endpoints verify signatures before processing

---

## 15. Common Mistakes and Troubleshooting

### Common Superpowers Usage Mistakes

**Mistake: Writing vague prompts without providing the spec as context.**

Superpowers generates much better code when it can see your architecture spec. Always start prompts with "Read architecture.md." If the generated code does not match your conventions, it is almost always because the spec was not included.

**Fix:** Add "Read architecture.md and [relevant-module-spec].md." to the beginning of every substantial prompt.

---

**Mistake: Asking Superpowers to generate everything in one prompt.**

Generating all ten modules plus the frontend in one go produces inconsistent, unreviewed code that is hard to debug.

**Fix:** Generate one module at a time. Review each one before moving on. Use the review prompt from Section 8 after each module.

---

### Common Modular Monolith Mistakes

**Mistake: One module calling another module's internal classes directly.**

```csharp
// WRONG — Cart module directly using Catalog's DbContext
public class CartService
{
    private readonly CatalogDbContext _catalogContext; // Do not do this
}
```

**Fix:** Use domain events for cross-module side effects. For synchronous cross-module reads (e.g. the Cart needs a product name), expose a thin public query in the Catalog module and call it via MediatR.

---

**Mistake: Creating foreign keys that cross schema boundaries in the database.**

If `cart.cart_items` has a real database foreign key to `catalog.products`, you have created a hidden coupling that will prevent you from ever separating the modules.

**Fix:** Store only the ID value. Validate that the product exists by querying the Catalog module's read model, not by a database constraint.

---

**Mistake: Putting all modules' entity types in a single shared DbContext.**

**Fix:** Each module gets its own DbContext. The API host calls each module's migration separately.

---

### Common CQRS Mistakes

**Mistake: Putting query logic inside command handlers.**

```csharp
// WRONG — a command handler that also returns a rich DTO
public async Task<ProductDto> Handle(CreateProductCommand cmd, CancellationToken ct)
{
    var product = Product.Create(...);
    await _repo.AddAsync(product, ct);
    return ProductDto.From(product); // This is query behaviour
}
```

**Fix:** Commands return only the new entity's ID (or nothing). If the caller needs the full object, they follow up with a query.

---

**Mistake: Adding business logic to query handlers.**

Query handlers should only read and map data. All business rules live in the domain model.

---

**Mistake: Using the same model for commands and queries.**

Sharing a domain aggregate as both the write model and the read model leads to complex mapping code and anemic aggregates.

**Fix:** Commands work on domain aggregates. Queries return lightweight DTOs built directly from the database.

---

### Common Frontend/Backend Integration Mistakes

**Mistake: Hardcoding the API base URL.**

```typescript
// WRONG
const { data } = await axios.get("http://localhost:5000/api/v1/products");
```

**Fix:** Always use `process.env.NEXT_PUBLIC_API_URL`.

---

**Mistake: Not handling HTTP errors in API client functions.**

If the backend returns a 422 with validation errors and the frontend does not read the error body, the user sees a generic failure message with no guidance.

**Fix:** In the Axios response interceptor, extract the error body and surface the field-level messages to the form via `react-hook-form`'s `setError`.

---

**Mistake: Not invalidating TanStack Query cache after mutations.**

After adding an item to the cart, the cart page still shows the old data because the query cache was not refreshed.

**Fix:** Every `useMutation` that changes data should call `queryClient.invalidateQueries` in its `onSuccess` callback.

---

### Debugging Steps

1. Check the browser network tab — is the request being sent? Is the URL correct?
2. Check the backend logs — did the request arrive? Did it fail validation or throw an exception?
3. Run the backend with `dotnet run --verbosity detailed` to see EF Core SQL.
4. Use the Swagger UI at `/swagger` to test the API endpoint in isolation.
5. If a domain event is not being handled, verify that both the event class and the handler are in the same MediatR scan assembly.

---

## 16. Final Review Checklist

### Architecture Checklist

- [ ] Each module is a separate C# class library project
- [ ] No module references another module's project directly
- [ ] Modules communicate only through domain events (MediatR `INotification`)
- [ ] Each module has its own DbContext and its own database schema
- [ ] No cross-schema foreign key constraints in the database
- [ ] The API host project is the only project that references all modules
- [ ] A Shared kernel exists for base classes and shared value objects only

### Module Checklist

For each of the ten modules:

- [ ] Domain aggregates have private setters and factory methods
- [ ] All business rules are enforced inside the aggregate, not in handlers or services
- [ ] Every command that changes state raises at least one domain event
- [ ] Repository interfaces are defined in the Domain layer
- [ ] Repository implementations are in the Infrastructure layer
- [ ] The module exposes a `AddXxxModule` extension method for service registration

### CQRS Checklist

- [ ] Commands are separate classes from queries
- [ ] Commands return only an ID or void; they do not return rich domain objects
- [ ] Query handlers do not modify any state
- [ ] Handlers do not contain business logic (that belongs in the aggregate)
- [ ] FluentValidation validators exist for every command
- [ ] The validation pipeline behaviour is registered in MediatR

### API Checklist

- [ ] All endpoints follow the `/api/v1/[resource]` pattern
- [ ] POST returns `201 Created` with a `Location` header
- [ ] GET returns `404 Not Found` when the resource does not exist
- [ ] Authentication-required endpoints have `[Authorize]` attributes
- [ ] Admin-only endpoints have `[Authorize(Roles = "Admin")]`
- [ ] All responses use the consistent JSON envelope format
- [ ] Validation errors return `422 Unprocessable Entity` with field-level messages

### Frontend Checklist

- [ ] The API base URL comes from an environment variable
- [ ] Every query has loading and error states
- [ ] Every mutation invalidates the relevant query cache
- [ ] Forms use `react-hook-form` with Zod schema validation
- [ ] The auth token is stored securely and sent with every authenticated request
- [ ] The app redirects to `/login` on 401 responses

### Testing Checklist

- [ ] Unit tests exist for all domain aggregates and value objects
- [ ] Unit tests exist for all command handlers
- [ ] Unit tests exist for all query handlers
- [ ] Integration tests exist for at least one happy-path and one error-path per module
- [ ] Frontend component tests cover rendering and basic interaction
- [ ] At least one end-to-end test covers the complete purchase flow
- [ ] All tests pass with `dotnet test` and `npm test`

---

*Built with [Superpowers](https://github.com/obra/superpowers) — spec-driven AI-assisted development.*
