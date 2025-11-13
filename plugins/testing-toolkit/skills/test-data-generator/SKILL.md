---
name: test-data-generator
description: Creates test fixtures, mock data, and test scenarios for unit and integration tests. Use when setting up test data, creating mocks, or generating test fixtures.
---

# Test Data Generator

Generate test data, fixtures, and mocks for testing.

## Quick Start

Create a simple fixture:
```javascript
const mockUser = {
  id: 1,
  name: 'Test User',
  email: 'test@example.com'
}
```

## Instructions

### Factory Functions

Create reusable data generators:

```javascript
function createUser(overrides = {}) {
  return {
    id: Math.floor(Math.random() * 1000),
    name: 'Test User',
    email: 'test@example.com',
    role: 'user',
    createdAt: new Date().toISOString(),
    ...overrides
  }
}

// Usage
const admin = createUser({ role: 'admin', name: 'Admin User' })
const regularUser = createUser()
```

```python
def create_user(**kwargs):
    defaults = {
        'id': random.randint(1, 1000),
        'name': 'Test User',
        'email': 'test@example.com',
        'role': 'user',
        'created_at': datetime.now()
    }
    return {**defaults, **kwargs}

# Usage
admin = create_user(role='admin', name='Admin User')
```

### Builder Pattern

For complex objects:

```javascript
class UserBuilder {
  constructor() {
    this.user = {
      id: 1,
      name: 'Test User',
      email: 'test@example.com',
      role: 'user',
      preferences: {}
    }
  }
  
  withId(id) {
    this.user.id = id
    return this
  }
  
  withName(name) {
    this.user.name = name
    return this
  }
  
  withRole(role) {
    this.user.role = role
    return this
  }
  
  withPreferences(preferences) {
    this.user.preferences = preferences
    return this
  }
  
  build() {
    return this.user
  }
}

// Usage
const user = new UserBuilder()
  .withId(123)
  .withName('John Doe')
  .withRole('admin')
  .withPreferences({ theme: 'dark' })
  .build()
```

### Test Fixtures

**JavaScript/TypeScript**:
```javascript
// fixtures/users.js
export const users = {
  admin: {
    id: 1,
    name: 'Admin User',
    email: 'admin@example.com',
    role: 'admin'
  },
  regular: {
    id: 2,
    name: 'Regular User',
    email: 'user@example.com',
    role: 'user'
  }
}

// In tests
import { users } from './fixtures/users'

test('admin can delete posts', () => {
  expect(canDelete(users.admin)).toBe(true)
})
```

**Python (pytest)**:
```python
# conftest.py
import pytest

@pytest.fixture
def sample_user():
    return {
        'id': 1,
        'name': 'Test User',
        'email': 'test@example.com',
        'role': 'user'
    }

@pytest.fixture
def admin_user():
    return {
        'id': 2,
        'name': 'Admin User',
        'email': 'admin@example.com',
        'role': 'admin'
    }

# In tests
def test_user_permissions(sample_user, admin_user):
    assert can_delete(admin_user)
    assert not can_delete(sample_user)
```

### Mock Data

**API Responses**:
```javascript
const mockApiResponse = {
  data: {
    users: [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' }
    ]
  },
  status: 200,
  statusText: 'OK'
}

// Mock fetch
global.fetch = jest.fn(() =>
  Promise.resolve({
    json: () => Promise.resolve(mockApiResponse.data)
  })
)
```

**Database Records**:
```python
mock_db_records = [
    {'id': 1, 'name': 'Product A', 'price': 10.99},
    {'id': 2, 'name': 'Product B', 'price': 20.99},
    {'id': 3, 'name': 'Product C', 'price': 15.99}
]

@patch('app.database.query')
def test_get_products(mock_query):
    mock_query.return_value = mock_db_records
    products = get_products()
    assert len(products) == 3
```

### Random Data Generation

