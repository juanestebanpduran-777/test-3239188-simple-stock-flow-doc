# Requirements — Simple Stock Flow

**Primary source:** `spec/data-model.md`  
**Convention:** derived requirements include traceability. When the model does not determine complete behavior, it is marked as an assumption.

## 1. Derived user stories

### US-01 — Browse products
As an internal operator, I want to search and view active products so that I can check their price, stock, and category.  
**Traceability:** §6.1, Q1–Q3; §§1 and 2.2.  
**Acceptance criteria:**
- The query supports partial-text and category filters and sorts by name (Q1).
- Products marked as soft-deleted are excluded when that state is applied.
- A product can be retrieved by identifier (Q2).
- Active products can be retrieved by a batch of identifiers (Q3).

### US-02 — Browse categories
As an internal operator, I want to view available categories to understand how products are classified.  
**Traceability:** §§2.1, 6.1 Q4–Q5, and 9.1.  
**Acceptance criteria:**
- Categories can be listed in name order.
- A category can be retrieved by identifier.
- Category creation, editing, and deletion are not offered because the repository is read-only.

### US-03 — Maintain products
As an internal operator, I want to create and update the allowed product data so that the catalog remains accurate.  
**Traceability:** §§1, 2.2, 3, and 6.1.  
**Acceptance criteria:**
- Product attributes are limited to name, price, stock, category, and optional image.
- The name is trimmed and cannot be empty.
- Price must be strictly positive.
- Stock must never become negative, and a withdrawal cannot exceed available stock.
- Category is required and must exist.
- An image is represented by an opaque key; absence is represented by `NULL`, not an empty string.
- Product removal is logical (soft deletion), not physical deletion.
**Note:** the model distinguishes rules enforced by the domain from those enforced by the database; do not assume all rules already have SQL constraints (§§2.2 and 4).

### US-04 — Record a sale
As an authenticated user with a valid internal role, I want to record a sale with its lines so that the commercial event is preserved and stock is updated.  
**Traceability:** §§1, 2.3–2.4, 5, and 7.1.  
**Acceptance criteria:**
- A confirmable sale contains at least one line.
- The same product cannot appear more than once in one sale.
- Each line quantity is greater than zero.
- A sale cannot withdraw more stock than is available.
- Adding a line and withdrawing stock are coordinated as one domain operation.
- The line preserves the product name, price, and category at the time of sale, according to the implementation state documented by the model.
- A recorded sale cannot be edited or deleted.
- The sale total is calculated from line subtotals; it is not stored as a column.

### US-05 — Browse sales by date
As an authorized operator, I want to query sales within a date range so that I can review commercial activity.  
**Traceability:** §§1 and 6.1 Q6–Q8.  
**Acceptance criteria:**
- A sale can be retrieved with its lines.
- Sales can be listed within a date range, ordered by descending date and paginated.
- The range end cannot be earlier than the range start (§1).
**Assumption:** the user interface for entering the range is not defined by the data model.

### US-06 — Generate an aggregated sales report
As an authorized operator, I want a sales report grouped by product so that I can analyze quantities and amounts sold during a date range.  
**Traceability:** §§1, 6.1 Q9, and 7.  
**Acceptance criteria:**
- The report is calculated in the database through a read port.
- Results are grouped by product and use the frozen values stored in sale lines.
- The report is not persisted as a table.
- No seller-level breakdown is included, according to DP-02 and §7.
- The date range must be valid.

### US-07 — Authenticate
As an internal operator, I want to sign in with my username and credential so that I can access the system.  
**Traceability:** §§1, 2.5, 6.1 Q10, 7, and 9.2.  
**Acceptance criteria:**
- User lookup uses an exact username match.
- The username is normalized to lowercase and trimmed.
- The password is verified through the hashing port; the domain does not receive or store the plaintext password.
- `password_hash` is not exposed in responses, logs, projections, or errors.
- Valid domain roles are `admin` and `seller`.
**Explicit risk:** §9.2 identifies anonymous user registration as an authorization defect. The model does not prove that this defect has been fixed.

## 2. Derived non-functional requirements

| ID | Requirement | Rationale and traceability |
|---|---|---|
| NFR-01 Stock integrity | The system must prevent valid operations from leaving stock below zero and detect writes that bypass the adapter. | §2.2; §4; ADR-002 referenced by the model. |
| NFR-02 Sale integrity | Recorded sales and their lines must remain consistent; orphaned lines and duplicate products within one sale must be prevented. | §§2.3–2.4, §4, and FK-2. |
| NFR-03 Historical consistency | Line names, prices, and category must reflect the sale-time values and must not change when the catalog changes. | §§1, 2.4, 5, and ADR-004. |
| NFR-04 Credential protection | The domain works only with hashes; hashes must not be logged or exposed and must not be indexed. | §§2.5 and 7. |
| NFR-05 Personal-data protection | Access to `username` and sale authorship must be restricted; reports must not expose operator personal data. | §7 and DP-02. |
| NFR-06 Retention | Sales and lines are retained indefinitely; products are not physically deleted; image binaries are removed according to the documented procedure. | §7.1. |
| NFR-07 Concurrency | Stock-changing operations must be protected against concurrent writes and follow the mechanism described by ADR-002/D-04. | §§2.2 and 6.1 Q3. The full implementation detail is not specified in this model. |
| NFR-08 Query performance | Frequent catalog, sales, and report queries must guide index selection; pending indexes must not be described as existing. | §§6.1–6.3. No response-time threshold is specified. |
| NFR-09 Monetary precision | Amounts must use two decimal places and keep `Money` rounding aligned with `numeric(18,2)`. | §2.2. |
| NFR-10 Verifiability | Schema documentation must be checkable against the verification queries described in §10. | §10. |

## 3. Constraints and exclusions

- There is no customer/buyer entity or buyer data (§§1 and 7).
- There is no currency column; the system is single-currency by design (§§1, 3, and D-05).
- Product description, SKU, and reference code are not included (DP-03, §1).
- Sale totals, subtotals, reports, and value objects are not stored in tables (§§1–2).
- `created_at` and `updated_at` columns are not included (§8).
- Atomicity between the database transaction and external image storage is not promised (§7.1).

## 4. Assumptions

- **ASSUMPTION-REQ-01:** an “internal operator” uses the application through an interface not described by the model.
- **ASSUMPTION-REQ-02:** catalog and sales operations are exposed through application use cases; endpoint names are not defined here.
- **ASSUMPTION-REQ-03:** criteria depending on fields marked as pending become enforceable only after the corresponding schema task is implemented.

## 5. Traceability

Each user story and NFR cites the model sections that support it. Unspecified needs are marked as assumptions rather than confirmed commitments.
