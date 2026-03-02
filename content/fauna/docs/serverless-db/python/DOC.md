---
name: serverless-db
description: "Fauna Database Python SDK for serverless database operations with GraphQL and distributed queries"
metadata:
  languages: "python"
  versions: "2.4.0"
  updated-on: "2026-03-02"
  source: maintainer
  tags: "fauna,serverless-db,database,graphql,distributed"
---

# Fauna Database Python SDK v2.4.0

## Golden Rule

**CRITICAL**: Use the official `fauna` package (v2.4.0+) for all Fauna v10 projects.

```bash
pip install fauna
```

**DO NOT USE**:
- `faunadb` package (deprecated, FQL v4 only, EOL June 30, 2025)
- Any unofficial Fauna packages

The `fauna` package is the only supported SDK for Fauna v10 and FQL v10. It is NOT compatible with earlier FQL versions.

**Important Service Notice**: The Fauna service will be ending on May 30, 2025.

---

## Installation

### Using pip

```bash
pip install fauna
```

### Using pip with specific version

```bash
pip install fauna==2.4.0
```

### Environment Setup

Set your Fauna secret as an environment variable:

```bash
export FAUNA_SECRET="your_secret_key_here"
```

Or create a `.env` file:

```
FAUNA_SECRET=your_secret_key_here
FAUNA_ENDPOINT=https://db.fauna.com
```

### Load Environment Variables

```python
import os
from dotenv import load_dotenv

load_dotenv()

FAUNA_SECRET = os.getenv('FAUNA_SECRET')
```

---

## Supported Python Versions

- Python 3.9
- Python 3.10
- Python 3.11
- Python 3.12

---

## Initialization

### Basic Client Setup

```python
from fauna.client import Client

# Uses FAUNA_SECRET environment variable by default
client = Client()
```

### Client with Explicit Secret

```python
from fauna.client import Client

client = Client(secret="your_secret_key_here")
```

### Client with Custom Endpoint

```python
import os
from fauna.client import Client

client = Client(
    secret=os.getenv('FAUNA_SECRET'),
    endpoint=os.getenv('FAUNA_ENDPOINT', 'https://db.fauna.com')
)
```

### Advanced Client Configuration

```python
from datetime import timedelta
from fauna.client import Client

client = Client(
    secret="your_secret_key_here",
    query_timeout=timedelta(seconds=60),
    client_buffer_timeout=timedelta(seconds=10),
    http_read_timeout=timedelta(seconds=30),
    http_write_timeout=timedelta(seconds=10),
    http_connect_timeout=timedelta(seconds=10),
    http_pool_timeout=timedelta(seconds=10),
    http_idle_timeout=timedelta(seconds=10),
    max_attempts=3,
    max_backoff=timedelta(seconds=20)
)
```

---

## Core API Surfaces

### FQL Query Construction

```python
from fauna import fql
from fauna.client import Client
from fauna.encoding import QuerySuccess

client = Client()

# Basic query
query = fql('Dogs.create({ name: "Scout" })')
result: QuerySuccess = client.query(query)

print(result.data)
```

### Variable Interpolation

```python
from fauna import fql

name = "Scout"
age = 3

query = fql('Dogs.create({ name: ${name}, age: ${age} })', name=name, age=age)
result = client.query(query)

print(result.data)
```

### Query Composition

```python
from fauna import fql

def add_two(x):
    return fql('${x} + 2', x=x)

query = fql('${y} + 4', y=add_two(2))
result = client.query(query)

print(result.data)  # 8
```

---

## Collection Operations

### Create Collection

```python
from fauna import fql

query = fql('Collection.create({ name: "Products" })')
result = client.query(query)

print(result.data)
```

### Check Collection Exists

```python
from fauna import fql

query = fql('Collection.byName("Products") != null')
result = client.query(query)

print(result.data)  # True or False
```

### Get Collection

```python
from fauna import fql

query = fql('Collection.byName("Products")')
result = client.query(query)

print(result.data)
```

---

## Document CRUD Operations

### Create Document

**Minimal**:
```python
from fauna import fql

doc = {
    "name": "Laptop",
    "price": 999,
    "stock": 50
}

query = fql('Products.create(${doc})', doc=doc)
result = client.query(query)

print(result.data['id'])
```

