# Testing Patterns and Best Practices

## Contents
- Test Organization Patterns
- Coverage Improvement Strategies
- Testing Anti-Patterns
- Test Data Patterns
- Mocking Patterns
- Assertion Patterns

## Test Organization Patterns

### Arrange-Act-Assert (AAA)

```javascript
test('calculates total price with tax', () => {
  // Arrange
  const items = [{ price: 10 }, { price: 20 }]
  const taxRate = 0.1
  
  // Act
  const total = calculateTotal(items, taxRate)
  
  // Assert
  expect(total).toBe(33)
})
```

### Given-When-Then (BDD)

```python
def test_user_login():
    # Given
    user = User(email="test@example.com", password="secret")
    user.save()
    
    # When
    result = authenticate(email="test@example.com", password="secret")
    
    # Then
    assert result.is_authenticated
    assert result.user.email == "test@example.com"
```

### Test Fixtures

```python
import pytest

@pytest.fixture
def sample_user():
    return User(name="Test User", email="test@example.com")

@pytest.fixture
def authenticated_client(sample_user):
    client = TestClient()
    client.login(sample_user)
    return client

def test_profile_access(authenticated_client, sample_user):
    response = authenticated_client.get('/profile')
    assert response.status_code == 200
```

## Coverage Improvement Strategies

### Strategy 1: Test Uncovered Functions First

```bash
# Find functions with 0% coverage
npm test -- --coverage --collectCoverageFrom='src/**/*.js' | grep "0%"
```

Then write tests for each:
```javascript
// Before: 0% coverage
function validateEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}

// Add test
test('validates email format', () => {
  expect(validateEmail('test@example.com')).toBe(true)
  expect(validateEmail('invalid')).toBe(false)
  expect(validateEmail('')).toBe(false)
})
```

### Strategy 2: Cover All Branches

```javascript
// Function with branches
function getDiscount(user) {
  if (user.isPremium) {
    return 0.2
  } else if (user.isRegular) {
    return 0.1
  }
  return 0
}

// Test all branches
test('premium users get 20% discount', () => {
  expect(getDiscount({ isPremium: true })).toBe(0.2)
})

test('regular users get 10% discount', () => {
  expect(getDiscount({ isRegular: true })).toBe(0.1)
})

test('new users get no discount', () => {
  expect(getDiscount({})).toBe(0)
})
```

### Strategy 3: Test Error Paths

```javascript
async function fetchUser(id) {
  try {
    const response = await api.get(`/users/${id}`)
    return response.data
  } catch (error) {
    if (error.status === 404) {
      throw new UserNotFoundError()
    }
    throw new ApiError()
  }
}

// Test success path
test('fetches user successfully', async () => {
  api.get.mockResolvedValue({ data: { id: 1, name: 'Test' } })
  const user = await fetchUser(1)
  expect(user.name).toBe('Test')
})

// Test error paths
test('throws UserNotFoundError for 404', async () => {
  api.get.mockRejectedValue({ status: 404 })
  await expect(fetchUser(999)).rejects.toThrow(UserNotFoundError)
})

test('throws ApiError for other errors', async () => {
  api.get.mockRejectedValue({ status: 500 })
  await expect(fetchUser(1)).rejects.toThrow(ApiError)
})
```

### Strategy 4: Parameterized Tests

```python
import pytest

@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("WORLD", "WORLD"),
    ("", ""),
    ("123", "123"),
    ("Hello World", "HELLO WORLD"),
])
def test_uppercase(input, expected):
    assert uppercase(input) == expected
```

```javascript
test.each([
  [1, 1, 2],
  [2, 2, 4],
  [3, 5, 8],
  [-1, 1, 0],
])('adds %i + %i to equal %i', (a, b, expected) => {
  expect(add(a, b)).toBe(expected)
})
```

## Testing Anti-Patterns

### Anti-Pattern 1: Testing Implementation Details

```javascript
// Bad: Testing internal state
test('counter increments internal value', () => {
  const counter = new Counter()
  counter.increment()
  expect(counter._value).toBe(1) // Don't test private properties
})

// Good: Testing behavior
test('counter displays incremented value', () => {
  const counter = new Counter()
  counter.increment()
  expect(counter.getValue()).toBe(1)
})
```

### Anti-Pattern 2: Fragile Tests

```javascript
// Bad: Depends on exact order
test('returns users in order', () => {
  const users = getUsers()
  expect(users[0].name).toBe('Alice')
  expect(users[1].name).toBe('Bob')
})

// Good: Tests the requirement
test('returns all users', () => {
  const users = getUsers()
  expect(users).toHaveLength(2)
  expect(users.map(u => u.name)).toContain('Alice')
  expect(users.map(u => u.name)).toContain('Bob')
})
```

### Anti-Pattern 3: Test Interdependence

