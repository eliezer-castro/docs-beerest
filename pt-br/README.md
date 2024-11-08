# Beerest

> Uma biblioteca Python elegante para testes de API com fluent interface.

## 🚀 Introdução

Beerest é uma biblioteca de testes de API que combina simplicidade, robustez e elegância, oferecendo uma experiência fluente de escrita de testes. Com Beerest, você pode escrever testes legíveis e manuteníveis com facilidade.

## 📦 Instalação

```bash
pip install beerest
```

## 🌟 Características principais

- Fluent Interface para escrita de testes intuitiva
- Assertions expressivas e encadeáveis
- Suporte completo a JSON Schema validation
- Validação de tempo de resposta
- Gerenciamento flexível de headers e query parameters
- Suporte a múltiplos métodos de autenticação
- Tratamento robusto de erros
- Suporte a timeout customizado
- Contexto de testes para melhor organização
- Validações customizáveis via predicados

## 🔧 Uso básico

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

## 📚 Guia de APIs

### Request

O objeto `Request` é o ponto de partida para fazer chamadas HTTP.

#### Métodos principais

| Método | Descrição |
|--------|-----------|
| `to(endpoint)` | Define o endpoint da requisição |
| `with_headers(headers)` | Adiciona headers à requisição |
| `with_body(data)` | Define o corpo da requisição |
| `with_query(params)` | Adiciona query parameters |
| `with_timeout(timeout)` | Define timeout da requisição |
| `with_basic_auth(username, password)` | Adiciona autenticação básica |
| `with_digest_auth(username, password)` | Adiciona autenticação digest |
| `with_bearer_token(token)` | Adiciona autenticação Bearer token |
| `with_custom_auth(auth)` | Adiciona autenticação customizada |
| `get()` | Executa requisição GET |
| `post()` | Executa requisição POST |
| `put()` | Executa requisição PUT |
| `delete()` | Executa requisição DELETE |

```python
response = self.request \
    .to("/users") \
    .with_headers({"Authorization": "Bearer token"}) \
    .with_query({"active": True}) \
    .get()
```

### Autenticação

Beerest suporta múltiplos métodos de autenticação:

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

O objeto `Response` encapsula a resposta da API e fornece acesso fácil aos dados:

| Atributo | Descrição |
|----------|-----------|
| `status_code` | Código de status HTTP |
| `headers` | Headers da resposta |
| `json_data` | Dados JSON parseados (se disponível) |
| `text` | Corpo da resposta como texto |
| `elapsed_time` | Tempo de resposta em milissegundos |

### Expect

O objeto `Expect` fornece uma interface fluente para assertions.

#### Métodos de validação

| Método | Descrição |
|--------|-----------|
| `status(code)` | Valida status code |
| `body(path)` | Seleciona elemento do body (suporta JSONPath) |
| `header(name)` | Valida header específico |
| `time()` | Valida tempo de resposta |
| `equals(value)` | Compara valor exato |
| `contains(value)` | Verifica se contém valor |
| `matches(pattern)` | Valida com regex |
| `less_than(value)` | Compara menor que |
| `greater_than(value)` | Compara maior que |
| `has_length(length)` | Valida tamanho |
| `is_json()` | Valida se é JSON válido |
| `has_keys(*keys)` | Valida presença de chaves |
| `matches_schema(schema)` | Valida contra JSON Schema |
| `is_not_empty()` | Verifica se valor não é vazio |
| `is_in(collection)` | Verifica se valor está na coleção |
| `satisfies(predicate)` | Valida usando predicado customizado |
| `that(context)` | Define contexto para validações |
| `has_type(type)` | Valida o tipo do valor |
| `has_array_items(schema)` | Valida items de um array |

```python
Expect(response) \
    .status(200) \
    .body("data.users") \
    .has_length(10) \
    .body("data.users[0].email") \
    .matches(r"^[\w\.-]+@[\w\.-]+\.\w+$")
```

### Contexto e Validações Agrupadas

O Beerest permite definir contextos para agrupar validações relacionadas:

```python
Expect(response) \
    .that("Status e formato") \
        .status(200) \
        .is_json() \
    .that("Dados do usuário") \
        .body("user.name").equals("John") \
        .body("user.email").matches(r"^[\w\.-]+@[\w\.-]+\.\w+$") \
    .that("Métricas") \
        .time().less_than(1000)
```

### Validações Customizadas

O método `satisfies` permite criar validações customizadas usando predicados:

```python
def is_valid_price(price):
    return isinstance(price, (int, float)) and price > 0

Expect(response) \
    .body("price") \
    .satisfies(is_valid_price, "price should be a positive number")
```

## Validação de Schema

Beerest oferece suporte robusto à validação de JSON Schema.

### Schema básico

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

### Validação de array

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

### Formatos suportados

O validador de schema suporta os seguintes formatos:
- email
- date-time-iso
- uuid
- url

## Assertions tradicionais

Além da interface fluente, Beerest fornece assertions tradicionais:

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

## Exemplos completos

### Teste básico GET

```python
def test_get_posts(self):
    response = self.request.to("/posts").get()
    
    Expect(response) \
        .status(200) \
        .is_json() \
        .body() \
        .has_length(100)
```

### POST com validação complexa

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

### Teste com autenticação e contexto

```python
def test_protected_endpoint(self):
    response = self.request \
        .to("/protected") \
        .with_bearer_token("your-token") \
        .get()
        
    Expect(response) \
        .that("Autenticação") \
            .status(200) \
            .header("Authorization").is_not_empty() \
        .that("Dados retornados") \
            .body("data").is_not_empty() \
            .body("permissions").has_type("array")
```

### Validação complexa com predicados

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

### Teste de performance

```python
def test_response_time(self):
    response = self.request.to("/posts/1").get()
    
    Expect(response) \
        .time() \
        .less_than(1000)  # menos que 1000ms
```

## Lifecycle dos testes

Beerest fornece hooks de setup e teardown:

```python
class TestApi(Test):
    def setup_method(self):
        super().setup_method()
        self.request.base_url = "https://api.example.com"
        # Configuração adicional

    def teardown_method(self):
        super().teardown_method()
        # Limpeza após cada teste
```

## Boas práticas

1. **Organização dos testes**
   - Agrupe testes relacionados na mesma classe
   - Use nomes descritivos para os métodos de teste
   - Documente casos de teste complexos

2. **Reutilização**
   - Crie helpers para operações comuns
   - Extraia schemas reutilizáveis para arquivos separados
   - Use setup_method para configuração comum

3. **Assertions**
   - Prefira a interface fluente para melhor legibilidade
   - Combine assertions quando apropriado
   - Valide todos os aspectos relevantes da resposta

4. **Performance**
   - Configure timeouts apropriados
   - Inclua assertions de tempo quando relevante
   - Agrupe testes relacionados para otimizar execução

5. **Autenticação**
   - Use o método de autenticação mais apropriado para cada caso
   - Encapsule lógica de autenticação complexa em classes customizadas
   - Mantenha credenciais sensíveis em variáveis de ambiente

## Tratamento de Erros

Beerest fornece tratamento robusto de erros com mensagens claras:

- Validação de URL inválida
- Timeout de requisição
- Erros de schema
- Falhas de assertion com mensagens detalhadas
- Problemas de autenticação

## Contribuindo

Contribuições são bem-vindas! Por favor, siga estes passos:

1. Fork o repositório
2. Crie um branch para sua feature (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -m 'Adiciona nova feature'`)
4. Push para o branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request

## Licença

Este projeto está licenciado sob a licença MIT - veja o arquivo [LICENSE](LICENSE) para detalhes.