# Fake External API

API externa simulada para o laboratório de integração com Craft CMS.

Este projeto representa um sistema externo, como um ERP simplificado, responsável por produtos, estoque, clientes e pedidos.

## Arquitetura

```text
Craft CMS
    ⇄
Integration API
    ⇄ HTTP/JSON
Fake External API
    ⇄
Banco de dados
```

O Craft CMS será responsável pelo catálogo e pelo conteúdo editorial. A Fake API será responsável pelos dados operacionais, como estoque, clientes e pedidos.

## Responsabilidades

- Produtos e categorias operacionais;
- Controle de estoque;
- Clientes;
- Pedidos;
- Exposição de endpoints HTTP/JSON;
- Persistência em banco de dados;
- Validação dos dados recebidos;
- Integração com a `integration-api`.

O `sku` será o identificador compartilhado entre o Craft CMS e a Fake API.

## Arquitetura interna

```text
Route
   ↓
Controller
   ↓
Validator
   ↓
Service
   ↓
RepositoryInterface
   ↓
DAO
   ↓
Banco de dados
```

- **Routes:** definem os endpoints HTTP.
- **Controllers:** recebem requisições e encaminham o fluxo.
- **Validators:** verificam os dados de entrada.
- **Services:** aplicam as regras de negócio.
- **Repositories:** definem contratos de persistência.
- **DAOs:** executam operações no banco.
- **Models:** representam as entidades do domínio.
- **Renderers:** formatam respostas JSON ou HTML.

## Estrutura

```text
fake-external-api/
├── config/
├── db/
│   ├── migrations/
│   └── seeds/
├── docs/
│   └── api.md
├── php/
│   └── src/
│       ├── Controller/
│       ├── DAO/
│       ├── Middleware/
│       ├── Model/
│       ├── Renderer/
│       ├── Repository/
│       ├── Service/
│       ├── Utils/
│       └── Validator/
├── public/
│   ├── assets/
│   ├── .htaccess
│   └── index.php
├── routes/
├── templates/
├── test/
├── bootstrap.php
├── composer.json
├── Dockerfile
└── docker-compose.yml
```

A estrutura inicial foi criada para organizar o projeto. As funcionalidades serão implementadas gradualmente.

## Requisitos

- PHP 8.2 ou superior;
- Composer 2;
- Slim Framework;
- banco de dados compatível;
- Docker, opcionalmente.

## Instalação

Clone o repositório:

```bash
git clone git@github.com:mariaclaramonteirop/fake-external-api.git
cd fake-external-api
```

Instale as dependências:

```bash
composer install
```

Gere ou atualize o autoload:

```bash
composer dump-autoload
```

## Composer e autoload

O namespace planejado da aplicação é:

```text
FakeExternalApi\\
```

As classes devem seguir a correspondência entre namespace e caminho do arquivo:

```text
FakeExternalApi\\Service\\ProductService
        ↓
php/src/Service/ProductService.php
```

Antes de implementar as classes, o caminho configurado no `composer.json` deve ser alinhado com a pasta utilizada pelo projeto: `src/` ou `php/src/`.

## Rotas planejadas

```text
GET    /health
GET    /products
GET    /products/{id}
POST   /products
PUT    /products/{id}
DELETE /products/{id}
GET    /stock/{sku}
GET    /customers
GET    /orders
```

## Documentação da API

A documentação detalhada está em [docs/api.md](docs/api.md).

Ela contém:

- endpoints;
- parâmetros;
- payloads;
- respostas JSON;
- códigos HTTP;
- regras de validação;
- idempotência;
- integração com o Craft CMS;
- versionamento.

## Twig

Twig poderá ser utilizado futuramente para páginas HTML, telas de consulta ou uma área administrativa.

A API REST continuará retornando JSON e não dependerá de Twig para os endpoints públicos.

## Estado atual

O projeto está em fase de estruturação. As pastas e os arquivos iniciais foram criados, mas as classes, rotas, banco de dados, testes e containers ainda serão implementados gradualmente.

## Próximas etapas

1. Alinhar o autoload do Composer.
2. Instalar e configurar o Slim Framework.
3. Criar o ponto de entrada em `public/index.php`.
4. Implementar a rota `/health`.
5. Configurar o banco de dados.
6. Implementar produtos e estoque.
7. Adicionar validações e testes.
8. Configurar Docker.
9. Integrar com a `integration-api`.
