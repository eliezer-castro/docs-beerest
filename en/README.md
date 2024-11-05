# Beerest

> An elegant Python library for API testing with fluent interface.

## 🚀 Introduction

Beerest is an API testing library that combines simplicity, robustness, and elegance, offering a fluent test writing experience. With Beerest, you can write readable and maintainable tests with ease.

## 📦 Installation

```bash
pip install beerest
```

## 🌟 Key Features

- Fluent Interface for intuitive test writing
- Expressive and chainable assertions
- Full JSON Schema validation support
- Response time validation
- Flexible headers and query parameters management
- Robust error handling
- Custom timeout support

## 🔧 Basic Usage

```python
from beerest import Test, Expect

class TestApi(Test):
    def setup_method(self):
        super().setup_method()
        self.request.base_url = "https://api.example.com"

    def test_get_user(self):
        response = self.request.to("/users/1").get()
        
        Expect(response) \
            .status(200) \
            .is_json() \
            .body("name").equals("John Doe") \
            .body("email").matches(r"^[\w\.-]+@[\w\.-]+\.\w+$")
```

## 📚 API Guide

### Request

The `Request` object is the starting point for making HTTP calls.

#### Main Methods

| Method | Description |
|--------|-------------|
| `to(endpoint)` | Sets the request endpoint |
| `with_headers(headers)` | Adds headers to the request |
| `with_body(data)` | Sets the request body |
| `with_query(params)` | Adds query parameters |
| `with_timeout(timeout)` | Sets request timeout |
| `get()` | Executes GET request |
| `post()` | Executes POST request |
| `put()` | Executes PUT request |
| `delete()` | Executes DELETE request |

```python
response = self.request \
    .to("/users") \
    .with_headers({"Authorization": "Bearer token"}) \
    .with_query({"active": True}) \
    .get()
```

### Expect

The `Expect` object provides a fluent interface for assertions.

#### Validation Methods

| Method | Description |
|--------|-------------|
| `status(code)` | Validates status code |
| `body(path)` | Selects body element (supports JSONPath) |
| `header(name)` | Validates specific header |
| `time()` | Validates response time |
| `equals(value)` | Compares exact value |
| `contains(value)` | Checks if contains value |
| `matches(pattern)` | Validates with regex |
| `less_than(value)` | Compares less than |
| `greater_than(value)` | Compares greater than |
| `has_length(length)` | Validates length |
| `is_json()` | Validates if JSON is valid |
| `has_keys(*keys)` | Validates presence of keys |
| `matches_schema(schema)` | Validates against JSON Schema |

```python
Expect(response) \
    .status(200) \
    .body("data.users") \
    .has_length(10) \
    .body("data.users[0].email") \
    .matches(r"^[\w\.-]+@[\w\.-]+\.\w+$")
```

## Schema Validation

Beerest offers robust support for JSON Schema validation.

### Basic Schema

```python
user_schema = {
    "type": "object",
    "required": ["id", "name", "email"],
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string"},
        "email": {"type": "string", "format": "email"}
    }
}

Expect(response) \
    .body() \
    .matches_schema(user_schema)
```

### Array Validation

```python
item_schema = {
    "type": "object",
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string"}
    }
}

Expect(response) \
    .body("items") \
    .has_array_items(item_schema)
```

### Supported Formats

- email
- date-time-iso
- uuid
- url

## Traditional Assertions

Besides the fluent interface, Beerest provides traditional assertions:

```python
from beerest import Assertions

Assertions.assertEqual(actual, expected)
Assertions.assertTrue(condition)
Assertions.assertFalse(condition)
Assertions.assertNotNull(value)
Assertions.assertLess(a, b)
Assertions.assertGreater(a, b)
Assertions.assertIn(element, collection)
```

## Complete Examples

### Basic GET Test

```python
def test_get_posts(self):
    response = self.request.to("/posts").get()
    
    Expect(response) \
        .status(200) \
        .is_json() \
        .body() \
        .has_length(100)
```

### POST with Complex Validation

```python
def test_create_post(self):
    new_post = {
        "title": "Test Post",
        "body": "This is a test post",
        "userId": 1
    }
    
    response = self.request \
        .to("/posts") \
        .with_headers({"Content-Type": "application/json"}) \
        .with_body(new_post) \
        .post()
        
    Expect(response) \
        .status(201) \
        .body("title").equals("Test Post") \
        .body("id").satisfies(lambda x: isinstance(x, int))
```

### Performance Test

```python
def test_response_time(self):
    response = self.request.to("/posts/1").get()
    
    Expect(response) \
        .time() \
        .less_than(1000)  # less than 1000ms
```

## Test Lifecycle

Beerest provides setup and teardown hooks:

```python
class TestApi(Test):
    def setup_method(self):
        super().setup_method()
        self.request.base_url = "https://api.example.com"
        # Additional setup

    def teardown_method(self):
        super().teardown_method()
        # Cleanup after each test
```

## Best Practices

1. **Test Organization**
   - Group related tests in the same class
   - Use descriptive names for test methods
   - Document complex test cases

2. **Reusability**
   - Create helpers for common operations
   - Extract reusable schemas to separate files
   - Use setup_method for common configuration

3. **Assertions**
   - Prefer fluent interface for better readability
   - Combine assertions when appropriate
   - Validate all relevant aspects of the response

4. **Performance**
   - Configure appropriate timeouts
   - Include time assertions when relevant
   - Group related tests to optimize execution

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a branch for your feature (`git checkout -b feature/new-feature`)
3. Commit your changes (`git commit -m 'Add new feature'`)
4. Push to the branch (`git push origin feature/new-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.