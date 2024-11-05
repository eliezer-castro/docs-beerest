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
- Tratamento robusto de erros
- Suporte a timeout customizado

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

```python
Expect(response) \
    .status(200) \
    .body("data.users") \
    .has_length(10) \
    .body("data.users[0].email") \
    .matches(r"^[\w\.-]+@[\w\.-]+\.\w+$")
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

## Contribuindo

Contribuições são bem-vindas! Por favor, siga estes passos:

1. Fork o repositório
2. Crie um branch para sua feature (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -m 'Adiciona nova feature'`)
4. Push para o branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request

## Licença

Este projeto está licenciado sob a licença MIT - veja o arquivo [LICENSE](LICENSE) para detalhes.