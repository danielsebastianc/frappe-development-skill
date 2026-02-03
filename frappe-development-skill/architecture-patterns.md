# Architecture Patterns

Use when implementing business logic with Controller-Service-Repository (CSR) architecture, API endpoints, or extending standard ERPNext DocTypes.

## CSR Architecture Overview

The CSR pattern separates concerns into three layers:
- **Controller**: API entry points, request/response handling
- **Service**: Business logic, orchestration
- **Repository**: Database operations, Frappe document CRUD

### Directory Structure

```
app/
├── controller/
│   └── doctype_name/
│       └── doctype_controller.py
├── service/
│   └── doctype_name/
│       └── doctype_service.py
└── repository/
    └── doctype_name/
        └── doctype_repository.py
```

## Controller Patterns

### API Endpoint with Validation

```python
# controller/my_controller.py
import frappe
from app.service.my_service import MyService

@frappe.whitelist()
def api_endpoint(param):
    try:
        if not param:
            return {"success": False, "message": "Param required"}
        
        service = MyService()
        result = service.process(param)
        
        return {
            "success": True,
            "message": "Success",
            "data": result
        }
    except Exception as e:
        frappe.log_error(title="API Error", message=frappe.get_traceback())
        return {"success": False, "message": str(e)}
```

## Service Patterns

### Service with Doc Reference

```python
# service/my_service.py
import frappe
from app.repository.my_repository import MyRepository

class MyService:
    def __init__(self, doc):
        self.doc = doc
        self.repository = MyRepository()
    
    def process(self):
        data = self.repository.get_data()
        return self._transform(data)
    
    def _transform(self, data):
        # Private helper method
        return data
```

### Service with Multiple Repositories

```python
class OrderService:
    def __init__(self):
        self.order_repo = OrderRepository()
        self.customer_repo = CustomerRepository()
    
    def process_order(self, order_id):
        order = self.order_repo.get(order_id)
        customer = self.customer_repo.get(order.customer_id)
        # Business logic here
        return result
```

## Repository Patterns

### Static Method Repository

```python
# repository/my_repository.py
import frappe

class MyRepository:
    @staticmethod
    def get_data(filters=None):
        return frappe.get_all("MyDocType", filters=filters, fields=["*"])
    
    @staticmethod
    def update_status(name, status):
        frappe.db.set_value("MyDocType", name, "status", status)
```

### Repository with Complex Queries

```python
class OrderRepository:
    @staticmethod
    def get_orders_with_items(customer):
        """Complex query using SQL"""
        query = """
            SELECT o.name, o.total, i.item_code, i.qty
            FROM `tabSales Order` o
            JOIN `tabSales Order Item` i ON o.name = i.parent
            WHERE o.customer = %s AND o.docstatus = 1
        """
        return frappe.db.sql(query, (customer,), as_dict=True)
```

## DocType Override Patterns

### Extending Standard Classes

```python
# overrides/sales_invoice.py
from erpnext.accounts.doctype.sales_invoice.sales_invoice import SalesInvoice

class CustomSalesInvoice(SalesInvoice):
    def validate(self):
        super().validate()
        # Additional validation
        self._validate_custom_fields()
    
    def _validate_custom_fields(self):
        if self.custom_field and self.custom_field < 0:
            frappe.throw("Custom field cannot be negative")
```

### hooks.py Configuration

```python
# hooks.py
override_doctype_class = {
    "Sales Invoice": "app.overrides.sales_invoice.CustomSalesInvoice",
}

doc_events = {
    "Sales Invoice": {
        "on_submit": "app.doc_events.sales_invoice.on_submit_handler",
    }
}
```

## Best Practices

1. **Keep Controllers Thin** - Delegate to services
2. **Services Handle Business Logic** - Coordinate repositories
3. **Repositories Handle Data** - Database operations only
4. **Consistent API Responses** - Always use `{success, message, data}`
5. **Error Handling** - Log in controller, return safe messages
6. **Static Methods in Repositories** - Unless instance state needed

## Quick Reference

| Layer | Responsibility | Key Patterns |
|-------|---------------|--------------|
| **Controller** | API endpoints | `@frappe.whitelist()`, error handling |
| **Service** | Business logic | `__init__(self, doc)`, private helpers |
| **Repository** | Data access | `@staticmethod`, SQL/ORM queries |
