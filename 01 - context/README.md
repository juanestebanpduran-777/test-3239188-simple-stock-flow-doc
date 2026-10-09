# Context and Scope — Simple Stock Flow

## 1. Overview

Simple Stock Flow manages a product catalog, fixed categories, inventory levels, and sales recorded by internal users. Each sale preserves when it occurred, which operator recorded it, and its lines; the lines retain copies of the relevant commercial values at the time of sale. The system supports date-range sales queries and a product-aggregated report (§§1, 2, 5, and 6).

## 2. System context

### Main inputs
- Allowed product data: name, price, stock, category, and optional image (§1 and DP-03).
- Internal operator credentials, processed through the hashing port (§§2.5 and 9.2).
- A request to record a sale with one or more lines (§2.3).
- A date range for querying sales and generating reports (§§1 and 6.1).

### Main outputs
- Searchable catalog and available categories (§6.1 Q1–Q5).
- Sale with its lines (§6.1 Q6).
- Sales list by date range (§6.1 Q7).
- Product-aggregated report calculated by the database (§6.1 Q9).
- Authentication result without exposing hashes or credentials (§7).

The concrete request, response, and error formats are not defined in the data model; the model's own exclusions indicate that these belong in a separate API contract.

## 3. Actors

| Actor | Description | Evidence |
|---|---|---|
| Internal operator | User who signs in and records sales. | §1 |
| `admin` | A role allowed for a user. Specific permissions are not described here. | §§1 and 2.5 |
| `seller` | A role allowed for a user. Specific permissions are not described here. | §§1 and 2.5 |
| Report consumer | Person or process that queries the sales aggregate. **Assumption:** not formally identified in the model. | §6.1 Q9 |

The model explicitly states that there is no customer or buyer entity (§1).

## 4. In scope

- Product catalog with name, price, stock, category, and optional image.
- Product queries for active items, by identifier, and by batch of identifiers.
- Read-only access to seeded categories.
- Internal-user authentication and the `admin` / `seller` role set.
- Sales recording with at least one line.
- Stock withdrawal associated with adding sale lines.
- Preservation of historical product names and prices in sale lines.
- Sales queries by date range and product-aggregated reporting.
- Image handling through an opaque key to external storage.
- Indefinite retention of sales and their lines.

**Traceability:** §§1–2, 6–7, and 9.

## 5. Out of scope or unsupported

- Final customer/buyer management.
- Payments, cards, payment gateways, and electronic invoicing.
- Multiple currencies.
- Category CRUD.
- Editing or deleting recorded sales.
- Reports broken down by seller or exposing operator personal data.
- Product fields such as description, SKU, or reference code.
- `created_at` and `updated_at` columns.
- A persisted table for reports, totals, or subtotals.
- Physical deletion of products.
- Guarantees of atomicity between the database and image storage.

**Traceability:** §§1, 7, 7.1, 8, and DP-02/DP-03.

## 6. Context constraints

1. PostgreSQL 16.14, database `simple_stock_flow`, schema `sales`, UTC server, according to the recorded verification (§§3 and 10).
2. The five categories are seeded by the initial migration and are not managed through CRUD (§§2.1 and 9.1).
3. Entity Framework migrations own the DDL (§3.2).
4. Not all invariants are enforced by the database: each rule is marked `database`, `domain only`, or `pending` (§§1 and 4).
5. Sales and sale lines are retained indefinitely (§7.1).

## 7. Assumptions

- **ASSUMPTION-CONT-01:** the system is used by a business that sells products; the business type is not identified.
- **ASSUMPTION-CONT-02:** operators access the system through an application interface; the specific channel is not established by the model.
- **ASSUMPTION-CONT-03:** report consumers have internal authorization; specific permissions must be confirmed in the application/API contract.

## 8. References

- `spec/data-model.md` §1: glossary and purpose of concepts.
- §§2–5: entities, invariants, and relationships.
- §6: query patterns.
- §§7–9: privacy, retention, audit, and seed data.
- §10: physical-schema verification.
