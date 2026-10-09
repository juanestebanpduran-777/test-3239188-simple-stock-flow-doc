# Product Vision — Simple Stock Flow

## 1. Problem

The data model describes a solution that maintains a product catalog, stock levels, and sales recorded by internal operators. A sale preserves who recorded it, when it occurred, and which products were sold, including quantities and values frozen at the time of the commercial event (§§1–3).

**Inferred problem:** without consistent records of the catalog, inventory, and sales, a business would have difficulty knowing available stock and reviewing commercial history. This is a reasonable inference from the entities and queries described, not a direct statement from a system user.

## 2. Vision

Simple Stock Flow is a catalog, inventory, and sales management system that allows users to browse products and categories, record sales attributed to internal operators, protect stock integrity, and generate product-aggregated reports for a date range.

**Traceability:** §§1, 2, 5, 6.1, and 7.

## 3. Users and stakeholders

| Actor | Relationship to the system | Evidence |
|---|---|---|
| Internal operator | Signs in and records sales. | §§1 and 2.5 |
| User with `admin` role | One of the two valid roles. Specific permissions are not detailed in the model. | §§1, 2.5, and 9.2 |
| User with `seller` role | One of the two valid roles. Specific permissions are not detailed in the model. | §§1 and 2.5 |
| Business/report stakeholder | May benefit from viewing the aggregated report. **Assumption:** the model does not formally identify this person as an actor. | §6.1 Q9 |

The model explicitly states that there is no customer or buyer entity (§1).

## 4. Product objectives

1. Maintain a product catalog with products assigned to predefined categories (§§2.1–2.2 and 9.1).
2. Protect stock integrity and prevent negative stock (§2.2 and §4).
3. Record sales with at least one line and without duplicating a product within the same sale (§2.3).
4. Preserve historical product values even when the catalog changes (§§1 and 2.4).
5. Query sales by date range and generate product-aggregated reports (§6.1 Q6–Q9).
6. Protect credentials and restrict exposure of personal data (§7).

## 5. Functional scope

### Included
- Product catalog with name, price, stock, category, and optional image.
- Read-only access to the five seeded categories; no category CRUD.
- Authentication for internal operators and the `admin` / `seller` role set.
- Immutable sales and sale lines.
- Stock withdrawal associated with adding sale lines.
- Sales queries by date range.
- Product-aggregated reports calculated by the database.
- External image handling through an opaque key.

**Traceability:** §§1–2, 6–7, and 9.

### Excluded or unsupported by the model
- Customer/buyer management.
- Payments, cards, payment gateways, and electronic invoicing.
- Multiple currencies.
- Product attributes such as description, SKU, or reference code.
- Reports broken down by seller.
- Category CRUD.
- Editing or deleting recorded sales.
- `created_at` / `updated_at` audit columns.
- External analytics or exports not described in the model.

**Traceability:** §§1, 7, 8, and DP-02/DP-03.

## 6. Derived success indicators

- Domain operations do not leave stock below zero (§2.2).
- Confirmed sales contain at least one line and preserve historical values (§§2.3–2.4).
- Reports are calculated by product and date range without persisting a report table (§§1 and 6.1).
- Credentials are not exposed in logs or responses (§7).

No percentages or response-time targets are assigned because the data model provides no quantitative objectives.

## 7. Assumptions

- **ASSUMPTION-PROD-01:** the system is used by a business that sells products; the specific business sector is not identified.
- **ASSUMPTION-PROD-02:** the report supports operational decision-making; the model specifies the report but does not define its consumer.
