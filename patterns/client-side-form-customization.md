# Client-Side Form Customization

Use when dynamically setting child table field properties based on parent document state.

## Pattern: Child Table Field Properties

**Correct: Using `frappe.meta.get_docfield()`**
```javascript
frappe.ui.form.on('Sales Order', {
    refresh: function(frm) {
        var read_only = 0;
        if (frm.doc.custom_sales_order_type === "Retail") {
            read_only = 1;
        } else if (frm.doc.custom_sales_order_type === "Project") {
            read_only = frm.doc.docstatus === 1 ? 1 : 0;
        }
        
        // Set property for each row in the child table
        $.each(frm.doc.items || [], function(i, row) {
            frappe.meta.get_docfield('Sales Order Item', 'warehouse', row.name).read_only = read_only;
        });
        
        // Refresh the grid to apply changes
        frm.fields_dict.items.grid.refresh();
    },
    custom_sales_order_type: function(frm) {
        frm.trigger('refresh'); // Re-apply when type changes
    }
});
```

**Key points:**
- Use `frappe.meta.get_docfield(doctype, fieldname, docname)` to get the field definition
- Iterate through each row in the child table (`frm.doc.items`)
- Call `grid.refresh()` after setting properties to apply changes
- Hook into both `refresh` and relevant field change events

## Alternative: Grid `fields_map`

```javascript
var grid = frm.fields_dict.items.grid;
if (grid && grid.fields_map && grid.fields_map.warehouse) {
    grid.fields_map.warehouse.read_only = 1;
}
frm.refresh_field('items');
```

## Server-Side Enforcement (doc_events)

Use server-side validation for stricter enforcement.

```python
doc_events = {
    "Sales Order": {
        "before_save": "module.path.to.validate_warehouse"
    }
}

def validate_warehouse(doc, method):
    if doc.custom_sales_order_type == "Retail":
        for item in doc.items:
            # Prevent warehouse changes or set default
            pass
```

## Common Mistakes

- Setting child field properties without refreshing the grid or field
- Updating parent field state without reapplying to each child row
