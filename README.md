
# Tarefa — Space Weather Monitor API com .NET 8

## Contexto

Você vai criar uma pequena API em .NET 8 que consome a API pública da NASA DONKI.

O DONKI é uma base de dados da NASA voltada para eventos de clima espacial, como ejeções de massa coronal, tempestades geomagnéticas, erupções solares e notificações relacionadas.

Nesta tarefa, o objetivo não é apenas “chamar uma API externa”. O objetivo é praticar estrutura de aplicação, separação de responsabilidades, validação, tratamento de erro, testes de integração e uso básico de Docker.

---

## Objetivo principal

Criar uma Web API chamada **SpaceWeather.Api** que permite consultar eventos de clima espacial da NASA de forma simplificada.

A API deve consumir pelo menos estes endpoints da NASA DONKI:

- Solar Flare — `DONKI/FLR`
- Geomagnetic Storm — `DONKI/GST`
- Notifications — `DONKI/notifications`

### Referências da NASA API

- Documentação geral da NASA API: https://api.nasa.gov/
- Documentação e cadastro de API Key: https://api.nasa.gov/
- Endpoint Solar Flare: `https://api.nasa.gov/DONKI/FLR`
- Endpoint Geomagnetic Storm: `https://api.nasa.gov/DONKI/GST`
- Endpoint Notifications: `https://api.nasa.gov/DONKI/notifications`

Exemplos de chamadas:

```http
GET https://api.nasa.gov/DONKI/FLR?startDate=2024-01-01&endDate=2024-01-31&api_key=DEMO_KEY
GET https://api.nasa.gov/DONKI/GST?startDate=2024-01-01&endDate=2024-01-31&api_key=DEMO_KEY
GET https://api.nasa.gov/DONKI/notifications?startDate=2024-01-01&endDate=2024-01-07&type=all&api_key=DEMO_KEY
```

---

## Requisitos funcionais

### 1. Consultar erupções solares

Criar o endpoint:

```http
GET /api/space-weather/solar-flares?startDate=2024-01-01&endDate=2024-01-31
```

A API deve:

- Receber `startDate` e `endDate`.
- Consultar o endpoint `DONKI/FLR` da NASA.
- Retornar uma lista simplificada de eventos.

Resposta esperada:

```json
[
  {
    "id": "2024-01-01T00:00:00-FLR-001",
    "beginTime": "2024-01-01T00:00:00Z",
    "peakTime": "2024-01-01T00:10:00Z",
    "endTime": "2024-01-01T00:20:00Z",
    "classType": "M1.2",
    "sourceLocation": "N10W20"
  }
]
```

---

### 2. Consultar tempestades geomagnéticas

Criar o endpoint:

```http
GET /api/space-weather/geomagnetic-storms?startDate=2024-01-01&endDate=2024-01-31
```

A API deve:

- Receber `startDate` e `endDate`.
- Consultar o endpoint `DONKI/GST`.
- Retornar dados simplificados da tempestade geomagnética.

Resposta esperada:

```json
[
  {
    "id": "2024-01-01T00:00:00-GST-001",
    "startTime": "2024-01-01T00:00:00Z",
    "kpIndex": 5.67,
    "linkedEventsCount": 2
  }
]
```

---

### 3. Consultar notificações

Criar o endpoint:

```http
GET /api/space-weather/notifications?startDate=2024-01-01&endDate=2024-01-07&type=all
```

A API deve:

- Receber `startDate`, `endDate` e `type`.
- Consultar o endpoint `DONKI/notifications`.
- Retornar notificações resumidas.

Resposta esperada:

```json
[
  {
    "messageId": "20240101-AL-001",
    "messageType": "Space Weather Notification",
    "messageIssueTime": "2024-01-01T10:00:00Z",
    "bodyPreview": "Geomagnetic storm observed..."
  }
]
```

---

## Requisitos técnicos

A aplicação deve ser criada usando:

- .NET 8
- ASP.NET Core Web API
- HttpClientFactory
- FluentValidation
- AutoMapper
- Docker
- Docker Compose
- Testes de integração primeiro

---

## Estrutura esperada da solução