**With Field Selection**:
```python
from fauna import fql

doc = {
    "name": "Laptop",
    "price": 999,
    "stock": 50
}

query = fql('''
    Products.create(${doc}) {
        id,
        ts,
        name,
        price
    }
''', doc=doc)

result = client.query(query)
print(result.data)
```

### Read Document by ID

```python
from fauna import fql

doc_id = "123456789"

query = fql('Products.byId(${id})', id=doc_id)
result = client.query(query)

print(result.data)
```

### Read All Documents

```python
from fauna import fql

query = fql('Products.all() { id, name, price }')
result = client.query(query)

print(result.data)
```

### Update Document

**Using firstWhere**:
```python
from fauna import fql

query = fql('''
    Products.firstWhere(.name == "Laptop")?.update({
        price: 899
    })
''')

result = client.query(query)
print(result.data)
```

**By ID**:
```python
from fauna import fql

doc_id = "123456789"
updates = {"price": 799, "stock": 45}

query = fql('''
    Products.byId(${id})?.update(${updates})
''', id=doc_id, updates=updates)

result = client.query(query)
print(result.data)
```

**With Field Selection**:
```python
from fauna import fql

query = fql('''
    Products.firstWhere(.name == "Laptop")?.update({
        price: 899
    }) { id, name, price, ts }
''')

result = client.query(query)
```

### Replace Document

```python
from fauna import fql

doc_id = "123456789"
new_data = {
    "name": "Gaming Laptop",
    "price": 1299,
    "stock": 30,
    "category": "Electronics"
}

query = fql('''
    Products.byId(${id})?.replace(${data})
''', id=doc_id, data=new_data)

result = client.query(query)
```

### Delete Document

**Single Document**:
```python
from fauna import fql

query = fql('Products.byId("123456789")?.delete()')
result = client.query(query)

print(result.data)
```

**Multiple Documents**:
```python
from fauna import fql

query = fql('''
    Products.where(.price < 10).forEach(.delete())
''')

result = client.query(query)
```

**Using firstWhere**:
```python
from fauna import fql

query = fql('''
    Products.firstWhere(.name == "Laptop")?.delete()
''')

result = client.query(query)
```

---

## Querying and Filtering

### Query All Documents

```python
from fauna import fql

query = fql('Products.all() { name, price, stock }')
result = client.query(query)

for product in result.data:
    print(f"{product['name']}: ${product['price']}")
```

### Filter with where()

**Note**: `where()` scans entire collection. Use indexes for better performance on large collections.

```python
from fauna import fql

min_price = 500
max_price = 1500

query = fql('''
    Products.where(.price >= ${min_price} && .price <= ${max_price}) {
        name,
        price
    }
''', min_price=min_price, max_price=max_price)

result = client.query(query)
```

**Complex Conditions**:
```python
from fauna import fql

category = "Electronics"
min_stock = 10

query = fql('''
    Products.where(.category == ${category} && .stock > ${min_stock}) {
        name,
        price,
        stock
    }
''', category=category, min_stock=min_stock)

result = client.query(query)
```

### First Matching Document

```python
from fauna import fql

query = fql('Products.firstWhere(.name == "Laptop")')
result = client.query(query)

print(result.data)
```

**Note**: `firstWhere()` can cause a full collection scan. For better performance, use an index:

```python
from fauna import fql

query = fql('Products.byName("Laptop").first()')
result = client.query(query)
```

### Sort Documents

```python
from fauna import fql

query = fql('''
    Products.all().order(.price desc) { name, price }
''')

result = client.query(query)
```

### Limit Results

```python
from fauna import fql

query = fql('''
    Products.all().pageSize(10) { name, price }
''')

result = client.query(query)
```

---

## Pagination

### Basic Pagination

```python
from fauna import fql
from fauna.client import Client

client = Client()

query = fql('''
    Products.all().pageSize(50) { id, name, price }
''')

pages = client.paginate(query)

for products_page in pages:
    for product in products_page:
        print(f"{product['name']}: ${product['price']}")
```

### Pagination with Stats

```python
from fauna import fql

query = fql('''
    Product
        .byName("limes")
        .pageSize(60) { description }
''')

pages = client.paginate(query)

for page in pages:
    print(f'Page has {len(page)} items')
    for product in page:
        print(product)
```

### Flatten Pagination

Iterate individual items instead of pages:

