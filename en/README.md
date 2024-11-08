# Beerest

> An elegant Python library for API testing with a fluent interface.

## 🚀 Introduction

Beerest is an API testing library that combines simplicity, robustness, and elegance, offering a fluent experience for writing tests. With Beerest, you can write readable and maintainable tests with ease.

## 📦 Installation

```bash
pip install beerest
```

## 🌟 Key Features

- Fluent Interface for intuitive test writing
- Expressive and chainable assertions
- Complete JSON Schema validation support
- Response time validation
- Flexible header and query parameter management
- Multiple authentication methods support
- Robust error handling
- Custom timeout support
- Test context for better organization
- Custom validations via predicates

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
|--------|------------|
| `to(endpoint)` | Sets the request endpoint |
| `with_headers(headers)` | Adds headers to the request |
| `with_body(data)` | Sets the request body |
| `with_query(params)` | Adds query parameters |
| `with_timeout(timeout)` | Sets request timeout |
| `with_basic_auth(username, password)` | Adds basic authentication |
| `with_digest_auth(username, password)` | Adds digest authentication |
| `with_bearer_token(token)` | Adds Bearer token authentication |
| `with_custom_auth(auth)` | Adds custom authentication |
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

### Authentication

Beerest supports multiple authentication methods:

```python
# Basic Auth
response = self.request \
    .with_basic_auth("username", "password") \
    .to("/protected").get()

# Digest Auth
response = self.request \
    .with_digest_auth("username", "password") \
    .to("/protected").get()

# Bearer Token
response = self.request \
    .with_bearer_token("your-token") \
    .to("/protected").get()

# Custom Auth
class CustomAuth(Authentication):
    def apply(self, headers: Dict[str, str], auth: Optional[Any]) -> Tuple[Dict[str, str], Optional[Any]]:
        headers['X-Custom-Auth'] = 'custom-value'
        return headers, auth

response = self.request \
    .with_custom_auth(CustomAuth()) \
    .to("/protected").get()
```

### Response

The `Response` object encapsulates the API response and provides easy access to data:

| Attribute | Description |
|-----------|-------------|
| `status_code` | HTTP status code |
| `headers` | Response headers |
| `json_data` | Parsed JSON data (if available) |
| `text` | Response body as text |
| `elapsed_time` | Response time in milliseconds |

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
| `is_not_empty()` | Checks if value is not empty |
| `is_in(collection)` | Checks if value is in collection |
| `satisfies(predicate)` | Validates using custom predicate |
| `that(context)` | Sets context for validations |
| `has_type(type)` | Validates value type |
| `has_array_items(schema)` | Validates array items |

```python
Expect(response) \
    .status(200) \
    .body("data.users") \
    .has_length(10) \
    .body("data.users[0].email") \
    .matches(r"^[\w\.-]+@[\w\.-]+\.\w+$")
```

### Context and Grouped Validations

Beerest allows defining contexts to group related validations:

```python
Expect(response) \
    .that("Status and format") \
        .status(200) \
        .is_json() \
    .that("User data") \
        .body("user.name").equals("John") \
        .body("user.email").matches(r"^[\w\.-]+@[\w\.-]+\.\w+$") \
    .that("Metrics") \
        .time().less_than(1000)
```

### Custom Validations

The `satisfies` method allows creating custom validations using predicates:

```python
def is_valid_price(price):
    return isinstance(price, (int, float)) and price > 0

Expect(response) \
    .body("price") \
    .satisfies(is_valid_price, "price should be a positive number")
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

The schema validator supports the following formats:
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

### Test with Authentication and Context

```python
def test_protected_endpoint(self):
    response = self.request \
        .to("/protected") \
        .with_bearer_token("your-token") \
        .get()
        
    Expect(response) \
        .that("Authentication") \
            .status(200) \
            .header("Authorization").is_not_empty() \
        .that("Returned data") \
            .body("data").is_not_empty() \
            .body("permissions").has_type("array")
```

### Complex Validation with Predicates

```python
def test_product_data(self):
    def valid_product(product):
        return all([
            isinstance(product.get("id"), int),
            isinstance(product.get("price"), (int, float)),
            product.get("price") > 0,
            isinstance(product.get("stock"), int),
            product.get("stock") >= 0
        ])
    
    response = self.request.to("/products/1").get()
    
    Expect(response) \
        .status(200) \
        .body() \
        .satisfies(valid_product, "product data should be valid")
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

5. **Authentication**
   - Use the most appropriate authentication method for each case
   - Encapsulate complex authentication logic in custom classes
   - Keep sensitive credentials in environment variables

## Error Handling

Beerest provides robust error handling with clear messages:

- Invalid URL validation
- Request timeout
- Schema errors
- Assertion failures with detailed messages
- Authentication issues

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-feature`)
3. Commit your changes (`git commit -m 'Add new feature'`)
4. Push to the branch (`git push origin feature/new-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.