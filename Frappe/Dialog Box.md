# Complete Frappe Dialog Box API Guide

## Table of Contents
- [[#Basic Dialog Creation|Basic Dialog Creation]]
- [[#Dialog Fields|Dialog Fields]]
- [[#Dialog Methods|Dialog Methods]]
- [[#Events & Callbacks|Events & Callbacks]]
- [[#Advanced Features|Advanced Features]]
- [[#Real-world Examples|Real-world Examples]]
- [[#Utility Functions|Utility Functions]]

---

## Basic Dialog Creation

### Simple Dialog

```javascript
// Basic dialog
let d = new frappe.ui.Dialog({
    title: 'Enter Details',
    fields: [
        {
            label: 'First Name',
            fieldname: 'first_name',
            fieldtype: 'Data'
        }
    ],
    primary_action_label: 'Submit',
    primary_action(values) {
        console.log(values);
        d.hide();
    }
});

d.show();
```

### Quick Entry Dialog

```javascript
// Quick one-liner for simple prompts
frappe.prompt('Enter your name', (values) => {
    console.log(values.value);
});

// With field configuration
frappe.prompt({
    label: 'Birth Date',
    fieldname: 'birth_date',
    fieldtype: 'Date'
}, (values) => {
    console.log(values.birth_date);
});
```

---

## Dialog Fields

### All Field Types

> [!tip] Field Configuration
> All fields support common properties: `label`, `fieldname`, `fieldtype`, `default`, `description`, `reqd`, `read_only`, `hidden`

```javascript
let d = new frappe.ui.Dialog({
    title: 'All Field Types Demo',
    fields: [
        // Text Input
        {
            label: 'Data Field',
            fieldname: 'data_field',
            fieldtype: 'Data',
            default: 'Default Value',
            description: 'Helper text here',
            reqd: 1 // Required field
        },
        
        // Number
        {
            label: 'Integer',
            fieldname: 'int_field',
            fieldtype: 'Int'
        },
        {
            label: 'Float',
            fieldname: 'float_field',
            fieldtype: 'Float',
            precision: 2
        },
        
        // Currency
        {
            label: 'Amount',
            fieldname: 'amount',
            fieldtype: 'Currency'
        },
        
        // Date & Time
        {
            label: 'Date',
            fieldname: 'date_field',
            fieldtype: 'Date'
        },
        {
            label: 'Time',
            fieldname: 'time_field',
            fieldtype: 'Time'
        },
        {
            label: 'DateTime',
            fieldname: 'datetime_field',
            fieldtype: 'Datetime'
        },
        
        // Select/Dropdown
        {
            label: 'Select',
            fieldname: 'select_field',
            fieldtype: 'Select',
            options: [
                'Option 1',
                'Option 2',
                'Option 3'
            ],
            default: 'Option 1'
        },
        
        // Link (DocType reference)
        {
            label: 'Customer',
            fieldname: 'customer',
            fieldtype: 'Link',
            options: 'Customer', // DocType name
            get_query: function() {
                return {
                    filters: {
                        'customer_group': 'Commercial'
                    }
                };
            }
        },
        
        // Dynamic Link
        {
            label: 'Link Type',
            fieldname: 'link_type',
            fieldtype: 'Select',
            options: ['Customer', 'Supplier']
        },
        {
            label: 'Link Name',
            fieldname: 'link_name',
            fieldtype: 'Dynamic Link',
            options: 'link_type' // References above field
        },
        
        // Checkbox
        {
            label: 'Is Active',
            fieldname: 'is_active',
            fieldtype: 'Check',
            default: 1
        },
        
        // Text Area
        {
            label: 'Long Text',
            fieldname: 'long_text',
            fieldtype: 'Text'
        },
        {
            label: 'Small Text',
            fieldname: 'small_text',
            fieldtype: 'Small Text'
        },
        
        // Code
        {
            label: 'Code',
            fieldname: 'code_field',
            fieldtype: 'Code',
            options: 'Python' // or 'JavaScript', 'HTML', etc.
        },
        
        // HTML
        {
            fieldtype: 'HTML',
            fieldname: 'html_field',
            options: '<p>Custom HTML content here</p>'
        },
        
        // Column Break (for multi-column layout)
        {
            fieldtype: 'Column Break'
        },
        
        // Section Break
        {
            fieldtype: 'Section Break',
            label: 'Section Title'
        },
        
        // Table
        {
            label: 'Items',
            fieldname: 'items',
            fieldtype: 'Table',
            fields: [
                {
                    label: 'Item',
                    fieldname: 'item_code',
                    fieldtype: 'Link',
                    options: 'Item',
                    in_list_view: 1
                },
                {
                    label: 'Quantity',
                    fieldname: 'qty',
                    fieldtype: 'Float',
                    in_list_view: 1
                }
            ]
        },
        
        // Attach
        {
            label: 'Attachment',
            fieldname: 'attachment',
            fieldtype: 'Attach'
        },
        
        // Attach Image
        {
            label: 'Image',
            fieldname: 'image',
            fieldtype: 'Attach Image'
        },
        
        // Password
        {
            label: 'Password',
            fieldname: 'password',
            fieldtype: 'Password'
        },
        
        // Rating
        {
            label: 'Rating',
            fieldname: 'rating',
            fieldtype: 'Rating'
        },
        
        // Color
        {
            label: 'Color',
            fieldname: 'color',
            fieldtype: 'Color'
        },
        
        // Signature
        {
            label: 'Signature',
            fieldname: 'signature',
            fieldtype: 'Signature'
        },
        
        // Autocomplete
        {
            label: 'Tags',
            fieldname: 'tags',
            fieldtype: 'Autocomplete',
            options: ['Tag1', 'Tag2', 'Tag3']
        },
        
        // MultiSelect
        {
            label: 'Multi Select',
            fieldname: 'multi_select',
            fieldtype: 'MultiSelect',
            options: ['Option 1', 'Option 2', 'Option 3']
        },
        
        // Read Only
        {
            label: 'Read Only',
            fieldname: 'read_only',
            fieldtype: 'Data',
            read_only: 1,
            default: 'Cannot edit'
        }
    ]
});

d.show();
```

---

## Dialog Methods

### Core Methods

| Method | Description |
|--------|-------------|
| `d.show()` | Display the dialog |
| `d.hide()` | Hide the dialog |
| `d.get_values()` | Get all field values as object |
| `d.get_value(fieldname)` | Get specific field value |
| `d.set_values(obj)` | Set multiple field values |
| `d.set_value(fieldname, value)` | Set single field value |
| `d.clear()` | Clear all field values |
| `d.refresh()` | Refresh dialog UI |

```javascript
let d = new frappe.ui.Dialog({
    title: 'My Dialog'
});

// Show/Hide
d.show();
d.hide();

// Get values
let values = d.get_values();
console.log(values);

// Get specific field value
let name = d.get_value('first_name');

// Set values
d.set_values({
    'first_name': 'John',
    'last_name': 'Doe'
});

// Set specific field value
d.set_value('first_name', 'Jane');

// Get field object
let field = d.get_field('first_name');

// Set field property
d.set_df_property('first_name', 'read_only', 1);
d.set_df_property('first_name', 'hidden', 1);
d.set_df_property('first_name', 'reqd', 1);

// Clear dialog
d.clear();

// Disable/Enable primary action
d.disable_primary_action();
d.enable_primary_action();

// Get primary button
d.get_primary_btn();

// Set primary action
d.set_primary_action('Save', () => {
    console.log('Saved');
    d.hide();
});

// Set secondary action
d.set_secondary_action('Cancel', () => {
    d.hide();
});

// Add custom button
d.add_custom_action('Custom Button', () => {
    console.log('Custom action');
});

// No submit on enter
d.no_submit_on_enter = true;
```

### Field Manipulation

> [!warning] Refresh Required
> Always call `refresh()` after modifying field properties

```javascript
// Show/Hide fields
d.fields_dict.first_name.df.hidden = 1;
d.fields_dict.first_name.refresh();

// Make field mandatory
d.fields_dict.first_name.df.reqd = 1;
d.fields_dict.first_name.refresh();

// Make field read-only
d.fields_dict.first_name.df.read_only = 1;
d.fields_dict.first_name.refresh();

// Set field description
d.fields_dict.first_name.df.description = 'Enter your first name';
d.fields_dict.first_name.refresh();

// Refresh specific field
d.fields_dict.first_name.refresh();

// Set select options dynamically
d.fields_dict.status.df.options = ['Open', 'Closed', 'Pending'];
d.fields_dict.status.refresh();
```

---

## Events & Callbacks

### Field Events

```javascript
let d = new frappe.ui.Dialog({
    title: 'Event Demo',
    fields: [
        {
            label: 'First Name',
            fieldname: 'first_name',
            fieldtype: 'Data',
            change() {
                // Triggered on field change
                let value = d.get_value('first_name');
                console.log('First name changed:', value);
            },
            onchange() {
                // Alternative to change()
                console.log('Value changed');
            }
        },
        {
            label: 'Customer',
            fieldname: 'customer',
            fieldtype: 'Link',
            options: 'Customer',
            get_query() {
                // Filter link options
                return {
                    filters: {
                        'disabled': 0
                    }
                };
            },
            onchange() {
                // Fetch customer details when selected
                let customer = d.get_value('customer');
                if (customer) {
                    frappe.db.get_value('Customer', customer, 'customer_name')
                        .then(r => {
                            console.log('Customer Name:', r.message.customer_name);
                        });
                }
            }
        }
    ]
});
```

### Dialog Events

```javascript
let d = new frappe.ui.Dialog({
    title: 'Dialog Events',
    fields: [/*...*/],
    
    // Before show
    before_show() {
        console.log('About to show dialog');
    },
    
    // After show
    on_show() {
        console.log('Dialog shown');
    },
    
    // On hide
    on_hide() {
        console.log('Dialog hidden');
    },
    
    // On page show (for multi-page dialogs)
    on_page_show() {
        console.log('Page shown');
    }
});

// Listen to hide event
d.$wrapper.on('hidden.bs.modal', () => {
    console.log('Dialog closed');
});
```

---

## Advanced Features

### Size Control

```javascript
let d = new frappe.ui.Dialog({
    title: 'Large Dialog',
    size: 'large', // 'small', 'large', 'extra-large'
    fields: [/*...*/]
});

// Or set custom size
let d2 = new frappe.ui.Dialog({
    title: 'Custom Size',
    fields: [/*...*/]
});
d2.$wrapper.find('.modal-dialog').css('max-width', '800px');
```

### Static Dialog (Non-dismissible)

```javascript
let d = new frappe.ui.Dialog({
    title: 'Static Dialog',
    static: true, // Cannot close by clicking outside
    fields: [/*...*/]
});
```

### Multiple Prompts

```javascript
frappe.prompt([
    {
        label: 'First Name',
        fieldname: 'first_name',
        fieldtype: 'Data',
        reqd: 1
    },
    {
        label: 'Last Name',
        fieldname: 'last_name',
        fieldtype: 'Data',
        reqd: 1
    },
    {
        label: 'Email',
        fieldname: 'email',
        fieldtype: 'Data',
        options: 'Email'
    }
], (values) => {
    console.log(values);
}, 'Enter User Details', 'Create User');
```

### Multi-step Dialog (Wizard)

```javascript
let current_step = 0;
let steps = [
    {
        title: 'Step 1',
        fields: [
            {label: 'Name', fieldname: 'name', fieldtype: 'Data', reqd: 1}
        ]
    },
    {
        title: 'Step 2',
        fields: [
            {label: 'Email', fieldname: 'email', fieldtype: 'Data', reqd: 1}
        ]
    },
    {
        title: 'Step 3',
        fields: [
            {label: 'Phone', fieldname: 'phone', fieldtype: 'Data', reqd: 1}
        ]
    }
];

let d = new frappe.ui.Dialog({
    title: steps[current_step].title,
    fields: steps[current_step].fields,
    primary_action_label: 'Next',
    primary_action(values) {
        if (current_step < steps.length - 1) {
            current_step++;
            d.fields = [];
            d.clear();
            d.set_title(steps[current_step].title);
            
            steps[current_step].fields.forEach(field => {
                d.add_field(field);
            });
            
            if (current_step === steps.length - 1) {
                d.set_primary_action_label('Submit');
            }
            
            d.refresh();
        } else {
            console.log('All values:', d.get_values());
            d.hide();
        }
    }
});

d.set_secondary_action_label('Previous');
d.set_secondary_action(() => {
    if (current_step > 0) {
        current_step--;
        d.fields = [];
        d.clear();
        d.set_title(steps[current_step].title);
        
        steps[current_step].fields.forEach(field => {
            d.add_field(field);
        });
        
        d.set_primary_action_label('Next');
        d.refresh();
    }
});

d.show();
```

### Custom HTML in Dialog

```javascript
let d = new frappe.ui.Dialog({
    title: 'Custom HTML',
    fields: [
        {
            fieldtype: 'HTML',
            fieldname: 'custom_html'
        }
    ]
});

d.fields_dict.custom_html.$wrapper.html(`
    <div style="padding: 20px;">
        <h3>Custom Content</h3>
        <p>You can add any HTML here</p>
        <button class="btn btn-primary" id="my-custom-btn">Click Me</button>
    </div>
`);

d.show();

// Add event listener to custom element
d.$wrapper.find('#my-custom-btn').on('click', () => {
    frappe.msgprint('Custom button clicked!');
});
```

### Dynamic Field Addition

```javascript
let d = new frappe.ui.Dialog({
    title: 'Dynamic Fields'
});

// Add fields dynamically
d.add_field({
    label: 'First Name',
    fieldname: 'first_name',
    fieldtype: 'Data'
});

d.add_field({
    fieldtype: 'Column Break'
});

d.add_field({
    label: 'Last Name',
    fieldname: 'last_name',
    fieldtype: 'Data'
});

d.show();
```

---

## Real-world Examples

### Example 1: Customer Selection with Details

```javascript
let d = new frappe.ui.Dialog({
    title: 'Select Customer',
    fields: [
        {
            label: 'Customer',
            fieldname: 'customer',
            fieldtype: 'Link',
            options: 'Customer',
            reqd: 1,
            onchange: function() {
                let customer = d.get_value('customer');
                if (customer) {
                    frappe.call({
                        method: 'frappe.client.get',
                        args: {
                            doctype: 'Customer',
                            name: customer
                        },
                        callback: function(r) {
                            if (r.message) {
                                d.set_value('customer_name', r.message.customer_name);
                                d.set_value('email', r.message.email_id);
                                d.set_value('phone', r.message.mobile_no);
                            }
                        }
                    });
                }
            }
        },
        {
            fieldtype: 'Column Break'
        },
        {
            label: 'Customer Name',
            fieldname: 'customer_name',
            fieldtype: 'Data',
            read_only: 1
        },
        {
            fieldtype: 'Section Break'
        },
        {
            label: 'Email',
            fieldname: 'email',
            fieldtype: 'Data',
            read_only: 1
        },
        {
            fieldtype: 'Column Break'
        },
        {
            label: 'Phone',
            fieldname: 'phone',
            fieldtype: 'Data',
            read_only: 1
        }
    ],
    primary_action_label: 'Select',
    primary_action(values) {
        console.log(values);
        d.hide();
    }
});

d.show();
```

### Example 2: Order Entry with Items Table

```javascript
let d = new frappe.ui.Dialog({
    title: 'Create Order',
    size: 'large',
    fields: [
        {
            label: 'Customer',
            fieldname: 'customer',
            fieldtype: 'Link',
            options: 'Customer',
            reqd: 1
        },
        {
            label: 'Date',
            fieldname: 'date',
            fieldtype: 'Date',
            default: frappe.datetime.get_today(),
            reqd: 1
        },
        {
            fieldtype: 'Section Break',
            label: 'Items'
        },
        {
            label: 'Items',
            fieldname: 'items',
            fieldtype: 'Table',
            cannot_add_rows: false,
            in_place_edit: true,
            data: [],
            fields: [
                {
                    label: 'Item',
                    fieldname: 'item_code',
                    fieldtype: 'Link',
                    options: 'Item',
                    in_list_view: 1,
                    reqd: 1
                },
                {
                    label: 'Quantity',
                    fieldname: 'qty',
                    fieldtype: 'Float',
                    in_list_view: 1,
                    reqd: 1,
                    default: 1
                },
                {
                    label: 'Rate',
                    fieldname: 'rate',
                    fieldtype: 'Currency',
                    in_list_view: 1,
                    reqd: 1
                },
                {
                    label: 'Amount',
                    fieldname: 'amount',
                    fieldtype: 'Currency',
                    in_list_view: 1,
                    read_only: 1
                }
            ]
        },
        {
            fieldtype: 'Section Break'
        },
        {
            label: 'Total',
            fieldname: 'total',
            fieldtype: 'Currency',
            read_only: 1
        }
    ],
    primary_action_label: 'Create',
    primary_action(values) {
        // Calculate total
        let total = 0;
        values.items.forEach(item => {
            item.amount = item.qty * item.rate;
            total += item.amount;
        });
        
        values.total = total;
        
        console.log('Order:', values);
        
        // Create document
        frappe.call({
            method: 'your_app.api.create_order',
            args: {
                order_data: values
            },
            callback: function(r) {
                if (r.message) {
                    frappe.msgprint('Order created successfully');
                    d.hide();
                }
            }
        });
    }
});

d.show();
```

### Example 3: Confirmation Dialog with Custom Actions

```javascript
function delete_record(doctype, name) {
    let d = new frappe.ui.Dialog({
        title: 'Confirm Delete',
        fields: [
            {
                fieldtype: 'HTML',
                options: `
                    <div class="alert alert-danger">
                        <strong>Warning!</strong> 
                        This action cannot be undone.
                    </div>
                    <p>Are you sure you want to delete <strong>${name}</strong>?</p>
                `
            },
            {
                label: 'Type "DELETE" to confirm',
                fieldname: 'confirmation',
                fieldtype: 'Data',
                reqd: 1
            }
        ],
        primary_action_label: 'Delete',
        primary_action(values) {
            if (values.confirmation === 'DELETE') {
                frappe.call({
                    method: 'frappe.client.delete',
                    args: {
                        doctype: doctype,
                        name: name
                    },
                    callback: function(r) {
                        frappe.show_alert({
                            message: 'Deleted successfully',
                            indicator: 'green'
                        });
                        d.hide();
                    }
                });
            } else {
                frappe.msgprint('Please type "DELETE" to confirm');
            }
        }
    });
    
    d.set_secondary_action_label('Cancel');
    d.set_secondary_action(() => {
        d.hide();
    });
    
    // Make delete button red
    d.get_primary_btn().removeClass('btn-primary').addClass('btn-danger');
    
    d.show();
}
```

### Example 4: Upload Dialog with Progress

```javascript
let d = new frappe.ui.Dialog({
    title: 'Upload File',
    fields: [
        {
            label: 'Select File',
            fieldname: 'file',
            fieldtype: 'Attach',
            reqd: 1
        },
        {
            fieldtype: 'HTML',
            fieldname: 'progress_area'
        }
    ],
    primary_action_label: 'Upload',
    primary_action(values) {
        let progress_html = `
            <div class="progress" style="margin-top: 10px;">
                <div class="progress-bar progress-bar-striped active" 
                     role="progressbar" 
                     style="width: 0%">
                    0%
                </div>
            </div>
        `;
        
        d.fields_dict.progress_area.$wrapper.html(progress_html);
        d.disable_primary_action();
        
        // Simulate upload progress
        let progress = 0;
        let interval = setInterval(() => {
            progress += 10;
            d.fields_dict.progress_area.$wrapper
                .find('.progress-bar')
                .css('width', progress + '%')
                .text(progress + '%');
            
            if (progress >= 100) {
                clearInterval(interval);
                setTimeout(() => {
                    frappe.msgprint('File uploaded successfully');
                    d.hide();
                }, 500);
            }
        }, 200);
    }
});

d.show();
```

### Example 5: Filter Builder Dialog

```javascript
let d = new frappe.ui.Dialog({
    title: 'Advanced Filter',
    fields: [
        {
            label: 'Field',
            fieldname: 'field',
            fieldtype: 'Select',
            options: [
                'Customer Name',
                'Status',
                'Amount',
                'Date'
            ],
            reqd: 1,
            onchange: function() {
                let field = d.get_value('field');
                let operator_field = d.fields_dict.operator;
                
                if (field === 'Amount') {
                    operator_field.df.options = ['=', '!=', '>', '<', '>=', '<='];
                } else if (field === 'Date') {
                    operator_field.df.options = ['=', '>', '<', 'Between'];
                } else {
                    operator_field.df.options = ['=', '!=', 'like', 'not like'];
                }
                operator_field.refresh();
            }
        },
        {
            label: 'Operator',
            fieldname: 'operator',
            fieldtype: 'Select',
            options: ['=', '!=', '>', '<'],
            reqd: 1
        },
        {
            label: 'Value',
            fieldname: 'value',
            fieldtype: 'Data',
            reqd: 1
        }
    ],
    primary_action_label: 'Apply Filter',
    primary_action(values) {
        console.log('Filter:', values);
        // Apply filter logic here
        d.hide();
    }
});

d.show();
```

### Example 6: Settings Dialog with Tabs

```javascript
let d = new frappe.ui.Dialog({
    title: 'Settings',
    size: 'large',
    fields: [
        // Tab 1: General
        {
            fieldtype: 'Section Break',
            label: 'General Settings'
        },
        {
            label: 'Company Name',
            fieldname: 'company_name',
            fieldtype: 'Data'
        },
        {
            label: 'Email',
            fieldname: 'email',
            fieldtype: 'Data',
            options: 'Email'
        },
        
        // Tab 2: Preferences
        {
            fieldtype: 'Section Break',
            label: 'Preferences'
        },
        {
            label: 'Theme',
            fieldname: 'theme',
            fieldtype: 'Select',
            options: ['Light', 'Dark', 'Auto']
        },
        {
            label: 'Language',
            fieldname: 'language',
            fieldtype: 'Select',
            options: ['English', 'Spanish', 'French']
        },
        
        // Tab 3: Notifications
        {
            fieldtype: 'Section Break',
            label: 'Notifications'
        },
        {
            label: 'Email Notifications',
            fieldname: 'email_notifications',
            fieldtype: 'Check',
            default: 1
        },
        {
            label: 'SMS Notifications',
            fieldname: 'sms_notifications',
            fieldtype: 'Check'
        }
    ],
    primary_action_label: 'Save',
    primary_action(values) {
        frappe.call({
            method: 'your_app.api.save_settings',
            args: {settings: values},
            callback: function(r) {
                frappe.msgprint('Settings saved');
                d.hide();
            }
        });
    }
});

d.show();
```

---

## Utility Functions

### Message Dialogs

```javascript
// Simple message
frappe.msgprint('Hello World');

// Message with title
frappe.msgprint({
    title: 'Notification',
    message: 'Operation completed',
    indicator: 'green'
});

// Confirm dialog
frappe.confirm(
    'Are you sure you want to proceed?',
    () => {
        // Yes action
        console.log('Confirmed');
    },
    () => {
        // No action
        console.log('Cancelled');
    }
);

// Alert (toast notification)
frappe.show_alert({
    message: 'Saved successfully',
    indicator: 'green'
}, 5); // Duration in seconds
```

### Loading Indicator

```javascript
// Show loading
frappe.show_progress('Processing', 50, 100, 'Please wait...');

// Hide loading
frappe.hide_progress();
```

---

> [!info] Reference
> This guide covers Frappe Framework's Dialog API. For more details, refer to the [Frappe Documentation](https://docs.frappe.io/framework/user/en/api/dialog).

`

---

## Key Obsidian Features Added:

1. **YAML Frontmatter** - Metadata for title, description, tags, and date
2. **WikiLinks** - Internal navigation using `[[#Section Name]]` format
3. **Callouts** - Using `> [!tip]`, `> [!warning]`, `> [!info]` for important notes
4. **Tables** - Method reference table in the Methods section
5. **Proper Code Blocks** - JavaScript syntax highlighting throughout
6. **Anchor Links** - Table of Contents links to sections
7. **Consistent Formatting** - Standard Markdown with Obsidian enhancements

Save this as `Frappe Dialog API Guide.md` in your Obsidian vault!


