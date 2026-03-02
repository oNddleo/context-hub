---
name: serverless-db
description: "Fauna Database JavaScript SDK for serverless database operations with GraphQL and distributed queries"
metadata:
  languages: "javascript"
  versions: "2.5.0"
  updated-on: "2026-03-02"
  source: maintainer
  tags: "fauna,serverless-db,database,graphql,distributed"
---

# Fauna Database JavaScript SDK v2.5.0

## Golden Rule

**CRITICAL**: Use the official `fauna` package (v2.5.0+) for all Fauna v10 projects.

```bash
npm install fauna
```

**DO NOT USE**:
- `faunadb` package (deprecated, FQL v4 only, EOL June 30, 2025)
- Any unofficial Fauna packages

The `fauna` package is the only supported SDK for Fauna v10 and FQL v10. It is NOT compatible with earlier FQL versions.

**Important Service Notice**: The Fauna service will be ending on May 30, 2025.

---

## Installation

### Node.js

```bash
npm install fauna
```

### Browser (CDN)

```html
<script type="module">
  import * as fauna from "https://cdn.jsdelivr.net/npm/fauna@latest/dist/browser/index.js";
</script>
```

### Environment Setup

Set your Fauna secret as an environment variable:

```bash
export FAUNA_SECRET="your_secret_key_here"
```

Or create a `.env` file:

```
FAUNA_SECRET=your_secret_key_here
```

---

## Supported Runtimes

**Server-side**: Node.js v18+ (Current, LTS, and Maintenance versions)

**Cloud platforms**: Cloudflare Workers, AWS Lambda, Netlify, Vercel

**Browsers**: Chrome 69+, Firefox 62+, Safari 12.1+, Edge 79+

---

## Initialization

### Basic Client Setup

```javascript
import { Client } from "fauna";

// Uses FAUNA_SECRET environment variable by default
const client = new Client();
```

### Client with Explicit Secret

```javascript
import { Client } from "fauna";

const client = new Client({
  secret: "your_secret_key_here"
});
```

### Advanced Client Configuration

```javascript
import { Client, endpoints } from "fauna";

const client = new Client({
  secret: process.env.FAUNA_SECRET,
  endpoint: endpoints.default,
  client_timeout_buffer_ms: 5000,
  http2_max_streams: 100,
  http2_session_idle_ms: 5000,
  fetch_keepalive: false,
  // Default query options
  format: "tagged",
  long_type: "number",
  linearized: false,
  max_attempts: 3,
  max_backoff_ms: 1000,
  query_timeout_ms: 60000
});
```

### Client Lifecycle

```javascript
import { Client, fql, FaunaError } from "fauna";

const client = new Client();

try {
  // Use client for queries
  const result = await client.query(fql`Product.all()`);
  console.log(result.data);
} catch (error) {
  if (error instanceof FaunaError) {
    console.error("Fauna error:", error);
  }
} finally {
  // Close client when done
  client.close();
}
```

---

## Core API Surfaces

### FQL Template Literals

The `fql` function creates safe, composable queries. Interpolated variables are treated as data values, preventing injection attacks.

```javascript
import { Client, fql } from "fauna";

const client = new Client();

// Basic query
const result = await client.query(
  fql`Collection.create({ name: "Dogs" })`
);
```

### Variable Interpolation

```javascript
const dog = { name: "Scout", age: 3 };

const result = await client.query(
  fql`Dogs.create(${dog}) { id, ts, name, age }`
);

console.log(result.data);
// { id: "123", ts: "...", name: "Scout", age: 3 }
```

### Reusable Subqueries

```javascript
const collectionExists = (name) =>
  fql`Collection.byName(${name}) != null`;

const query = fql`
  if (${collectionExists("Pets")}) {
    "Collection exists"
  } else {
    Collection.create({ name: "Pets" })
  }
`;

const result = await client.query(query);
```

---

## Collection Operations

### Create Collection

```javascript
const createCollection = fql`
  Collection.create({ name: "Products" })
`;

const result = await client.query(createCollection);
```

### Check Collection Exists

```javascript
const exists = fql`
  Collection.byName("Products") != null
`;

const result = await client.query(exists);
console.log(result.data); // true or false
```

### Get Collection

```javascript
const getCollection = fql`
  Collection.byName("Products")
`;

const result = await client.query(getCollection);
```

---

## Document CRUD Operations

### Create Document

**Minimal**:
```javascript
const doc = { name: "Laptop", price: 999 };

const createDoc = fql`
  Products.create(${doc})
`;

const result = await client.query(createDoc);
console.log(result.data.id);
```

