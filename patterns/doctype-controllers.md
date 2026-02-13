# DocType Controller Patterns

Use when building or modifying DocType controllers, implementing business logic, validations, or document lifecycle hooks.

## Document Lifecycle Methods

```python
from frappe.model.document import Document
from frappe import _

class MyDoctype(Document):
    def before_insert(self):
        """Before inserting - set initial values"""
        pass
    
    def validate(self):
        """Main validation - ALWAYS use this"""
        if not self.required_field:
            frappe.throw(_("Field is required"))
    
    def before_save(self):
        """Before saving - pre-save logic"""
        pass
    
    def on_update(self):
        """After update - check field changes"""
        if self.has_value_changed("status"):
            self.notify_status_change()
    
    def before_submit(self):
        """Before submitting"""
        pass
    
    def on_submit(self):
        """After submitting"""
        pass
    
    def before_cancel(self):
        """Before cancelling"""
        pass
    
    def on_trash(self):
        """Before deleting - cleanup"""
        pass
```

## Common Patterns

### Child Table Operations

```python
def validate(self):
    # Iterate child table
    for item in self.items:
        item.amount = item.qty * item.rate
    
    # Add row
    self.append("items", {
        "item_code": "ITEM-001",
        "qty": 5,
        "rate": 100
    })
```

### Field Change Detection

```python
def on_update(self):
    if self.has_value_changed("status"):
        # Status changed - trigger action
        self.create_status_log()
```

### Silent Updates

```python
def update_status(self, status):
    """Update without triggering validate"""
    self.db_set("status", status, update_modified=True)
```

### Permission Checks

```python
def validate(self):
    self.check_permission("write")
```

## Common Mistakes

**Wrong: Validating in wrong method**
```python
def before_save(self):
    if not self.email:  # Don't validate here
        frappe.throw("Email required")

def validate(self):  # Use this instead
    if not self.email:
        frappe.throw("Email required")
```
