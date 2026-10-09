# Domain Model — Simple Stock Flow

**Source:** `spec/data-model.md`  
**Purpose:** describe the business language, entities, relationships, invariants, and deducible events without adding unsupported concepts.

## 1. Glossary

| Term | Definition | Evidence |
|---|---|---|
| Product (`Product`) | Catalog item with name, price, stock, category, and optional image. No additional attributes are included. | §1, DP-03 |
| Category (`Category`) | Fixed product classification; five seeded rows exist and the repository is read-only. | §§1, 2.1, and 9.1 |
| Price (`Money`) | Current product catalog amount; strictly positive and rounded to two decimal places. | §§1 and 2.2 |
| Stock | Available units; it must never be negative. | §§1 and 2.2 |
| Image | Binary stored externally and identified in the database by an opaque `image_key`. | §§1, 3, and 7.1 |
| Sale (`Sale`) | Immutable commercial fact recording who sold, when, and what was sold. | §§1 and 2.3 |
| Sale line (`SaleItem`) | Item within a sale, with product, quantity, and values frozen at sale time. | §§1 and 2.4 |
| Quantity (`Quantity`) | Number of units sold in a line; must be positive. | §§1 and 2.4 |
| Line subtotal | Unit price multiplied by quantity; calculated, not stored. | §1 |
| Sale total | Sum of line subtotals; calculated, not stored. | §1 |
| User (`User`) | Internal operator who signs in and records sales; not the final buyer. | §§1 and 2.5 |
| Role | A user's single role, limited in the domain to `admin` or `seller`. | §§1 and 2.5 |
| Password hash | One-way credential hash; the domain never sees the plaintext password. | §§1, 2.5, and 7 |
| Date range | Interval used to query sales/reports; the end cannot precede the start. | §1 |
| Aggregated report | Result grouped by product and date range; not persisted in a table. | §§1 and 6.1 |

## 2. Entities and aggregates

### `Category`
- Reference entity, not an aggregate root.
- Name is required, trimmed, and unique.
- No lifecycle is managed through ports; its five rows are created by the initial migration.
- **Traceability:** §§2.1 and 9.1.

### `Product` — aggregate root
- Data: identifier, name, price, stock, category, and optional image; the schema also documents technical soft-delete fields according to their implementation status.
- Invariants: name is non-empty and trimmed; price is greater than zero; stock is non-negative; withdrawals cannot exceed available stock; category is required and must exist; absent image is represented by `NULL`; removal is logical rather than physical.
- **Traceability:** §§2.2–3 and §4.

### `Sale` — aggregate root
- Represents an immutable commercial event.
- Records the operator and sale time.
- Must contain at least one line to be confirmed.
- The same product cannot occur more than once within a sale.
- Adding a line and withdrawing stock are coordinated domain operations.
- No edit/delete ports exist.
- **Traceability:** §2.3.

### `SaleItem` — entity inside `Sale`
- Cannot legitimately be created independently.
- Includes product, quantity, unit price, and frozen copies of the product name and category.
- Quantity must be positive.
- It cannot exist outside its sale.
- **Traceability:** §2.4 and FK-2 in §5.

### `User` — aggregate root
- Data: identifier, username, password hash, and role.
- Username is unique, lowercase, and trimmed.
- The hash is required; plaintext password does not enter the domain.
- Domain roles are `admin` and `seller`.
- **Traceability:** §2.5 and §7.

## 3. Relationships and cardinalities

| Relationship | Cardinality | Rule |
|---|---|---|
| `Category` → `Product` | 1:N | Each product has exactly one required category; a category may have no products. |
| `Sale` → `SaleItem` | 1:N | A confirmable sale has at least one line; lines cannot exist outside it. |
| `Product` → `SaleItem` | 1:N from product to sale lines | Each line references an existing product that is not soft-deleted at sale time. |
| `User` → `Sale` | Conceptual 1:N | Each sale is attributed to an existing operator; the `sold_by_user_id` FK is listed as pending T-12 in the model. |
| `Sale` ↔ `Product` | N:M through `SaleItem` | The associative line stores its own quantity and historical values. |

**Traceability:** §5, cardinality table, and FK-1 through FK-4.

## 4. Business rules and enforcement

Rules enforced by the domain must be distinguished from constraints currently active in the database.

| Rule | Enforcement according to the model | Evidence |
|---|---|---|
| Stock is non-negative | Database `CHECK` | §§2.2 and 4 |
| Sufficient stock before withdrawal | Domain | §2.2 |
| Positive price | Domain only; task T-20 identified | §§2.2 and 4 |
| Positive quantity | Domain only; task T-20 identified | §§2.4 and 4 |
| Unique category name | Database unique index | §§2.1 and 4 |
| Unique username | Database unique index | §§2.5 and 4 |
| A line cannot be separated from its sale | FK-2 and composition | §§2.4 and 5 |
| Product cannot repeat within a sale | Domain and composite index described by the model; verify physical state against §10 | §§2.3 and 4 |
| Sale is immutable | No edit/delete port | §§2.3 and 7.1 |
| Sale has at least one line | Domain | §2.3 |
| Valid role | Domain only in the described state | §§2.5 and 4 |
| Sale authorship linked to `User` | FK-4 pending T-12 | §§3 and 5 |

## 5. Domain events

The model does not list a formal catalog of domain events. The following are **proposed conceptual events**, inferred from described operations; they are not claimed to exist as implemented classes or messages.

| Proposed event | When it would occur | Basis / limitation |
|---|---|---|
| `ProductCreated` | A product is added to the catalog. | **Inference** from `Product`; the event is not formally declared. |
| `ProductPriceChanged` | The current product price changes. | `Product.ChangePrice` appears in §2.2; the event is proposed. |
| `StockRestocked` | Product stock increases. | `Product.Restock` appears in §2.2; the event is proposed. |
| `StockWithdrawn` | Stock is deducted when a sale line is added. | `Product.Withdraw` and `Sale.AddItem`, §§2.2–2.3; the event is proposed. |
| `SaleRecorded` | A sale with its lines is confirmed. | A sale is an immutable completed fact, §§1 and 2.3; the event name is proposed. |
| `ProductSoftDeleted` | A product is soft-deleted. | §§2.2 and 7.1; the event is proposed. |

Do not implement integration events, queues, or messaging based only on this table.

## 6. Value objects

- `Money`: represents amounts; rounds to two decimal places using `AwayFromZero`; database column is `numeric(18,2)` (§2.2).
- `Quantity`: enforces a strictly positive quantity (§§1 and 2.4).
- Date range: application-layer value object; end cannot precede start (§1).
- Sale total and line subtotal: calculated values, not persisted columns (§1).

## 7. Assumptions and boundaries

- Events in §5 are vocabulary proposals, not confirmed implemented features.
- There is no final buyer/customer entity (§1).
- No N:M relationship between user and role is inferred: each user has one role from a closed set (§5).
- Pending states must retain the status given by the model until implementation is verified (§§3–5 and 10).