```python
from fauna import fql

query = fql('''
    Products.all().pageSize(100) { name, price }
''')

pages = client.paginate(query)

for product in pages.flatten():
    print(f"{product['name']}: ${product['price']}")
```

### Manual Pagination Control

```python
from fauna import fql

after = None
has_more = True

while has_more:
    if after:
        query = fql('''
            Products.all().pageSize(20).after(${after}) { name, price }
        ''', after=after)
    else:
        query = fql('Products.all().pageSize(20) { name, price }')

    result = client.query(query)

    # Process results
    for product in result.data['data']:
        print(product)

    after = result.data.get('after')
    has_more = after is not None
```

---

## Indexes

### Query by Index

**Note**: You must create indexes using the Fauna dashboard or schema files before querying them.

```python
from fauna import fql

# Assuming you have a 'byName' index on Products
query = fql('Products.byName("Laptop") { name, price }')
result = client.query(query)

print(result.data)
```

### Index with Range Query

```python
from fauna import fql

query = fql('''
    Products.byPriceRange({
        from: 100,
        to: 500
    }) { name, price }
''')

result = client.query(query)
```

### Index with Time Range

```python
from fauna import fql

query = fql('''
    Orders.orderedByCreated({
        from: Time("2025-01-01T00:00:00Z"),
        to: Time("2025-12-31T23:59:59Z")
    }) { id, total, createdAt }
''')

result = client.query(query)
```

### First Result from Index

```python
from fauna import fql

query = fql('Products.byName("Laptop").first()')
result = client.query(query)

print(result.data)
```

---

## Query Statistics

### Access Performance Metrics

```python
from fauna import fql
from fauna.encoding import QueryStats

query = fql('Products.all() { name, price }')
result = client.query(query)

stats = result.stats

print(f"Compute ops: {stats.compute_ops}")
print(f"Read ops: {stats.read_ops}")
print(f"Write ops: {stats.write_ops}")
print(f"Query time: {stats.query_time_ms}ms")
print(f"Storage read: {stats.storage_bytes_read} bytes")
print(f"Storage write: {stats.storage_bytes_write} bytes")
print(f"Retries: {stats.contention_retries}")
```

### Stats from Error

```python
from fauna import fql
from fauna.errors import ServiceError

try:
    result = client.query(query)
except ServiceError as e:
    if e.stats:
        print(f"Failed after {e.stats.query_time_ms}ms")
        print(f"Compute ops used: {e.stats.compute_ops}")
```

---

## Event Feeds

### Creating an Event Feed from Query

```python
from fauna import fql

# Create event source from query result
response = client.query(fql('''
    let set = Product.all()
    {
        initialPage: set.pageSize(10),
        eventSource: set.eventSource()
    }
'''))

event_source = response.data['eventSource']
feed = client.feed(event_source)
```

### Direct Event Feed from Query

```python
from fauna import fql

query = fql('Product.all().eventsOn(.price, .stock)')
feed = client.feed(query)
```

### Iterating Events by Pages

```python
from fauna import fql

query = fql('Product.all().eventsOn(.price, .stock)')
feed = client.feed(query)

for page in feed:
    print(f'Page stats: {page.stats}')

    for event in page:
        event_type = event['type']

        if event_type == 'add':
            print('Add event:', event)
        elif event_type == 'update':
            print('Update event:', event)
        elif event_type == 'remove':
            print('Remove event:', event)
```

### Flatten Events

```python
from fauna import fql

query = fql('Product.all().eventsOn(.price, .stock)')
feed = client.feed(query)

for event in feed.flatten():
    print(f"Event type: {event['type']}")
    print(f"Event data: {event.get('data')}")
```

### Event Feed Error Handling

```python
from fauna import fql
from fauna.errors import FaunaException

try:
    query = fql('Product.all().eventsOn(.price, .stock)')
    feed = client.feed(query)

    for event in feed.flatten():
        print(event)
except FaunaException as e:
    print(f'Event feed error: {e}')
```

---

## Error Handling

### Basic Error Handling

```python
from fauna import fql
from fauna.errors import FaunaException

try:
    query = fql('Products.byId("invalid_id")')
    result = client.query(query)
except FaunaException as e:
    print(f'Fauna error: {e}')
```

### Specific Error Types

