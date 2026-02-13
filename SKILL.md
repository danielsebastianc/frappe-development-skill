---
name: developing-frappe-apps
description: Build custom Frappe/ERPNext applications with proper architecture patterns. Use when creating DocTypes, implementing CSR (Controller-Service-Repository) architecture, writing business logic, querying Frappe database, handling errors, or writing tests. Triggers on phrases like "create DocType", "Frappe API", "ERPNext custom app", "validate document", "Frappe controller", "Frappe test".
license: MIT
compatibility: Requires Frappe/ERPNext framework environment with Python, bench CLI, and frappe app dependencies. Compatible with Frappe v14+ and ERPNext v14+.
metadata:
  author: OpenCode Community
  version: 2.0.0
  category: backend-development
  tags: [frappe, erpnext, python, doctype, testing]
  documentation: https://docs.frappe.io/framework/user/en
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

## Common Mistakes

### Wrong: Validating in `before_save()` instead of `validate()`
```python
# WRONG - Don't validate in before_save
def before_save(self):
    if not self.email:
        frappe.throw("Email required")

# CORRECT - Always validate in validate()
def validate(self):
    if not self.email:
        frappe.throw(_("Email is required"))
```

### Wrong: Using `get_all()` without permission checks
```python
# WRONG - Bypasses permissions
data = frappe.get_all("Customer", filters={"territory": "India"})

# CORRECT - Respects permissions
data = frappe.get_list("Customer", filters={"territory": "India"})
```

### Wrong: Mutating document after save
```python
# WRONG - Changes won't persist
def on_submit(self):
    self.status = "Processing"  # Too late!

# CORRECT - Use before_submit
def before_submit(self):
    self.status = "Processing"  # Will be saved with submission
```

## Troubleshooting

### DocType Not Found
**Symptom:** `frappe.exceptions.DoesNotExistError: DocType MyDocType not found`

**Cause:** 
- DocType not installed in site
- Missing `bench migrate` after creating DocType
- Typo in DocType name

**Solution:**
1. Run `bench migrate` to create tables
2. Check DocType exists: `frappe.db.exists("DocType", "MyDocType")`
3. Verify installed apps: `bench --site [site] list-apps`

### Permission Denied
**Symptom:** `frappe.exceptions.PermissionError: Not permitted`

**Cause:**
- User lacks role permissions
- DocType permissions not configured
- Server script trying to access restricted data

**Solution:**
1. Check role permissions in DocType settings
2. Use `ignore_permissions=True` for server-side operations only
3. Verify user has required roles

### Validation Errors
**Symptom:** `frappe.exceptions.ValidationError: [message]`

**Cause:**
- Missing required fields
- Business logic validation failed
- Custom validation in `validate()` method

**Solution:**
1. Check all mandatory fields are populated
2. Review `validate()` method in DocType controller
3. Check field types and formats (dates, numbers, etc.)

### API Not Responding
**Symptom:** `@frappe.whitelist()` endpoint returns 403 or 500

**Cause:**
- Missing `@frappe.whitelist()` decorator
- Permission check failing
- Exception in controller code

**Solution:**
1. Ensure method has `@frappe.whitelist()` decorator
2. Check user has "Desk Access" permission
3. Review `frappe.log_error()` output for stack trace

### Tests Not Running
**Symptom:** `pytest` can't find or run Frappe tests

**Cause:**
- Not using Frappe test runner
- Missing test fixtures
- Database not initialized

**Solution:**
1. Run with bench: `bench --site [site] run-tests --app [app]`
2. Ensure `test_records` is defined in test file
3. Check `bench doctor` for database connectivity

## Validation Checklist

Before deploying Frappe code to production:

### DocType Controllers
- [ ] All validations in `validate()` method, not `before_save()`
- [ ] Translation function `_()` used for all user-facing messages
- [ ] Child table operations handle edge cases (empty tables, large datasets)
- [ ] Field change detection uses `has_value_changed()`
- [ ] Silent updates use `db_set()` to bypass validation loops

### CSR Architecture
- [ ] Controllers are thin - only handle request/response
- [ ] Services contain business logic, not data access
- [ ] Repositories handle all database operations
- [ ] API responses follow consistent `{success, message, data}` format
- [ ] All API endpoints have `@frappe.whitelist()` decorator

### Database Queries
- [ ] Use `frappe.get_list()` instead of `get_all()` (unless admin)
- [ ] Complex queries use `frappe.qb` or raw SQL with proper escaping
- [ ] Check document existence before operations
- [ ] Use `db.get_value()` for single value retrieval
- [ ] Handle empty result sets gracefully

### Error Handling
- [ ] Use specific exception types (ValidationError, PermissionError)
- [ ] Log errors with `frappe.log_error()` for debugging
- [ ] Return safe error messages to users (no stack traces)
- [ ] Handle MCP/external API failures gracefully
- [ ] Validate inputs before processing

### Testing
- [ ] All service methods have unit tests
- [ ] DocType controllers have integration tests
- [ ] Mock external dependencies (database, APIs)
- [ ] Test fixtures are cleaned up after tests
- [ ] Edge cases and error scenarios are covered
