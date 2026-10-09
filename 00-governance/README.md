# Architecture — Simple Stock Flow

**Status:** Proposal derived from the data model  
**Primary source:** `spec/data-model.md`  
**Traceability:** §§1–6 and §§7–9, especially §§2.2–2.5 and §§4–6.

## 1. Inferred architectural style

The model describes a domain-centered architecture with separation between the business core and infrastructure adapters, compatible with a **hexagonal architecture (ports and adapters)**. Direct evidence includes references to aggregates, read ports, persistence adapters, a hashing port, and rules enforced by the domain (§§1, 2, 6, and 9.2).

The data model does not describe every deployment detail. The concrete runtime architecture must be checked against implementation documents, which are outside the input provided for this challenge.

## 2. Aggregates and responsibilities

| Element | Inferred responsibility | Evidence |
|---|---|---|
| `Product` | Catalog aggregate root; protects positive price, stock withdrawal/restocking, and category association. | §2.2 |
| `Sale` | Sales aggregate root; records the sale and coordinates line creation and stock withdrawal. | §2.3 |
| `SaleItem` | Entity contained within `Sale`; cannot legitimately be created independently. It freezes the sold product name, price, and category. | §§2.3–2.4 |
| `User` | Identity aggregate root; normalizes the username and stores password hash and role. | §2.5 |
| `Category` | Read-only reference entity; its five rows are created by the initial migration. | §§2.1 and 9 |

The value objects identified by the model are `Money`, `Quantity`, and the report date range. The sale total and line subtotal are calculated, not stored as columns (§1).

## 3. Logical components

1. **Domain:** entities, aggregates, value objects, and invariants. It owns rules such as `price > 0`, `quantity > 0`, sufficient stock, and a sale containing at least one line (§§2.1–2.5).
2. **Application:** coordinates use cases and defines the report date range; the model identifies that range as an application-layer value object (§1). The complete use-case list is a proposal to validate.
3. **Inbound ports:** conceptual interfaces for catalog, sales, authentication, and reporting operations. **Design assumption:** their exact names and contracts are not specified in the data model.
4. **Outbound ports:** repositories/queries for products, categories, sales, and users; a read port for the aggregated report; a hashing port for credentials; and access to external image storage. Evidence: §§1, 6, 7, and 9.2.
5. **Persistence adapters:** map aggregates to PostgreSQL and translate class/collection names into tables and columns. Entity Framework migrations own the DDL (§§0 and 3.1–3.2).
6. **Image-storage adapter:** stores/retrieves the external binary through `image_key`; the database stores an opaque key, not the file or its path (§§1, 3, and 7.1).
7. **Database engine:** PostgreSQL 16.14, database `simple_stock_flow`, schema `sales`, according to the verification recorded in the document (§§3 and 10).

## 4. Where each rule is enforced

| Rule | Current owner | Status / evidence |
|---|---|---|
| `product.stock >= 0` | PostgreSQL `CHECK` constraint | Enforced by the database; §4 |
| Withdrawing more units than available | Domain (`Product.Withdraw`) | Domain only; §2.2 |
| `product.price > 0` | Domain (`Product.ChangePrice`) | Domain only; task T-20 plans database enforcement; §§2.2 and 4 |
| `sale_item.quantity > 0` | Domain (`Quantity`) | Domain only; T-20; §§2.4 and 4 |
| Sale must have at least one line | Domain (`Sale.EnsureConfirmable`) | Domain only; §2.3 |
| Category required and existing for a product | Domain + FK-1 | FK enforced by database; §5 |
| Sales cannot be edited or deleted | No edit/delete ports by design | Domain design; §§2.3 and 7.1 |
| Password hashing; domain never sees plaintext password | Hashing port | Hexagonal design; §§2.5, 7, and 9.2 |
| Unique `username` and `category.name` | Unique indexes | Database; §4 |
| Sale-author attribution FK (`sale.sold_by_user_id`) | Planned constraint | Pending T-12; §§3 and 5 |
| Product soft deletion | `deleted_at` and global filter | Described in §§2.2 and 3; verify the current schema against §10 before claiming deployment status |



## 5. Persistence and queries

Documented access patterns include product search/listing for active products, product by ID, products by batch of IDs, category listing, sale with lines, sales by date range, an aggregated report by product, and user lookup by username (§6.1).

The report is calculated in PostgreSQL through a read port and is not stored in a report table (§§1 and 6.1). Additional indexes listed in §6.2 must be treated as pending, not as already existing.

## 6. Security and retention

- `password_hash` must never appear in logs, responses, projections, or errors; it is not indexed (§7).
- `username` and sale authorship are classified as personal data and require restricted access (§7).
- Sales and sale lines are retained indefinitely and are not edited or deleted (§7.1).
- Products are soft-deleted rather than physically deleted (§§2.2 and 7.1).
- The image binary is deleted after the database key has been cleared; atomicity between PostgreSQL and external storage is not promised (§7.1).

## 7. Explicit assumptions

- **ASSUMPTION-ARQ-01:** the application exposes use cases through an API. The model refers to an API/contract in its exclusions, but does not define endpoints here.
- **ASSUMPTION-ARQ-02:** concrete port interfaces will follow the implementation repository's conventions; the data model only supports their responsibilities.
- **ASSUMPTION-ARQ-03:** authentication and authorization run in the application/API layer. The model says authorization belongs to the API (§9.2), but does not specify every flow here.

## 8. Traceability references

- `spec/data-model.md` §§0–2: naming conventions, glossary, aggregates, and invariants.
- §§3–5: physical schema, constraints, and foreign keys.
- §6: access patterns and indexes.
- §§7–9: privacy, retention, audit, and seed data.
- §10: queries used to verify the physical schema.

## 9. Closing review: consistency check

Before considering the documentation complete, verify that:

- [ ] `Product`, `Sale`, `SaleItem`, `User`, and `Category` are the only domain entities described (§2).
- [ ] Sale totals and line subtotals are calculated; no report tables or columns are invented (§1).
- [ ] `SaleItem` remains inside the `Sale` aggregate (§§2.3–2.4).
- [ ] The report is calculated in the database through a read port and does not expose a seller breakdown (§§1, 6, and 7).
- [ ] Rules are correctly labeled as database-enforced, domain-only, or pending (§§1 and 4).
- [ ] No customers, payments, multiple currencies, SKUs, or `created_at`/`updated_at` audit columns are added (§§1, 7, and 8).
- [ ] The architecture does not claim pending foreign keys, indexes, or other tasks are implemented without checking §10.
- [ ] Assumptions are clearly identified and not presented as confirmed requirements.