```python
from fauna import fql
from fauna.errors import (
    FaunaException,
    ServiceError,
    AuthenticationError,
    AuthorizationError,
    QueryCheckError,
    QueryRuntimeError,
    ThrottlingError,
    NetworkError
)

try:
    result = client.query(query)
except AuthenticationError as e:
    print(f'Authentication failed: {e}')
except AuthorizationError as e:
    print(f'Authorization failed: {e}')
except QueryCheckError as e:
    print(f'Query validation failed: {e}')
except QueryRuntimeError as e:
    print(f'Query execution failed: {e}')
except ThrottlingError as e:
    print(f'Request throttled: {e}')
except NetworkError as e:
    print(f'Network error: {e}')
except ServiceError as e:
    print(f'Service error: {e}')
except FaunaException as e:
    print(f'Fauna error: {e}')
```

### Error with Stats

```python
from fauna import fql
from fauna.errors import ServiceError

try:
    result = client.query(query)
except ServiceError as e:
    print(f'Query failed: {e}')

    if e.stats:
        print(f'Query time: {e.stats.query_time_ms}ms')
        print(f'Compute ops: {e.stats.compute_ops}')

    if hasattr(e, 'query_info') and e.query_info:
        print(f'Query: {e.query_info}')
```

---

## Advanced Query Patterns

### Conditional Logic

```python
from fauna import fql

status = "active"

query = fql('''
    if (${status} == "active") {
        Products.where(.status == "active")
    } else {
        Products.all()
    }
''', status=status)

result = client.query(query)
```

### Computed Fields

```python
from fauna import fql

query = fql('''
    Products.all() {
        name,
        price,
        discountPrice: .price * 0.9,
        inStock: .stock > 0
    }
''')

result = client.query(query)
```

### Nested Projections

```python
from fauna import fql

query = fql('''
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
''')

result = client.query(query)
```

### Aggregations

```python
from fauna import fql

query = fql('''
    {
        totalProducts: Products.all().count(),
        totalValue: Products.all().fold(0, (sum, product) => sum + product.price),
        avgPrice: Products.all().fold(0, (sum, p) => sum + p.price) / Products.all().count()
    }
''')

result = client.query(query)

print(f"Total products: {result.data['totalProducts']}")
print(f"Total value: {result.data['totalValue']}")
print(f"Average price: {result.data['avgPrice']}")
```

### forEach Operations

```python
from fauna import fql

updates = [
    {"id": "1", "price": 100},
    {"id": "2", "price": 200}
]

query = fql('''
    ${updates}.forEach(item => {
        Products.byId(item.id)?.update({ price: item.price })
    })
''', updates=updates)

result = client.query(query)
```

### Map Operations

```python
from fauna import fql

query = fql('''
    Products.all().map(product => {
        {
            name: product.name,
            discounted: product.price * 0.8
        }
    })
''')

result = client.query(query)
```

---

## Client Configuration Details

### Timeout Configuration

```python
from datetime import timedelta
from fauna.client import Client

client = Client(
    query_timeout=timedelta(seconds=60),
    client_buffer_timeout=timedelta(seconds=10),
    http_read_timeout=timedelta(seconds=30),
    http_write_timeout=timedelta(seconds=10),
    http_connect_timeout=timedelta(seconds=10),
    http_pool_timeout=timedelta(seconds=10),
    http_idle_timeout=timedelta(seconds=10)
)
```

**Timeout Types**:
- `query_timeout`: Server-side execution limit (default: 5 seconds)
- `client_buffer_timeout`: Network buffer (default: 5 seconds)
- `http_read_timeout`: Data reception (default: None)
- `http_write_timeout`: Data transmission (default: 5 seconds)
- `http_connect_timeout`: Connection establishment (default: 5 seconds)
- `http_pool_timeout`: Connection pool wait (default: 5 seconds)
- `http_idle_timeout`: Session idle duration (default: 5 seconds)

### Retry Configuration

```python
from datetime import timedelta
from fauna.client import Client

client = Client(
    max_attempts=3,
    max_backoff=timedelta(seconds=20)
)
```

**Default**: 3 attempts with exponential backoff up to 20 seconds.

Set `max_attempts` to 1 or less to disable retries.

---

## Serialization and Deserialization

### Converting Custom Classes

Custom class serialization is not yet supported. Convert classes to dictionaries before querying:

