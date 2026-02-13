# Database Query Patterns

Use when reading or writing data to the Frappe database.

## Get Single Document

```python
# Get by name
doc = frappe.get_doc("User", "test@example.com")

# Get by filters
doc = frappe.get_doc("User", {"email": "test@example.com"})

# Create new
doc = frappe.new_doc("User")
doc.first_name = "John"
doc.insert()
```

## Query Multiple Documents

### get_list() - Respects Permissions

```python
users = frappe.get_list("User",
    filters={"enabled": 1},
    fields=["name", "email"],
    order_by="creation desc",
    limit=10
)
```

### get_value() - Single Values

```python
# Single value
email = frappe.db.get_value("User", "test", "email")

# Multiple values
first_name, email = frappe.db.get_value("User", "test", ["first_name", "email"])

# As dict
user = frappe.db.get_value("User", "test", "*", as_dict=True)
```

### exists() and count()

```python
if frappe.db.exists("User", "test@example.com"):
    pass

count = frappe.db.count("User", filters={"enabled": 1})
```

## Query Builder

### Basic Select

```python
from frappe.query_builder import DocType

User = DocType("User")
results = (frappe.qb.from_(User)
    .select(User.name, User.email)
    .where(User.enabled == 1)
    .run()
)
```

### Joins

```python
User = DocType("User")
Role = DocType("Has Role")

results = (frappe.qb.from_(User)
    .join(Role).on(User.name == Role.parent)
    .select(User.name, Role.role)
    .where(Role.role == "System Manager")
    .run()
)
```

### Aggregations

```python
from frappe.query_builder.functions import Count, Sum

# Count
count = (frappe.qb.from_(User)
    .select(Count("*"))
    .where(User.enabled == 1)
    .run(pluck=True)
)[0]

# Sum
total = (frappe.qb.from_(Invoice)
    .select(Sum(Invoice.grand_total))
    .where(Invoice.docstatus == 1)
    .run()
)
```

## Direct SQL

```python
# Use when query builder is insufficient
results = frappe.db.sql("""
    SELECT name, email 
    FROM `tabUser` 
    WHERE enabled = %s
"", (1,), as_dict=True)
```

## Filter Operators

```python
filters = {
    "field": "value",                 # Equals
    "field": ("!=", "value"),         # Not equals
    "field": ("in", ["a", "b"]),      # In list
    "field": ("like", "%value%"),     # Like
    "field": (">", 10),               # Greater than
    "field": ("between", [1, 10]),    # Between
}
```

## Performance Tips

- Limit results: `limit=100`
- Select specific fields, not `*`
- Use `exists()` instead of `get_doc` for checks
- Use query builder for complex queries