**With Field Selection**:
```javascript
const doc = {
  name: "Laptop",
  price: 999,
  stock: 50
};

const createDoc = fql`
  Products.create(${doc}) {
    id,
    ts,
    name,
    price
  }
`;

const result = await client.query(createDoc);
console.log(result.data);
// { id: "...", ts: "...", name: "Laptop", price: 999 }
```

### Read Document by ID

```javascript
const docId = "123456789";

const getDoc = fql`
  Products.byId(${docId})
`;

const result = await client.query(getDoc);
console.log(result.data);
```

### Read All Documents

```javascript
const getAllDocs = fql`
  Products.all() { id, name, price }
`;

const result = await client.query(getAllDocs);
console.log(result.data);
```

### Update Document

**Using firstWhere**:
```javascript
const updateDoc = fql`
  Products.firstWhere(.name == "Laptop")?.update({
    price: 899
  })
`;

const result = await client.query(updateDoc);
```

**By ID**:
```javascript
const docId = "123456789";
const updates = { price: 799, stock: 45 };

const updateDoc = fql`
  Products.byId(${docId})?.update(${updates})
`;

const result = await client.query(updateDoc);
```

**With Field Selection**:
```javascript
const updateDoc = fql`
  Products.firstWhere(.name == "Laptop")?.update({
    price: 899
  }) { id, name, price, ts }
`;

const result = await client.query(updateDoc);
```

### Replace Document

```javascript
const docId = "123456789";
const newData = {
  name: "Gaming Laptop",
  price: 1299,
  stock: 30,
  category: "Electronics"
};

const replaceDoc = fql`
  Products.byId(${docId})?.replace(${newData})
`;

const result = await client.query(replaceDoc);
```

### Delete Document

**Single Document**:
```javascript
const deleteDoc = fql`
  Products.byId("123456789")?.delete()
`;

const result = await client.query(deleteDoc);
```

**Multiple Documents**:
```javascript
const deleteDocs = fql`
  Products.where(.price < 10).forEach(.delete())
`;

const result = await client.query(deleteDocs);
```

**Using firstWhere**:
```javascript
const deleteDoc = fql`
  Products.firstWhere(.name == "Laptop")?.delete()
`;

const result = await client.query(deleteDoc);
```

---

## Querying and Filtering

### Query All Documents

```javascript
const query = fql`
  Products.all() { name, price, stock }
`;

const result = await client.query(query);
```

### Filter with where()

**Note**: `where()` scans entire collection. Use indexes for better performance on large collections.

```javascript
const minPrice = 500;
const maxPrice = 1500;

const query = fql`
  Products.where(.price >= ${minPrice} && .price <= ${maxPrice}) {
    name,
    price
  }
`;

const result = await client.query(query);
```

**Complex Conditions**:
```javascript
const category = "Electronics";
const minStock = 10;

const query = fql`
  Products.where(.category == ${category} && .stock > ${minStock}) {
    name,
    price,
    stock
  }
`;

const result = await client.query(query);
```

### First Matching Document

```javascript
const query = fql`
  Products.firstWhere(.name == "Laptop")
`;

const result = await client.query(query);
console.log(result.data);
```

**Note**: `firstWhere()` can cause a full collection scan. For better performance, use an index:

```javascript
const query = fql`
  Products.byName("Laptop").first()
`;

const result = await client.query(query);
```

### Sort Documents

```javascript
const query = fql`
  Products.all().order(.price desc) { name, price }
`;

const result = await client.query(query);
```

### Limit Results

```javascript
const query = fql`
  Products.all().pageSize(10) { name, price }
`;

const result = await client.query(query);
```

---

## Pagination

### Basic Pagination

```javascript
import { Client, fql } from "fauna";

const client = new Client();

const query = fql`
  Products.all().pageSize(20) { name, price }
`;

const pages = client.paginate(query);

for await (const products of pages) {
  for (const product of products) {
    console.log(product);
  }
}
```

### Pagination with Query Options

```javascript
const query = fql`
  Products.where(.price > 100).pageSize(50) {
    name,
    price,
    stock
  }
`;

const pages = client.paginate(query, {
  query_timeout_ms: 60000
});

for await (const productsPage of pages) {
  console.log(`Page has ${productsPage.length} items`);
  for (const product of productsPage) {
    console.log(product.name, product.price);
  }
}
```

### Flatten Pagination

Iterate individual items instead of pages:

```javascript
const query = fql`
  Products.all().pageSize(100) { name, price }
`;

const pages = client.paginate(query);

for await (const product of pages.flatten()) {
  console.log(product.name, product.price);
}
```

### Manual Pagination Control

