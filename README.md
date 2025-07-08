# MassTransit + RabbitMQ + Retry + CQRS (Exemplo em .NET 8)

Este é um projeto de exemplo com as seguintes tecnologias:

- ✅ **.NET 8**
- ✅ **MassTransit** (abstração de mensageria)
- ✅ **RabbitMQ** (broker de mensagens)
- ✅ **CQRS** (Command Query Responsibility Segregation)
- ✅ **Retry automático** com intervalo configurável
- ✅ **Três projetos separados**: API, Worker e Contratos

---

## 📁 Estrutura do projeto

```
MassTransitExample/
├── MassTransitExample.API         → API que publica comandos
├── MassTransitExample.Worker      → Worker que consome da fila
├── MassTransitExample.Contracts   → Projeto compartilhado com mensagens (comandos)
└── README.md
```

---

## 🚀 Tecnologias utilizadas

| Tecnologia         | Descrição |
|--------------------|-----------|
| **.NET 8**         | Framework moderno, rápido e multiplataforma da Microsoft |
| **MassTransit**    | Biblioteca que abstrai brokers como RabbitMQ, Azure Service Bus, etc |
| **RabbitMQ**       | Broker de mensagens open-source (AMQP) |
| **CQRS**           | Separação de leitura (Query) e escrita (Command) |
| **Retry automático** | Reenvio automático de mensagens em caso de erro no processamento |

---

## ⚙️ Como executar

### 1. Requisitos

- [.NET 8 SDK](https://dotnet.microsoft.com/en-us/download)
- [Docker + Docker Compose](https://docs.docker.com/get-docker/) (ou uma instância RabbitMQ local)

---

### 2. Subir o RabbitMQ (com painel)

Crie um arquivo `docker-compose.yml`:

```yaml
version: '3.4'

services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"     # comunicação com apps
      - "15672:15672"   # painel web
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
```

Execute:

```bash
docker-compose up -d
```

Acesse o painel do RabbitMQ:
👉 http://localhost:15672  
(login: `guest`, senha: `guest`)

---

### 3. Restaurar e compilar o projeto

Na raiz do projeto:

```bash
dotnet restore
dotnet build
```

---

### 4. Executar os projetos

**Terminal 1 – API**:

```bash
dotnet run --project MassTransitExample.API
```

**Terminal 2 – Worker**:

```bash
dotnet run --project MassTransitExample.Worker
```

---

## 🧪 Como testar

### ✅ Cenário 1: sucesso

Use o Postman ou `curl` para enviar um comando:

```bash
curl -X POST http://localhost:5000/api/orders \
-H "Content-Type: application/json" \
-d '{ "orderId": "00000000-0000-0000-0000-000000000001", "customerName": "João" }'
```

✅ Você verá a mensagem sendo processada no console do Worker.

---

### ❌ Cenário 2: falha com retry

Envie um nome com `"erro"`:

```bash
curl -X POST http://localhost:5000/api/orders \
-H "Content-Type: application/json" \
-d '{ "orderId": "00000000-0000-0000-0000-000000000002", "customerName": "cliente erro" }'
```

⏳ O Worker vai:
- Lançar uma exceção proposital
- Repetir 3 vezes com intervalo de 5 segundos (configurado com `.UseMessageRetry()`)

---

## 🛠 Explicação técnica

### 📤 Publicação do comando
A API usa `IPublishEndpoint.Publish()` para enviar o `PlaceOrderCommand` ao RabbitMQ.

### 📥 Consumo do comando
O Worker escuta a fila `order-queue` e executa o `PlaceOrderConsumer`.

### 🔁 Retry automático
Usamos MassTransit com retry configurado via:

```csharp
cfg.UseMessageRetry(r => r.Interval(3, TimeSpan.FromSeconds(5)));
```

---

## 📚 Próximos passos

- [ ] Persistir pedidos no banco com Entity Framework Core
- [ ] Adicionar autenticação e validação
- [ ] Usar OpenTelemetry para tracing
- [ ] Implementar outros comandos e eventos com pub/sub

---

## 🧠 Dica

Se você quiser simular falhas reais, pode colocar um `throw new Exception()` no consumer ou derrubar o banco de dados intencionalmente.