Criar a solução seguindo esta organização:

```txt
SpaceWeather.sln

src/
  SpaceWeather.Api/
  SpaceWeather.Application/
  SpaceWeather.Domain/
  SpaceWeather.Infrastructure/

tests/
  SpaceWeather.IntegrationTests/
```

---

## Responsabilidade de cada projeto

### SpaceWeather.Api

Responsável por:

- Controllers ou Minimal APIs
- Configuração da aplicação
- Injeção de dependência
- Middlewares
- Swagger
- Entrada e saída HTTP

Não deve conter regra de negócio.

---

### SpaceWeather.Application

Responsável por:

- Casos de uso
- Commands
- Queries
- Handlers
- DTOs
- Validators
- Interfaces da aplicação

Exemplo de organização:

```txt
Application/
  SpaceWeather/
    SolarFlares/
      GetSolarFlaresQuery.cs
      GetSolarFlaresQueryHandler.cs
      GetSolarFlaresQueryValidator.cs

    GeomagneticStorms/
      GetGeomagneticStormsQuery.cs
      GetGeomagneticStormsQueryHandler.cs
      GetGeomagneticStormsQueryValidator.cs

    Notifications/
      GetNotificationsQuery.cs
      GetNotificationsQueryHandler.cs
      GetNotificationsQueryValidator.cs
```

---

### SpaceWeather.Domain

Responsável por:

- Entidades de domínio
- Value Objects
- Enums
- Regras puras de negócio

Para esta tarefa, o domínio pode ser simples.

Exemplo:

```txt
Domain/
  SpaceWeatherEvent.cs
  SpaceWeatherEventType.cs
  DateRange.cs
```

---

### SpaceWeather.Infrastructure

Responsável por:

- Cliente HTTP da NASA
- Implementações concretas de interfaces
- Mapeamento de respostas externas
- Configuração de serviços externos

Exemplo:

```txt
Infrastructure/
  NASA/
    NasaDonkiClient.cs
    NasaDonkiOptions.cs
    Responses/
      NasaSolarFlareResponse.cs
      NasaGeomagneticStormResponse.cs
      NasaNotificationResponse.cs
```

---

## CQS — Command Query Separation

A aplicação deve seguir a ideia de CQS.

Como esta tarefa é apenas de consulta, você provavelmente criará apenas **queries**.

Exemplo:

```csharp
public sealed record GetSolarFlaresQuery(
    DateOnly StartDate,
    DateOnly EndDate
);
```

O handler deve conter a orquestração do caso de uso:

```csharp
public sealed class GetSolarFlaresQueryHandler
{
    public async Task<IReadOnlyList<SolarFlareDto>> HandleAsync(
        GetSolarFlaresQuery query,
        CancellationToken cancellationToken)
    {
        // validar entrada
        // chamar client da NASA
        // mapear resposta
        // retornar DTO simplificado
    }
}
```

---

## Validações obrigatórias

Usar FluentValidation para validar:

- `startDate` é obrigatório.
- `endDate` é obrigatório.
- `startDate` não pode ser maior que `endDate`.
- O intervalo entre `startDate` e `endDate` não pode ser maior que 30 dias.
- `type` em notificações deve aceitar inicialmente:
  - `all`
  - `FLR`
  - `GST`

Quando a validação falhar, a API deve retornar HTTP `400 Bad Request`.

---

## Configuração da API Key

A chave da NASA não deve ficar hardcoded no código.

Usar configuração via:

```json
{
  "Nasa": {
    "BaseUrl": "https://api.nasa.gov",
    "ApiKey": "DEMO_KEY"
  }
}
```

Permitir sobrescrever via variável de ambiente:

```bash
Nasa__ApiKey=DEMO_KEY
```

---

## Tratamento de erro

A aplicação deve tratar pelo menos estes cenários:

### NASA fora do ar ou indisponível

Retornar:

```http
503 Service Unavailable
```

### NASA retorna erro de limite de requisição

Retornar:

```http
429 Too Many Requests
```

### Entrada inválida

Retornar:

```http
400 Bad Request
```

### Erro inesperado

Retornar:

