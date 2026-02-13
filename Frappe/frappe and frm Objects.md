
# Complete Guide to `frappe` and `frm` Objects

> [!NOTE] Ultimate reference for Frappe Framework client-side JavaScript APIs. Optimized for Obsidian with collapsible callouts, proper code blocks, and internal linking.

## Table of Contents
- [[#1-the-frappe-object-global|1. The `frappe` Object (Global)]]
- [[#2-the-frm-object-form-specific|2. The `frm` Object (Form-Specific)]]
- [[#3-deep-comparison|3. Deep Comparison]]
- [[#4-advanced-usage-patterns|4. Advanced Usage Patterns]]

---


### 1.1 Session & User Information

#### `frappe.session`
Contains current user session data.

```javascript
// Current user
frappe.session.user           // "john@example.com"
frappe.session.user_fullname  // "John Doe"
frappe.session.user_email     // "john@example.com"

// User image
frappe.session.user_image     // "/files/user_image.jpg"

// Session info
frappe.session.sid            // Session ID
frappe.session.logged_in      // true/false

// Check if user is logged in
if (frappe.session.user !== 'Guest') {
	console.log('User is logged in');
}
```

#### `frappe.user`
User-specific methods and properties.

```javascript
// Check if user has role
frappe.user.has_role('Sales Manager')  // true/false
frappe.user.has_role(['Sales Manager', 'Sales User'])  // true if any role matches

// Get user roles
frappe.user.get_roles()  // ['System Manager', 'Sales User', ...]

// Check permissions
frappe.user.has_perm('Sales Order', 'read')   // true/false
frappe.user.has_perm('Sales Order', 'write')  // true/false

// Get user defaults
frappe.user.get_default('company')  // Default company
frappe.user.get_default('fiscal_year')

// Set user defaults
frappe.user.set_default('company', 'My Company')

// User info
frappe.user.name              // "john@example.com"
frappe.user_info()            // Full user object
frappe.user_info('john@example.com')  // Specific user info
```

> [!EXAMPLE] Role-based UI
> ```javascript
> frappe.ui.form.on('Sales Order', {
> 	refresh: function(frm) {
> 		// Show button only for managers
> 		if (frappe.user.has_role('Sales Manager')) {
> 			frm.add_custom_button('Approve', function() {
> 				// Approval logic
> 			});
> 		}
> 		
> 		// Different behavior for different roles
> 		if (frappe.user.has_role('System Manager')) {
> 			frm.set_df_property('discount_percentage', 'read_only', 0);
> 		} else {
> 			frm.set_df_property('discount_percentage', 'read_only', 1);
> 		}
> 	}
> });
> ```

### 1.2 Site & System Information

```javascript
// Site name
frappe.boot.sitename          // "mysite.erpnext.com"

// System settings
frappe.sys_defaults           // System defaults object
frappe.sys_defaults.company   // Default company
frappe.sys_defaults.currency  // Default currency

// Boot info (loaded on page load)
frappe.boot                   // Contains all boot data
frappe.boot.user              // User details
frappe.boot.sysdefaults       // System defaults
frappe.boot.versions          // App versions

// App version
frappe.boot.versions.frappe   // "15.0.0"
frappe.boot.versions.erpnext  // "15.0.0"

// System language
frappe.boot.lang              // "en"

// Timezone
frappe.sys_defaults.time_zone // "Asia/Kolkata"
```

### 1.3 Database Operations

#### `frappe.db.get_value()`
```javascript
// Get single value
frappe.db.get_value('Customer', 'CUST-001', 'customer_name')
	.then(r => {
		console.log(r.message.customer_name);
	});

// Get multiple fields
frappe.db.get_value('Customer', 'CUST-001', ['customer_name', 'territory'])
	.then(r => {
		console.log(r.message.customer_name);
		console.log(r.message.territory);
	});

// With filters
frappe.db.get_value('Customer', {
	customer_group: 'Commercial',
	territory: 'India'
}, 'customer_name')
	.then(r => {
		console.log(r.message.customer_name);
	});

// Async/await syntax (modern)
async function getCustomerName() {
	let r = await frappe.db.get_value('Customer', 'CUST-001', 'customer_name');
	return r.message.customer_name;
}
```

#### `frappe.db.get_list()`
```javascript
// Get list of documents
frappe.db.get_list('Customer', {
	fields: ['name', 'customer_name', 'territory'],
	filters: {
		customer_group: 'Commercial',
		disabled: 0
	},
	order_by: 'creation desc',
	limit: 20
}).then(r => {
	console.log(r);  // Array of objects
});

// With complex filters
frappe.db.get_list('Sales Order', {
	fields: ['name', 'customer', 'grand_total'],
	filters: [
		['docstatus', '=', 1],
		['grand_total', '>', 10000],
		['transaction_date', 'Between', ['2024-01-01', '2024-12-31']]
	],
	order_by: 'grand_total desc',
	limit_page_length: 50
}).then(r => {
	console.log(`Found ${r.length} orders`);
});

// OR filters
frappe.db.get_list('Item', {
	fields: ['name', 'item_name'],
	filters: {
		item_group: 'Products',
		disabled: 0
	},
	or_filters: [
		['is_stock_item', '=', 1],
		['is_sales_item', '=', 1]
	]
}).then(r => {
	console.log(r);
});
```

#### Other Database Methods
```javascript
// Count documents
frappe.db.count('Customer', {
	filters: {
		disabled: 0,
		customer_group: 'Commercial'
	}
}).then(r => {
	console.log(`Total customers: ${r.message}`);
});

// Get full document
frappe.db.get_doc('Sales Order', 'SO-00001')
	.then(doc => {
		console.log(doc);
		console.log(doc.customer);
		console.log(doc.items);  // Child table
	});

// Update single field
frappe.db.set_value('Customer', 'CUST-001', 'territory', 'India')
	.then(r => {
		console.log('Updated successfully');
	});

// Delete document
frappe.db.delete_doc('Customer', 'CUST-001')
	.then(r => {
		console.log('Deleted successfully');
	});

// Insert new document
frappe.db.insert({
	doctype: 'Customer',
	customer_name: 'New Customer',
	customer_type: 'Company',
	territory: 'India'
}).then(doc => {
	console.log('Created:', doc.name);
});
```

### 1.4 Server Calls

#### `frappe.call()`
```javascript
// Basic call
frappe.call({
	method: 'frappe.client.get_value',
	args: {
		doctype: 'Customer',
		filters: { name: 'CUST-001' },
		fieldname: 'customer_name'
	},
	callback: function(r) {
		if (!r.exc) {
			console.log(r.message);
		}
	}
});

// Call with freeze (loading indicator)
frappe.call({
	method: 'your_app.api.long_process',
	args: { data: 'something' },
	freeze: true,
	freeze_message: __('Processing...'),
	callback: function(r) {
		frappe.msgprint('Completed!');
	}
});

// Call doc method (method defined in Python DocType class)
frappe.call({
	method: 'calculate_taxes',
	doc: frm.doc,  // Current document
	callback: function(r) {
		if (!r.exc) {
			frm.refresh_field('taxes');
		}
	}
});

// Error handling
frappe.call({
	method: 'your_app.api.risky_method',
	args: { data: 'test' },
	callback: function(r) {
		console.log('Success:', r.message);
	},
	error: function(r) {
		console.error('Error:', r);
		frappe.msgprint({
			title: __('Error'),
			indicator: 'red',
			message: r.message
		});
	}
});

// Async/await pattern
async function fetchData() {
	try {
		let r = await frappe.call({
			method: 'your_app.api.get_data',
			args: { id: 123 }
		});
		return r.message;
	} catch (error) {
		console.error(error);
		throw error;
	}
}
```

#### `frappe.xcall()` - Modern Promise-based Call
```javascript
// Returns a promise directly
async function getData() {
	let result = await frappe.xcall('your_app.api.get_data', {
		id: 123
	});
	return result;
}

// Shorter syntax
frappe.xcall('frappe.client.get_value', {
	doctype: 'Customer',
	filters: { name: 'CUST-001' },
	fieldname: 'customer_name'
}).then(r => {
	console.log(r.customer_name);
});
```

### 1.5 UI Components

#### Messages & Alerts
```javascript
// Simple message
frappe.msgprint('Hello World');
frappe.msgprint(__('Translated message'));

// Formatted message
frappe.msgprint({
	title: __('Success'),
	message: __('Data saved successfully'),
	indicator: 'green'
});

// Alert (auto-hide)
frappe.show_alert({
	message: __('Item added to cart'),
	indicator: 'green'
}, 5);  // 5 seconds

// Different indicators
frappe.show_alert({ message: 'Info', indicator: 'blue' });
frappe.show_alert({ message: 'Warning', indicator: 'orange' });
frappe.show_alert({ message: 'Error', indicator: 'red' });
frappe.show_alert({ message: 'Success', indicator: 'green' });

// Throw error (stops execution)
frappe.throw(__('This is an error'));
frappe.throw({
	title: __('Validation Error'),
	message: __('Invalid data'),
	indicator: 'red'
});

// Confirm dialog
frappe.confirm(
	'Are you sure you want to proceed?',
	function() {
		// Yes callback
		console.log('User clicked Yes');
	},
	function() {
		// No callback (optional)
		console.log('User clicked No');
	}
);
```

#### Prompt Dialogs
```javascript
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
		reqd: 1,
		default: 0
	},
	{
		label: 'Date',
		fieldname: 'date',
		fieldtype: 'Date',
		default: frappe.datetime.get_today()
	}
], function(values) {
	console.log('Customer:', values.customer);
	// Process data
}, __('Create Invoice'));
```

### 1.6 Routing & Navigation
```javascript
// Get current route
frappe.get_route()           // ['List', 'Customer']
frappe.get_route_str()       // "List/Customer"

// Set route (navigate)
frappe.set_route('List', 'Customer');
frappe.set_route('Form', 'Customer', 'CUST-001');
frappe.set_route('List', 'Sales Order', {'customer': 'CUST-001'});

// Print view
frappe.set_route('print', 'Sales Order', 'SO-00001');

// Report
frappe.set_route('query-report', 'Sales Analytics');
```

### 1.7 Date & Time Utilities
```javascript
// Current date
frappe.datetime.get_today()              // "2024-01-15"

// Add days
frappe.datetime.add_days('2024-01-15', 7)         // "2024-01-22"
frappe.datetime.add_days('2024-01-15', -7)        // "2024-01-08"

// Add months
frappe.datetime.add_months('2024-01-15', 1)       // "2024-02-15"

// Get day difference
frappe.datetime.get_day_diff('2024-01-22', '2024-01-15')  // 7

// Get first/last day of month
frappe.datetime.get_first_day('2024-01-15')      // "2024-01-01"
frappe.datetime.get_last_day('2024-01-15')       // "2024-01-31"

// Pretty date
frappe.datetime.prettyDate('2024-01-15')         // "15 Jan 2024"
frappe.datetime.comment_when('2024-01-15 10:30:00')  // "2 hours ago"
```

### 1.8 Utilities
```javascript
// Translation
__('Hello')                              // Translated string
__('Hello {0}', ['World'])              // "Hello World"

// Format currency
format_currency(1000, 'USD')            // "$1,000.00"
format_currency(1000)                   // Uses default currency

// Parse number
flt(value)                              // Convert to float
cint(value)                             // Convert to int

// Scrub/unscrub
frappe.scrub('Customer Name')           // "customer_name"
frappe.unscrub('customer_name')         // "Customer Name"

// Debounce/throttle
let debounced = frappe.utils.debounce(function() {
	console.log('Called after 300ms of inactivity');
}, 300);

let throttled = frappe.utils.throttle(function() {
	console.log('Called max once per 500ms');
}, 500);
```

---

## 2. The `frm` Object (Form-Specific)

The `frm` object represents a **specific form instance** and is available in all form events.

### 2.1 Document Properties
```javascript
// Basic properties
frm.doc                    // The document object
frm.doc.name              // Document name (e.g., "SO-00001")
frm.doc.doctype           // "Sales Order"
frm.doc.docstatus         // 0=Draft, 1=Submitted, 2=Cancelled

// Form properties
frm.doctype               // "Sales Order"
frm.docname               // Same as frm.doc.name
frm.meta                  // DocType metadata
frm.is_new()              // true if unsaved document
frm.is_dirty()            // true if has unsaved changes

// Examples
if (frm.is_new()) {
	console.log('This is a new document');
}
if (frm.doc.docstatus === 1) {
	console.log('Document is submitted');
}
```

### 2.2 Field Operations

#### Get/Set Values
```javascript
// Get field value
let customer = frm.get_value('customer');
let values = frm.get_values(['customer', 'transaction_date']);

// Set single value
frm.set_value('customer', 'CUST-001');

// Set multiple values
frm.set_value({
	customer: 'CUST-001',
	transaction_date: frappe.datetime.get_today()
});

// Refresh field (re-render)
frm.refresh_field('customer');
frm.refresh_field('items');  // Refresh child table
```

#### Field Properties
```javascript
// Show/Hide field
frm.toggle_display('custom_field', true);   // Show
frm.set_df_property('custom_field', 'hidden', 1);

// Enable/Disable
frm.toggle_enable('customer', false);       // Disable
frm.set_df_property('customer', 'read_only', 1);

// Make Required/Optional
frm.toggle_reqd('delivery_date', true);     // Required
frm.set_df_property('delivery_date', 'reqd', 1);

// Change label/description
frm.set_df_property('customer', 'label', 'Client Name');
frm.set_df_property('amount', 'description', 'Enter amount in USD');
```

### 2.3 Field Queries (Link Field Filters)
```javascript
// Simple filter
frm.set_query('customer', function() {
	return {
		filters: {
			disabled: 0,
			customer_type: 'Company'
		}
	};
});

// Dynamic filter based on form values
frm.set_query('customer', function() {
	return {
		filters: {
			territory: frm.doc.territory || '',
			customer_group: frm.doc.customer_group || ''
		}
	};
});

// Child table field query
frm.set_query('item_code', 'items', function(doc, cdt, cdn) {
	let row = locals[cdt][cdn];
	return {
		filters: {
			item_group: 'Products',
			disabled: 0
		}
	};
});
```

### 2.4 Buttons
```javascript
// Add custom button
frm.add_custom_button(__('Create Invoice'), function() {
	// Button click logic
}, __('Actions')).addClass('btn-primary');

// Conditional buttons
if (frm.doc.docstatus === 1 && frm.doc.status !== 'Completed') {
	frm.add_custom_button(__('Complete'), function() {
		// Completion logic
	});
}

// Remove button
frm.remove_custom_button('Create Invoice');
frm.remove_custom_button('Sales Invoice', 'Create');  // From group
```

### 2.5 Child Table Operations

#### Access & Iterate
```javascript
// Get child table rows
frm.doc.items                   // Array of child rows
frm.doc.items.length           // Number of rows

// Iterate through rows
frm.doc.items.forEach(function(row) {
	console.log(row.item_code, row.qty, row.rate);
});
```

#### Add/Update/Remove Rows
```javascript
// Add empty row
let row = frm.add_child('items');
row.item_code = 'ITEM-001';
row.qty = 1;
frm.refresh_field('items');

// Add row with data
let row = frm.add_child('items', {
	item_code: 'ITEM-001',
	qty: 1,
	rate: 100
});
frm.refresh_field('items');

// Clear all rows
frm.clear_table('items');
frm.refresh_field('items');

// Remove by condition
frm.doc.items = frm.doc.items.filter(row => row.qty > 0);
frm.refresh_field('items');
```

### 2.6 Save & Submit Operations
```javascript
// Save document
frm.save();

// Save with callback
frm.save('Save', function() {
	frappe.msgprint('Saved successfully');
});

// Submit document
frm.submit();

// Cancel document
frm.cancel();

// Reload document
frm.reload_doc();
```

### 2.7 Dashboard & UI Elements
```javascript
// Set intro message
frm.set_intro('Please fill all mandatory fields', 'blue');
frm.set_intro('Document is locked', 'red');

// Dashboard indicators
frm.dashboard.add_indicator(__('Status: Open'), 'orange');
frm.dashboard.add_indicator(__('High Priority'), 'red');

// Progress bar
frm.dashboard.show_progress(__('Completion'), 75, __('75% Complete'));

// Clear indicators
frm.dashboard.clear_headline();
```

### 2.8 Server Method Calls (Doc Methods)
```javascript
// Call doc method
frm.call({
	method: 'calculate_taxes',
	doc: frm.doc,
	callback: function(r) {
		if (!r.exc) {
			frm.refresh_field('taxes');
			frappe.show_alert('Taxes calculated');
		}
	}
});

// Async/await
async function fetchDetails(frm) {
	let r = await frm.call({
		method: 'get_details',
		args: { id: frm.doc.name }
	});
	return r.message;
}
```

---

## 3. Deep Comparison

### When to Use `frappe` vs `frm`

| Use Case                     | Use `frappe`                          | Use `frm`                             |
|------------------------------|---------------------------------------|---------------------------------------|
| Database queries             | ✅ `frappe.db.get_value()`            | ❌                                    |
| Server calls (generic)       | ✅ `frappe.call()`                    | ❌                                    |
| Server calls (doc methods)   | ❌                                    | ✅ `frm.call()`                       |
| User info                    | ✅ `frappe.session.user`              | ❌                                    |
| Messages/Alerts              | ✅ `frappe.msgprint()`                | ❌                                    |
| Navigation                   | ✅ `frappe.set_route()`               | ❌                                    |
| Field operations             | ❌                                    | ✅ `frm.set_value()`                  |
| Form buttons                 | ❌                                    | ✅ `frm.add_custom_button()`          |
| Child tables                 | ❌                                    | ✅ `frm.add_child()`                  |
| Document data                | ❌                                    | ✅ `frm.doc`                          |
| Save/Submit                  | ❌                                    | ✅ `frm.save()`                       |

---

## 4. Advanced Usage Patterns

### Pattern 1: Form with Server Validation
```javascript
frappe.ui.form.on('Sales Order', {
	validate: function(frm) {
		// Client-side validation
		if (!frm.doc.items || frm.doc.items.length === 0) {
			frappe.msgprint(__('Please add items'));
			frappe.validated = false;
			return;
		}
		
		// Server-side async validation
		return new Promise((resolve, reject) => {
			frappe.call({
				method: 'your_app.api.validate_order',
				args: {
					customer: frm.doc.customer,
					total: frm.doc.grand_total
				},
				callback: function(r) {
					if (r.message.valid) {
						resolve();
					} else {
						frappe.msgprint(r.message.error);
						frappe.validated = false;
						reject();
					}
				}
			});
		});
	}
});
```

### Pattern 2: Dynamic Pricing Engine
```javascript
frappe.ui.form.on('Sales Order Item', {
	item_code: frappe.utils.debounce(function(frm, cdt, cdn) {
		let row = locals[cdt][cdn];
		if (!row.item_code) return;
		
		frappe.call({
			method: 'your_app.pricing.get_price',
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
				}
			}
		});
	}, 300)
});
```

### Pattern 3: Bulk Operations
```javascript
frappe.ui.form.on('Sales Order', {
	refresh: function(frm) {
		frm.add_custom_button(__('Bulk Update Items'), function() {
			show_bulk_update_dialog(frm);
		});
	}
});

function show_bulk_update_dialog(frm) {
	let d = new frappe.ui.Dialog({
		title: __('Bulk Update'),
		fields: [
			{
				label: 'Field to Update',
				fieldname: 'field',
				fieldtype: 'Select',
				options: ['Rate', 'Discount %', 'Warehouse'],
				reqd: 1
			},
			{
				label: 'Value',
				fieldname: 'value',
				fieldtype: 'Data',
				reqd: 1
			}
		],
		primary_action_label: __('Update'),
		primary_action: function(values) {
			bulk_update_items(frm, values);
			d.hide();
		}
	});
	d.show();
}

function bulk_update_items(frm, values) {
	let field_map = {
		'Rate': 'rate',
		'Discount %': 'discount_percentage',
		'Warehouse': 'warehouse'
	};
	let field = field_map[values.field];
	let updated = 0;
	
	frm.doc.items.forEach(function(row) {
		frappe.model.set_value(row.doctype, row.name, field, values.value);
		updated++;
	});
	
	frm.refresh_field('items');
	frappe.show_alert(__('Updated {0} items', [updated]), 5);
}
```

---

## Quick Reference Tables

### `frappe` Object - Key Methods

| Category         | Methods                                                                 |
|------------------|-------------------------------------------------------------------------|
| **User & Session** | `frappe.session.user`, `frappe.user.has_role()`, `frappe.user.get_roles()` |
| **Database**       | `frappe.db.get_value()`, `frappe.db.get_list()`, `frappe.db.set_value()` |
| **Server Calls**   | `frappe.call()`, `frappe.xcall()`                                      |
| **UI Messages**    | `frappe.msgprint()`, `frappe.show_alert()`, `frappe.throw()`           |
| **Date/Time**      | `frappe.datetime.get_today()`, `frappe.datetime.add_days()`            |
| **Navigation**     | `frappe.set_route()`, `frappe.get_route()`                             |
| **Utilities**      | `__()`, `format_currency()`, `frappe.utils.debounce()`                 |

### `frm` Object - Key Methods

| Category        | Methods                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **Document**      | `frm.doc`, `frm.is_new()`, `frm.is_dirty()`, `frm.reload_doc()`        |
| **Fields**        | `frm.set_value()`, `frm.get_value()`, `frm.toggle_display()`           |
| **Queries**       | `frm.set_query()`                                                      |
| **Buttons**       | `frm.add_custom_button()`, `frm.remove_custom_button()`                |
| **Child Tables**  | `frm.add_child()`, `frm.clear_table()`, `frm.refresh_field('items')`   |
| **Save/Submit**   | `frm.save()`, `frm.submit()`, `frm.cancel()`                           |
| **UI**            | `frm.set_intro()`, `frm.dashboard.add_indicator()`, `frm.set_focus()`  |
| **Server Calls**  | `frm.call()`                                                           |
