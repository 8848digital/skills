# Whitelisted APIs

## Preferred: Methods in DocType controllers

Place whitelisted methods in the controller file — either as Document class methods (doc-level) or as module-level functions (doctype-level). This avoids needing full dotted paths to call them.

### Doc-level methods (on a specific document)

```python
# apps/<app>/<app>/<module>/doctype/expense/expense.py

import frappe
from frappe.model.document import Document

class Expense(Document):
    @frappe.whitelist()
    def approve(self):
        self.status = "Approved"
        self.save()
        return self.status
```

Call from client JS:
```javascript
frappe.call({
    method: "approve",       // just the method name
    doc: frm.doc,
    callback(r) { console.log(r.message); }
});
// or
frm.call("approve");
```

Call from HTTP (v2 API):
```
POST /api/v2/document/Expense/EXP-0001/method/approve
```

### DocType-level functions (module root)

```python
# apps/<app>/<app>/<module>/doctype/expense/expense.py

import frappe

@frappe.whitelist()
def get_expense_summary(status=None):
    filters = {"status": status} if status else {}
    return frappe.db.get_all("Expense", filters=filters, fields=["name", "title", "amount"])
```

Call from client JS:
```javascript
frappe.call({
    method: "myapp.mymodule.doctype.expense.expense.get_expense_summary",
    args: { status: "Draft" },
    callback(r) { console.log(r.message); }
});
```

Call from HTTP:
```
POST /api/v2/method/Expense/get_expense_summary
```

## Standalone API files (for non-DocType logic)

Use only when logic doesn't belong to any DocType:

```python
# apps/<app>/<app>/api.py
import frappe

@frappe.whitelist()
def get_dashboard_data():
    return {"total": frappe.db.count("Expense")}
```

For larger apps, organize by feature:
```
apps/<app>/<app>/api/
    __init__.py
    expenses.py
    reports.py
```

### Doctype-scoped `api.py`

When an app needs custom whitelisted endpoints tied to one doctype but separate from the controller (e.g. a REST-style resource layer), place `api.py` inside that doctype's own folder — not in a shared `api/` directory.

```
apps/<app>/<app>/<module>/doctype/expense/
    expense.py         # controller (Document class, doc hooks)
    api.py              # thin whitelisted entry points only
    expense_utils.py    # business logic, called by api.py
```

- `api.py` holds only `@frappe.whitelist()` functions that validate input, call logic elsewhere, and return a response. No business logic lives here.
- Non-whitelisted helper/business-logic functions live in other files in the same doctype folder (e.g. `expense_utils.py`), not inside `api.py`.
- This keeps whitelisted surface area easy to audit — you can scan `api.py` and see every entry point without wading through implementation.

## Allow guest access

```python
@frappe.whitelist(allow_guest=True)
def public_endpoint():
    return {"message": "Hello"}
```

Without `allow_guest=True`, the endpoint requires authentication.

## Argument handling

- **Always add type hints** to whitelisted method parameters. Frappe validates and casts arguments based on type hints, preventing type-confusion attacks:
```python
@frappe.whitelist()
def create_expense(title: str, amount: float, tags: list | None = None):
    # title is guaranteed to be str, amount is cast to float
    # Without type hints, all args arrive as untrusted strings
    ...
```

- Use `frappe.form_dict` for raw request data:
```python
data = frappe.form_dict
```

## Return values

- Return a dict/list → auto-serialized to JSON under `{"message": <return_value>}`
- For custom HTTP responses:
```python
frappe.response["meta"] = meta
```

## API response standard

Every custom-built API (in `api.py`, `api/`, or versioned frontend endpoints) must return a consistent response shape rather than a raw dict, so frontend/client code can rely on one contract.

### Standard shape

```json
{
    "status": true,
    "status_code": 200,
    "message": "Success",
    "data": {},
    "errors": null
}
```

- `status` (bool, required) — success or failure
- `status_code` (int, required) — HTTP status code, must match the actual response status
- `message` (str, required) — human-readable, safe to show to a user
- `data` (object/array/null, required) — payload on success; `null` on failure
- `errors` (object/array/null, optional) — validation/business-rule error detail; `null` on success

### Examples

Success:
```json
{
    "status": true,
    "status_code": 200,
    "message": "Customer details fetched successfully",
    "data": { "customer": "CUST-0001", "customer_name": "ABC Pvt Ltd" },
    "errors": null
}
```

Validation error:
```json
{
    "status": false,
    "status_code": 400,
    "message": "Validation failed",
    "data": null,
    "errors": { "customer": "Customer is mandatory" }
}
```

### Status codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Record created |
| 204 | No content |
| 400 | Validation error |
| 401 | Authentication failed |
| 403 | Permission denied |
| 404 | Record not found |
| 409 | Duplicate record / conflict |
| 422 | Business rule validation failed |
| 429 | Too many requests |
| 500 | Internal server error |

