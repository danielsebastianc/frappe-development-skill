---
name: developing-frappe-apps
description: Covers developing custom Frappe/ERPNext applications with Controller-Service-Repository (CSR) architecture, DocType controllers, database queries, error handling, and testing. Use when building Frappe apps, implementing business logic, creating API endpoints, writing tests, or extending ERPNext.
---

# Developing Frappe Apps

## Overview

Reference guide for building custom Frappe applications with clean architecture patterns.

## When to Use

- Building custom DocType controllers
- Implementing CSR architecture (Controllers, Services, Repositories)
- Creating API endpoints with `@frappe.whitelist()`
- Querying Frappe database (get_doc, get_list, query builder)
- Implementing validations and business logic
- Handling errors with frappe.throw() and logging
- Writing tests with pytest and MagicMock
- Extending standard ERPNext DocTypes via overrides
- Hooking into document events

## Skill Files

| File | Purpose | When to Load |
|------|---------|--------------|
| [architecture-patterns.md](architecture-patterns.md) | CSR architecture, API controllers, services, repositories, overrides | Implementing business logic |
| [doctype-controllers.md](doctype-controllers.md) | DocType lifecycle, validation, flags, child tables | Building DocType controllers |
| [database-queries.md](database-queries.md) | Query patterns: get_doc, get_list, frappe.qb, filters | Database operations |
| [error-handling.md](error-handling.md) | Error handling: frappe.throw(), exceptions, logging | Validation and errors |
| [testing-patterns.md](testing-patterns.md) | Testing basics with pytest and MagicMock | Writing tests |
| [testing-advanced.md](testing-advanced.md) | Advanced testing scenarios | Complex tests |

## Workflows

### CSR Architecture Workflow

Copy this checklist when implementing CSR architecture:

```
CSR Implementation:
- [ ] Step 1: Create Repository with data access methods
- [ ] Step 2: Create Service with business logic
- [ ] Step 3: Create Controller with @frappe.whitelist()
- [ ] Step 4: Add consistent API response structure
- [ ] Step 5: Write tests with mocked dependencies
- [ ] Step 6: Update hooks.py if needed
```

**Step 1: Repository**
Create `repository/doctype_name/doctype_repository.py`:
```python
class MyRepository:
    @staticmethod
    def get_data(filters):
        return frappe.get_all("MyDocType", filters=filters)
```

**Step 2: Service**
Create `service/doctype_name/doctype_service.py`:
```python
class MyService:
    def __init__(self, doc):
        self.doc = doc
        self.repo = MyRepository()
    
    def process(self):
        # Business logic here
        pass
```

**Step 3: Controller**
Create `controller/doctype_name/doctype_controller.py`:
```python
@frappe.whitelist()
def api_endpoint(param):
    try:
        service = MyService(None)
        result = service.process()
        return {"success": True, "data": result}
    except Exception as e:
        frappe.log_error()
        return {"success": False, "error": str(e)}
```

**Step 4: API Response Structure**
Always return:
```python
return {
    "success": True/False,
    "message": "Description",
    "data": result_dict
}
```

**Step 5: Tests**
See [testing-patterns.md](testing-patterns.md).

### Creating New DocType Workflow

Copy this checklist when creating a new DocType:

```
New DocType:
- [ ] Step 1: Create directory structure
- [ ] Step 2: Define DocType in .json file
- [ ] Step 3: Create Document class in .py file
- [ ] Step 4: Add validation methods
- [ ] Step 5: Create service layer if complex logic
- [ ] Step 6: Add tests
- [ ] Step 7: Update hooks.py if events needed
```

**Step 1: Directory Structure**
```
doctype/
└── my_doctype/
    ├── __init__.py
    ├── my_doctype.json
    ├── my_doctype.py
    └── test_my_doctype.py
```

**Step 2: DocType JSON**
Define fields, permissions, and settings.

**Step 3: Document Class**
```python
from frappe.model.document import Document

class MyDocType(Document):
    def validate(self):
        # Validation logic
        pass
    
    def before_submit(self):
        # Pre-submit logic
        pass
```

**Step 4: Validation**
Use `validate()` method for all validations.

**Step 5-7:** Continue with service layer, tests, and hooks as needed.

## Quick Reference

| Operation | Method | Example |
|-----------|--------|---------|
| Get document | `frappe.get_doc()` | `doc = frappe.get_doc("User", "test@example.com")` |
| Get value | `frappe.db.get_value()` | `email = frappe.db.get_value("User", name, "email")` |
| Check exists | `frappe.db.exists()` | `if frappe.db.exists("User", email):` |
| Throw error | `frappe.throw()` | `frappe.throw(_("Error"))` |
| Log error | `frappe.log_error()` | `frappe.log_error(title="Error")` |
| Set value | `self.db_set()` | `self.db_set("status", "Active")` |

## Best Practices

1. **Use `validate()` for validations** - Not before_save or on_update
2. **Use `_()` for user messages** - Enables translation
3. **Use `get_list()` not `get_all()`** - Unless Administrator
4. **Use `db_set()` for silent updates** - Bypasses validation
5. **Check `has_value_changed()`** - Before triggering actions
6. **Use specific exception types** - Not generic ValidationError
7. **Clean up test fixtures** - Reset mocks after tests
8. **Separate concerns with CSR** - Controllers/Services/Repositories
9. **Consistent API responses** - Use `{success, message, data}`
10. **Patch at right location** - Where function uses it, not where defined
