# Automação de Testes de API — Banco Carrefour

[![API Tests](https://github.com/AlanJung/carrefour-api-automation/actions/workflows/api-tests.yml/badge.svg)](https://github.com/AlanJung/carrefour-api-automation/actions/workflows/api-tests.yml)

Suite completa de testes automatizados para a API [ServeRest](https://serverest.dev), cobrindo 100% dos endpoints de gerenciamento de usuários.

---

## Tecnologias

| Componente | Tecnologia |
|---|---|
| Testes | [Newman](https://www.npmjs.com/package/newman) (CLI do Postman) |
| Relatório HTML | [newman-reporter-htmlextra](https://www.npmjs.com/package/newman-reporter-htmlextra) |
| Relatório CI | JUnit XML |
| CI/CD | GitHub Actions |
| API Alvo | ServeRest — https://serverest.dev |

---

## Pré-requisitos

- [Node.js](https://nodejs.org/) >= 18

---

## Instalação

```bash
npm install
```

---

## Executar os Testes

```bash
npm test
```

Os relatórios serão gerados na pasta `reports/`:

| Arquivo | Descrição |
|---|---|
| `reports/html-report.html` | Relatório visual completo (abrir no navegador) |
| `reports/junit-report.xml` | Relatório JUnit para integração com CI |

---

## Estrutura do Projeto

```
Carrefour-automation/
├── .github/
│   └── workflows/
│       └── api-tests.yml          # Pipeline GitHub Actions
├── collections/
│   └── ServeRest_Usuarios.postman_collection.json
├── environments/
│   └── serverest.postman_environment.json
├── reports/                       # Gerado em runtime (gitignored)
├── package.json
└── README.md
```

---

## Casos de Teste

A suite possui **20 casos de teste** organizados em 8 pastas:

### 1. Setup — Criar Usuários para Testes
| # | Request | Status Esperado | Descrição |
|---|---|---|---|
| 1 | `POST /usuarios` | 201 | Cria usuário admin com email único (timestamp) |
| 2 | `POST /usuarios` | 201 | Cria segundo usuário não-admin |

### 2. Autenticação (`/login`)
| # | Request | Status Esperado | Descrição |
|---|---|---|---|
| 3 | `POST /login` | 200 | Login com credenciais válidas — salva token JWT |
| 4 | `POST /login` | 401 | Login com senha incorreta |

### 3. Criar Usuário (`POST /usuarios`)
| # | Request | Status Esperado | Descrição |
|---|---|---|---|
| 5 | `POST /usuarios` | 400 | Email já cadastrado |
| 6 | `POST /usuarios` | 400 | Campo `nome` ausente |
| 7 | `POST /usuarios` | 400 | Campo `email` ausente |
| 8 | `POST /usuarios` | 400 | Campo `password` ausente |
| 9 | `POST /usuarios` | 400 | Campo `administrador` ausente |

### 4. Listar Usuários (`GET /usuarios`)
| # | Request | Status Esperado | Descrição |
|---|---|---|---|
| 10 | `GET /usuarios` | 200 | Retorna lista com todos os campos |
| 11 | `GET /usuarios?nome=` | 200 | Filtra usuários por nome |

### 5. Buscar por ID (`GET /usuarios/:id`)
| # | Request | Status Esperado | Descrição |
|---|---|---|---|
| 12 | `GET /usuarios/:id` | 200 | Retorna usuário pelo ID correto |
| 13 | `GET /usuarios/:id` | 400 | ID inválido/inexistente |

### 6. Atualizar Usuário (`PUT /usuarios/:id`)
| # | Request | Status Esperado | Descrição |
|---|---|---|---|
| 14 | `PUT /usuarios/:id` | 200 | Atualiza dados do usuário existente |
| 15 | `PUT /usuarios/:id` | 400 | Tenta usar email já pertencente a outro usuário |
| 16 | `PUT /usuarios/:id` | 201 | ID inexistente — ServeRest cria novo usuário (upsert) |

### 7. Excluir Usuário (`DELETE /usuarios/:id`)
| # | Request | Status Esperado | Descrição |
|---|---|---|---|
| 17 | `DELETE /usuarios/:id` | 200 | Exclui usuário existente |
| 18 | `DELETE /usuarios/:id` | 200 | ID inexistente — retorna "Nenhum registro excluído" |

### 8. Teardown — Limpeza
| # | Request | Status Esperado | Descrição |
|---|---|---|---|
| 19 | `DELETE /usuarios/:id` | 200 | Remove userId2 criado no setup |
| 20 | `DELETE /usuarios/:id` | 200 | Remove usuário criado via upsert |

---

## Observações

### Mapeamento de endpoints
O desafio referencia os endpoints como `/users`. A ServeRest implementa esses endpoints como `/usuarios`. A collection está mapeada corretamente para `/usuarios`.

### Rate Limiting
A ServeRest limita a 100 requisições por minuto. Esta suite executa 20 requests, muito abaixo do limite. Em pipelines com múltiplos jobs paralelos, considere adicionar delay entre runs.

### Emails únicos
Cada execução gera emails únicos usando `Date.now()` no Pre-request Script, garantindo que os testes sejam idempotentes e não colidam entre execuções paralelas.

### Token JWT
O token JWT é obtido no teste de login e salvo como variável de coleção `{{token}}`. Ele expira em 600 segundos. A suite completa executa em menos de 1 minuto, então não há risco de expiração.

---

## CI/CD — GitHub Actions

A pipeline é disparada automaticamente em:
- Push para a branch `main`
- Pull Requests direcionados à `main`

### Artefatos gerados

Após cada execução, os relatórios ficam disponíveis como artefatos na aba **Actions** do GitHub por 30 dias:

- `newman-html-report` → `html-report.html`
- `newman-junit-report` → `junit-report.xml`

### Como acessar os artefatos

1. Acesse a aba **Actions** no repositório
2. Clique na execução desejada
3. Na seção **Artifacts**, baixe `newman-html-report`
4. Abra o arquivo `html-report.html` no navegador

---

## Importar no Postman

Para executar os testes diretamente pelo Postman:

1. Abra o Postman
2. **Import** → selecione `collections/ServeRest_Usuarios.postman_collection.json`
3. **Import** → selecione `environments/serverest.postman_environment.json`
4. Selecione o ambiente **ServeRest - Ambiente de Testes**
5. Execute a collection com o botão **Run**
