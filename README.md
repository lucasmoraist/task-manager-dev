# task-manager-dev

Este projeto representa uma **Arquitetura de Microsserviços para um Sistema de Gerenciamento de Tarefas**, organizada como um **mono-repositório** utilizando **Git Submodules**.

O sistema é composto por dois microsserviços principais desenvolvidos em Spring Boot:

1.  **`task-ms`**: O core do sistema, responsável pela gestão de usuários, tarefas, autenticação e integração com o serviço de pagamento.
2.  **`payment-ms`**: Serviço focado em processar e validar transações de pagamento, garantindo a integridade dos dados através de assinatura digital.

-----

## 🏗️ Arquitetura do Sistema

O projeto adota uma arquitetura distribuída, com os serviços comunicando-se via REST (Feign Client) e Mensageria (RabbitMQ).

### 🛠️ Tecnologias Principais

| Categoria | Tecnologia | Uso |
| :--- | :--- | :--- |
| **Backend** | Java 21, Spring Boot 3 | Desenvolvimento dos microsserviços. |
| **Containerização** | Docker, Docker Compose | Orquestração do ambiente de desenvolvimento. |
| **Autenticação** | Keycloak (OAuth2) | Servidor de Autorização centralizado. |
| **Comunicação** | OpenFeign, RabbitMQ | Comunicação síncrona (REST) e assíncrona (Mensageria para notificações). |
| **Segurança** | JWT (RSA), Assinatura de Payload | Segurança baseada em Tokens e validação de integridade de dados. |

### 📁 Estrutura do Repositório

O repositório principal contém os seguintes submódulos:

  * **`task-ms`**: Microsserviço de Gerenciamento de Tarefas e Usuários.
  * **`payment-ms`**: Microsserviço dedicado ao Processamento Seguro de Pagamentos.

-----

## 🚀 Configuração e Execução

### 1\. Clonagem e Inicialização dos Submódulos

Como este é um repositório com submódulos, a clonagem deve ser feita de forma recursiva:

```bash
# 1. Clonar o repositório principal e inicializar os submódulos
git clone --recurse-submodules https://github.com/lucasmoraist/task-manager-dev.git
cd task-manager-dev

# (Se você clonou sem a flag, use:)
# git submodule update --init --recursive
```

### 2\. Inicializar Infraestrutura com Docker Compose

A infraestrutura compartilhada (Keycloak, Banco de Dados, RabbitMQ) está definida no arquivo `compose.yml` localizado dentro do submódulo `task-ms`.

Execute os serviços de apoio a partir do diretório `task-ms`:

```bash
cd task-ms
docker-compose -f compose.yml up -d
```

**Serviços em execução:**
| Serviço | Porta | Descrição |
| :--- | :--- | :--- |
| **PostgreSQL** (`db`) | `5432` | Banco de dados principal. |
| **Keycloak** (`keycloak`) | `8082` | Servidor de Autorização. |
| **RabbitMQ** (`rabbitmq`) | `15672` | Mensageria para notificações. |

### 3\. Execução dos Microsserviços

Os dois microsserviços devem ser executados separadamente. Certifique-se de estar usando a **Java 21**.

#### A. Executar `payment-ms`

O serviço de pagamento deve rodar na porta `8083` (configuração padrão em `application-default.yml`).

```bash
cd ../payment-ms
./gradlew bootRun
```

#### B. Executar `task-ms`

O serviço de tarefas deve rodar na porta `8080` (configuração padrão em `application-default.yml`).

```bash
cd ../task-ms
./gradlew bootRun
```

-----

## 📝 Detalhes dos Microsserviços

### 1\. `task-ms` (Porta: 8080)

Gerencia usuários e tarefas, atuando como o principal ponto de entrada.

| Funcionalidade | Endpoint (Exemplo) | Detalhes |
| :--- | :--- | :--- |
| **Autenticação** | `POST /v1/auth/login` | Gera JWT para acesso. |
| **Usuários** | `POST /v1/user/signup` | Criação de usuário (possibilidade de criar `ADMIN` com `x-application-key`). |
| **Tarefas** | `POST /v1/task/create` | Cria e gerencia tarefas. |
| **Premium** | `POST /v1/premium/subscribe` | Inicia o processo de pagamento, chamando o `payment-ms`. |

### 2\. `payment-ms` (Porta: 8083)

Protege a lógica de processamento de pagamentos.

| Funcionalidade | Endpoint | Segurança e Validações |
| :--- | :--- | :--- |
| **Processar Pagamento** | `POST /v1/payments/create` | Requer autenticação **OAuth2** (Keycloak). |
| | | Valida a **assinatura digital** do payload via cabeçalho `x-payload-hash`. |
| | | Valida regras de negócio (ex: valor da transação `> 0`, data não futura). |

### 🗝️ Criação de Usuário Administrador (Admin Key)

Para criar um usuário com a role `ADMIN` no `task-ms`, inclua o cabeçalho `x-application-key` na rota `/v1/user/signup` com o valor: `94ab834f-43ca-4654-8ef5-1a6102fa156d`.