### Rules

- Always return JSON with `status`, `status_code`, and `message` present.
- Never expose Python tracebacks or raw database errors — catch and translate into `message`/`errors`.
- `data` is populated only on success; leave it `null` on failure.
- `errors` carries validation/business-rule failure detail; leave it `null` on success.
- Keep `message` human-readable — it may be shown directly in a UI.
- The HTTP status actually returned must match `status_code` in the body.

### Helper

Define once per app and reuse everywhere a whitelisted endpoint returns:

```python
def api_response(status=True, code=200, message="", data=None, errors=None):
    return {
        "status": status,
        "status_code": code,
        "message": message,
        "data": data,
        "errors": errors,
    }
```

## Frontend API structure (versioned APIs)

For apps serving a dedicated frontend (e.g. React), version the API surface separately from internal doctype APIs, so frontend contracts can evolve independently of internal logic.

```
apps/<app>/<app>/api/
    v1.py           # main v1 controller — routes/re-exports v1 endpoints
    v1/
        __init__.py
        cart.py
        item_list.py
        sales_order.py
```

- Version from the start (`v1/`, `v2/`, ...) even if only one version exists — retrofitting versioning later breaks existing frontend clients.
- Keep the top-level version file (e.g. `v1.py`) as the entry controller; it stays thin and routes to resource files under `v1/`.
- Split endpoints by resource/feature (`cart.py`, `sales_order.py`), not by HTTP verb or a single catch-all file.
- When introducing `v2`, keep `v1` intact and functioning — don't break existing frontend clients still pointed at v1. Only remove a version after all clients have migrated.
- Every endpoint under `api/vN/` follows the API response standard above.

## Built-in document APIs (v2)

Frappe provides CRUD APIs automatically via `/api/v2/document/` — no need to write them. Requires **Frappe v15+**.

```
GET    /api/v2/document/<DocType>                          # list (with filters, fields, order_by, limit)
POST   /api/v2/document/<DocType>                          # create
GET    /api/v2/document/<DocType>/<name>/                  # read
PUT    /api/v2/document/<DocType>/<name>/                  # update
DELETE /api/v2/document/<DocType>/<name>/                  # delete
GET    /api/v2/document/<DocType>/<name>/copy              # copy doc
POST   /api/v2/document/<DocType>/<name>/method/<method>/  # call doc method
POST   /api/v2/method/<DocType>/<method>                   # call doctype level method
GET    /api/v2/doctype/<DocType>/meta                      # get DocType meta
GET    /api/v2/doctype/<DocType>/count                     # count records
```

### List query params
`fields` (JSON list), `filters` (JSON dict/list), `order_by`, `start`, `limit` (default 20), `group_by`.

Response includes `has_next_page` boolean for pagination.

### Bulk operations
```
POST /api/v2/document/<DocType>/bulk_delete   # body: {"names": [...]}
POST /api/v2/document/<DocType>/bulk_update   # body: {"docs": [{"name": "...", ...fields}]}
```

Large bulk operations (>20 items by default) are automatically enqueued as background jobs.

Only create custom `@frappe.whitelist()` endpoints for logic that goes beyond CRUD.

## Specify HTTP methods

Always declare allowed HTTP methods explicitly. Frappe auto-commits only for POST/PUT — GET requests do not commit.

```python
@frappe.whitelist(methods=["GET"])
def get_dashboard_data(): ...

@frappe.whitelist(methods=["POST"])
def submit_entry(name: str): ...

@frappe.whitelist(methods=["GET", "POST"])
def get_or_create_token(): ...
```

## Anti-patterns

- **Don't wrap doc methods in standalone APIs.** If the controller has `@frappe.whitelist()` on a method, clients call it directly via `frm.call("approve")` or `POST /api/v2/document/Expense/EXP-001/method/approve`. Don't create a separate `api.py` function that just fetches the doc and calls the same method.
- **Don't put doc-scoped logic in standalone APIs.** If the function fetches one doc, validates the caller, and acts on that doc — it belongs as a doc-level `@frappe.whitelist()` method, not in `api/`. Reserve standalone APIs for cross-document operations, aggregations, or endpoints with no document context.
- **Don't write business logic inside `api.py`.** `api.py` (whether app-level or doctype-scoped) should only contain thin whitelisted functions that validate, delegate, and format the response. Put actual logic in a sibling module and import it.
- **Don't return raw dicts or unstructured errors from custom endpoints.** Use the API response standard consistently so frontend code doesn't need per-endpoint special-casing.
- **Don't leak sensitive fields in guest APIs.** With `allow_guest=True`, only return fields guests need. Never expose `user` (email), internal IDs, or permission-sensitive data.