```javascript
// Bad: Tests depend on each other
let userId
test('creates user', () => {
  userId = createUser({ name: 'Test' })
  expect(userId).toBeDefined()
})

test('updates user', () => {
  updateUser(userId, { name: 'Updated' }) // Depends on previous test
  expect(getUser(userId).name).toBe('Updated')
})

// Good: Independent tests
test('creates user', () => {
  const userId = createUser({ name: 'Test' })
  expect(userId).toBeDefined()
})

test('updates user', () => {
  const userId = createUser({ name: 'Test' })
  updateUser(userId, { name: 'Updated' })
  expect(getUser(userId).name).toBe('Updated')
})
```

## Test Data Patterns

### Pattern 1: Factory Functions

```javascript
function createUser(overrides = {}) {
  return {
    id: Math.random(),
    name: 'Test User',
    email: 'test@example.com',
    role: 'user',
    ...overrides
  }
}

test('admin users can delete posts', () => {
  const admin = createUser({ role: 'admin' })
  expect(canDelete(admin)).toBe(true)
})
```

### Pattern 2: Fixtures

```python
# conftest.py
@pytest.fixture
def sample_data():
    return {
        'users': [
            {'id': 1, 'name': 'Alice'},
            {'id': 2, 'name': 'Bob'}
        ],
        'posts': [
            {'id': 1, 'user_id': 1, 'title': 'First Post'}
        ]
    }

def test_user_posts(sample_data):
    user = sample_data['users'][0]
    posts = get_user_posts(user['id'])
    assert len(posts) == 1
```

### Pattern 3: Builders

```javascript
class UserBuilder {
  constructor() {
    this.user = {
      name: 'Test User',
      email: 'test@example.com',
      role: 'user'
    }
  }
  
  withName(name) {
    this.user.name = name
    return this
  }
  
  withRole(role) {
    this.user.role = role
    return this
  }
  
  build() {
    return this.user
  }
}

test('admin users have special permissions', () => {
  const admin = new UserBuilder()
    .withName('Admin User')
    .withRole('admin')
    .build()
  
  expect(hasPermission(admin, 'delete')).toBe(true)
})
```

## Mocking Patterns

### Pattern 1: Dependency Injection

```javascript
// Good: Inject dependencies
class UserService {
  constructor(api) {
    this.api = api
  }
  
  async getUser(id) {
    return this.api.get(`/users/${id}`)
  }
}

// Easy to test
test('fetches user from API', async () => {
  const mockApi = { get: jest.fn().mockResolvedValue({ id: 1 }) }
  const service = new UserService(mockApi)
  
  await service.getUser(1)
  expect(mockApi.get).toHaveBeenCalledWith('/users/1')
})
```

### Pattern 2: Spy on Methods

```javascript
test('logs errors when API fails', async () => {
  const consoleSpy = jest.spyOn(console, 'error').mockImplementation()
  
  await fetchData() // This will fail and log error
  
  expect(consoleSpy).toHaveBeenCalled()
  consoleSpy.mockRestore()
})
```

### Pattern 3: Mock Modules

```javascript
// __mocks__/api.js
export const get = jest.fn()
export const post = jest.fn()

// test file
jest.mock('./api')
import { get } from './api'

test('uses API', async () => {
  get.mockResolvedValue({ data: 'test' })
  const result = await fetchData()
  expect(result).toBe('test')
})
```

## Assertion Patterns

### Pattern 1: Specific Assertions

```javascript
// Bad: Too generic
expect(result).toBeTruthy()

// Good: Specific
expect(result).toBe(true)
expect(result).toEqual({ id: 1, name: 'Test' })
```

### Pattern 2: Snapshot Testing

```javascript
test('renders component correctly', () => {
  const tree = renderer.create(<UserProfile user={mockUser} />).toJSON()
  expect(tree).toMatchSnapshot()
})
```

### Pattern 3: Custom Matchers

```javascript
expect.extend({
  toBeValidEmail(received) {
    const pass = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(received)
    return {
      pass,
      message: () => `expected ${received} to be a valid email`
    }
  }
})

test('validates email', () => {
  expect('test@example.com').toBeValidEmail()
})
```

## Coverage-Driven Development

### Workflow

1. **Write failing test**
2. **Run coverage** - Should show 0% for new code
3. **Implement feature**
4. **Run tests** - Should pass
5. **Run coverage** - Should show 100% for new code
6. **Refactor** - Coverage should remain 100%

### Example

```javascript
// Step 1: Write test
test('calculates compound interest', () => {
  expect(calculateCompoundInterest(1000, 0.05, 2)).toBe(1102.5)
})

// Step 2: Run coverage - shows calculateCompoundInterest at 0%

// Step 3: Implement
function calculateCompoundInterest(principal, rate, years) {
  return principal * Math.pow(1 + rate, years)
}

// Step 4: Test passes

// Step 5: Coverage shows 100% for calculateCompoundInterest

// Step 6: Refactor if needed, tests still pass
```
