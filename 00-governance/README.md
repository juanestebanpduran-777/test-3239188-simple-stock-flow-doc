# 00 — Documentation Governance

## 1. Purpose

This folder defines the documentation principles used to reconstruct the **Simple Stock Flow** system from the supplied data model.

The purpose of this evaluation is to demonstrate Specification-Driven Development (SDD): understanding an existing specification, deriving consistent documentation from it, and maintaining traceability between the source and every documented decision.

## 2. Source of truth

The authoritative input for this exercise is [`spec/data-model.md`](../spec/data-model.md).

All documentation in this repository must be derived from that file. References to source sections, entities, columns, constraints, methods, and task identifiers must be precise enough to support verification.

If the source does not establish a fact, it must not be presented as confirmed.

Use the following classifications:

- **Confirmed:** explicitly described by the source.
- **Domain rule:** enforced by domain behavior but not necessarily by the database.
- **Database constraint:** currently enforced by PostgreSQL.
- **Pending:** explicitly identified as unfinished in the source.
- **Assumption:** a reasonable interpretation that the source does not confirm. State it explicitly.

## 3. Documentation sequence

The evaluation requires the following reconstruction sequence:

1. `05-architecture/`: infer the architecture and responsibilities from the data model.
2. `04-requirements/`: derive functional and non-functional requirements.
3. `03-product/`: define the product problem and vision.
4. `02-domain/`: document the domain language, entities, rules, and relevant events.
5. `01-context/`: describe the system boundary and overall context.
6. `05-architecture/`: review the architecture against the other documents and the source.

The folders are numbered for navigation, but the analysis must follow the sequence above.

## 4. Traceability rules

Every important statement must be traceable to an identifiable source element.

Acceptable references include:

- A section of `spec/data-model.md`, such as `§2.2`.
- A database object, such as `ck_product_stock_non_negative`.
- A domain method, such as `Sale.EnsureConfirmable`.
- A task identifier, such as `T-12` or `T-20`.

Do not claim that a pending task has been completed unless the source explicitly confirms it.

Do not treat a domain rule as a database constraint. These enforcement mechanisms have different guarantees.

## 5. Scope restrictions

The documentation must reflect the supplied model, not an unrelated project or a hypothetical future system.

The model describes five tables: `category`, `product`, `sale`, `sale_item`, and `user`, within the PostgreSQL `sales` schema.

Do not introduce unsupported features such as customer accounts, product SKUs, product descriptions, seller-based reporting, category CRUD, microservices, HTTP endpoints, or message brokers as confirmed capabilities.

Possible future improvements may be mentioned only when clearly identified as proposals or unresolved decisions.

## 6. Language and consistency

All documentation in this evaluation is written in English.

Use the source's technical identifiers exactly as written. Database table and column names are case-sensitive in documentation, even where PostgreSQL itself may handle identifiers differently.

Use consistent terminology across context, product, domain, requirements, and architecture.

## 7. Data-model exclusion

Do not create a `06-data/` folder or duplicate the supplied data model. The source file already serves as the data-model specification.

## 8. Completion criteria

The documentation is ready for submission when:

- All required folders from `00-governance/` through `05-architecture/` contain their intended `README.md`.
- The documents consistently describe Simple Stock Flow.
- Every significant claim is traceable or explicitly marked as an assumption.
- Pending constraints and relationships remain pending where the source says they are pending.
- No unsupported architecture or product features are stated as facts.
- The architecture has been reviewed against the other documents and the source model.
- The final changes are committed to the evaluation fork before the deadline.