```http
500 Internal Server Error
```

---

## Testes de integração

Priorizar testes de integração, não testes unitários.

Criar testes em:

```txt
tests/SpaceWeather.IntegrationTests
```

Os testes devem subir a aplicação em memória usando `WebApplicationFactory`.

Os testes devem validar pelo menos:

### Solar flares com período válido

Dado um `startDate` e `endDate` válidos, quando chamar:

```http
GET /api/space-weather/solar-flares
```

Então deve retornar HTTP `200 OK`.

---

### Datas inválidas

Dado `startDate` maior que `endDate`, quando chamar qualquer endpoint, então deve retornar HTTP `400 Bad Request`.

---

### Intervalo maior que 30 dias

Dado um intervalo maior que 30 dias, então deve retornar HTTP `400 Bad Request`.

---

### NASA indisponível

Simular falha no client externo e garantir que a API retorna HTTP `503 Service Unavailable`.

---

## Importante sobre testes

Não criar muitos testes unitários apenas para “aumentar cobertura”.

A prioridade desta tarefa é provar que a API funciona de ponta a ponta.

Testes unitários só devem ser criados se realmente ajudarem a testar uma regra isolada, como `DateRange`.

---

## Docker

A aplicação deve ter um `Dockerfile`.

Também deve existir um `docker-compose.yml` para rodar a API localmente.

Exemplo de execução esperada:

```bash
docker compose up --build
```

Depois disso, a API deve estar disponível em:

```txt
http://localhost:8080/swagger
```

---

## Critérios de aceite

A tarefa será considerada concluída quando:

- A aplicação compilar sem erros.
- O Swagger abrir corretamente.
- Os três endpoints estiverem disponíveis.
- A API consumir a NASA DONKI API.
- A API Key estiver configurada por variável de ambiente ou `appsettings`.
- As validações estiverem implementadas com FluentValidation.
- O AutoMapper estiver sendo usado para transformar respostas externas em DTOs internos.
- Os testes de integração principais estiverem passando.
- O projeto rodar via Docker Compose.
- O README explicar como rodar o projeto.

---

## README esperado

O projeto deve conter um `README.md` com:

- Descrição do projeto.
- Como configurar a NASA API Key.
- Como rodar localmente.
- Como rodar com Docker.
- Como executar os testes.
- Lista dos endpoints disponíveis.
- Decisões técnicas tomadas.
- O que poderia ser melhorado em uma próxima versão.

---

## Desafio extra opcional

Caso termine a tarefa principal, implementar um endpoint de resumo:

```http
GET /api/space-weather/summary?startDate=2024-01-01&endDate=2024-01-31
```

Resposta esperada:

```json
{
  "startDate": "2024-01-01",
  "endDate": "2024-01-31",
  "solarFlaresCount": 10,
  "geomagneticStormsCount": 2,
  "notificationsCount": 5,
  "mostRelevantEventType": "FLR"
}
```

Esse endpoint deve chamar internamente os handlers já existentes, sem duplicar lógica.

---

## O que você deve estudar durante a tarefa

Durante a execução, estudar e anotar:

- O que é ASP.NET Core Web API.
- O que é HttpClientFactory.
- Por que não devemos instanciar `HttpClient` manualmente em todo lugar.
- O que é CQS.
- Diferença entre DTO interno e resposta de API externa.
- O que é FluentValidation.
- O que é AutoMapper.
- O que é teste de integração.
- Diferença entre teste unitário e teste de integração.
- Como funciona configuração por variável de ambiente no .NET.
- Como containerizar uma aplicação .NET.

---

## Perguntas para responder no final

Ao concluir, responda no README ou em um arquivo `LEARNINGS.md`:

1. Qual foi a maior dificuldade da tarefa?
2. O que você entendeu sobre separação entre API, Application, Domain e Infrastructure?
3. Por que a API Key não deve ficar hardcoded?
4. Qual a diferença entre testar um handler isolado e testar o endpoint completo?
5. O que você melhoraria se essa API fosse para produção?
6. Onde você usaria cache nesta aplicação?
7. Como você lidaria com rate limit da NASA em uma aplicação real?
