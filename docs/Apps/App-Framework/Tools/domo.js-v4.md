# ryuu.js v4 and Earlier Documentation

> A powerful JavaScript SDK for building custom applications within the Domo platform

[![npm version](https://img.shields.io/npm/v/ryuu.js.svg)](https://www.npmjs.com/package/ryuu.js)
[![License](https://img.shields.io/badge/license-SEE%20LICENSE-blue.svg)](./LICENSE)
[![TypeScript](https://img.shields.io/badge/TypeScript-Ready-blue.svg)](https://www.typescriptlang.org/)

> **Note:** This documentation is for ryuu.js v4.x and earlier. For the latest version, see [v5+ Documentation](./domo.js-v5.md).

## Quick Start

Get started with ryuu.js in just a few lines:

```javascript
import domo from 'ryuu.js';

// Fetch data from a dataset
const data = await domo.get('/data/v1/sales');
console.log(data); // Array of objects with your data

// Listen for dataset updates
domo.onDataUpdate((alias) => {
  console.log(`Dataset ${alias} was updated!`);
  // Refresh your visualization
});

// Listen for filter changes
domo.onFiltersUpdate((filters) => {
  console.log('Filters changed:', filters);
  // Apply filters to your data
});
```

---

## Features

- **HTTP API Access** - Authenticated requests to Domo datasets, datastores, and APIs
- **Real-time Events** - Listen for dataset updates, filter changes, and variable updates
- **Filter Management** - Get and set page-level filters programmatically
- **Variable Management** - Access and update page variables
- **Custom App Data** - Send custom data between apps on the same page
- **Mobile Support** - iOS and Android compatibility
- **TypeScript Ready** - Type definitions included
- **Zero Dependencies** - Minimal bundle size

---

## Installation

### NPM

```bash
npm install ryuu.js@4
```

Then import in your application:

```javascript
// ES modules
import domo from 'ryuu.js';

// CommonJS
const domo = require('ryuu.js');
```

### CDN / Script Tag

```html
<script src="https://unpkg.com/ryuu.js@4"></script>
<script>
  // domo is available globally
  domo.get('/data/v1/sales').then(data => {
    console.log(data);
  });
</script>
```

### TypeScript

TypeScript definitions are included automatically:

```typescript
import domo, { Filter, Variable, RequestOptions } from 'ryuu.js';
```

---

## Core Concepts

### Communication Architecture

Your custom app runs in an iframe within the Domo platform. ryuu.js v4 establishes bidirectional communication using:

- **window.postMessage API** - Primary communication mechanism for desktop/web
- **webkit.messageHandlers** - iOS native integration
- **Global objects** - Android integration

### Environment Context

Access information about the current user and environment via `domo.env`:

```javascript
console.log(domo.env.userId);      // Current user ID
console.log(domo.env.customer);    // Customer name
console.log(domo.env.pageId);      // Current page ID
console.log(domo.env.locale);      // Locale (e.g., 'en-US')
console.log(domo.env.platform);    // Platform (e.g., 'desktop', 'mobile')
```

**Security Note:** Environment properties come from URL parameters and can be spoofed. Always verify with the API for secure operations:

```javascript
const authenticatedUser = await domo.get('/domo/environment/v1/');
```

---

## API Reference

### HTTP Methods

All HTTP methods return Promises and support multiple data formats.

#### `domo.get(url, options?)`

Fetch data from a Domo dataset or API endpoint.

**Parameters:**
- `url` (string) - API endpoint URL
- `options` (object, optional) - Request options

**Returns:** `Promise<ResponseBody>`

**Basic Usage:**

```javascript
// Returns array of objects by default
const data = await domo.get('/data/v1/sales');
console.log(data); // [{ id: 1, amount: 100, ... }, ...]
```

**Format Options:**

```javascript
// CSV format
const csv = await domo.get('/data/v1/sales', { format: 'csv' });

// Array of arrays with metadata
const arrayData = await domo.get('/data/v1/sales', { format: 'array-of-arrays' });
console.log(arrayData.columns);  // ['id', 'amount', 'date']
console.log(arrayData.rows);     // [[1, 100, '2024-01-01'], ...]

// Excel format (returns Blob)
const excel = await domo.get('/data/v1/sales', { format: 'excel' });
```

**Supported Formats:**
- `'array-of-objects'` (default) - Returns `ObjectResponseBody[]`
- `'array-of-arrays'` - Returns `ArrayResponseBody` with metadata
- `'csv'` - Returns CSV string
- `'excel'` - Returns Excel Blob
- `'plain'` - Returns plain text string

**Query Parameters:**

```javascript
const data = await domo.get('/data/v1/sales', {
  query: {
    limit: 100,
    offset: 0,
    fields: 'id,amount,date'
  }
});
```

---

#### `domo.getAll(urls, options?)`

Fetch multiple datasets or endpoints in parallel.

**Parameters:**
- `urls` (string[]) - Array of API endpoint URLs
- `options` (object, optional) - Request options applied to all requests

**Returns:** `Promise<ResponseBody[]>`

**Example:**

```javascript
const [sales, inventory, customers] = await domo.getAll([
  '/data/v1/sales',
  '/data/v1/inventory',
  '/data/v1/customers'
]);
```

---

#### `domo.post(url, body?, options?)`

Send a POST request to create data.

**Parameters:**
- `url` (string) - API endpoint URL
- `body` (object | string, optional) - Request body
- `options` (object, optional) - Request options

**Returns:** `Promise<ResponseBody>`

**Example:**

```javascript
// Create a document in Domo DataStore
const result = await domo.post(
  '/domo/datastores/v1/collections/users/documents/',
  {
    name: 'John Doe',
    email: 'john@example.com',
    role: 'Admin'
  }
);
```

---

#### `domo.put(url, body?, options?)`

Send a PUT request to update data.

**Example:**

```javascript
// Update an existing document
const result = await domo.put(
  '/domo/datastores/v1/collections/users/documents/abc123',
  {
    name: 'Jane Doe',
    role: 'Manager'
  }
);
```

---

#### `domo.delete(url, options?)`

Send a DELETE request to remove data.

**Example:**

```javascript
// Delete a document
await domo.delete('/domo/datastores/v1/collections/users/documents/abc123');
```

---

### Event Listeners

Event listeners enable real-time reactivity to changes in the Domo platform. All listener methods return an **unsubscribe function**.

**Important: Message Acknowledgement**

Event messages from the Domo platform require acknowledgement. If a message is not acknowledged, Domo will force a refresh of your app. The ryuu.js library **automatically handles acknowledgement** for all event listeners documented below, so you don't need to worry about this when using the standard event listener methods.

However, if you are implementing custom event handling or bypassing the library's event system, you must manually acknowledge messages to prevent unwanted app refreshes.

**Custom Implementation Examples (v4 postMessage Pattern):**

If you're implementing custom event handling in v4, you need to send an acknowledgement by posting a JSON stringified acknowledgement back to the parent. The acknowledgement format varies by event type:

**For onDataUpdate events:**

```javascript
// Custom event handler example (not needed when using domo.onDataUpdate)
window.addEventListener('message', (event) => {
  // Verify origin for security
  if (!event.origin.includes('domo.com')) return;

  if (typeof event.data === 'string' && event.data.length > 0) {
    try {
      const message = JSON.parse(event.data);

      // Check if this is a data update message
      if (message.hasOwnProperty('alias')) {
        // Send acknowledgement - must include BOTH event and alias for onDataUpdate
        const ack = JSON.stringify({ event: 'ack', alias: message.alias });
        if (event.source && typeof event.source.postMessage === 'function') {
          event.source.postMessage(ack, event.origin);
        }

        // Handle your custom logic
        console.log('Dataset updated:', message.alias);
      }
    } catch (err) {
      console.warn('Error parsing message:', err);
    }
  }
});
```

**For onFiltersUpdate and onAppData events:**

```javascript
// Custom event handler example (not needed when using domo.onFiltersUpdate or domo.onAppData)
window.addEventListener('message', (event) => {
  // Verify origin for security
  if (!event.origin.includes('domo.com')) return;

  if (typeof event.data === 'string' && event.data.length > 0) {
    try {
      const message = JSON.parse(event.data);

      // Check if this is a filters update or app data message
      if (message.hasOwnProperty('filters') || message.hasOwnProperty('appData')) {
        // Send acknowledgement - only event property needed (no alias)
        const ack = JSON.stringify({ event: 'ack' });
        if (event.source && typeof event.source.postMessage === 'function') {
          event.source.postMessage(ack, event.origin);
        }

        // Handle your custom logic
        if (message.filters) {
          console.log('Filters updated:', message.filters);
        }
        if (message.appData) {
          console.log('App data received:', message.appData);
        }
      }
    } catch (err) {
      console.warn('Error parsing message:', err);
    }
  }
});
```

**Acknowledgement Summary:**
- **onDataUpdate**: `{ event: 'ack', alias: '<dataset-alias>' }`
- **onFiltersUpdate**: `{ event: 'ack' }`
- **onAppData**: `{ event: 'ack' }`

**Note:** This manual acknowledgement is only necessary for custom implementations. When using the standard `domo.onDataUpdate()`, `domo.onFiltersUpdate()`, and `domo.onAppData()` methods, acknowledgement is handled automatically.

---

#### `domo.onDataUpdate(callback)`

Listen for dataset update events.

**Parameters:**
- `callback` (function) - Called when a dataset is updated
  - Receives: `datasetAlias` (string) - Alias of the updated dataset

**Returns:** `function` - Unsubscribe function

**Example:**

```javascript
const unsubscribe = domo.onDataUpdate((datasetAlias) => {
  console.log(`Dataset ${datasetAlias} was updated`);
  // Reload data for this dataset
  loadData();
});

// Later, stop listening
unsubscribe();
```

---

#### `domo.onFiltersUpdate(callback)`

Listen for page filter changes.

**Parameters:**
- `callback` (function) - Called when filters change
  - Receives: `filters` (Filter[]) - Array of filter objects

**Returns:** `function` - Unsubscribe function

**Example:**

```javascript
domo.onFiltersUpdate((filters) => {
  console.log('Filters updated:', filters);

  // Find specific filter
  const categoryFilter = filters.find(f => f.column === 'category');

  if (categoryFilter) {
    console.log('Category filter:', categoryFilter.values);
    // Apply filter to your visualization
    applyFilters(categoryFilter.values);
  }
});
```

**Filter Object Structure:**

```javascript
{
  column: "category",              // Column name being filtered
  operator: "IN",                  // Filter operator
  values: ["ALERT", "WARNING"],    // Array of filter values
  dataType: "STRING",              // Data type: STRING, NUMERIC, DATE, DATETIME
  dataSourceId: "46d91556-...",    // Source dataset ID (optional)
  label: "category"                // Display label (optional)
}
```

**Supported Operators:**

**String Operators:**
- `"IN"` / `"NOT_IN"`
- `"CONTAINS"` / `"NOT_CONTAINS"`
- `"STARTS_WITH"` / `"NOT_STARTS_WITH"`
- `"ENDS_WITH"` / `"NOT_ENDS_WITH"`

**Numeric/Date Operators:**
- `"EQUALS"` / `"NOT_EQUALS"`
- `"GREATER_THAN"` / `"GREAT_THAN_EQUALS_TO"`
- `"LESS_THAN"` / `"LESS_THAN_EQUALS_TO"`
- `"BETWEEN"`

---

#### `domo.onAppData(callback)`

Listen for custom app data updates.

**Parameters:**
- `callback` (function) - Called when app data is received
  - Receives: `data` (any) - Custom data object

**Returns:** `function` - Unsubscribe function

**Example:**

```javascript
domo.onAppData((data) => {
  console.log('Received app data:', data);

  // Handle custom data from other apps
  if (data.action === 'highlight') {
    highlightRow(data.rowId);
  }
});
```

---

### Emitters

Emitters send messages to the parent Domo platform to trigger actions or update state.

#### `domo.filterContainer(filters, pageStateUpdate?)`

Update page-level filters programmatically.

**Parameters:**
- `filters` (Filter[]) - Array of filter objects
- `pageStateUpdate` (boolean, optional) - Whether to update page state (default: true)

**Returns:** `void`

**Example:**

```javascript
// Basic usage
domo.filterContainer([
  {
    column: 'category',
    operator: 'IN',
    values: ['ALERT', 'WARNING'],
    dataType: 'STRING'
  },
  {
    column: 'amount',
    operator: 'GREATER_THAN',
    values: [1000],
    dataType: 'NUMERIC'
  }
]);
```

---

#### `domo.sendVariables(variables)`

Update page variables programmatically.

**Parameters:**
- `variables` (Variable[]) - Array of variable objects

**Returns:** `void`

**Example:**

```javascript
domo.sendVariables([
  {
    functionId: 123,
    value: 100
  },
  {
    functionId: 124,
    value: 'dark'
  }
]);
```

**Variable Object Structure:**

```javascript
{
  functionId: number,  // Variable function ID from Domo
  value: any          // New value for the variable
}
```

---

#### `domo.sendAppData(data)`

Send custom app data to other apps on the same page.

**Parameters:**
- `data` (any) - Custom data object

**Returns:** `void`

**Example:**

```javascript
// Send custom data
domo.sendAppData({
  action: 'highlight',
  rowId: 123,
  timestamp: Date.now()
});
```

---

### Navigation

Navigation in v4 is available through the parent window. For programmatic navigation, you can use:

```javascript
// Navigate within Domo
window.parent.postMessage({
  event: 'navigate',
  data: { route: '/page/123456789' }
}, '*');
```

---

### Environment

#### `domo.env`

Access environment information about the current context.

**Type:** `QueryParams` (object)

**Available Properties:**

```javascript
domo.env.pageId        // Current page ID
domo.env.userId        // Current user ID
domo.env.customer      // Customer name
domo.env.locale        // Locale (e.g., 'en-US')
domo.env.environment   // Environment (e.g., 'dev3', 'prod')
domo.env.platform      // Platform (e.g., 'desktop', 'mobile')
```

---

## TypeScript Support

ryuu.js v4 includes TypeScript definitions.

### Importing Types

```typescript
import domo, {
  // Interfaces
  Filter,
  Variable,
  RequestOptions,
  ObjectResponseBody,
  ArrayResponseBody,
  QueryParams,

  // Enums
  DataFormats,
  DomoDataTypes,
  RequestMethods
} from 'ryuu.js';
```

### Typed Requests

```typescript
// Type-safe request options
const options: RequestOptions<'array-of-objects'> = {
  format: 'array-of-objects',
  query: { limit: 100 }
};

// Return type is automatically inferred
const data: ObjectResponseBody[] = await domo.get('/data/v1/sales', options);

// Format-specific types
const csv: string = await domo.get('/data/v1/sales', { format: 'csv' });
const arrays: ArrayResponseBody = await domo.get('/data/v1/sales', { format: 'array-of-arrays' });
```

### Typed Filters

```typescript
const filters: Filter[] = [
  {
    column: 'category',
    operator: 'IN',
    values: ['ALERT', 'WARNING'],
    dataType: 'STRING'
  }
];

domo.filterContainer(filters);
```

---

## Mobile Platform Support

ryuu.js v4 provides support for iOS and Android mobile platforms.

### iOS Integration

On iOS, ryuu.js uses **webkit.messageHandlers** for native communication:

```javascript
// Automatically handled by ryuu.js
domo.filterContainer(filters);
// Internally uses: webkit.messageHandlers.domofilter.postMessage()

domo.sendVariables(variables);
// Internally uses: webkit.messageHandlers.domovariable.postMessage()
```

### Android Integration

On Android, ryuu.js uses global objects injected by the native app:

```javascript
// Automatically handled by ryuu.js
domo.sendVariables(variables);
// Internally uses: window.domovariable.postMessage()

domo.filterContainer(filters);
// Internally uses: window.domofilter.postMessage()
```

---

## Error Handling

All HTTP methods return Promises. Use `try/catch` with async/await for error handling.

### Basic Error Handling

```javascript
try {
  const data = await domo.get('/data/v1/sales');
  console.log('Data loaded:', data);
} catch (error) {
  console.error('Failed to load data:', error);
}
```

### Error Object Structure

```javascript
try {
  const data = await domo.get('/data/v1/nonexistent');
} catch (error) {
  console.error('Status:', error.status);         // 404
  console.error('Status Text:', error.statusText); // 'Not Found'
  console.error('Message:', error.message);        // Error description
}
```

### Handling Specific Error Types

```javascript
try {
  const data = await domo.get('/data/v1/sales');
} catch (error) {
  if (error.status === 404) {
    console.error('Dataset not found');
  } else if (error.status === 403) {
    console.error('Access denied');
  } else if (error.status === 401) {
    console.error('Authentication failed');
  } else if (error.status >= 500) {
    console.error('Server error');
  }
}
```

---

## Complete Example

Here's a comprehensive example showing a real-world custom app:

```javascript
import domo from 'ryuu.js';

class SalesDashboard {
  constructor() {
    this.data = [];
    this.filters = [];
    this.initialize();
  }

  async initialize() {
    // Set up event listeners
    this.setupEventListeners();

    // Load initial data
    await this.loadData();

    // Render dashboard
    this.render();
  }

  setupEventListeners() {
    // Listen for dataset updates
    domo.onDataUpdate((datasetAlias) => {
      console.log(`Dataset ${datasetAlias} updated`);
      this.loadData();
    });

    // Listen for filter changes
    domo.onFiltersUpdate((filters) => {
      console.log('Filters updated:', filters);
      this.filters = filters;
      this.applyFilters();
    });

    // Set up UI event handlers
    document.getElementById('refreshBtn').addEventListener('click', () => {
      this.loadData();
    });

    document.getElementById('exportBtn').addEventListener('click', () => {
      this.exportData();
    });

    document.getElementById('filterBtn').addEventListener('click', () => {
      this.updateFilters();
    });
  }

  async loadData() {
    try {
      // Load multiple datasets in parallel
      const [sales, customers, products] = await domo.getAll([
        '/data/v1/sales',
        '/data/v1/customers',
        '/data/v1/products'
      ]);

      this.data = {
        sales,
        customers,
        products
      };

      this.render();
    } catch (error) {
      console.error('Failed to load data:', error);
      this.showError('Unable to load dashboard data');
    }
  }

  applyFilters() {
    // Find category filter
    const categoryFilter = this.filters.find(f => f.column === 'category');

    if (categoryFilter && categoryFilter.values.length > 0) {
      // Filter local data
      const filteredSales = this.data.sales.filter(sale =>
        categoryFilter.values.includes(sale.category)
      );

      this.renderSales(filteredSales);
    } else {
      this.renderSales(this.data.sales);
    }
  }

  updateFilters() {
    // Get selected categories from UI
    const selectedCategories = Array.from(
      document.querySelectorAll('.category-checkbox:checked')
    ).map(cb => cb.value);

    // Update page filters
    domo.filterContainer([
      {
        column: 'category',
        operator: 'IN',
        values: selectedCategories,
        dataType: 'STRING'
      }
    ]);
  }

  async exportData() {
    try {
      // Export as CSV
      const csv = await domo.get('/data/v1/sales', { format: 'csv' });

      // Download file
      const blob = new Blob([csv], { type: 'text/csv' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `sales-export-${Date.now()}.csv`;
      a.click();
      URL.revokeObjectURL(url);

      console.log('Export successful');
    } catch (error) {
      console.error('Export failed:', error);
      this.showError('Failed to export data');
    }
  }

  render() {
    if (!this.data.sales) return;

    // Render sales chart
    this.renderSales(this.data.sales);

    // Render customer stats
    this.renderCustomerStats(this.data.customers);

    // Render product list
    this.renderProducts(this.data.products);
  }

  renderSales(sales) {
    const salesContainer = document.getElementById('salesChart');

    // Calculate totals
    const total = sales.reduce((sum, sale) => sum + sale.amount, 0);
    const count = sales.length;
    const average = total / count;

    salesContainer.innerHTML = `
      <div class="stats">
        <div class="stat">
          <h3>Total Sales</h3>
          <p>${total.toLocaleString()}</p>
        </div>
        <div class="stat">
          <h3>Number of Sales</h3>
          <p>${count.toLocaleString()}</p>
        </div>
        <div class="stat">
          <h3>Average Sale</h3>
          <p>${average.toFixed(2)}</p>
        </div>
      </div>
    `;
  }

  renderCustomerStats(customers) {
    const customerContainer = document.getElementById('customerStats');

    customerContainer.innerHTML = `
      <h3>Total Customers: ${customers.length}</h3>
      <ul>
        ${customers.slice(0, 10).map(c => `
          <li>${c.name} - ${c.email}</li>
        `).join('')}
      </ul>
    `;
  }

  renderProducts(products) {
    const productContainer = document.getElementById('productList');

    productContainer.innerHTML = `
      <table>
        <thead>
          <tr>
            <th>Product</th>
            <th>Price</th>
            <th>Stock</th>
          </tr>
        </thead>
        <tbody>
          ${products.map(p => `
            <tr>
              <td>${p.name}</td>
              <td>${p.price}</td>
              <td>${p.stock}</td>
            </tr>
          `).join('')}
        </tbody>
      </table>
    `;
  }

  showError(message) {
    const errorContainer = document.getElementById('error');
    errorContainer.textContent = message;
    errorContainer.style.display = 'block';

    setTimeout(() => {
      errorContainer.style.display = 'none';
    }, 5000);
  }
}

// Initialize app
new SalesDashboard();
```

---

## Upgrading to v5+

If you're using v4 and want to upgrade to the latest v5+, here's what you need to know.

### Breaking Changes

v5 introduced method renames for consistency. All old method names are still supported as deprecated aliases, so your code will continue to work.

### Method Name Changes

| v4 Method | v5 Method | Description |
|-----------|-----------|-------------|
| `domo.onDataUpdate` | `Domo.onDataUpdated` | Listen for dataset changes |
| `domo.onFiltersUpdate` | `Domo.onFiltersUpdated` | Listen for filter changes |
| `domo.onAppData` | `Domo.onAppDataUpdated` | Listen for app data changes |
| `domo.filterContainer` | `Domo.requestFiltersUpdate` | Set page filters |
| `domo.sendVariables` | `Domo.requestVariablesUpdate` | Update page variables |
| `domo.sendAppData` | `Domo.requestAppDataUpdate` | Send custom app data |

### Class Name Change

The main class was renamed from `domo` (lowercase) to `Domo` (PascalCase):

```javascript
// v4
import domo from 'ryuu.js';
domo.get('/data/v1/sales');

// v5
import Domo from 'ryuu.js';
Domo.get('/data/v1/sales');
```

### New Features in v5

- **Request Tracking** - `Domo.getRequests()` and `Domo.getRequest(id)` for tracking async operations
- **Navigate Method** - `Domo.navigate(route, isNewWindow)` for programmatic navigation
- **Variables Event** - `Domo.onVariablesUpdated()` listener
- **Extend Method** - `Domo.extend()` for overriding methods
- **Better Mobile Support** - Enhanced iOS and Android integration
- **Fetch API** - Modern replacement for XMLHttpRequest
- **MessageChannel** - More reliable bidirectional communication

### Migration Steps

1. **Update Package:**
   ```bash
   npm install ryuu.js@latest
   ```

2. **Update Imports:**
   ```javascript
   // Old
   import domo from 'ryuu.js';

   // New
   import Domo from 'ryuu.js';
   ```

3. **Update Method Names (Optional):**
   ```javascript
   // Old (still works)
   domo.onDataUpdate((alias) => { ... });
   domo.onFiltersUpdate((filters) => { ... });
   domo.filterContainer(filters);

   // New (recommended)
   Domo.onDataUpdated((alias) => { ... });
   Domo.onFiltersUpdated((filters) => { ... });
   Domo.requestFiltersUpdate(filters);
   ```

4. **Test Thoroughly** - After migration, test all functionality.

For complete v5 documentation, see [v5+ Documentation](./domo.js-v5.md).

---

## Contributing

Contributions are welcome! Please follow these guidelines:

1. **Fork the repository**
2. **Create a feature branch:** `git checkout -b feature/my-feature`
3. **Make your changes**
4. **Add tests** for new functionality
5. **Run tests:** `npm test`
6. **Build:** `npm run build`
7. **Commit:** `git commit -m "Add my feature"`
8. **Push:** `git push origin feature/my-feature`
9. **Open a Pull Request**

---

## License

SEE LICENSE IN [LICENSE](./LICENSE)

Copyright (c) Domo

---

[← Back to main domo.js page](./domo.js.md)