**Using faker (JavaScript)**:
```javascript
import { faker } from '@faker-js/faker'

function generateUser() {
  return {
    id: faker.string.uuid(),
    name: faker.person.fullName(),
    email: faker.internet.email(),
    avatar: faker.image.avatar(),
    address: {
      street: faker.location.streetAddress(),
      city: faker.location.city(),
      country: faker.location.country()
    }
  }
}

// Generate multiple users
const users = Array.from({ length: 10 }, generateUser)
```

**Using Faker (Python)**:
```python
from faker import Faker

fake = Faker()

def generate_user():
    return {
        'id': fake.uuid4(),
        'name': fake.name(),
        'email': fake.email(),
        'phone': fake.phone_number(),
        'address': {
            'street': fake.street_address(),
            'city': fake.city(),
            'country': fake.country()
        }
    }

# Generate multiple users
users = [generate_user() for _ in range(10)]
```

### Seeded Random Data

For reproducible tests:

```javascript
import { faker } from '@faker-js/faker'

test('generates consistent data with seed', () => {
  faker.seed(123)
  const user1 = generateUser()
  
  faker.seed(123)
  const user2 = generateUser()
  
  expect(user1).toEqual(user2)
})
```

```python
from faker import Faker

def test_consistent_data():
    fake = Faker()
    Faker.seed(123)
    user1 = generate_user(fake)
    
    Faker.seed(123)
    user2 = generate_user(fake)
    
    assert user1 == user2
```

### Mock Functions

**Jest**:
```javascript
const mockCallback = jest.fn(x => x * 2)

test('calls callback for each item', () => {
  const items = [1, 2, 3]
  items.forEach(mockCallback)
  
  expect(mockCallback).toHaveBeenCalledTimes(3)
  expect(mockCallback).toHaveBeenCalledWith(1)
  expect(mockCallback).toHaveBeenCalledWith(2)
  expect(mockCallback).toHaveBeenCalledWith(3)
})
```

**Python (unittest.mock)**:
```python
from unittest.mock import Mock

def test_callback():
    mock_callback = Mock(return_value=42)
    
    result = process_data([1, 2, 3], mock_callback)
    
    assert mock_callback.call_count == 3
    mock_callback.assert_called_with(3)
```

### Time-based Data

**Freeze time**:
```javascript
import { jest } from '@jest/globals'

test('creates timestamp', () => {
  const mockDate = new Date('2024-01-01T00:00:00Z')
  jest.useFakeTimers()
  jest.setSystemTime(mockDate)
  
  const record = createRecord()
  expect(record.createdAt).toBe('2024-01-01T00:00:00.000Z')
  
  jest.useRealTimers()
})
```

```python
from freezegun import freeze_time

@freeze_time("2024-01-01 00:00:00")
def test_timestamp():
    record = create_record()
    assert record['created_at'] == datetime(2024, 1, 1, 0, 0, 0)
```

## Common Patterns

**Pattern: Array of test cases**:
```javascript
const testCases = [
  { input: 'hello', expected: 'HELLO' },
  { input: 'world', expected: 'WORLD' },
  { input: '', expected: '' }
]

testCases.forEach(({ input, expected }) => {
  test(`converts "${input}" to "${expected}"`, () => {
    expect(toUpperCase(input)).toBe(expected)
  })
})
```

**Pattern: Shared setup**:
```javascript
describe('UserService', () => {
  let service
  let mockDb
  
  beforeEach(() => {
    mockDb = {
      query: jest.fn(),
      insert: jest.fn()
    }
    service = new UserService(mockDb)
  })
  
  test('fetches users', async () => {
    mockDb.query.mockResolvedValue([{ id: 1 }])
    const users = await service.getUsers()
    expect(users).toHaveLength(1)
  })
})
```

**Pattern: Test data files**:
```javascript
// testData/users.json
[
  {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com"
  },
  {
    "id": 2,
    "name": "Bob",
    "email": "bob@example.com"
  }
]

// In tests
import users from './testData/users.json'

test('processes user data', () => {
  const result = processUsers(users)
  expect(result).toHaveLength(2)
})
```

## Advanced

For complex scenarios:
- Use factory libraries (factory-bot, fishery, rosie)
- Generate GraphQL mock data
- Create database seeders
- Build test data pipelines
