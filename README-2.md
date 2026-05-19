# Building a Full-Stack E-Commerce Store with Superpowers

> A beginner-friendly, step-by-step tutorial for building a complete online store using the [Superpowers agentic skills framework](https://github.com/obra/superpowers), Next.js (App Router), and C# / .NET — guided from idea to production by an AI coding agent.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [What Is Superpowers?](#2-what-is-superpowers)
3. [Prerequisites](#3-prerequisites)
4. [Architecture Overview](#4-architecture-overview)
   - [Modular Monolith](#modular-monolith)
   - [Domain-Driven Design (DDD)](#domain-driven-design-ddd)
   - [CQRS](#cqrs)
   - [The Four Architecture Rules](#the-four-architecture-rules)
5. [Project Layout at a Glance](#5-project-layout-at-a-glance)
6. [The Nine Bounded Contexts (Modules)](#6-the-nine-bounded-contexts-modules)
7. [Suggested Build Order and Reasoning](#7-suggested-build-order-and-reasoning)
8. [The Superpowers Workflow — Eight Stages](#8-the-superpowers-workflow--eight-stages)
   - [Stage 1 — Installation](#stage-1--installation)
   - [Stage 2 — Brainstorming](#stage-2--brainstorming)
   - [Stage 3 — Git Worktrees](#stage-3--git-worktrees)
   - [Stage 4 — Writing Plans](#stage-4--writing-plans)
   - [Stage 5 — Subagent-Driven Development](#stage-5--subagent-driven-development)
   - [Stage 6 — Test-Driven Development](#stage-6--test-driven-development)
   - [Stage 7 — Code Review](#stage-7--code-review)
   - [Stage 8 — Finishing a Branch](#stage-8--finishing-a-branch)
9. [Module-by-Module Build Guide](#9-module-by-module-build-guide)
   - [Module 1 — Identity & Access](#module-1--identity--access)
   - [Module 2 — Catalog](#module-2--catalog)
   - [Module 3 — Inventory](#module-3--inventory)
   - [Module 4 — Customers](#module-4--customers)
   - [Module 5 — Cart](#module-5--cart)
   - [Module 6 — Ordering](#module-6--ordering)
   - [Module 7 — Payments](#module-7--payments)
   - [Module 8 — Shipping](#module-8--shipping)
   - [Module 9 — Notifications](#module-9--notifications)
10. [What You've Learned](#10-what-youve-learned)
11. [Next Steps](#11-next-steps)

---

## 1. Introduction

This tutorial walks you through building **ShopNow** — a complete, production-grade e-commerce platform — using an AI coding agent supercharged by the [Superpowers framework](https://github.com/obra/superpowers).

By the end, you will have:

- A **Next.js 14 frontend** (App Router, TypeScript) that lets customers browse products, manage a cart, and place orders.
- A **C# / .NET 8 backend** with nine independently deployable modules, each owning its own data and logic.
- A **repeatable workflow** for using an AI agent to build large systems reliably — without the agent going off-rails or making inconsistent decisions.

**Who is this for?** Developers who know basic Next.js and C# but are new to:
- Domain-Driven Design (DDD)
- CQRS (Command Query Responsibility Segregation)
- The Superpowers agentic skills framework

Every concept is defined the first time it appears. You do not need prior experience with any of them.

---

## 2. What Is Superpowers?

**Superpowers** ([github.com/obra/superpowers](https://github.com/obra/superpowers)) is a **skills framework for AI coding agents** — specifically for agents running inside Claude Code. Think of it as a collection of structured workflows (called *skills*) that the agent follows automatically when you trigger them.

Without Superpowers, an AI agent will often:
- Misunderstand vague requirements.
- Write code before a plan exists.
- Skip tests or reviews.
- Make architecture decisions inconsistently across a large project.

With Superpowers, the agent follows a disciplined pipeline:

```
Brainstorm → Plan → Implement (with subagents) → Test → Review → Merge
```

Each step in that pipeline is a *skill* — a precise set of instructions the agent loads and executes. You trigger skills with slash commands (e.g., `/brainstorm`, `/plan`, `/implement`). The agent handles the rest.

**Key ideas:**

| Term | Plain-English Meaning |
|---|---|
| Skill | A reusable workflow the agent follows for one part of the build (brainstorming, planning, coding, reviewing, etc.) |
| Spec | A written description of what you want to build — produced during brainstorming |
| Plan | A breakdown of the spec into small, ordered, implementable tasks |
| Subagent | A second AI instance the agent spawns to execute one task in isolation |
| Worktree | An isolated Git checkout where one module is developed without affecting other modules |

---

## 3. Prerequisites

Before starting, make sure you have the following installed:

| Tool | Version | Purpose |
|---|---|---|
| [Node.js](https://nodejs.org) | 20+ | Run Next.js and npm |
| [.NET SDK](https://dotnet.microsoft.com/download) | 8.0+ | Build the C# backend |
| [Git](https://git-scm.com) | 2.35+ | Version control and worktrees |
| [Claude Code](https://claude.ai/code) | Latest | The AI coding agent CLI |
| [Docker Desktop](https://www.docker.com/products/docker-desktop/) | Latest | Run databases locally |
| A code editor | VS Code recommended | Edit code and review agent output |

Verify your installs:

```bash
node --version       # should print v20.x or higher
dotnet --version     # should print 8.x.x
git --version        # should print 2.35 or higher
claude --version     # should print a version number
docker --version     # should print 24.x or higher
```

---

## 4. Architecture Overview

Before writing a single line of code, you need to understand the three architectural ideas that shape the entire project. The agent will use these as constraints whenever it makes design decisions — so understanding them helps you catch mistakes early.

### Modular Monolith

A **monolith** is a single deployable application that contains all the code. A **microservices** architecture splits the code into many small services that communicate over a network.

A **modular monolith** sits in between: it is a single deployable application, but the code is internally divided into strict, self-contained modules that behave *as if* they were microservices. Each module:

- Owns its own database tables (or schema).
- Has its own public API surface (interfaces, events).
- Cannot directly call another module's internal code.

This gives you the organizational benefits of microservices without the operational complexity of running and coordinating dozens of networked services.

**Why a modular monolith for this tutorial?** Because it is easier to start, easier to test, and can be split into actual microservices later if needed — without rewriting the domain logic.

### Domain-Driven Design (DDD)

**Domain-Driven Design** is a way of structuring software so that the code closely mirrors the real-world business it models. It has two levels:

**Strategic DDD** (the big picture):
- You divide the business into **Bounded Contexts** — areas with clear, non-overlapping responsibilities. In this project, each Bounded Context becomes one module (e.g., `Catalog`, `Inventory`, `Cart`).
- A **Ubiquitous Language** is established for each context: the words used in code match the words used by the business. "Product" means the same thing to a developer and to a product manager inside the Catalog context.

**Tactical DDD** (inside each module):
- **Aggregate** — a cluster of objects treated as a single unit for data changes. An `Order` aggregate, for example, contains `OrderLine` entities. You always change an aggregate as a whole and save it as a whole.
- **Entity** — an object with a unique identity that persists over time. A `Customer` is an entity (it has an ID). Two customers are different even if they have the same name.
- **Value Object** — an object defined entirely by its values, with no identity. A `Money` value object (`{ amount: 19.99, currency: "USD" }`) is equal to any other `Money` with the same values.
- **Domain Event** — a record that something important happened in the domain (e.g., `OrderPlaced`, `PaymentSucceeded`). Other modules listen to these events to react without being tightly coupled.
- **Repository** — an abstraction over the database for loading and saving aggregates.
- **Domain Service** — a service that handles business logic that doesn't naturally belong to a single aggregate.

### CQRS

**CQRS** stands for *Command Query Responsibility Segregation*. It means you separate every operation into one of two types:

- **Command** — an operation that *changes* state (e.g., `PlaceOrder`, `AddItemToCart`). Commands do not return data (or return only an ID/acknowledgement). They represent *intent*: "I want to do X."
- **Query** — an operation that *reads* state (e.g., `GetProductById`, `ListOrdersByCustomer`). Queries never change data.

Each command and each query gets its own handler class. This makes the codebase easy to navigate: to understand how an order is placed, you open `PlaceOrderCommandHandler.cs`. To understand how orders are fetched, you open `GetOrdersQueryHandler.cs`.

### The Four Architecture Rules

These rules apply to the entire project. The agent must follow them; do not let it break them:

> **Rule 1 — Bounded Contexts as modules.** The system is divided into modules using DDD strategic design. Each module corresponds to exactly one Bounded Context. Modules are: Catalog, Inventory, Cart, Ordering, Payments, Customers, Identity & Access, Shipping, and Notifications.

> **Rule 2 — Tactical DDD inside modules.** The internals of each module are built using DDD tactical patterns: Aggregates, Entities, Value Objects, Domain Events, Repositories, and Domain Services.

> **Rule 3 — Contracts only, no internal access.** Modules communicate only through well-defined contracts: public interfaces, integration events, and shared DTOs. No module may reach into another module's internal classes, domain objects, or database tables.

> **Rule 4 — Module data ownership.** Each module owns its own data and exposes its own commands and queries. If Module A needs data owned by Module B, it must ask through Module B's public query API — it cannot query Module B's tables directly.

---

## 5. Project Layout at a Glance

Here is the high-level folder structure you will build. The agent will create these directories as it works through each module.

### Frontend — `ecommerce-frontend/`

```
ecommerce-frontend/
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── (shop)/                   # Public-facing shop pages
│   │   │   ├── page.tsx              # Home / featured products
│   │   │   ├── products/
│   │   │   │   ├── page.tsx          # Product listing
│   │   │   │   └── [slug]/
│   │   │   │       └── page.tsx      # Product detail
│   │   │   ├── cart/
│   │   │   │   └── page.tsx
│   │   │   ├── checkout/
│   │   │   │   └── page.tsx
│   │   │   └── orders/
│   │   │       ├── page.tsx          # Order history
│   │   │       └── [orderId]/
│   │   │           └── page.tsx      # Order detail
│   │   ├── (auth)/                   # Auth-related pages
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── register/
│   │   │       └── page.tsx
│   │   ├── account/                  # Customer account pages
│   │   │   └── page.tsx
│   │   └── api/                      # Next.js API routes (BFF layer)
│   │       ├── auth/
│   │       └── [...proxy]/
│   ├── modules/                      # Frontend module boundaries
│   │   ├── catalog/
│   │   │   ├── components/           # ProductCard, CategoryNav, etc.
│   │   │   ├── hooks/                # useProducts, useProductDetail
│   │   │   ├── services/             # catalogApi.ts (HTTP client)
│   │   │   └── types.ts              # Product, Category types
│   │   ├── cart/
│   │   │   ├── components/           # CartDrawer, CartItem, etc.
│   │   │   ├── hooks/                # useCart
│   │   │   ├── store/                # Zustand cart store
│   │   │   └── types.ts
│   │   ├── checkout/
│   │   ├── ordering/
│   │   ├── identity/
│   │   └── customer/
│   └── shared/
│       ├── components/               # Button, Input, Modal, etc.
│       ├── hooks/                    # useAuth, useFetch
│       └── types/                    # Common types (Money, Address, etc.)
├── public/
├── next.config.ts
├── tsconfig.json
└── package.json
```

### Backend — `EcommercePlatform/`

```
EcommercePlatform/
├── EcommercePlatform.sln
├── src/
│   ├── EcommercePlatform.Api/               # ASP.NET Core entry point
│   │   ├── Program.cs
│   │   ├── appsettings.json
│   │   └── appsettings.Development.json
│   ├── Modules/
│   │   ├── Catalog/
│   │   │   ├── Catalog.Domain/
│   │   │   │   ├── Aggregates/
│   │   │   │   │   └── Product.cs
│   │   │   │   ├── Entities/
│   │   │   │   │   └── Category.cs
│   │   │   │   ├── ValueObjects/
│   │   │   │   │   ├── Money.cs
│   │   │   │   │   └── ProductSlug.cs
│   │   │   │   ├── Events/
│   │   │   │   │   └── ProductPublishedEvent.cs
│   │   │   │   └── Repositories/
│   │   │   │       └── IProductRepository.cs
│   │   │   ├── Catalog.Application/
│   │   │   │   ├── Commands/
│   │   │   │   │   ├── CreateProduct/
│   │   │   │   │   │   ├── CreateProductCommand.cs
│   │   │   │   │   │   └── CreateProductCommandHandler.cs
│   │   │   │   │   └── PublishProduct/
│   │   │   │   ├── Queries/
│   │   │   │   │   ├── GetProductBySlug/
│   │   │   │   │   │   ├── GetProductBySlugQuery.cs
│   │   │   │   │   │   └── GetProductBySlugQueryHandler.cs
│   │   │   │   │   └── SearchProducts/
│   │   │   │   └── Contracts/          # Public DTOs other modules may use
│   │   │   │       └── ProductSummaryDto.cs
│   │   │   └── Catalog.Infrastructure/
│   │   │       ├── Persistence/
│   │   │       │   └── CatalogDbContext.cs
│   │   │       └── Repositories/
│   │   │           └── EfProductRepository.cs
│   │   ├── Inventory/
│   │   │   └── (same three-layer structure)
│   │   ├── Cart/
│   │   ├── Ordering/
│   │   ├── Payments/
│   │   ├── Customers/
│   │   ├── Identity/
│   │   ├── Shipping/
│   │   └── Notifications/
│   └── Shared/
│       ├── Shared.Kernel/             # Shared base classes (Entity, AggregateRoot, etc.)
│       └── Shared.Infrastructure/    # Shared infrastructure (event bus, outbox, etc.)
└── tests/
    └── Modules/
        ├── Catalog.Tests/
        ├── Inventory.Tests/
        └── (one test project per module)
```

Each backend module has **three layers**:

| Layer | What it contains |
|---|---|
| `Domain` | Aggregates, Entities, Value Objects, Domain Events, Repository *interfaces* |
| `Application` | Command handlers, Query handlers, public Contracts (DTOs) |
| `Infrastructure` | EF Core DbContext, Repository *implementations*, external service adapters |

---

## 6. The Nine Bounded Contexts (Modules)

Below is the full definition of every module. Read this section carefully before starting — it gives you and the agent a shared understanding of the entire system.

---

### Catalog

**Responsibility:** Owns the product catalog. Knows what is for sale, what it looks like, and how it is organized.

**Features:**
1. **Create and manage products** — store name, description, slug (URL-friendly identifier), images, attributes (size, color, etc.), and base price.
2. **Product categories** — hierarchical categories (e.g., Electronics → Phones → Smartphones). A product belongs to one or more categories.
3. **Publish / unpublish products** — a product is either in "Draft" or "Published" state. Only published products are visible to customers.
4. **Product search** — full-text search by name and description; filter by category, price range, and attributes.
5. **Product detail page data** — a query that returns everything the frontend needs for a single product detail page.
6. **Featured products** — the ability to flag products as "featured" for homepage display.

---

### Inventory

**Responsibility:** Tracks how many units of each product are physically available. It does not know about prices, descriptions, or customers — just stock numbers.

**Features:**
1. **Stock levels** — track quantity on hand for each product variant (identified by SKU).
2. **Stock reservations** — temporarily reserve units when a customer adds an item to their cart (so two customers cannot both think the last item is available).
3. **Reservation expiry** — automatically release reservations that are not converted to orders within a configurable timeout (e.g., 30 minutes).
4. **Availability check** — a query that answers: "Is SKU X available in quantity Y?" — used by the Cart and Ordering modules.
5. **Restock events** — emit a `StockReplenished` domain event when stock is added, so Notifications can alert waiting customers.
6. **Low-stock threshold** — flag a SKU as "low stock" when it falls below a configurable threshold.

---

### Cart

**Responsibility:** Manages a customer's in-progress shopping session before checkout. Owns everything about what the customer intends to buy, at what price.

**Features:**
1. **Add items to cart** — add a product variant (SKU) with a quantity. The cart checks availability with Inventory before confirming.
2. **Remove items / change quantities** — update or remove individual line items.
3. **Cart persistence** — a guest cart is stored in a browser session; a logged-in customer's cart is persisted server-side and is accessible across devices.
4. **Price snapshot** — when an item is added, the cart records the current price from the Catalog. This price is locked in for the duration of the session (so a price change mid-session does not surprise the customer).
5. **Cart summary** — a query returning line items, per-item prices, subtotal, and estimated taxes.
6. **Merge carts** — when a guest logs in and already has a server-side cart, merge the two carts intelligently (sum quantities, keep the lower of conflicting prices).

---

### Ordering

**Responsibility:** Converts a cart into a formal order and manages the order through its entire lifecycle.

**Features:**
1. **Checkout / place order** — validate the cart, confirm inventory with the Inventory module, record the order, and emit an `OrderPlaced` event.
2. **Order lifecycle states** — `Pending` → `Confirmed` → `Processing` → `Shipped` → `Delivered` / `Cancelled` / `Refunded`.
3. **Order line items** — each order records exactly what was ordered, at what price, in what quantity (an immutable snapshot).
4. **Order cancellation** — a customer or admin can cancel an order before it enters "Processing"; triggers a `OrderCancelled` event for Inventory to release reservations.
5. **Order history** — query all orders for a given customer, sorted by date, with pagination.
6. **Order detail** — query returning the full order, its status, its line items, shipping address, and payment summary.

---

### Payments

**Responsibility:** Handles all money movement. Knows nothing about products or customers — just payment intents, charges, and refunds.

**Features:**
1. **Create payment intent** — when an order is placed, create a payment intent (integrates with Stripe or a stub in development).
2. **Confirm payment** — handle the payment provider's callback (webhook) confirming a charge succeeded; emit `PaymentSucceeded` event.
3. **Payment failure handling** — record failed payment attempts; emit `PaymentFailed` event so Ordering can react.
4. **Refunds** — initiate a full or partial refund for a payment; emit `RefundIssued` event.
5. **Payment status query** — return the current status of a payment for a given order.
6. **Payment method storage** — optionally store tokenized payment methods per customer (never store raw card data).

---

### Customers

**Responsibility:** Owns the customer's profile and address book. Distinct from Identity (which handles login credentials) — a customer can exist without an identity (e.g., guest checkouts).

**Features:**
1. **Customer registration** — create a customer profile (name, email, phone) when a user signs up.
2. **Profile management** — update name, email, and phone number.
3. **Address book** — add, edit, and delete shipping/billing addresses per customer. Mark one address as default.
4. **Guest customers** — create a minimal customer record for guests at checkout (no password required).
5. **Customer profile query** — return a customer's full profile with their address book.
6. **Link customer to identity** — when a guest account is later registered, link the guest customer record to the new identity.

---

### Identity & Access

**Responsibility:** Handles who can log in and what they are allowed to do. Does not know about products, orders, or customer profiles — just usernames, passwords, tokens, and roles.

**Features:**
1. **User registration** — create a user account with email and password; hash passwords securely (bcrypt or Argon2).
2. **Login / session management** — authenticate with email+password; issue a JWT access token and a refresh token.
3. **Token refresh** — exchange a valid refresh token for a new access token (rotate refresh tokens on use).
4. **Logout** — revoke the refresh token.
5. **Role-based access** — roles include `Customer`, `Admin`, and `StoreManager`. Enforce roles on API endpoints with authorization middleware.
6. **Password reset** — initiate a password-reset flow (token emailed to the user); verify token and set a new password.

---

### Shipping

**Responsibility:** Knows about delivery options and tracks shipments after an order is placed.

**Features:**
1. **Delivery options** — given a shipping address and a list of items, return available delivery methods with estimated delivery dates and costs (integrate with a carrier API or use a configurable rate table in development).
2. **Create shipment** — when an order enters "Processing", create a shipment record linked to the order.
3. **Shipment tracking** — store and expose carrier tracking numbers; poll or receive webhooks to update tracking status.
4. **Shipment status** — query the current status of a shipment for a given order.
5. **Shipping address validation** — validate that a given address is deliverable before an order is placed.
6. **Emit shipping events** — emit `ShipmentDispatched` and `ShipmentDelivered` events for Notifications and Ordering to consume.

---

### Notifications

**Responsibility:** Sends messages to customers (and internal staff) when notable events occur. Does not own any business logic — it only listens to events from other modules and sends the appropriate message.

**Features:**
1. **Order confirmation email** — send a formatted email when `OrderPlaced` is received.
2. **Payment confirmation email** — send a receipt when `PaymentSucceeded` is received.
3. **Shipment dispatched email/SMS** — notify the customer with a tracking number when `ShipmentDispatched` is received.
4. **Order cancellation email** — send a cancellation notice when `OrderCancelled` is received.
5. **Password reset email** — send the password-reset link when the Identity module requests it.
6. **Low-stock alert (internal)** — send an email to store admins when `LowStockThresholdReached` is received from Inventory.
7. **Notification preferences** — respect customer opt-out preferences for marketing vs. transactional messages.

---

## 7. Suggested Build Order and Reasoning

Build modules in this order:

```
Identity & Access  →  Catalog  →  Inventory  →  Customers
      →  Cart  →  Ordering  →  Payments  →  Shipping  →  Notifications
```

**Why this order?**

| Module | Why it comes here |
|---|---|
| **Identity & Access** | Every other module's API is protected by auth. Build auth first so you can immediately add security to each subsequent module, rather than retrofitting it later. |
| **Catalog** | Products are the center of the system. Inventory, Cart, and Ordering all reference product data. Build the catalog before anything that needs to look up a product. |
| **Inventory** | Cart and Ordering both need stock checks. Build inventory before cart so you can wire up the availability check as soon as the cart is built. |
| **Customers** | Ordering and Cart both reference a customer record. Building Customers before Cart ensures the data model is ready. |
| **Cart** | Depends on Catalog (for prices) and Inventory (for availability). Now that both exist, the cart can be built with real integrations. |
| **Ordering** | Depends on Cart (for line items), Inventory (to confirm reservations), and Customers (for addresses). |
| **Payments** | Depends on Ordering (payment is triggered by an order). The integration is simpler once the order model is defined. |
| **Shipping** | Depends on Ordering (a shipment is linked to an order). Can be developed in parallel with Payments once Ordering is stable. |
| **Notifications** | Purely reactive — it only listens to events. All the events it needs come from earlier modules. Build it last so the event contracts are already defined. |

---

## 8. The Superpowers Workflow — Eight Stages

This section teaches you the Superpowers workflow in detail. For each stage you will see:
- **What the stage does** — a plain-English explanation.
- **How to trigger it** — the slash command or prompt you type.
- **What the agent does** — what happens automatically.
- **A worked example** — applied to this e-commerce project.

---

### Stage 1 — Installation

**What it does:** Installs the Superpowers plugin into Claude Code so all the skills become available.

**How to install:**

**Option A — Claude Code marketplace (recommended):**

```bash
# Open a terminal and run:
claude mcp add superpowers --from https://github.com/obra/superpowers
```

**Option B — Manual clone and link:**

```bash
git clone https://github.com/obra/superpowers.git ~/.claude/plugins/superpowers
claude mcp add superpowers --local ~/.claude/plugins/superpowers
```

**Verify the install:**

```bash
claude
# Inside the Claude Code REPL, type:
/help
# You should see Superpowers skills listed (brainstorm, plan, implement, review, etc.)
```

**Start your project:**

```bash
mkdir shopnow && cd shopnow
git init
claude
```

You are now inside the Claude Code agent with Superpowers active.

---

### Stage 2 — Brainstorming

**What it does:** Turns a rough idea into a precise, actionable specification. The agent uses **Socratic questioning** — it asks you clarifying questions one at a time, then synthesizes your answers into a written spec.

> **Why this matters:** An agent given vague instructions produces vague code. Brainstorming forces ambiguity into the open *before* any code is written.

**How to trigger it:**

```
/brainstorm
```

After triggering, describe your project in a few sentences. The agent will ask follow-up questions like:
- "Who are the end users — consumers, businesses, or both?"
- "Do you need multi-vendor (marketplace) or single-vendor?"
- "Should guest checkout be supported?"
- "Which payment providers do you want to integrate?"

**What the agent produces:** A `SPEC.md` file — a written description of the system to build. This spec becomes the source of truth for all future planning.

**Worked Example — What to type:**

```
/brainstorm

I want to build a full-stack e-commerce store. The frontend should use
Next.js 14 with the App Router and TypeScript. The backend should be
C# / .NET 8. I want to follow Domain-Driven Design with a modular monolith
architecture, where each module is a Bounded Context. I also want to use
CQRS inside each module. The system needs nine modules: Catalog, Inventory,
Cart, Ordering, Payments, Customers, Identity & Access, Shipping, and
Notifications.
```

**Expect the agent to ask:**

```
Clarifying question 1: Should the store support multiple sellers
(marketplace model) or a single seller?
```

```
Your answer: Single seller for now.
```

```
Clarifying question 2: For the Cart module, should carts persist
for logged-out guests, and if so, for how long?
```

```
Your answer: Yes. Guest carts should persist for 30 days using
a browser cookie with an anonymous cart ID.
```

After 5–10 questions, the agent writes `SPEC.md` to your project root. Review it carefully — this document drives everything downstream. If something is wrong, say:

```
The spec says X, but I actually want Y. Please update the spec.
```

Do not proceed to planning until the spec accurately captures your intent.

---

### Stage 3 — Git Worktrees

**What it does:** Creates an isolated Git workspace for each module, so you can develop and test one module without risking changes to modules that are already working.

> **What is a Git worktree?** Normally, a Git repository has one working directory. `git worktree` lets you check out multiple branches simultaneously, each in its own folder. You can work on `feature/catalog` in `../shopnow-catalog/` while `main` stays clean in `shopnow/`.

**How to trigger it:**

```
/worktree create catalog
```

The Superpowers agent runs:

```bash
git worktree add ../shopnow-catalog feature/catalog
```

This creates:
- A new folder `../shopnow-catalog/` checked out to branch `feature/catalog`.
- An isolated environment where you develop the Catalog module.
- A clean `main` branch that accumulates finished, reviewed modules.

**Worked Example — Setting up worktrees for all modules:**

```
/worktree create identity
/worktree create catalog
/worktree create inventory
/worktree create customers
/worktree create cart
/worktree create ordering
/worktree create payments
/worktree create shipping
/worktree create notifications
```

> **Tip:** You do not create all worktrees at the start. Create each worktree as you begin that module — follow the build order from Section 7.

**Switching between worktrees:**

```
/worktree switch identity
# The agent cd's to the identity worktree and begins working there
```

---

### Stage 4 — Writing Plans

**What it does:** Takes the spec for one module and breaks it down into a list of small, concrete, independently implementable tasks. The agent outputs a `PLAN.md` file in the current worktree.

> **Why small tasks?** Large tasks confuse AI agents. A task like "build the Catalog module" is too big — the agent will make architectural decisions without consulting you. A task like "Create the `Product` aggregate with `ProductId`, `Name`, `Description`, `Slug`, `BasePrice`, and `Status` properties" is specific enough that the agent can implement it confidently and you can verify it quickly.

**How to trigger it:**

```
/plan
```

The agent reads `SPEC.md` and the four architecture rules, then writes a `PLAN.md` for the current module. Each task in the plan should be:
- **Atomic** — one clear thing to build or test.
- **Ordered** — listed in the sequence they must be built (dependencies first).
- **Verifiable** — you can tell by inspection whether it is done.

**Worked Example — Planning the Catalog module:**

```
/worktree switch catalog
/plan

Use the spec and the architecture rules to create a detailed plan for
the Catalog bounded context. The module should have Domain, Application,
and Infrastructure layers. Tasks should be ordered: domain models first,
then repository interfaces, then command handlers, then query handlers,
then infrastructure, then API endpoints, then tests.
```

**Example `PLAN.md` output (abbreviated):**

```markdown
# Catalog Module Plan

## Phase 1: Domain Layer

- [ ] Task 1: Create `Catalog.Domain` project.
- [ ] Task 2: Create `ProductId` value object (wraps a `Guid`).
- [ ] Task 3: Create `Money` value object with `Amount` (decimal) and `Currency` (string, ISO 4217).
- [ ] Task 4: Create `ProductSlug` value object (lowercase alphanumeric with hyphens).
- [ ] Task 5: Create `ProductStatus` enum: `Draft`, `Published`, `Archived`.
- [ ] Task 6: Create `Category` entity with `CategoryId`, `Name`, `ParentCategoryId?`.
- [ ] Task 7: Create `Product` aggregate root with: `ProductId`, `Name`, `Description`,
             `Slug` (ProductSlug), `BasePrice` (Money), `Status` (ProductStatus),
             `CategoryIds` (list), `Images` (list of URLs), `IsFeatured` (bool).
- [ ] Task 8: Add `Publish()` method to `Product` that transitions status to Published
             and raises a `ProductPublishedEvent`.
- [ ] Task 9: Create `ProductPublishedEvent` domain event.
- [ ] Task 10: Create `IProductRepository` interface in the Domain layer.

## Phase 2: Application Layer

- [ ] Task 11: Create `Catalog.Application` project.
- [ ] Task 12: Add MediatR dependency for command/query dispatching.
- [ ] Task 13: Create `CreateProductCommand` and `CreateProductCommandHandler`.
- [ ] Task 14: Create `PublishProductCommand` and `PublishProductCommandHandler`.
- [ ] Task 15: Create `GetProductBySlugQuery` and `GetProductBySlugQueryHandler`.
- [ ] Task 16: Create `SearchProductsQuery` and `SearchProductsQueryHandler`.
- [ ] Task 17: Create `GetFeaturedProductsQuery` and handler.
- [ ] Task 18: Create public `ProductSummaryDto` and `ProductDetailDto` in `Catalog.Application.Contracts`.

## Phase 3: Infrastructure Layer

- [ ] Task 19: Create `Catalog.Infrastructure` project.
- [ ] Task 20: Add Entity Framework Core with PostgreSQL provider.
- [ ] Task 21: Create `CatalogDbContext` with `Products` and `Categories` DbSets.
- [ ] Task 22: Configure EF mappings for `Product` aggregate (owned value objects,
              enum storage as string).
- [ ] Task 23: Implement `EfProductRepository` satisfying `IProductRepository`.

## Phase 4: API Endpoints

- [ ] Task 24: Register Catalog module in `EcommercePlatform.Api/Program.cs`.
- [ ] Task 25: Create `CatalogController` with endpoints:
              GET /api/catalog/products (search), GET /api/catalog/products/{slug},
              GET /api/catalog/featured, POST /api/catalog/products (admin),
              PUT /api/catalog/products/{id}/publish (admin).

## Phase 5: Tests

- [ ] Task 26: Create `Catalog.Tests` project.
- [ ] Task 27: Write unit tests for `Product` aggregate methods.
- [ ] Task 28: Write unit tests for each command handler (mock repository).
- [ ] Task 29: Write integration tests for `EfProductRepository` against a real test database.
```

Review this plan before execution. Add, remove, or reword tasks as needed. Only when you are satisfied, move to Stage 5.

---

### Stage 5 — Subagent-Driven Development

**What it does:** Executes the plan, task by task, using subagents. Each task gets its own subagent — an isolated AI instance that focuses on exactly one task, then hands control back for review.

> **What is a subagent?** It is a second Claude instance spawned by the main agent to work on a single, well-defined task. Using subagents provides two benefits:
> 1. **Isolation** — the subagent starts with a clean context, so it cannot "remember" earlier bad decisions and repeat them.
> 2. **Two-stage review** — the subagent implements, the main agent reviews. This catches errors before they compound.

**How to trigger it:**

```
/implement
```

The agent reads `PLAN.md`, finds the first unchecked task, spawns a subagent to implement it, waits for the result, reviews it, and marks the task complete (or asks you to approve) before moving on.

**What a subagent implementation cycle looks like:**

```
[Main agent] → Reading Task 7: Create Product aggregate root
[Main agent] → Spawning subagent for Task 7
[Subagent]   → Reading SPEC.md and PLAN.md context for Task 7
[Subagent]   → Writing src/Modules/Catalog/Catalog.Domain/Aggregates/Product.cs
[Subagent]   → Done. Returning to main agent.
[Main agent] → Reviewing generated Product.cs ...
[Main agent] → ✓ Product.cs looks correct. Marking Task 7 complete.
[Main agent] → Proceeding to Task 8.
```

**Worked Example — Implementing the Product aggregate:**

```
/worktree switch catalog
/implement

Execute the plan in PLAN.md one task at a time. For each task:
1. Spawn a subagent to implement it.
2. Review the output for correctness against the spec and architecture rules.
3. Flag any issues before marking the task done.
4. Pause and ask me to review after every 5 tasks, or any time you are unsure.
```

**Example generated code — `Product.cs`:**

```csharp
// src/Modules/Catalog/Catalog.Domain/Aggregates/Product.cs

namespace EcommercePlatform.Catalog.Domain.Aggregates;

public sealed class Product : AggregateRoot<ProductId>
{
    private readonly List<Guid> _categoryIds = new();
    private readonly List<string> _imageUrls = new();

    public string Name { get; private set; }
    public string Description { get; private set; }
    public ProductSlug Slug { get; private set; }
    public Money BasePrice { get; private set; }
    public ProductStatus Status { get; private set; }
    public bool IsFeatured { get; private set; }
    public IReadOnlyList<Guid> CategoryIds => _categoryIds.AsReadOnly();
    public IReadOnlyList<string> ImageUrls => _imageUrls.AsReadOnly();

    private Product() { } // Required by EF Core

    public static Product Create(
        string name,
        string description,
        string slug,
        decimal basePrice,
        string currency)
    {
        var product = new Product
        {
            Id = ProductId.New(),
            Name = name,
            Description = description,
            Slug = new ProductSlug(slug),
            BasePrice = new Money(basePrice, currency),
            Status = ProductStatus.Draft,
            IsFeatured = false
        };

        product.RaiseDomainEvent(new ProductCreatedEvent(product.Id, product.Name));
        return product;
    }

    public void Publish()
    {
        if (Status == ProductStatus.Published)
            return;

        Status = ProductStatus.Published;
        RaiseDomainEvent(new ProductPublishedEvent(Id, Slug));
    }

    public void AddCategory(Guid categoryId)
    {
        if (!_categoryIds.Contains(categoryId))
            _categoryIds.Add(categoryId);
    }

    public void SetFeatured(bool isFeatured)
    {
        IsFeatured = isFeatured;
    }
}
```

**Example generated code — `Money.cs` (Value Object):**

```csharp
// src/Modules/Catalog/Catalog.Domain/ValueObjects/Money.cs

namespace EcommercePlatform.Catalog.Domain.ValueObjects;

public sealed record Money(decimal Amount, string Currency)
{
    public static Money Zero(string currency) => new(0m, currency);

    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException(
                $"Cannot add {Currency} and {other.Currency}");
        return this with { Amount = Amount + other.Amount };
    }

    public Money Multiply(int quantity) =>
        this with { Amount = Amount * quantity };
}
```

**Example generated code — `CreateProductCommandHandler.cs`:**

```csharp
// src/Modules/Catalog/Catalog.Application/Commands/CreateProduct/CreateProductCommandHandler.cs

namespace EcommercePlatform.Catalog.Application.Commands.CreateProduct;

public sealed class CreateProductCommandHandler
    : IRequestHandler<CreateProductCommand, ProductId>
{
    private readonly IProductRepository _products;

    public CreateProductCommandHandler(IProductRepository products)
        => _products = products;

    public async Task<ProductId> Handle(
        CreateProductCommand command,
        CancellationToken cancellationToken)
    {
        var product = Product.Create(
            command.Name,
            command.Description,
            command.Slug,
            command.BasePrice,
            command.Currency);

        await _products.AddAsync(product, cancellationToken);
        return product.Id;
    }
}
```

> **When the agent gets stuck:** If the agent produces incorrect code or makes an architecture decision you disagree with, say:
> ```
> Stop. Task 13 is wrong — the command handler should not query the
> database for a product that doesn't exist yet. Please re-read Task 13
> in the plan and try again.
> ```
> Superpowers' plan-tracking means the agent knows exactly which task it is on and can retry without losing its place.

---

### Stage 6 — Test-Driven Development

**What it does:** Enforces the **RED → GREEN → REFACTOR** cycle for every feature. This ensures that tests are written *before* implementation, not as an afterthought.

> **The RED-GREEN-REFACTOR cycle:**
> - **RED** — Write a test that describes desired behavior. Run it. It fails (because the feature doesn't exist yet). This proves the test actually tests something.
> - **GREEN** — Write the minimum code to make the test pass. Do not over-engineer.
> - **REFACTOR** — Clean up the code (rename, extract, simplify) while the test continues to pass.

**How to trigger TDD mode:**

```
/tdd Task 27
```

This tells the agent to implement Task 27 using the RED-GREEN-REFACTOR cycle explicitly.

**Worked Example — TDD for the Product aggregate's `Publish()` method:**

```
/tdd Task 27

Write a failing test for the Product.Publish() method first,
confirm it fails, then implement Publish(), confirm the test
passes, then refactor if needed.
```

**RED — The agent writes the failing test first:**

```csharp
// tests/Modules/Catalog.Tests/Domain/ProductTests.cs

public class ProductTests
{
    [Fact]
    public void Publish_WhenProductIsDraft_ShouldChangeStatusToPublished()
    {
        // Arrange
        var product = Product.Create(
            "Test Widget", "A great widget", "test-widget", 9.99m, "USD");

        // Act
        product.Publish();

        // Assert
        product.Status.Should().Be(ProductStatus.Published);
    }

    [Fact]
    public void Publish_WhenProductIsDraft_ShouldRaiseProductPublishedEvent()
    {
        // Arrange
        var product = Product.Create(
            "Test Widget", "A great widget", "test-widget", 9.99m, "USD");

        // Act
        product.Publish();

        // Assert
        product.DomainEvents.Should()
            .ContainSingle(e => e is ProductPublishedEvent);
    }

    [Fact]
    public void Publish_WhenAlreadyPublished_ShouldNotRaiseDuplicateEvent()
    {
        // Arrange
        var product = Product.Create(
            "Test Widget", "A great widget", "test-widget", 9.99m, "USD");
        product.Publish(); // first publish
        product.ClearDomainEvents();

        // Act
        product.Publish(); // second publish — should be a no-op

        // Assert
        product.DomainEvents.Should().BeEmpty();
    }
}
```

**The agent runs the tests:**

```bash
dotnet test tests/Modules/Catalog.Tests/
# Expected: 3 tests FAIL (Product.Publish() not yet implemented)
```

**GREEN — The agent implements `Publish()`:**

```csharp
public void Publish()
{
    if (Status == ProductStatus.Published)
        return;

    Status = ProductStatus.Published;
    RaiseDomainEvent(new ProductPublishedEvent(Id, Slug));
}
```

```bash
dotnet test tests/Modules/Catalog.Tests/
# Expected: 3 tests PASS
```

**REFACTOR — The agent checks if anything can be simplified.** In this case the code is already clean, so no refactoring is needed.

> **Tip:** Ask the agent to apply TDD to every task in the "Tests" phase of your plan, and optionally to the command and query handlers too. Tests written this way act as living documentation of intent.

---

### Stage 7 — Code Review

**What it does:** Runs a structured review of all the code in the current worktree before it is merged into `main`. The agent reads every file it wrote, checks it against the spec and the architecture rules, and produces a review report.

**How to trigger it:**

```
/review
```

The review checks for:
- **Spec compliance** — does the code implement everything in the spec?
- **Architecture rule violations** — does any module access another module's internals?
- **CQRS purity** — do command handlers return data beyond a plain ID? (They should not.) Do query handlers write anything?
- **Domain model correctness** — are aggregates protecting their invariants? Are value objects immutable?
- **Missing tests** — is every public behavior covered by at least one unit test?
- **Security** — are all admin endpoints protected with role-based authorization? Is any sensitive data (passwords, tokens) ever logged?

**Worked Example:**

```
/review

Review all code in the current worktree (feature/catalog) against
SPEC.md, PLAN.md, and the four architecture rules. Produce a
REVIEW.md report that lists: (1) compliant items, (2) issues that
must be fixed before merge, (3) suggestions (optional improvements).
```

**Example `REVIEW.md` excerpt:**

```markdown
# Catalog Module Code Review

## ✅ Compliant

- Product aggregate correctly uses private setters and exposes
  behavior only through methods.
- Money value object is a record (immutable by default in C#).
- IProductRepository is defined in the Domain layer (correct —
  infrastructure must depend on domain, not vice versa).
- CreateProductCommandHandler does not query the database
  for the product after inserting it (CQRS-compliant).
- All admin endpoints require the `Admin` role.

## ❌ Must Fix Before Merge

1. **Architecture rule violation:** `SearchProductsQueryHandler`
   calls `_inventoryService.GetAvailability()` directly,
   importing `Inventory.Application` namespace.
   The Catalog module must not depend on the Inventory module.
   Fix: Remove the availability check from this query.
   Availability is Inventory's concern, not Catalog's.

2. **Missing domain event:** `Product.AddCategory()` modifies the
   aggregate but raises no domain event. If other modules need to
   react to category changes, add `ProductCategoryChangedEvent`.

## 💡 Suggestions

- Consider adding a `CreatedAt` and `UpdatedAt` timestamp to Product
  for future admin filtering needs (not required by spec, optional).
```

**Fix every "Must Fix" issue, then re-run `/review` to confirm they are resolved before merging.**

---

### Stage 8 — Finishing a Branch

**What it does:** Merges the completed, reviewed module branch into `main` and cleans up the worktree.

**How to trigger it:**

```
/finish
```

The agent:
1. Runs all tests one final time to confirm everything passes.
2. Creates a commit on the feature branch summarizing what was built.
3. Merges the feature branch into `main`.
4. Removes the worktree folder.
5. Optionally tags the merge commit with the module name (e.g., `catalog-v1`).

**Worked Example:**

```
/finish

The catalog module review is complete and all issues are resolved.
Run all tests, then merge feature/catalog into main. Use the commit
message: "feat(catalog): implement Catalog bounded context — products,
categories, search, and publish workflow".
```

> **Before finishing, confirm:**
> - All tasks in `PLAN.md` are checked off.
> - `/review` produced no "Must Fix" issues.
> - All tests are green.
> - The code builds without warnings.

---

## 9. Module-by-Module Build Guide

Now we apply the full eight-stage workflow to each module, in build order. For each module you will find: the key prompts to give the agent, a structural sketch, and the most important things to get right.

---

### Module 1 — Identity & Access

**Set up:**

```
/worktree create identity
/worktree switch identity
```

**Brainstorm prompt (inside this worktree):**

```
/brainstorm

Spec out the Identity & Access module for our e-commerce platform.
It should handle: user registration (email + password), login with JWT
(access token, 15-minute expiry) and refresh token (7-day expiry, rotated
on use), logout (revoke refresh token), role-based access (Customer, Admin,
StoreManager roles enforced via ASP.NET Core authorization), and a
password-reset flow (token emailed via the Notifications module's event bus).

The module must never store raw passwords. Use ASP.NET Core Identity with
a custom user store backed by PostgreSQL. The module exposes: RegisterUser
command, Login command (returns tokens), RefreshToken command, Logout
command, and RequestPasswordReset command. It owns its own database schema
("identity").
```

**Plan prompt:**

```
/plan

Create a detailed task plan for the Identity module. Start with the domain
(User aggregate, Role entity, RefreshToken value object). Then application
layer (commands and handlers). Then infrastructure (ASP.NET Core Identity
integration, JWT generation, EF Core user store). Then API endpoints.
Then tests (unit tests for token generation logic, integration tests for
the registration and login flows). Order tasks with domain first.
```

**Key structural points to check during implementation:**

```
/implement

Implement the Identity module. Important constraints:
- Passwords must be hashed with ASP.NET Core Identity's PasswordHasher.
- JWT access tokens must include: userId, email, and roles as claims.
- Refresh tokens must be stored in the database (not in the JWT) so they
  can be revoked.
- The RefreshToken entity must record: tokenHash, userId, expiresAt,
  and revokedAt (nullable).
- Do NOT expose ASP.NET Core Identity's internal types (IdentityUser,
  IdentityRole) outside the Identity.Infrastructure layer.
  Wrap them in your own domain types.
```

**Example domain structure:**

```csharp
// Identity.Domain/Aggregates/User.cs
public sealed class User : AggregateRoot<UserId>
{
    public string Email { get; private set; }
    public string PasswordHash { get; private set; }
    public UserStatus Status { get; private set; } // Active, Suspended

    private readonly List<string> _roles = new();
    public IReadOnlyList<string> Roles => _roles.AsReadOnly();

    public static User Register(string email, string passwordHash)
    {
        var user = new User { Id = UserId.New(), Email = email,
            PasswordHash = passwordHash, Status = UserStatus.Active };
        user._roles.Add("Customer");
        user.RaiseDomainEvent(new UserRegisteredEvent(user.Id, user.Email));
        return user;
    }

    public void AssignRole(string role) { /* ... */ }
}
```

**Review prompt:**

```
/review

Review the Identity module. Extra checks:
1. Confirm that refresh tokens are revoked on logout (not just ignored).
2. Confirm that password reset tokens expire after 1 hour.
3. Confirm that the token rotation logic revokes the old refresh token
   when a new one is issued.
4. Confirm that no Identity.Infrastructure types are imported by
   Identity.Domain or Identity.Application.
```

```
/finish
```

---

### Module 2 — Catalog

**Set up:**

```
/worktree create catalog
/worktree switch catalog
```

**Brainstorm prompt:**

```
/brainstorm

Spec out the Catalog bounded context. It owns: products (name, description,
slug, images, base price, status: Draft/Published/Archived, isFeatured flag),
categories (hierarchical, one parent allowed), and product-category
associations (many-to-many).

Key behaviors: Create product (starts as Draft), Publish product (raises
ProductPublishedEvent), Archive product, Set featured flag, Add/remove
categories from a product, Search products (by text, category, price range,
status:Published only), Get product by slug, Get featured products.

The module's public contract (usable by other modules) exposes only:
ProductSummaryDto (id, name, slug, basePrice, primaryImageUrl) and
ProductDetailDto (all fields). It does NOT expose its domain objects.
```

**Key thing to enforce during `/implement`:**

```
/implement

Implement the Catalog module. Critical constraint: the Catalog module must
NOT check inventory or availability. That is Inventory's job. Search results
show all published products regardless of stock. The frontend will call
Inventory separately to overlay availability information.
```

**Example frontend module for Catalog (`src/modules/catalog/`):**

```typescript
// src/modules/catalog/services/catalogApi.ts

const BASE = process.env.NEXT_PUBLIC_API_URL;

export async function searchProducts(params: {
  query?: string;
  categoryId?: string;
  minPrice?: number;
  maxPrice?: number;
  page?: number;
}): Promise<{ products: ProductSummary[]; total: number }> {
  const qs = new URLSearchParams();
  if (params.query)      qs.set("q", params.query);
  if (params.categoryId) qs.set("categoryId", params.categoryId);
  if (params.minPrice)   qs.set("minPrice", String(params.minPrice));
  if (params.maxPrice)   qs.set("maxPrice", String(params.maxPrice));
  if (params.page)       qs.set("page", String(params.page));

  const res = await fetch(`${BASE}/catalog/products?${qs}`);
  if (!res.ok) throw new Error("Failed to fetch products");
  return res.json();
}

export async function getProductBySlug(slug: string): Promise<ProductDetail> {
  const res = await fetch(`${BASE}/catalog/products/${slug}`);
  if (!res.ok) throw new Error("Product not found");
  return res.json();
}
```

```
/finish
```

---

### Module 3 — Inventory

**Set up:**

```
/worktree create inventory
/worktree switch inventory
```

**Brainstorm prompt:**

```
/brainstorm

Spec out the Inventory bounded context. It manages stock for SKUs (Stock
Keeping Units). A SKU maps to a product variant (e.g., "Blue T-Shirt Size M").
The module does NOT know about product names or prices — it only knows SKUs
and quantities.

Commands: SetStockLevel(sku, quantity), ReserveStock(sku, quantity,
reservationId, expiresAt), ReleaseReservation(reservationId),
ConfirmReservation(reservationId) — converts a reservation into a permanent
deduction.

Queries: GetAvailability(sku) → { available: int, reserved: int, onHand: int },
CheckAvailability(sku, quantity) → { isAvailable: bool }.

Events emitted: StockReserved, ReservationReleased, ReservationConfirmed,
StockReplenished, LowStockThresholdReached.

Reservations expire automatically after 30 minutes if not confirmed.
Use a background service (IHostedService) to sweep and release expired
reservations every 5 minutes.
```

**Implement prompt (focus on reservation logic):**

```
/implement

Pay careful attention to the reservation system. A reservation must be
idempotent: calling ReserveStock twice with the same reservationId must
not double-deduct stock. Use the reservationId as a natural idempotency key
in the database. The available quantity is: onHand - sum(activeReservations).
Do not use a separate "available" column — calculate it at query time to
avoid race conditions.
```

**Key domain model:**

```csharp
// Inventory.Domain/Aggregates/StockItem.cs
public sealed class StockItem : AggregateRoot<SkuId>
{
    private readonly List<StockReservation> _reservations = new();

    public int QuantityOnHand { get; private set; }
    public int LowStockThreshold { get; private set; }
    public IReadOnlyList<StockReservation> Reservations
        => _reservations.AsReadOnly();

    public int QuantityAvailable =>
        QuantityOnHand - _reservations
            .Where(r => r.Status == ReservationStatus.Active)
            .Sum(r => r.Quantity);

    public void Reserve(ReservationId id, int quantity, DateTime expiresAt)
    {
        if (_reservations.Any(r => r.Id == id))
            return; // idempotent

        if (QuantityAvailable < quantity)
            throw new InsufficientStockException(Id, quantity, QuantityAvailable);

        _reservations.Add(StockReservation.Create(id, quantity, expiresAt));
        RaiseDomainEvent(new StockReservedEvent(Id, id, quantity));
    }

    public void ConfirmReservation(ReservationId id)
    {
        var reservation = _reservations.First(r => r.Id == id);
        reservation.Confirm();
        QuantityOnHand -= reservation.Quantity;
        RaiseDomainEvent(new ReservationConfirmedEvent(Id, id));
    }
}
```

```
/finish
```

---

### Module 4 — Customers

**Set up:**

```
/worktree create customers
/worktree switch customers
```

**Brainstorm prompt:**

```
/brainstorm

Spec out the Customers bounded context. It owns customer profiles and
address books. A customer is created either during Identity registration
(for registered users) or at guest checkout.

Commands: RegisterCustomer(firstName, lastName, email, identityUserId?),
UpdateProfile(customerId, firstName, lastName, phone),
AddAddress(customerId, address), SetDefaultAddress(customerId, addressId),
RemoveAddress(customerId, addressId),
LinkToIdentity(customerId, identityUserId).

Queries: GetCustomerProfile(customerId), GetCustomerByIdentityUserId(userId).

Address is a value object: {line1, line2?, city, state, postalCode, country}.
A customer may have multiple addresses; one is marked as default.

The module listens to UserRegisteredEvent (from Identity) and automatically
creates a customer profile for newly registered users.
```

```
/finish
```

---

### Module 5 — Cart

**Set up:**

```
/worktree create cart
/worktree switch cart
```

**Brainstorm prompt:**

```
/brainstorm

Spec out the Cart bounded context. It depends on Catalog (for price snapshots)
and Inventory (for availability checks). Importantly, it calls these as
external services via their public query contracts — it does NOT import
Catalog.Domain or Inventory.Domain.

Cart types: GuestCart (keyed by anonymousCartId in a cookie), CustomerCart
(keyed by customerId, server-side persistent).

Commands: AddItem(cartId, sku, quantity) — checks availability, snapshots price,
creates a stock reservation in Inventory.
RemoveItem(cartId, sku), UpdateQuantity(cartId, sku, quantity),
ClearCart(cartId), MergeGuestCart(guestCartId, customerId).

Queries: GetCart(cartId) → { items: [{sku, name, quantity, unitPrice, subtotal}],
subtotal, itemCount }.

When a reservation expires (Inventory emits ReservationExpiredEvent), the cart
automatically removes the affected item and marks the cart as "stale" so the
frontend can show a warning.
```

**Key cross-module communication pattern:**

```
/implement

For cross-module calls (Catalog price lookup, Inventory availability check),
inject interfaces defined in the Cart.Application layer, NOT the actual
Catalog or Inventory application classes. For example:

// Cart.Application/Ports/IProductPriceService.cs
public interface IProductPriceService {
    Task<Money?> GetCurrentPriceAsync(string sku, CancellationToken ct);
}

// Cart.Infrastructure/ExternalServices/CatalogProductPriceService.cs
// (implements the interface by calling the Catalog HTTP API or an in-process
//  query dispatcher — either way, Cart.Application does not know the difference)

This is the "port and adapter" pattern. Cart.Application defines the port
(interface). Cart.Infrastructure provides the adapter (implementation).
This ensures Cart never directly couples to Catalog's internal code.
```

**Frontend Cart module sketch:**

```typescript
// src/modules/cart/store/cartStore.ts  (Zustand)

interface CartStore {
  items: CartItem[];
  isStale: boolean;
  addItem: (sku: string, quantity: number) => Promise<void>;
  removeItem: (sku: string) => void;
  updateQuantity: (sku: string, quantity: number) => Promise<void>;
  subtotal: () => number;
}

export const useCartStore = create<CartStore>((set, get) => ({
  items: [],
  isStale: false,
  addItem: async (sku, quantity) => {
    const result = await cartApi.addItem(sku, quantity);
    if (result.ok) {
      set(state => ({ items: [...state.items, result.item] }));
    }
  },
  subtotal: () =>
    get().items.reduce((sum, item) => sum + item.unitPrice * item.quantity, 0),
  // ...
}));
```

```
/finish
```

---

### Module 6 — Ordering

**Set up:**

```
/worktree create ordering
/worktree switch ordering
```

**Brainstorm prompt:**

```
/brainstorm

Spec out the Ordering bounded context. It converts a cart into a permanent
order.

Commands:
PlaceOrder(cartId, customerId, shippingAddressId, selectedShippingMethodId)
→ creates the order, calls Inventory to confirm reservations, emits OrderPlaced.
CancelOrder(orderId, reason) → valid only in Pending/Confirmed states; emits
OrderCancelled.
UpdateOrderStatus(orderId, newStatus) → admin only; used to advance the
lifecycle (Confirmed → Processing → Shipped → Delivered).

Queries:
GetOrder(orderId), ListOrdersByCustomer(customerId, page, pageSize).

The Order aggregate records: orderId, customerId, shippingAddress (snapshot,
not a FK — addresses can change later but the order address must not),
orderLines [{sku, productName, quantity, unitPrice}], orderTotal (Money),
shippingCost (Money), status, placedAt.

Events emitted: OrderPlaced, OrderConfirmed, OrderCancelled, OrderShipped,
OrderDelivered.

The module listens to PaymentSucceeded (from Payments) → advances status
from Pending to Confirmed. Listens to PaymentFailed → cancels the order.
```

**Checkout flow on the frontend:**

```typescript
// src/app/(shop)/checkout/page.tsx  (simplified)

export default function CheckoutPage() {
  const { items } = useCartStore();
  const [step, setStep] = useState<"address" | "shipping" | "payment">("address");

  return (
    <CheckoutLayout>
      {step === "address"  && <AddressStep    onNext={() => setStep("shipping")} />}
      {step === "shipping" && <ShippingStep   onNext={() => setStep("payment")}  />}
      {step === "payment"  && <PaymentStep    />}
    </CheckoutLayout>
  );
}
```

```
/finish
```

---

### Module 7 — Payments

**Set up:**

```
/worktree create payments
/worktree switch payments
```

**Brainstorm prompt:**

```
/brainstorm

Spec out the Payments bounded context. For development, use a stub payment
provider (always succeeds or can be configured to fail). In production, the
stub is replaced with a Stripe adapter — the domain code must not change.

The module listens to OrderPlaced and creates a PaymentIntent.

Commands:
CreatePaymentIntent(orderId, amount, currency) → returns a clientSecret
(or payment URL) that the frontend uses to collect card details directly
with Stripe.js (no raw card data touches our server).
ConfirmPayment(paymentIntentId) → called via Stripe webhook when payment
succeeds; emits PaymentSucceeded.
FailPayment(paymentIntentId, reason) → called via Stripe webhook on failure;
emits PaymentFailed.
InitiateRefund(paymentId, amount?) → creates a refund; emits RefundIssued.

Queries: GetPaymentStatus(orderId) → { status, amount, lastAttemptAt }.

The payment provider is injected via IPaymentGateway interface (port and
adapter pattern — same as Cart used for IProductPriceService).
```

**Webhook security note:**

```
/implement

The Stripe webhook endpoint must verify the Stripe-Signature header using
the webhook secret. Do not process any webhook payload that fails signature
verification. The webhook handler should be exempt from the global
authentication middleware (it uses Stripe's signature as its own auth
mechanism), but ALL other Payments endpoints require authentication.
```

```
/finish
```

---

### Module 8 — Shipping

**Set up:**

```
/worktree create shipping
/worktree switch shipping
```

**Brainstorm prompt:**

```
/brainstorm

Spec out the Shipping bounded context. For development, use a configurable
rate table (e.g., Standard $5.99 / 5–7 days, Express $14.99 / 2–3 days,
Overnight $29.99 / next day). In production, this is replaced with a carrier
API adapter.

Commands:
CreateShipment(orderId, shippingAddress, items[], selectedMethodId) → creates
a shipment record; emits ShipmentCreated.
UpdateTrackingNumber(shipmentId, carrier, trackingNumber).
MarkDispatched(shipmentId) → emits ShipmentDispatched.
MarkDelivered(shipmentId) → emits ShipmentDelivered.

Queries:
GetShippingOptions(address, items[]) → list of available methods with cost
and estimated delivery date.
GetShipmentStatus(orderId) → { status, carrier, trackingNumber, estimatedDelivery }.

The module listens to OrderConfirmed (from Ordering) to automatically
create a shipment.
```

```
/finish
```

---

### Module 9 — Notifications

**Set up:**

```
/worktree create notifications
/worktree switch notifications
```

**Brainstorm prompt:**

```
/brainstorm

Spec out the Notifications bounded context. It is purely reactive — it has
no commands from end users. It listens to integration events and sends
messages.

For development, write emails/SMS to the application log instead of actually
sending them (use an INotificationSender abstraction; swap in a real provider
like SendGrid for production).

Events to handle and what to send:
- OrderPlaced → order confirmation email (order summary, shipping address,
  estimated delivery)
- PaymentSucceeded → payment receipt email (amount charged, last 4 of card
  if available)
- ShipmentDispatched → shipping notification email + optional SMS (tracking
  number, carrier, estimated delivery date)
- OrderCancelled → cancellation email (order ID, reason, refund status)
- UserRegisteredEvent → welcome email (from Identity module)
- PasswordResetRequested → password reset email with a 1-hour link
  (from Identity module)
- LowStockThresholdReached → internal alert email to store admin
  (from Inventory module)

Each handler must check the customer's notification preferences before
sending. If the customer has opted out of the relevant notification type,
silently skip the send (do NOT throw an error).
```

**Implement prompt:**

```
/implement

Implement Notifications. Key points:
1. Use an outbox pattern: store the notification intent in the database
   first, then send it. This ensures no notification is lost if the
   app crashes between receiving the event and sending the email.
2. Each INotificationHandler implementation should be tested with a mock
   INotificationSender — verify the correct template and recipient are used.
3. For the dev stub, log the full rendered email body (subject and HTML body)
   at Information level so developers can see what would be sent.
```

**Example handler:**

```csharp
// Notifications.Application/Handlers/OrderPlacedNotificationHandler.cs

public sealed class OrderPlacedNotificationHandler
    : IIntegrationEventHandler<OrderPlacedEvent>
{
    private readonly INotificationSender _sender;
    private readonly ICustomerPreferencesService _preferences;
    private readonly IEmailRenderer _renderer;

    public async Task Handle(OrderPlacedEvent evt, CancellationToken ct)
    {
        if (!await _preferences.AllowsEmailAsync(evt.CustomerId, "transactional", ct))
            return;

        var email = await _renderer.RenderAsync("order-confirmation", new {
            OrderId = evt.OrderId,
            CustomerName = evt.CustomerName,
            OrderLines = evt.OrderLines,
            Total = evt.OrderTotal,
            ShippingAddress = evt.ShippingAddress
        });

        await _sender.SendEmailAsync(evt.CustomerEmail, email.Subject, email.Body, ct);
    }
}
```

```
/review

Final review of the Notifications module. Confirm:
1. Every handler checks notification preferences before sending.
2. The outbox pattern is implemented (notification persisted before sent).
3. No handler directly queries another module's database — all data comes
   from the event payload.
4. Dev stub logs emails at Information level, not Debug (so they appear in
   the default log output).
```

```
/finish
```

---

## 10. What You've Learned

By completing this tutorial you have learned:

**Superpowers workflow:**
- How to install the Superpowers skills framework into Claude Code.
- How the eight-stage pipeline (Brainstorm → Plan → Worktree → Implement → TDD → Review → Finish) prevents AI agents from going off-rails on large projects.
- How subagents provide task isolation and a natural two-stage review.
- How to use `/brainstorm` to turn a vague idea into a precise spec, and why this step is non-negotiable.

**Domain-Driven Design:**
- The difference between strategic DDD (Bounded Contexts, Ubiquitous Language) and tactical DDD (Aggregates, Entities, Value Objects, Domain Events, Repositories).
- Why modules must communicate only through public contracts and events — not by reaching into each other's internals.
- How to model business behavior inside an Aggregate (`Product.Publish()`, `StockItem.Reserve()`).

**CQRS:**
- How to split every operation into a Command (changes state, returns nothing) or a Query (reads state, changes nothing).
- How MediatR wires commands and queries to their handlers without tight coupling.

**Modular Monolith:**
- How to get the organizational discipline of microservices inside a single deployable application.
- How the port-and-adapter pattern (e.g., `IProductPriceService`) lets a module depend on an abstraction it defines, not on another module's concrete code.

**Test-Driven Development:**
- How to apply RED-GREEN-REFACTOR: write failing tests, make them pass with minimum code, then refactor safely.
- Why tests written before implementation are more trustworthy than tests written after.

---

## 11. Next Steps

Now that the core platform is working, here are the logical next phases:

### Immediate Improvements

1. **Admin dashboard** — Build a Next.js admin area for managing products, viewing orders, and processing refunds. Use the same module structure; add an `admin` route group under `src/app/(admin)/`.
2. **Real payment integration** — Replace the Stripe stub with a real `StripePaymentGateway` adapter that implements `IPaymentGateway`. The domain code does not change.
3. **Real email delivery** — Replace the logging stub with a `SendGridNotificationSender` adapter. Again, no domain code changes.
4. **Search engine** — Replace the EF Core full-text search in Catalog with an Elasticsearch adapter behind the `IProductSearchService` port.

### Scaling the Architecture

5. **Outbox → message broker** — Replace the in-process event bus with RabbitMQ or Azure Service Bus. The domain event and integration event contracts do not change — only the infrastructure wiring does.
6. **Module extraction** — If one module (e.g., Payments) needs to scale independently, extract it into a microservice. Because it already owns its data and communicates only through public contracts, this is mostly an infrastructure change.
7. **Read-model projections** — Add dedicated read models (projection tables or a separate read database) for high-traffic queries like product search — a natural CQRS evolution.

### Platform Additions

8. **Product reviews and ratings** — A new `Reviews` bounded context. It listens to `OrderDelivered` to know which customers can leave reviews.
9. **Discounts and promotions** — A new `Promotions` bounded context. The Cart module consults `IPromotionService` (a port) to apply active discounts.
10. **Multi-currency** — Extend the `Money` value object and Catalog pricing to support multiple currencies with configurable exchange rates.

### Developer Experience

11. **Run `/brainstorm` on a new feature** — Practice the full pipeline with a feature like "wishlists" or "product bundles". The workflow generalizes to any feature of any size.
12. **Set up a CI pipeline** — Add a GitHub Actions workflow that runs `dotnet test` and `next build` on every push to `main`. The modular test structure makes it easy to run only the tests for changed modules.

---

*Built with [Superpowers](https://github.com/obra/superpowers) — the agentic skills framework for Claude Code.*