```javascript
let after = null;
let hasMore = true;

while (hasMore) {
  const query = after
    ? fql`Products.all().pageSize(20).after(${after}) { name, price }`
    : fql`Products.all().pageSize(20) { name, price }`;

  const result = await client.query(query);

  // Process results
  result.data.data.forEach(product => {
    console.log(product);
  });

  after = result.data.after;
  hasMore = after != null;
}
```

---

## Indexes

### Query by Index

**Note**: You must create indexes using the Fauna dashboard or schema files before querying them.

```javascript
// Assuming you have a 'byName' index on Products
const query = fql`
  Products.byName("Laptop") { name, price }
`;

const result = await client.query(query);
```

### Index with Range Query

```javascript
const query = fql`
  Products.byPriceRange({
    from: 100,
    to: 500
  }) { name, price }
`;

const result = await client.query(query);
```

### Index with Time Range

```javascript
const query = fql`
  Orders.orderedByCreated({
    from: Time("2025-01-01T00:00:00Z"),
    to: Time("2025-12-31T23:59:59Z")
  }) { id, total, createdAt }
`;

const result = await client.query(query);
```

### First Result from Index

```javascript
const query = fql`
  Products.byName("Laptop").first()
`;

const result = await client.query(query);
console.log(result.data);
```

---

## TypeScript Support

### Basic Type Safety

```typescript
import { Client, fql, type QuerySuccess } from "fauna";

interface Product {
  id: string;
  name: string;
  price: number;
  stock: number;
}

const client = new Client();

const query = fql<Product>`
  Products.byId("123")
`;

const response: QuerySuccess<Product> = await client.query(query);
const product: Product = response.data;

console.log(product.name); // Type-safe
```

### Automatic Type Inference

```typescript
interface Product {
  name: string;
  price: number;
  stock: number;
}

const query = fql<Product>`
  Products.create({
    name: "Laptop",
    price: 999,
    stock: 50
  })
`;

// response.data is automatically typed as Product
const response = await client.query(query);
console.log(response.data.price); // TypeScript knows this is a number
```

### Custom Interfaces

```typescript
import { type QueryValueObject } from "fauna";

interface User extends QueryValueObject {
  id: string;
  name: string;
  email: string;
  createdAt: string;
}

const query = fql<User>`
  Users.byEmail("user@example.com").first()
`;

const response = await client.query(query);
const user: User = response.data;
```

### Array Results

```typescript
interface Product {
  name: string;
  price: number;
}

const query = fql<Product[]>`
  Products.all() { name, price }
`;

const response = await client.query(query);
const products: Product[] = response.data;

products.forEach(p => console.log(p.name));
```

---

## Query Options

### Per-Query Configuration

```javascript
import { Client, fql, type QueryOptions } from "fauna";

const client = new Client();

const options = {
  query_timeout_ms: 60000,
  max_contention_retries: 5,
  typecheck: true,
  linearized: false,
  query_tags: { name: "product_search", version: "v1" }
};

const query = fql`Products.all() { name, price }`;

const result = await client.query(query, options);
```

### Arguments in Queries

