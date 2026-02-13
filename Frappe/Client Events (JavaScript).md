# Complete Guide to Frappe Client Events (JavaScript)

## Table of Contents

1. [What Are Client Events?](#1-what-are-client-events)
2. [Where to Write Client Scripts](#2-where-to-write-client-scripts)
3. [Complete Event Reference](#3-complete-event-reference)
4. [Field-Level Events](#4-field-level-events)
5. [Child Table Events](#5-child-table-events)
6. [Form Object (frm) Deep Dive](#6-form-object-frm-deep-dive)
7. [Advanced Techniques](#7-advanced-techniques)
8. [Real-World Examples](#8-real-world-examples)
9. [Performance & Best Practices](#9-performance--best-practices)
10. [Debugging & Troubleshooting](#10-debugging--troubleshooting)

---

## 1. What Are Client Events?

**Client Events** are JavaScript functions that execute in the browser when specific actions occur on a Frappe form (DocType).

### Key Characteristics:

- Run **client-side** (in user's browser)
- Execute **instantly** (no server delay)
- Can call server methods via `frappe.call()`
- Used for: validation, auto-fill, calculations, UI changes

### Event Execution Flow:

```
Page Load → before_load → onload → refresh → User Action → Field Events → validate → before_save → Save
```

---
## 2. The Master Syntax

```javascript
frappe.ui.form.on('Your DocType', {
    // Page load / Form load events
    refresh(frm) { },

    onload(frm) { },

    onload_post_render(frm) { },

    // Field-specific events
    fieldname(frm) { },

    another_field(frm) { },

    // Validation & Save events
    validate(frm) { },

    before_save(frm) { },
    on_submit(frm) { },
    after_save(frm) { },

    before_submit(frm) { },
    post_submit(frm) { },   // v15+

    // Cancel & Workflow
    before_cancel(frm) { },
    on_cancel(frm) { },

    // Workflow state change
    workflow_state(frm) { },

    // Timeline / Communication
    timeline_refresh(frm) { },

    // Special events
    setup(frm) { },                 // runs once when form is created
    render(frm) { },                // v15+ – after render
});
```

## 2. Where to Write Client Scripts

### Method 1: Custom Script (Recommended for Customization)

```
Desk → Customization → Custom Script → New
```

**Settings:**
- **DocType**: Select your DocType (e.g., Sales Order)
- **Script Type**: Client
- **Script**: Write your JavaScript

**Example:**

```javascript
frappe.ui.form.on('Sales Order', {
    refresh: function(frm) {
        console.log('Form loaded');
    }
});
```

### Method 2: DocType JS File (For App Development)

Create file in your app:

```
apps/your_app/your_app/your_module/doctype/your_doctype/your_doctype.js
```

**Example:**

```javascript
frappe.ui.form.on('Sales Order', {
    refresh(frm) {
        // Your code
    },
    customer(frm) {
        // When customer changes
    }
});
```

### Method 3: Inline in Form (Not Recommended)

Only for quick testing - doesn't persist.

---

## 3. Complete Event Reference

### A. Form Lifecycle Events

#### `before_load`

- **When**: Very first event, before any data is loaded
- **Use**: Initialize variables, set up listeners
- **Fires**: Once per form instance

```javascript
frappe.ui.form.on('Sales Order', {
    before_load: function(frm) {
        console.log('Before load - no data yet');
        // frm.doc may be empty
    }
});
```

---

#### `onload`

- **When**: After data is fetched, before rendering
- **Use**: Initial setup based on document data
- **Fires**: Once when form opens

```javascript
frappe.ui.form.on('Sales Order', {
    onload: function(frm) {
        if (frm.is_new()) {
            frm.set_value('transaction_date', frappe.datetime.get_today());
        }
        // Set query filters
        frm.set_query('customer', function() {
            return {
                filters: { 'customer_group': 'Commercial' }
            };
        });
    }
});
```

---

#### `refresh`

- **When**: After form renders, and on every refresh
- **Use**: Add buttons, set field properties, update UI
- **Fires**: Multiple times (after save, after field changes, etc.)

```javascript
frappe.ui.form.on('Sales Order', {
    refresh: function(frm) {
        // Most commonly used event
        // Hide field
        frm.set_df_property('custom_field', 'hidden', 1);
        // Make field read-only
        frm.set_df_property('customer', 'read_only', frm.doc.docstatus == 1);
        // Add custom button
        if (frm.doc.docstatus === 1) {
            frm.add_custom_button(__('Create Invoice'), function() {
                // Create invoice logic
            });
        }
        // Change field label
        frm.set_df_property('custom_field', 'label', 'New Label');
        // Set field description
        frm.set_df_property('total', 'description', 'Total amount including taxes');
        // Toggle field requirement
        frm.toggle_reqd('delivery_date', frm.doc.customer ? true : false);
    }
});
```

**Important:** Avoid expensive operations in `refresh` as it fires frequently.

---

#### `onload_post_render`

- **When**: After form is fully rendered
- **Use**: DOM manipulations, complex UI setup
- **Fires**: Once after complete rendering

```javascript
frappe.ui.form.on('Sales Order', {
    onload_post_render: function(frm) {
        // Access DOM elements safely
        frm.$wrapper.find('.custom-selector').on('click', function() {
            // Handle click
        });
    }
});
```

---

#### `validate`

- **When**: Before save, on both client and server
- **Use**: Data validation, calculations
- **Fires**: Every save attempt
- **Can block save**: Set `frappe.validated = false`

```javascript
frappe.ui.form.on('Sales Order', {
    validate: function(frm) {
        // Validation logic
        if (!frm.doc.items || frm.doc.items.length === 0) {
            frappe.msgprint(__('Please add at least one item'));
            frappe.validated = false;
            return;
        }
        // Calculate totals
        let total = 0;
        frm.doc.items.forEach(function(item) {
            total += item.amount;
        });
        frm.set_value('total', total);
        // Advanced validation
        if (frm.doc.delivery_date < frm.doc.transaction_date) {
            frappe.throw(__('Delivery date cannot be before transaction date'));
        }
    }
});
```

---

#### `before_save`

- **When**: Just before save request is sent to server
- **Use**: Last-minute calculations, confirmations
- **Fires**: Every save

```javascript
frappe.ui.form.on('Sales Order', {
    before_save: function(frm) {
        // Last chance to modify data before save
        frm.doc.modified_by_script = frappe.session.user;
        // Async operations - return promise
        return new Promise((resolve, reject) => {
            frappe.call({
                method: 'your_app.api.check_credit_limit',
                args: { customer: frm.doc.customer },
                callback: function(r) {
                    if (r.message.exceeded) {
                        frappe.msgprint('Credit limit exceeded!');
                        frappe.validated = false;
                        reject();
                    } else {
                        resolve();
                    }
                }
            });
        });
    }
});
```

---

#### `after_save`

- **When**: After successful save
- **Use**: Show messages, refresh other forms, analytics
- **Fires**: After each save

```javascript
frappe.ui.form.on('Sales Order', {
    after_save: function(frm) {
        frappe.show_alert({
            message: __('Order Saved Successfully'),
            indicator: 'green'
        }, 5);
        // Track in analytics
        if (window.gtag) {
            gtag('event', 'order_saved', {
                'value': frm.doc.grand_total
            });
        }
    }
});
```

---

#### `on_submit`

- **When**: Before document is submitted
- **Use**: Validation before submission
- **Fires**: On submit button click

```javascript
frappe.ui.form.on('Sales Order', {
    on_submit: function(frm) {
        // Confirm before submit
        if (frm.doc.grand_total > 100000) {
            frappe.confirm(
                'Order value is high. Are you sure you want to submit?',
                function() {
                    // Continue submission
                },
                function() {
                    frappe.validated = false;
                }
            );
        }
    }
});
```

---

#### `before_submit`

- **When**: Just before actual submission
- **Use**: Final checks
- **Fires**: During submission process

```javascript
frappe.ui.form.on('Sales Order', {
    before_submit: function(frm) {
        // Check stock availability
        if (!frm.doc.delivery_date) {
            frappe.throw(__('Delivery date is mandatory for submission'));
        }
    }
});
```

---

#### `after_cancel`

- **When**: After document is cancelled
- **Use**: Cleanup, notifications
- **Fires**: After cancellation

```javascript
frappe.ui.form.on('Sales Order', {
    after_cancel: function(frm) {
        frappe.show_alert({
            message: __('Order Cancelled'),
            indicator: 'red'
        });
        // Send notification
        frappe.call({
            method: 'frappe.desk.doctype.notification_log.notification_log.make_notification_logs',
            args: {
                subject: 'Order Cancelled: ' + frm.doc.name,
                for_user: frm.doc.owner
            }
        });
    }
});
```

---

#### `timeline_refresh`

- **When**: When timeline/comments section refreshes
- **Use**: Custom timeline modifications
- **Fires**: On timeline updates

```javascript
frappe.ui.form.on('Sales Order', {
    timeline_refresh: function(frm) {
        // Add custom timeline entry
        console.log('Timeline refreshed');
    }
});
```

---

## 4. Field-Level Events

### Basic Field Change Event

```javascript
frappe.ui.form.on('Sales Order', {
    customer: function(frm) {
        // Triggered when customer field changes
        if (frm.doc.customer) {
            // Fetch customer details
            frappe.call({
                method: 'frappe.client.get',
                args: {
                    doctype: 'Customer',
                    name: frm.doc.customer
                },
                callback: function(r) {
                    if (r.message) {
                        frm.set_value('customer_name', r.message.customer_name);
                        frm.set_value('territory', r.message.territory);
                    }
                }
            });
        } else {
            // Clear dependent fields
            frm.set_value('customer_name', '');
            frm.set_value('territory', '');
        }
    },
    delivery_date: function(frm) {
        // Validate delivery date
        if (frm.doc.delivery_date < frappe.datetime.get_today()) {
            frappe.msgprint(__('Delivery date cannot be in the past'));
            frm.set_value('delivery_date', '');
        }
    }
});
```

### Multiple Fields with Same Logic

```javascript
frappe.ui.form.on('Sales Order', {
    setup: function(frm) {
        // Setup runs once before onload
        ['discount_percentage', 'tax_rate'].forEach(function(field) {
            frm.fields_dict[field].$input.on('change', function() {
                frm.trigger('calculate_total');
            });
        });
    },
    calculate_total: function(frm) {
        // Shared calculation logic
        let total = 0;
        // ... calculation
        frm.set_value('grand_total', total);
    }
});
```

### Conditional Field Events

```javascript
frappe.ui.form.on('Sales Order', {
    payment_terms_template: function(frm) {
        if (frm.doc.payment_terms_template) {
            // Fetch and populate payment schedule
            frm.call({
                method: 'get_payment_schedule',
                doc: frm.doc,
                callback: function(r) {
                    frm.refresh_field('payment_schedule');
                }
            });
        }
    }
});
```

---

## 5. Child Table Events

### Child Table Structure

```javascript
// Parent DocType: Sales Order
// Child DocType: Sales Order Item
// Table Field: items
frappe.ui.form.on('Sales Order', {
    // Parent events
});
frappe.ui.form.on('Sales Order Item', {
    // Child table events
});
```

---

### A. Row-Level Events

#### Field Change in Child Table

```javascript
frappe.ui.form.on('Sales Order Item', {
    item_code: function(frm, cdt, cdn) {
        // cdt = child doctype name
        // cdn = child doc name (row identifier)
        let row = locals[cdt][cdn];
        if (row.item_code) {
            frappe.call({
                method: 'erpnext.stock.get_item_details.get_item_details',
                args: {
                    item_code: row.item_code,
                    company: frm.doc.company
                },
                callback: function(r) {
                    if (r.message) {
                        frappe.model.set_value(cdt, cdn, 'item_name', r.message.item_name);
                        frappe.model.set_value(cdt, cdn, 'rate', r.message.price_list_rate);
                        frappe.model.set_value(cdt, cdn, 'uom', r.message.stock_uom);
                    }
                }
            });
        }
    },
    qty: function(frm, cdt, cdn) {
        calculate_item_amount(frm, cdt, cdn);
    },
    rate: function(frm, cdt, cdn) {
        calculate_item_amount(frm, cdt, cdn);
    },
    discount_percentage: function(frm, cdt, cdn) {
        calculate_item_amount(frm, cdt, cdn);
    }
});

function calculate_item_amount(frm, cdt, cdn) {
    let row = locals[cdt][cdn];
    let amount = row.qty * row.rate;
    if (row.discount_percentage) {
        amount = amount * (1 - row.discount_percentage / 100);
    }
    frappe.model.set_value(cdt, cdn, 'amount', amount);
    // Recalculate parent total
    frm.trigger('calculate_totals');
}
```

---

#### Adding Rows

```javascript
frappe.ui.form.on('Sales Order Item', {
    items_add: function(frm, cdt, cdn) {
        // Triggered when new row is added
        let row = locals[cdt][cdn];
        // Set default values for new row
        frappe.model.set_value(cdt, cdn, 'qty', 1);
        frappe.model.set_value(cdt, cdn, 'warehouse', frm.doc.set_warehouse);
        console.log('New row added:', cdn);
    }
});
```

---

#### Removing Rows

```javascript
frappe.ui.form.on('Sales Order Item', {
    items_remove: function(frm, cdt, cdn) {
        // Triggered after row is removed
        frm.trigger('calculate_totals');
    },
    before_items_remove: function(frm, cdt, cdn) {
        // Triggered before row removal
        let row = locals[cdt][cdn];
        if (row.delivered_qty > 0) {
            frappe.msgprint(__('Cannot remove item that has been delivered'));
            frappe.validated = false;
        }
    }
});
```

---

### B. Table-Level Events

```javascript
frappe.ui.form.on('Sales Order', {
    items_move: function(frm, cdt, cdn, old_idx, new_idx) {
        // When rows are reordered
        console.log(`Row moved from ${old_idx} to ${new_idx}`);
    }
});
```

---

### C. Advanced Child Table Manipulation

#### Programmatically Add Rows

```javascript
frappe.ui.form.on('Sales Order', {
    add_multiple_items: function(frm) {
        let items = ['ITEM-001', 'ITEM-002', 'ITEM-003'];
        items.forEach(function(item_code) {
            let row = frm.add_child('items');
            row.item_code = item_code;
            row.qty = 1;
        });
        frm.refresh_field('items');
    }
});
```

#### Clear All Rows

```javascript
frappe.ui.form.on('Sales Order', {
    clear_items: function(frm) {
        frm.clear_table('items');
        frm.refresh_field('items');
    }
});
```

#### Filter Child Table Grid

```javascript
frappe.ui.form.on('Sales Order', {
    refresh: function(frm) {
        // Set filter for item_code field in child table
        frm.set_query('item_code', 'items', function(doc, cdt, cdn) {
            return {
                filters: {
                    'item_group': 'Products',
                    'disabled': 0
                }
            };
        });
    }
});
```

#### Iterate Through Child Rows

```javascript
frappe.ui.form.on('Sales Order', {
    calculate_totals: function(frm) {
        let total_qty = 0;
        let total_amount = 0;
        frm.doc.items.forEach(function(item) {
            total_qty += item.qty;
            total_amount += item.amount;
        });
        frm.set_value('total_qty', total_qty);
        frm.set_value('total', total_amount);
    }
});
```

---

## 6. Form Object (frm) Deep Dive

### A. Document Properties

```javascript
frm.doc                    // Current document object
frm.doc.name              // Document name
frm.doc.docstatus         // 0=Draft, 1=Submitted, 2=Cancelled
frm.doc.owner             // Document creator
frm.doc.modified_by       // Last modifier
frm.doc.creation          // Creation timestamp
frm.doc.modified          // Last modification timestamp
frm.doctype               // DocType name
frm.docname               // Document name (same as frm.doc.name)
```

### B. Form State Methods

```javascript
// Check if new document
frm.is_new()              // Returns true if unsaved
// Check if dirty (has unsaved changes)
frm.is_dirty()            // Returns true if modified
// Get field value
frm.doc.customer          // Direct access
let val = frm.get_value('customer');  // Method
// Set field value
frm.set_value('customer', 'CUST-001');
// Set multiple values
frm.set_value({
    'customer': 'CUST-001',
    'territory': 'India'
});
```

---

### C. Field Manipulation

```javascript
// Show/Hide field
frm.toggle_display('custom_field', true);  // Show
frm.toggle_display('custom_field', false); // Hide
// Enable/Disable field
frm.toggle_enable('customer', true);   // Enable
frm.toggle_enable('customer', false);  // Disable
// Make field mandatory/optional
frm.toggle_reqd('delivery_date', true);   // Required
frm.toggle_reqd('delivery_date', false);  // Optional
// Set field property
frm.set_df_property('customer', 'read_only', 1);
frm.set_df_property('amount', 'precision', 3);
frm.set_df_property('status', 'options', ['Open', 'Closed']);
frm.set_df_property('description', 'label', 'Remarks');
// Refresh field (re-render)
frm.refresh_field('customer');
frm.refresh_field('items');  // For child tables
```

---

### D. Field Queries (Filters in Link Fields)

```javascript
// Simple filter
frm.set_query('customer', function() {
    return {
        filters: {
            'customer_type': 'Company'
        }
    };
});

// Advanced filter
frm.set_query('item_code', 'items', function(doc, cdt, cdn) {
    let row = locals[cdt][cdn];
    return {
        filters: [
            ['Item', 'item_group', '=', 'Products'],
            ['Item', 'disabled', '=', 0]
        ],
        or_filters: [
            ['Item', 'is_stock_item', '=', 1],
            ['Item', 'is_sales_item', '=', 1]
        ]
    };
});

// Custom query with method
frm.set_query('warehouse', function() {
    return {
        query: 'erpnext.controllers.queries.warehouse_query',
        filters: {
            'company': frm.doc.company
        }
    };
});
```

---

### E. Buttons

```javascript
// Add custom button
frm.add_custom_button(__('Create Invoice'), function() {
    // Button click logic
    frappe.call({
        method: 'make_invoice',
        doc: frm.doc,
        callback: function(r) {
            if (r.message) {
                frappe.set_route('Form', 'Sales Invoice', r.message);
            }
        }
    });
});

// Add button in a group
frm.add_custom_button(__('Sales Invoice'), function() {
    // Logic
}, __('Create'));

// Remove button
frm.remove_custom_button('Create Invoice');

// Change button style
frm.add_custom_button(__('Submit'), function() {
    // Logic
}).addClass('btn-primary');

// Clear all custom buttons
frm.clear_custom_buttons();
```

---

### F. Messages & Alerts

```javascript
// Simple message
frappe.msgprint('Hello World');

// Formatted message
frappe.msgprint({
    title: __('Notification'),
    indicator: 'green',
    message: __('Data saved successfully')
});

// Alert (auto-hide)
frappe.show_alert({
    message: __('Item added'),
    indicator: 'green'
}, 5);  // 5 seconds

// Error message
frappe.throw(__('Invalid data'));

// Confirm dialog
frappe.confirm(
    'Are you sure you want to proceed?',
    function() {
        // Yes
        console.log('Confirmed');
    },
    function() {
        // No
        console.log('Cancelled');
    }
);

// Prompt dialog
frappe.prompt({
    label: 'Enter Remarks',
    fieldname: 'remarks',
    fieldtype: 'Data',
    reqd: 1
}, function(values) {
    console.log(values.remarks);
}, __('Enter Details'));

// Multi-field prompt
frappe.prompt([
    {
        label: 'Customer',
        fieldname: 'customer',
        fieldtype: 'Link',
        options: 'Customer',
        reqd: 1
    },
    {
        label: 'Amount',
        fieldname: 'amount',
        fieldtype: 'Currency',
        reqd: 1
    }
], function(values) {
    console.log(values.customer, values.amount);
});
```

---

### G. Saving & Reloading

```javascript
// Save document
frm.save();

// Save with options
frm.save('Save', function() {
    console.log('Saved successfully');
});

// Submit document
frm.submit();

// Cancel document
frm.cancel();

// Reload document
frm.reload_doc();

// Dirty (mark as changed)
frm.dirty();

// Save if dirty
if (frm.is_dirty()) {
    frm.save();
}
```

---

### H. Dashboard & Intro

```javascript
// Set intro message
frm.set_intro('Please verify all details before submission', 'blue');

// Clear intro
frm.set_intro('');

// Add to dashboard
frm.dashboard.add_indicator(__('Status: Open'), 'orange');

// Set headline
frm.layout.show_message('Special Discount Applied!', 'green');
```

---

## 7. Advanced Techniques

### A. Debouncing Events

Prevent excessive API calls:

```javascript
frappe.ui.form.on('Sales Order', {
    customer: frappe.utils.debounce(function(frm) {
        // This will only fire 300ms after user stops typing
        frappe.call({
            method: 'get_customer_details',
            args: { customer: frm.doc.customer }
        });
    }, 300)
});
```

---

### B. Throttling Events

Limit execution frequency:

```javascript
let throttled_calc = frappe.utils.throttle(function(frm) {
    // Calculate totals
}, 500);  // Max once per 500ms

frappe.ui.form.on('Sales Order Item', {
    qty: function(frm, cdt, cdn) {
        throttled_calc(frm);
    }
});
```

---

### C. Custom Formatters

```javascript
frappe.ui.form.on('Sales Order', {
    refresh: function(frm) {
        // Custom format for amount field
        frm.fields_dict['grand_total'].formatter = function(value, df, options, doc) {
            return `<strong style="color: green;">${format_currency(value)}</strong>`;
        };
    }
});
```

---

### D. Grid Customization

```javascript
frappe.ui.form.on('Sales Order', {
    refresh: function(frm) {
        // Customize grid buttons
        frm.fields_dict.items.grid.add_custom_button(__('Bulk Update'), function() {
            // Bulk update logic
        });

        // Hide add row button conditionally
        if (frm.doc.docstatus === 1) {
            frm.fields_dict.items.grid.cannot_add_rows = true;
            frm.fields_dict.items.grid.refresh();
        }

        // Grid row formatting
        frm.fields_dict.items.grid.wrapper.find('.grid-body .rows').find('.grid-row').each(function(i, row) {
            let $row = $(row);
            let item = frm.doc.items[i];
            if (item && item.qty > 100) {
                $row.css('background-color', '#fffacd');
            }
        });
    }
});
```

---

### E. Dynamic Field Creation

```javascript
frappe.ui.form.on('Sales Order', {
    onload: function(frm) {
        // Add custom HTML field
        if (!frm.fields_dict.custom_html) {
            frm.add_custom_button('Add HTML Section', function() {
                let $wrapper = frm.fields_dict.items.wrapper;
                $wrapper.append(`
                    <div class="custom-section">
                        <h4>Custom Section</h4>
                        <p>Additional information</p>
                    </div>
                `);
            });
        }
    }
});
```

---

### F. Conditional Rendering

```javascript
frappe.ui.form.on('Sales Order', {
    refresh: function(frm) {
        // Show section based on condition
        frm.toggle_display('section_break_1', frm.doc.order_type === 'Sales');

        // Multiple conditions
        let show_tax_section = frm.doc.customer && frm.doc.items.length > 0;
        frm.toggle_display('tax_section', show_tax_section);
    }
});
```

---

### G. Server Calls (frappe.call)

```javascript
// Basic call
frappe.call({
    method: 'frappe.client.get_value',
    args: {
        doctype: 'Customer',
        filters: { name: 'CUST-001' },
        fieldname: ['customer_name', 'territory']
    },
    callback: function(r) {
        console.log(r.message);
    }
});

// Call doc method
frappe.call({
    method: 'calculate_taxes',
    doc: frm.doc,
    callback: function(r) {
        frm.refresh_field('taxes');
    }
});

// Call with freeze (loading indicator)
frappe.call({
    method: 'long_running_method',
    args: { data: 'something' },
    freeze: true,
    freeze_message: __('Processing...'),
    callback: function(r) {
        frappe.msgprint('Done');
    }
});

// Async/await pattern
async function fetchData(frm) {
    let response = await frappe.call({
        method: 'get_data',
        args: { name: frm.doc.name }
    });
    return response.message;
}
```

---

### H. Route & Navigation

```javascript
// Navigate to form
frappe.set_route('Form', 'Customer', 'CUST-001');

// Navigate to list
frappe.set_route('List', 'Sales Order');

// Open in new tab
frappe.open_in_new_tab = true;
frappe.set_route('Form', 'Sales Invoice', 'INV-001');

// Go back
frappe.set_route(frappe.get_prev_route());

// Get current route
let route = frappe.get_route();
console.log(route);  // ['Form', 'Sales Order', 'SO-001']
```

---

### I. Local Storage

```javascript
frappe.ui.form.on('Sales Order', {
    customer: function(frm) {
        localStorage.setItem('last_customer', frm.doc.customer);
    },
    onload: function(frm) {
        // Restore from local storage
        if (frm.is_new()) {
            let last_customer = localStorage.getItem('last_customer');
            if (last_customer) {
                frm.set_value('customer', last_customer);
            }
        }
    }
});
```

---

### J. Custom Validators

```javascript
frappe.ui.form.on('Sales Order', {
    validate: function(frm) {
        // Email validation
        if (frm.doc.contact_email) {
            let email_regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            if (!email_regex.test(frm.doc.contact_email)) {
                frappe.msgprint(__('Invalid email format'));
                frappe.validated = false;
            }
        }

        // Phone validation
        if (frm.doc.contact_phone) {
            let phone_regex = /^[0-9]{10}$/;
            if (!phone_regex.test(frm.doc.contact_phone)) {
                frappe.msgprint(__('Phone must be 10 digits'));
                frappe.validated = false;
            }
        }

        // Custom business logic
        if (frm.doc.items) {
            let has_negative_qty = frm.doc.items.some(item => item.qty < 0);
            if (has_negative_qty) {
                frappe.throw(__('Quantity cannot be negative'));
            }
        }
    }
});
```

---

## 8. Real-World Examples

### Example 1: Auto-Calculate Discount

```javascript
frappe.ui.form.on('Sales Order', {
    refresh: function(frm) {
        frm.add_custom_button(__('Apply Discount'), function() {
            apply_bulk_discount(frm);
        });
    }
});

function apply_bulk_discount(frm) {
    frappe.prompt([
        {
            label: 'Discount %',
            fieldname: 'discount',
            fieldtype: 'Float',
            reqd: 1
        }
    ], function(values) {
        frm.doc.items.forEach(function(item) {
            let discount_amount = item.rate * values.discount / 100;
            let new_rate = item.rate - discount_amount;
            frappe.model.set_value(item.doctype, item.name, 'rate', new_rate);
            frappe.model.set_value(item.doctype, item.name, 'discount_percentage', values.discount);
        });
        frm.refresh_field('items');
        frappe.show_alert(__('Discount applied'), 3);
    }, __('Bulk Discount'));
}
```

---

### Example 2: Stock Availability Check

```javascript
frappe.ui.form.on('Sales Order Item', {
    item_code: function(frm, cdt, cdn) {
        let row = locals[cdt][cdn];
        if (row.item_code && row.warehouse) {
            frappe.call({
                method: 'erpnext.stock.utils.get_stock_balance',
                args: {
                    item_code: row.item_code,
                    warehouse: row.warehouse
                },
                callback: function(r) {
                    if (r.message) {
                        let stock_qty = r.message;
                        if (stock_qty < row.qty) {
                            frappe.msgprint({
                                title: __('Low Stock'),
                                indicator: 'orange',
                                message: __('Available: {0}, Required: {1}', [stock_qty, row.qty])
                            });
                        }
                        // Show in row
                        frappe.model.set_value(cdt, cdn, 'available_qty', stock_qty);
                    }
                }
            });
        }
    }
});
```

---

### Example 3: Customer Credit Limit

```javascript
frappe.ui.form.on('Sales Order', {
    customer: function(frm) {
        if (frm.doc.customer) {
            check_credit_limit(frm);
        }
    },
    validate: function(frm) {
        return check_credit_limit(frm);
    }
});

function check_credit_limit(frm) {
    return new Promise((resolve, reject) => {
        frappe.call({
            method: 'erpnext.selling.doctype.customer.customer.get_credit_limit',
            args: {
                customer: frm.doc.customer,
                company: frm.doc.company
            },
            callback: function(r) {
                if (r.message) {
                    let credit_limit = r.message.credit_limit;
                    let outstanding = r.message.outstanding_amt;
                    let available = credit_limit - outstanding;
                    if (frm.doc.grand_total > available) {
                        frappe.msgprint({
                            title: __('Credit Limit Exceeded'),
                            indicator: 'red',
                            message: __('Available Credit: {0}', [format_currency(available)])
                        });
                        frappe.validated = false;
                        reject();
                    } else {
                        resolve();
                    }
                }
            }
        });
    });
}
```

---

### Example 4: Dynamic Pricing

```javascript
frappe.ui.form.on('Sales Order Item', {
    item_code: function(frm, cdt, cdn) {
        calculate_dynamic_price(frm, cdt, cdn);
    },
    qty: function(frm, cdt, cdn) {
        calculate_dynamic_price(frm, cdt, cdn);
    }
});

function calculate_dynamic_price(frm, cdt, cdn) {
    let row = locals[cdt][cdn];
    if (!row.item_code) return;
    frappe.call({
        method: 'your_app.api.get_dynamic_price',
        args: {
            item_code: row.item_code,
            customer: frm.doc.customer,
            qty: row.qty,
            date: frm.doc.transaction_date
        },
        callback: function(r) {
            if (r.message) {
                frappe.model.set_value(cdt, cdn, 'rate', r.message.rate);
                frappe.model.set_value(cdt, cdn, 'discount_percentage', r.message.discount);
                // Show price breakdown
                let price_details = `
                    Base Price: ${r.message.base_price}<br>
                    Volume Discount: ${r.message.volume_discount}%<br>
                    Customer Discount: ${r.message.customer_discount}%<br>
                    Final Rate: ${r.message.rate}
                `;
                frappe.show_alert({
                    message: price_details,
                    indicator: 'blue'
                }, 10);
            }
        }
    });
}
```

---

### Example 5: Duplicate Detection

```javascript
frappe.ui.form.on('Sales Order', {
    validate: function(frm) {
        return check_duplicate_order(frm);
    }
});

function check_duplicate_order(frm) {
    return new Promise((resolve, reject) => {
        if (!frm.doc.customer || !frm.doc.items.length) {
            resolve();
            return;
        }
        let item_codes = frm.doc.items.map(i => i.item_code);
        frappe.call({
            method: 'your_app.api.check_duplicate_order',
            args: {
                customer: frm.doc.customer,
                items: item_codes,
                exclude: frm.doc.name
            },
            callback: function(r) {
                if (r.message && r.message.length > 0) {
                    let duplicates = r.message.map(d => d.name).join(', ');
                    frappe.confirm(
                        __('Similar orders found: {0}. Do you want to continue?', [duplicates]),
                        function() {
                            resolve();
                        },
                        function() {
                            frappe.validated = false;
                            reject();
                        }
                    );
                } else {
                    resolve();
                }
            }
        });
    });
}
```

---

## 9. Performance & Best Practices

### ✅ DO's

```javascript
// 1. Use debounce for API calls
customer: frappe.utils.debounce(function(frm) {
    // API call
}, 300)

// 2. Cache expensive calculations
let _cached_total = null;
function get_total(frm) {
    if (_cached_total === null) {
        _cached_total = calculate_total(frm);
    }
    return _cached_total;
}

// 3. Batch field updates
frappe.run_serially([
    () => frm.set_value('field1', 'value1'),
    () => frm.set_value('field2', 'value2'),
    () => frm.set_value('field3', 'value3')
]);

// 4. Use promises for async operations
validate: function(frm) {
    return new Promise((resolve, reject) => {
        // Async validation
    });
}

// 5. Minimize refresh calls
// Bad: frm.refresh_field() after each set_value
// Good: Set all values, then refresh once
frm.doc.items.forEach(item => {
    item.rate = new_rate;
});
frm.refresh_field('items');
```

---

### ❌ DON'Ts

```javascript
// 1. Don't do heavy operations in refresh
refresh: function(frm) {
    // BAD: This runs multiple times
    expensive_calculation();
}

// 2. Don't use synchronous calls
// BAD
let result = frappe.call({
    method: 'my_method',
    async: false  // DON'T DO THIS
});

// GOOD
frappe.call({
    method: 'my_method',
    callback: function(r) {
        // Handle result
    }
});

// 3. Don't manipulate frm.doc directly without set_value
// BAD
frm.doc.customer = 'CUST-001';

// GOOD
frm.set_value('customer', 'CUST-001');

// 4. Don't attach multiple listeners to same event
// BAD
customer: function(frm) { ... }
customer: function(frm) { ... }  // Overwrites previous

// GOOD: Use a single function
customer: function(frm) {
    handle_customer_change_part1(frm);
    handle_customer_change_part2(frm);
}

// 5. Don't use alerts excessively
// BAD
frappe.msgprint('Step 1');
frappe.msgprint('Step 2');
frappe.msgprint('Step 3');

// GOOD
frappe.msgprint('All steps completed successfully');
```

---

## 10. Debugging & Troubleshooting

### Console Logging

```javascript
frappe.ui.form.on('Sales Order', {
    refresh: function(frm) {
        console.log('Form object:', frm);
        console.log('Document:', frm.doc);
        console.log('DocType:', frm.doctype);
        console.log('Is new:', frm.is_new());
        console.log('Is dirty:', frm.is_dirty());
    },
    customer: function(frm) {
        console.log('Customer changed to:', frm.doc.customer);
        console.trace('Call stack');  // Shows execution path
    }
});
```

---

### Breakpoints

```javascript
frappe.ui.form.on('Sales Order', {
    validate: function(frm) {
        debugger;  // Execution will pause here
        // Inspect variables in browser console
        let total = calculate_total(frm);
        console.log('Total:', total);
    }
});
```

---

### Error Handling

```javascript
frappe.ui.form.on('Sales Order', {
    customer: function(frm) {
        try {
            // Risky operation
            let data = JSON.parse(frm.doc.custom_json_field);
            process_data(data);
        } catch(e) {
            console.error('Error parsing JSON:', e);
            frappe.msgprint({
                title: __('Error'),
                indicator: 'red',
                message: __('Invalid data format: {0}', [e.message])
            });
        }
    }
});
```

---

### Network Debugging

```javascript
frappe.call({
    method: 'my_method',
    args: { data: 'test' },
    callback: function(r) {
        console.log('Response:', r);
    },
    error: function(r) {
        console.error('Error:', r);
        frappe.msgprint({
            title: __('Server Error'),
            indicator: 'red',
            message: r.message || 'Unknown error'
        });
    }
});
```

---

### Testing Event Triggers

```javascript
// Manually trigger events from console
cur_frm.trigger('customer');
cur_frm.trigger('validate');
cur_frm.trigger('calculate_totals');

// Check if event exists
console.log(cur_frm.cscript);
console.log(cur_frm.events);
```

---

### Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Event not firing | Check script is saved and cleared cache (Ctrl+Shift+R) |
| `frm` is undefined | Ensure using correct syntax: `function(frm)` |
| Child table event not working | Use child DocType name, not table field name |
| Changes not saving | Use `frm.set_value()`, not direct `frm.doc.field = value` |
| Infinite loops | Check for circular event triggers (A triggers B, B triggers A) |
| Performance issues | Remove heavy operations from `refresh` event |

---

## Complete Template Example

```javascript
frappe.ui.form.on('Sales Order', {
    // Setup - runs once
    setup: function(frm) {
        // Set filters
        frm.set_query('customer', function() {
            return {
                filters: { 'disabled': 0 }
            };
        });
    },

    // Before load
    before_load: function(frm) {
        console.log('Before load');
    },

    // On load
    onload: function(frm) {
        if (frm.is_new()) {
            frm.set_value('transaction_date', frappe.datetime.get_today());
        }
    },

    // Refresh (most common)
    refresh: function(frm) {
        // Hide fields
        frm.toggle_display('section_break', frm.doc.customer);

        // Add buttons
        if (!frm.is_new() && frm.doc.docstatus === 1) {
            frm.add_custom_button(__('Create Invoice'), function() {
                create_invoice(frm);
            });
        }

        // Set intro
        if (frm.doc.docstatus === 0) {
            frm.set_intro(__('Please fill all required fields'), 'blue');
        }
    },

    // Field events
    customer: function(frm) {
        if (frm.doc.customer) {
            fetch_customer_details(frm);
        }
    },
    delivery_date: function(frm) {
        validate_delivery_date(frm);
    },

    // Validation
    validate: function(frm) {
        if (!frm.doc.items || frm.doc.items.length === 0) {
            frappe.msgprint(__('Please add items'));
            frappe.validated = false;
        }
        calculate_totals(frm);
    },

    // Before save
    before_save: function(frm) {
        return check_credit_limit(frm);
    },

    // After save
    after_save: function(frm) {
        frappe.show_alert(__('Saved successfully'), 3);
    },

    // Custom methods (can be triggered with frm.trigger('method_name'))
    calculate_totals: function(frm) {
        let total = 0;
        frm.doc.items.forEach(item => {
            total += item.amount;
        });
        frm.set_value('total', total);
    }
});

// Child table events
frappe.ui.form.on('Sales Order Item', {
    items_add: function(frm, cdt, cdn) {
        let row = locals[cdt][cdn];
        frappe.model.set_value(cdt, cdn, 'qty', 1);
    },
    item_code: function(frm, cdt, cdn) {
        fetch_item_details(frm, cdt, cdn);
    },
    qty: function(frm, cdt, cdn) {
        calculate_amount(frm, cdt, cdn);
    },
    rate: function(frm, cdt, cdn) {
        calculate_amount(frm, cdt, cdn);
    }
});

// Helper functions
function fetch_customer_details(frm) {
    frappe.call({
        method: 'frappe.client.get',
        args: {
            doctype: 'Customer',
            name: frm.doc.customer
        },
        callback: function(r) {
            if (r.message) {
                frm.set_value('customer_name', r.message.customer_name);
                frm.set_value('territory', r.message.territory);
            }
        }
    });
}

function calculate_amount(frm, cdt, cdn) {
    let row = locals[cdt][cdn];
    let amount = (row.qty || 0) * (row.rate || 0);
    frappe.model.set_value(cdt, cdn, 'amount', amount);
    frm.trigger('calculate_totals');
}
```

---

