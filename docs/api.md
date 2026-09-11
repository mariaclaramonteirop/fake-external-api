# Documentação da API

Fake External API para simular um sistema externo integrado ao Craft CMS.

## Status

Esta documentação define o contrato inicial da API. Os endpoints ainda serão implementados gradualmente.

## Base URL

Desenvolvimento local:

```text
http://localhost:8080
```

Quando executada em container, a URL e a porta dependerão da configuração do `docker-compose.yml`.

## Formato das requisições

As requisições que enviam dados devem utilizar JSON:

```http
Content-Type: application/json
Accept: application/json
```

## Formato das respostas

Respostas bem-sucedidas devem retornar JSON:

```json
{
  "data": {}
}
```

Respostas de erro devem seguir este formato:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Os dados enviados são inválidos.",
    "details": {}
  }
}
```

## Códigos HTTP

| Código | Significado |
| --- | --- |
| `200` | Requisição processada com sucesso |
| `201` | Recurso criado |
| `204` | Requisição processada sem conteúdo de resposta |
| `400` | Requisição inválida |
| `404` | Recurso não encontrado |
| `409` | Conflito, como SKU duplicado |
| `422` | Dados semanticamente inválidos |
| `500` | Erro interno |
| `503` | Serviço ou banco indisponível |

## Health check

### Verificar disponibilidade

```http
GET /health
```

Resposta esperada:

```json
{
  "data": {
    "status": "ok"
  }
}
```

## Produtos

O produto é o principal recurso compartilhado entre o Craft CMS e o Fake ERP.

O `sku` é o identificador de negócio usado para sincronização. O `id` é um identificador interno do Fake ERP.

### Listar produtos

```http
GET /products
```

Parâmetros opcionais de consulta:

| Parâmetro | Tipo | Descrição |
| --- | --- | --- |
| `page` | inteiro | número da página |
| `limit` | inteiro | quantidade de itens por página |
| `sku` | string | filtra por SKU |
| `active` | boolean | filtra produtos ativos |
| `category_id` | inteiro | filtra por categoria |

Resposta:

```json
{
  "data": [
    {
      "id": 1,
      "sku": "NOTE-001",
      "name": "Notebook X",
      "description": "Notebook para estudos e trabalho.",
      "price": 4500.00,
      "category_id": 1,
      "active": true,
      "created_at": "2026-09-10T10:00:00Z",
      "updated_at": "2026-09-10T10:00:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 1
  }
}
```

### Consultar produto por ID

```http
GET /products/{id}
```

Exemplo:

```http
GET /products/1
```

### Consultar produto por SKU

```http
GET /products/sku/{sku}
```

Exemplo:

```http
GET /products/sku/NOTE-001
```

### Criar produto

```http
POST /products
```

Payload:

```json
{
  "sku": "NOTE-001",
  "name": "Notebook X",
  "description": "Notebook para estudos e trabalho.",
  "price": 4500.00,
  "category_id": 1,
  "active": true
}
```

Regras:

- `sku` é obrigatório;
- `sku` deve ter no máximo 50 caracteres;
- `sku` deve ser único;
- `name` é obrigatório e deve ter no máximo 255 caracteres;
- `price` deve ser maior ou igual a zero;
- `category_id` deve referenciar uma categoria existente;
- `active` deve ser booleano.

Resposta esperada: `201 Created`.

### Atualizar produto

```http
PUT /products/{id}
```

Payload:

```json
{
  "name": "Notebook X Pro",
  "description": "Notebook atualizado.",
  "price": 5200.00,
  "category_id": 1,
  "active": true
}
```

O `sku` deve continuar sendo único. A alteração do SKU deve ser tratada como uma operação controlada de sincronização.

### Excluir produto

```http
DELETE /products/{id}
```

A regra inicial recomendada é desativar o produto em vez de apagá-lo fisicamente, preservando o histórico de pedidos.

## Estoque

O estoque pertence ao Fake ERP e não ao Craft CMS.

### Consultar estoque por produto

```http
GET /products/{id}/stock
```

Ou por SKU:

```http
GET /stock/{sku}
```

Resposta:

```json
{
  "data": {
    "product_id": 1,
    "sku": "NOTE-001",
    "quantity": 20,
    "reserved_quantity": 3,
    "available_quantity": 17,
    "updated_at": "2026-09-10T10:00:00Z"
  }
}
```

Regra:

```text
available_quantity = quantity - reserved_quantity
```

`quantity` e `reserved_quantity` não podem ser negativos.

### Atualizar estoque

```http
PUT /products/{id}/stock
```

Payload:

```json
{
  "quantity": 20,
  "reserved_quantity": 3
}
```

## Clientes

### Listar clientes

```http
GET /customers
```

### Consultar cliente

```http
GET /customers/{id}
```

### Criar cliente

```http
POST /customers
```

Payload:

```json
{
  "name": "Maria Clara",
  "email": "maria@example.com",
  "document": "12345678900"
}
```

Regras:

- `name` é obrigatório e deve ter no máximo 255 caracteres;
- `email` é obrigatório e deve ser único;
- `document` é opcional e deve ter no máximo 20 caracteres.

## Pedidos

### Listar pedidos

```http
GET /orders
```

### Consultar pedido

```http
GET /orders/{id}
```

### Criar pedido

```http
POST /orders
```

Payload:

```json
{
  "customer_id": 1,
  "items": [
    {
      "product_id": 1,
      "quantity": 2
    }
  ]
}
```

O preço dos itens deve ser obtido pelo sistema no momento da criação do pedido. O cliente não deve ser a fonte confiável do `unit_price` ou do `total`.

Status previstos:

```text
pending
confirmed
cancelled
completed
```

## Integração com o Craft CMS

Fluxo de envio do Craft para o Fake ERP:

```text
Produto publicado no Craft
        ↓
Webhook ou evento
        ↓
Integration API
        ↓
Validação e transformação
        ↓
POST ou PUT /products
        ↓
Fake External API
```

Payload de integração:

```json
{
  "sku": "NOTE-001",
  "name": "Notebook X",
  "description": "Notebook para estudos e trabalho.",
  "price": 4500.00,
  "category": "Eletrônicos",
  "active": true
}
```

O Craft é a origem editorial do produto. O Fake ERP é a origem operacional do estoque, dos clientes e dos pedidos.

## Idempotência

A sincronização de produtos deve ser idempotente. Enviar o mesmo produto mais de uma vez não deve criar duplicatas.

A regra inicial será:

```text
sku existente → atualizar produto
sku inexistente → criar produto
```

Em operações mais sensíveis, a `integration-api` poderá enviar um identificador de idempotência:

```http
Idempotency-Key: product-sync-NOTE-001-20260910
```

## Autenticação

A autenticação ainda será definida. Possibilidades para o laboratório:

- token fixo de desenvolvimento;
- API key;
- Bearer token;
- autenticação por ambiente Docker.

Credenciais não devem ser colocadas no código ou versionadas no Git. Devem ser lidas de variáveis de ambiente.

## Versionamento

Quando o contrato estiver estável, as rotas poderão ser versionadas:

```text
/api/v1/products
/api/v1/stock/NOTE-001
/api/v1/orders
```

A versão inicial pode ser usada sem prefixo enquanto a API estiver em desenvolvimento.
