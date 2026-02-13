# Error Handling Patterns

Use when implementing validation logic, handling errors, logging exceptions, or displaying messages.

## frappe.throw()

### Basic Usage

```python
from frappe import _

frappe.throw(_("Email is mandatory"))
frappe.throw(_("User not found"), frappe.DoesNotExistError)
frappe.throw(_("Invalid"), title=_("Error"))
```

### Multiple Errors

```python
def validate(self):
    errors = []
    
    if not self.start_date:
        errors.append(_("Start Date required"))
    if not self.end_date:
        errors.append(_("End Date required"))
    
    if errors:
        frappe.throw("<br>".join(errors), frappe.ValidationError)
```

## Exception Types

```python
frappe.ValidationError          # General validation
frappe.DoesNotExistError        # Document not found
frappe.PermissionError          # Permission denied
frappe.MandatoryError           # Mandatory field missing
frappe.LinkValidationError      # Invalid link field
frappe.DuplicateEntryError      # Duplicate entry
```

## Error Logging

```python
try:
    risky_operation()
except Exception:
    frappe.log_error(
        message=frappe.get_traceback(),
        title=_("Operation Failed")
    )
    frappe.throw(_("Operation failed. Please try again."))
```

## User Messages

```python
# Simple message
frappe.msgprint(_("Document saved"))

# With indicator
frappe.msgprint(_("Success"), indicator="green")
frappe.msgprint(_("Warning"), indicator="orange")

# With title
frappe.msgprint(_("Processed"), title=_("Done"), indicator="green")
```

## Best Practices

1. **Always use `_()` for translation**
2. **Use specific exception types**
3. **Log before throwing for internal errors**
4. **Don't expose internal details to users**
5. **Collect multiple errors when possible**