```javascript
const options = {
  arguments: {
    name: "Laptop",
    minPrice: 500
  }
};

const query = fql`
  Products.where(.name == #{name} && .price >= #{minPrice})
`;

const result = await client.query(query, options);
```

### Timeout Configuration

```javascript
const options = {
  query_timeout_ms: 30000, // 30 seconds server-side
  client_timeout_buffer_ms: 5000 // 5 seconds network buffer
};

const result = await client.query(query, options);
```

### Retry Configuration

```javascript
const options = {
  max_contention_retries: 10
};

const result = await client.query(query, options);
```

---

## Query Statistics

### Access Performance Metrics

```javascript
import { Client, fql, type QueryStats } from "fauna";

const client = new Client();

const query = fql`Products.all() { name, price }`;
const response = await client.query(query);

const stats = response.stats;

console.log(`Compute ops: ${stats.compute_ops}`);
console.log(`Read ops: ${stats.read_ops}`);
console.log(`Write ops: ${stats.write_ops}`);
console.log(`Query time: ${stats.query_time_ms}ms`);
console.log(`Storage read: ${stats.storage_bytes_read} bytes`);
console.log(`Storage write: ${stats.storage_bytes_write} bytes`);
console.log(`Retries: ${stats.contention_retries}`);
```

### Stats from Error

```javascript
import { ServiceError } from "fauna";

try {
  const result = await client.query(query);
} catch (error) {
  if (error instanceof ServiceError && error.stats) {
    console.log(`Failed after ${error.stats.query_time_ms}ms`);
    console.log(`Compute ops used: ${error.stats.compute_ops}`);
  }
}
```

---

## Event Feeds

### Create Event Feed

```javascript
const query = fql`
  Products.all().eventsOn(.price, .stock)
`;

const feed = await client.eventFeed(query);

for await (const event of feed) {
  console.log(`Event type: ${event.type}`);

  if (event.type === 'add') {
    console.log('New document:', event.data);
  } else if (event.type === 'update') {
    console.log('Updated document:', event.data);
  } else if (event.type === 'remove') {
    console.log('Removed document:', event.data);
  }
}
```

### Event Feed from Query Result

```javascript
const response = await client.query(fql`
  let set = Products.all()
  {
    initialPage: set.pageSize(10),
    eventSource: set.eventSource()
  }
`);

const eventSource = response.data.eventSource;
const feed = await client.eventFeed(eventSource);

for await (const event of feed) {
  console.log(event);
}
```

### Event Feed Error Handling

```javascript
import { FaunaError } from "fauna";

try {
  const feed = await client.eventFeed(query);

  for await (const event of feed) {
    console.log(event);
  }
} catch (error) {
  if (error instanceof FaunaError) {
    console.error('Event feed error:', error.message);
  }
}
```

---

## Event Streams

### Real-time Document Changes

```javascript
const query = fql`
  Products.byId("123")
`;

const stream = await client.stream(query);

for await (const event of stream) {
  console.log('Stream event:', event);

  if (event.type === 'update') {
    console.log('Document updated:', event.data);
  }
}
```

### Close Stream

```javascript
const stream = await client.stream(query);

// Listen for events
setTimeout(() => {
  stream.close();
  console.log('Stream closed');
}, 30000); // Close after 30 seconds

for await (const event of stream) {
  console.log(event);
}
```

### Stream Error Handling

```javascript
import { FaunaError } from "fauna";

try {
  const stream = await client.stream(query);

  for await (const event of stream) {
    console.log(event);
  }
} catch (error) {
  if (error instanceof FaunaError) {
    console.error('Stream error:', error.message);
  }
} finally {
  stream.close();
}
```

---

## Error Handling

### Basic Error Handling

```javascript
import { Client, fql, FaunaError } from "fauna";

const client = new Client();

try {
  const result = await client.query(fql`
    Products.byId("invalid_id")
  `);
} catch (error) {
  if (error instanceof FaunaError) {
    console.error('Fauna error:', error.message);
    console.error('Error code:', error.code);
  } else {
    console.error('Unknown error:', error);
  }
}
```

### Specific Error Types

```javascript
import {
  ServiceError,
  AuthenticationError,
  AuthorizationError,
  QueryCheckError,
  QueryRuntimeError,
  ThrottlingError,
  NetworkError
} from "fauna";

try {
  const result = await client.query(query);
} catch (error) {
  if (error instanceof AuthenticationError) {
    console.error('Authentication failed:', error.message);
  } else if (error instanceof AuthorizationError) {
    console.error('Authorization failed:', error.message);
  } else if (error instanceof QueryCheckError) {
    console.error('Query validation failed:', error.message);
  } else if (error instanceof QueryRuntimeError) {
    console.error('Query execution failed:', error.message);
  } else if (error instanceof ThrottlingError) {
    console.error('Request throttled:', error.message);
  } else if (error instanceof NetworkError) {
    console.error('Network error:', error.message);
  } else if (error instanceof ServiceError) {
    console.error('Service error:', error.message);
  }
}
```

### Error with Stats

```javascript
import { ServiceError } from "fauna";

try {
  const result = await client.query(query);
} catch (error) {
  if (error instanceof ServiceError) {
    console.error('Query failed:', error.message);

    if (error.stats) {
      console.error(`Query time: ${error.stats.query_time_ms}ms`);
      console.error(`Compute ops: ${error.stats.compute_ops}`);
    }

    if (error.queryInfo) {
      console.error('Query:', error.queryInfo.query);
    }
  }
}
```

---

## Advanced Query Patterns

### Conditional Logic

```javascript
const status = "active";

const query = fql`
  if (${status} == "active") {
    Products.where(.status == "active")
  } else {
    Products.all()
  }
`;

const result = await client.query(query);
```

### Computed Fields

```javascript
const query = fql`
  Products.all() {
    name,
    price,
    discountPrice: .price * 0.9,
    inStock: .stock > 0
  }
`;

const result = await client.query(query);
```

### Nested Projections

```javascript
const query = fql`
  Orders.all() {
    id,
    total,
    customer {
      name,
      email
    },
    items {
      product {
        name,
        price
      },
      quantity
    }
  }
`;

const result = await client.query(query);
```

### Aggregations

```javascript
const query = fql`
  {
    totalProducts: Products.all().count(),
    totalValue: Products.all().fold(0, (sum, product) => sum + product.price),
    avgPrice: Products.all().fold(0, (sum, p) => sum + p.price) / Products.all().count()
  }
`;

const result = await client.query(query);
```

### forEach Operations

```javascript
const updates = [
  { id: "1", price: 100 },
  { id: "2", price: 200 }
];

const query = fql`
  ${updates}.forEach(item => {
    Products.byId(item.id)?.update({ price: item.price })
  })
`;

const result = await client.query(query);
```

### Map Operations

```javascript
const query = fql`
  Products.all().map(product => {
    {
      name: product.name,
      discounted: product.price * 0.8
    }
  })
`;

const result = await client.query(query);
```

---

## Client Configuration Details

### HTTP/2 Configuration

```javascript
const client = new Client({
  http2_max_streams: 100,
  http2_session_idle_ms: 5000
});
```

### Retry Behavior

```javascript
const client = new Client({
  max_attempts: 3,
  max_backoff_ms: 1000
});
```

**Default**: 3 attempts with exponential backoff up to 1 second.

### Timeout Configuration

```javascript
const client = new Client({
  query_timeout_ms: 60000, // 60 seconds server-side
  client_timeout_buffer_ms: 5000 // 5 seconds network overhead
});
```

### Format Options

```javascript
const client = new Client({
  format: "tagged", // or "simple"
  long_type: "number" // or "bigint"
});
```

### Linearized Reads

```javascript
const client = new Client({
  linearized: true // Ensures strict consistency
});
```

---

## Complete Example Application

```javascript
import { Client, fql, FaunaError } from "fauna";

// Initialize client
const client = new Client({
  secret: process.env.FAUNA_SECRET,
  query_timeout_ms: 60000
});

async function createProduct(name, price, stock) {
  try {
    const result = await client.query(fql`
      Products.create({
        name: ${name},
        price: ${price},
        stock: ${stock}
      }) { id, name, price, stock, ts }
    `);

    console.log('Created product:', result.data);
    return result.data;
  } catch (error) {
    if (error instanceof FaunaError) {
      console.error('Failed to create product:', error.message);
    }
    throw error;
  }
}

async function getProductsByPriceRange(minPrice, maxPrice) {
  try {
    const query = fql`
      Products
        .where(.price >= ${minPrice} && .price <= ${maxPrice})
        .pageSize(50) { id, name, price, stock }
    `;

    const pages = client.paginate(query);
    const products = [];

    for await (const page of pages) {
      products.push(...page);
    }

    console.log(`Found ${products.length} products`);
    return products;
  } catch (error) {
    if (error instanceof FaunaError) {
      console.error('Failed to query products:', error.message);
    }
    throw error;
  }
}

async function updateProductPrice(productId, newPrice) {
  try {
    const result = await client.query(fql`
      Products.byId(${productId})?.update({
        price: ${newPrice}
      }) { id, name, price }
    `);

    console.log('Updated product:', result.data);
    return result.data;
  } catch (error) {
    if (error instanceof FaunaError) {
      console.error('Failed to update product:', error.message);
    }
    throw error;
  }
}

async function deleteProduct(productId) {
  try {
    const result = await client.query(fql`
      Products.byId(${productId})?.delete()
    `);

    console.log('Deleted product:', result.data);
    return result.data;
  } catch (error) {
    if (error instanceof FaunaError) {
      console.error('Failed to delete product:', error.message);
    }
    throw error;
  }
}

// Main execution
async function main() {
  try {
    // Create products
    await createProduct("Laptop", 999, 50);
    await createProduct("Mouse", 29, 200);
    await createProduct("Keyboard", 79, 150);

    // Query products
    const products = await getProductsByPriceRange(0, 100);
    console.log('Affordable products:', products);

    // Update a product
    if (products.length > 0) {
      await updateProductPrice(products[0].id, 25);
    }

    // Delete a product
    if (products.length > 1) {
      await deleteProduct(products[1].id);
    }
  } catch (error) {
    console.error('Application error:', error);
  } finally {
    // Clean up
    client.close();
  }
}

main();
```

---

## Environment Variables Reference

| Variable | Description | Required |
|----------|-------------|----------|
| `FAUNA_SECRET` | Authentication secret key | Yes |
| `FAUNA_ENDPOINT` | Custom endpoint URL | No |

---

## API Reference

Full documentation: https://fauna.github.io/fauna-js/

---

## License

Mozilla Public License 2.0 (MPL 2.0)