```python
from fauna import fql

class Product:
    def __init__(self, name, price, stock):
        self.name = name
        self.price = price
        self.stock = stock

    def to_dict(self):
        return {
            'name': self.name,
            'price': self.price,
            'stock': self.stock
        }

    @staticmethod
    def from_result(obj):
        return Product(
            name=obj['name'],
            price=obj['price'],
            stock=obj['stock']
        )

# Create document
product = Product("Laptop", 999, 50)
query = fql('Products.create(${doc})', doc=product.to_dict())
result = client.query(query)

# Rebuild from result
product_data = result.data
restored_product = Product.from_result(product_data)
```

### Working with Dates and Times

```python
from fauna import fql
from datetime import datetime

now = datetime.now().isoformat()

query = fql('''
    Products.create({
        name: "Laptop",
        createdAt: Time(${now})
    })
''', now=now)

result = client.query(query)
```

---

## Complete Example Application

```python
import os
from fauna import fql
from fauna.client import Client
from fauna.errors import FaunaException
from datetime import timedelta

# Initialize client
client = Client(
    secret=os.getenv('FAUNA_SECRET'),
    query_timeout=timedelta(seconds=60)
)

def create_product(name, price, stock):
    """Create a new product in the database."""
    try:
        query = fql('''
            Products.create({
                name: ${name},
                price: ${price},
                stock: ${stock}
            }) { id, name, price, stock, ts }
        ''', name=name, price=price, stock=stock)

        result = client.query(query)
        print(f'Created product: {result.data}')
        return result.data
    except FaunaException as e:
        print(f'Failed to create product: {e}')
        raise

def get_products_by_price_range(min_price, max_price):
    """Get all products within a price range."""
    try:
        query = fql('''
            Products
                .where(.price >= ${min_price} && .price <= ${max_price})
                .pageSize(50) { id, name, price, stock }
        ''', min_price=min_price, max_price=max_price)

        pages = client.paginate(query)
        products = []

        for page in pages:
            products.extend(page)

        print(f'Found {len(products)} products')
        return products
    except FaunaException as e:
        print(f'Failed to query products: {e}')
        raise

def update_product_price(product_id, new_price):
    """Update the price of a product."""
    try:
        query = fql('''
            Products.byId(${id})?.update({
                price: ${price}
            }) { id, name, price }
        ''', id=product_id, price=new_price)

        result = client.query(query)
        print(f'Updated product: {result.data}')
        return result.data
    except FaunaException as e:
        print(f'Failed to update product: {e}')
        raise

def delete_product(product_id):
    """Delete a product by ID."""
    try:
        query = fql('''
            Products.byId(${id})?.delete()
        ''', id=product_id)

        result = client.query(query)
        print(f'Deleted product: {result.data}')
        return result.data
    except FaunaException as e:
        print(f'Failed to delete product: {e}')
        raise

def get_product_stats():
    """Get aggregated statistics about products."""
    try:
        query = fql('''
            {
                total: Products.all().count(),
                totalValue: Products.all().fold(0, (sum, p) => sum + p.price)
            }
        ''')

        result = client.query(query)
        stats = result.data

        print(f"Total products: {stats['total']}")
        print(f"Total inventory value: ${stats['totalValue']}")

        return stats
    except FaunaException as e:
        print(f'Failed to get stats: {e}')
        raise

# Main execution
def main():
    try:
        # Create products
        create_product("Laptop", 999, 50)
        create_product("Mouse", 29, 200)
        create_product("Keyboard", 79, 150)

        # Query products
        products = get_products_by_price_range(0, 100)
        print(f'Affordable products: {products}')

        # Update a product
        if products:
            update_product_price(products[0]['id'], 25)

        # Delete a product
        if len(products) > 1:
            delete_product(products[1]['id'])

        # Get stats
        get_product_stats()

    except Exception as e:
        print(f'Application error: {e}')

if __name__ == '__main__':
    main()
```

---

## Environment Variables Reference

| Variable | Description | Required |
|----------|-------------|----------|
| `FAUNA_SECRET` | Authentication secret key | Yes |
| `FAUNA_ENDPOINT` | Custom endpoint URL | No |

---

## API Reference

Full documentation: https://fauna.github.io/fauna-python/

---

## License

Mozilla Public License 2.0 (MPL 2.0)
