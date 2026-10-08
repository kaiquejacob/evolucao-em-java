# 🍃 Spring Data Poliglota — Marketplace

![Java](https://img.shields.io/badge/Java-25-0B0F14?style=for-the-badge&logo=openjdk&logoColor=2F81F7)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.0.5-0B0F14?style=for-the-badge&logo=springboot&logoColor=6DB33F)
![MySQL](https://img.shields.io/badge/MySQL-9.6-0B0F14?style=for-the-badge&logo=mysql&logoColor=4479A1)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-18-0B0F14?style=for-the-badge&logo=postgresql&logoColor=4169E1)
![MongoDB](https://img.shields.io/badge/MongoDB-8.2-0B0F14?style=for-the-badge&logo=mongodb&logoColor=47A248)
![Redis](https://img.shields.io/badge/Redis-8.6-0B0F14?style=for-the-badge&logo=redis&logoColor=DC382D)

---

## `SOBRE`

Marketplace de ingressos organizado em **três contextos de negócio**, cada um com o banco que melhor serve ao seu problema. É o conceito de **persistência poliglota**: em vez de forçar tudo em um único banco relacional, escolhe-se a tecnologia de armazenamento pelo padrão de acesso dos dados.

| Contexto       | Armazenamento                     | Porta  | Por que essa escolha                                              |
| -------------- | --------------------------------- | ------ | ----------------------------------------------------------------- |
| `registration` | MySQL (JPA)                       | `3307` | Cadastro de clientes: dados estruturados e transacionais          |
| `catalog`      | MySQL (JPA) + MongoDB + Redis     | `3308` / `27018` / `6380` | Evento no relacional, metadados flexíveis no documento, vitrine em cache |
| `ticketing`    | PostgreSQL (JPA) + Redis          | `5433` / `6381` | Assentos no relacional, trava temporária com expiração no Redis |

Os contextos **não se chamam diretamente**. Eles se comunicam por eventos de aplicação.

---

## `ROADMAP`

```
SPRING DATA POLIGLOTA

[✓] 01. Spring Data JPA
    ├── Múltiplos DataSources e EntityManagers (um por contexto)
    ├── Query methods e projeções
    └── Entity listeners (@PostPersist, @PostUpdate, @PostRemove)

[✓] 02. Spring Data REST
    ├── Repositórios expostos como API (@RepositoryRestResource)
    ├── Controle do que é exposto (@RestResource(exported = false))
    └── Eventos de repositório (@RepositoryEventHandler)

[✓] 03. Spring Data MongoDB
    ├── Documentos com esquema flexível
    ├── Auditoria (@CreatedDate, @LastModifiedDate)
    └── Listener de ciclo de vida (AbstractMongoEventListener)

[✓] 04. Spring Data Redis
    ├── Cache declarativo (@Cacheable + RedisCacheManager)
    └── @RedisHash com TTL para trava de assentos

[✓] 05. Integração entre contextos
    ├── Eventos de aplicação (ApplicationEventPublisher, @EventListener)
    ├── Processamento assíncrono (@Async + CompletableFuture)
    └── Virtual threads
```

---

## `ESTRUTURA`

```
02-spring-data/
├── src/main/java/dio/marketplace/
│   ├── registration/                  # contexto: clientes (MySQL)
│   │   ├── domain/
│   │   └── infrastructure/
│   │       ├── persistence/           # entidades JPA, projeção, repositórios
│   │       ├── event/                 # CustomerEventHandler
│   │       └── RegistrationConfiguration.java
│   ├── catalog/                       # contexto: vitrine (MySQL + Mongo + Redis)
│   │   ├── application/               # BrowseShowcaseUseCase, EventEnricher
│   │   ├── domain/
│   │   └── infrastructure/
│   │       ├── persistence/           # entidade JPA, documento Mongo, adapters
│   │       ├── event/                 # EventListener (JPA), EventMetadataEventListener (Mongo)
│   │       ├── http/                  # ShowcaseController
│   │       └── CatalogConfiguration.java
│   ├── ticketing/                     # contexto: venda (PostgreSQL + Redis)
│   │   ├── application/               # SelectSeatUseCase, CreateEventUseCase, ...
│   │   ├── domain/
│   │   └── infrastructure/
│   │       ├── persistence/           # entidades, SeatLock (Redis), adapters
│   │       ├── event/                 # TicketingEventListener
│   │       ├── http/                  # SeatSelectionController
│   │       └── TicketingConfiguration.java
│   ├── common/infrastructure/event/dto/   # CustomerCreated, EventUpdated
│   └── MarketplaceApplication.java
├── compose.yml                        # 6 containers (bancos e caches)
├── build.gradle
└── README.md
```

---

## `ARQUITETURA`

Cada contexto tem seu próprio banco, seu próprio `DataSource` e suas próprias entidades. Para evitar acoplamento, **a comunicação entre eles é feita por eventos**, e os DTOs desses eventos ficam em `common/`.

```
 registration                catalog                     ticketing
 ┌────────────┐        ┌───────────────────┐        ┌─────────────────┐
 │ Customer   │        │ Event  (MySQL)    │        │ Customer        │
 │ (MySQL)    │        │ EventMetadata     │        │ Event/Seat      │
 └─────┬──────┘        │   (MongoDB)       │        │   (PostgreSQL)  │
       │               └─────────┬─────────┘        │ SeatLock        │
       │ CustomerCreated         │ EventUpdated     │   (Redis, 30s)  │
       └─────────────────────────┴────────────────▶ └─────────────────┘
                 ApplicationEventPublisher  →  @EventListener @Async
```

- Ao cadastrar um cliente, `registration` publica **`CustomerCreated`**, e `ticketing` cria sua própria cópia do cliente.
- Ao salvar os metadados de um evento no MongoDB, `catalog` publica **`EventUpdated`**, e `ticketing` cria os setores e assentos.

---

## `CONCEITOS E CÓDIGO`

### 1. Múltiplos DataSources

O Spring Boot configura **um** `DataSource` automaticamente. Com vários bancos, a auto-configuração precisa ser substituída por beans explícitos, e cada conjunto de repositórios é amarrado ao seu `EntityManagerFactory` e ao seu `TransactionManager`.

Um contexto é marcado como `@Primary` (o padrão). Os demais usam `defaultCandidate = false` + `@Qualifier`, para não serem injetados por engano.

```java
@Configuration(proxyBeanMethods = false)
@EnableJpaRepositories(
        basePackages = "dio.marketplace.registration",
        entityManagerFactoryRef = "registrationEntityManagerFactory",
        transactionManagerRef = "registrationTransactionManager")
public class RegistrationConfiguration {

    @Primary
    @Bean
    @ConfigurationProperties("registration.datasource")
    public DataSourceProperties registrationDataSourceProperties() {
        return new DataSourceProperties();
    }

    @Primary
    @Bean
    public LocalContainerEntityManagerFactoryBean registrationEntityManagerFactory(
            DataSource dataSource, JpaProperties jpaProperties) {
        // ...
        return builder
                .dataSource(dataSource)
                .packages("dio.marketplace.registration")   // só escaneia as entidades deste contexto
                .persistenceUnit("registration")
                .build();
    }
}
```

As propriedades usam um prefixo por contexto (`registration.datasource.*`, `catalog.datasource.*`, `ticketing.datasource.*`).

### 2. Spring Data JPA, query methods e projeções

Basta declarar uma interface: o Spring gera a implementação. O nome do método vira a consulta (`StartingWith` + `IgnoreCase` → `LIKE 'x%'` sem diferenciar maiúsculas).

Uma **projeção** devolve só os campos necessários, em vez da entidade inteira. Aqui ela é usada como *excerpt*, ou seja, a visão resumida padrão na listagem da API.

```java
@RepositoryRestResource(excerptProjection = CustomerExcerpt.class)
public interface CustomerEntityRepository
        extends PagingAndSortingRepository<Customer, UUID>, CrudRepository<Customer, UUID> {

    List<Customer> findByFirstNameStartingWithIgnoreCase(@Param("firstName") String firstName);

    @Override
    @RestResource(exported = false)     // DELETE não é exposto na API
    void deleteById(UUID id);
}
```

```java
@Projection(name = "excerpt", types = Customer.class)
public interface CustomerExcerpt {
    String getFirstName();
    String getLastName();

    @Value("#{target.address?.toString()}")   // SpEL: endereço formatado, null-safe
    String getAddress();
}
```

### 3. Spring Data REST

Com `@RepositoryRestResource`, o repositório vira um **endpoint HATEOAS** (paginação, links, busca) sem escrever controller. É ótimo para CRUD simples, mas expõe a entidade diretamente, por isso o `exported = false` é usado para esconder operações (`deleteById`) e repositórios internos (`RedisSeatLockRepository`, `EventCrudRepository`).

### 4. Eventos do ciclo de vida

Há dois mecanismos diferentes no módulo, e vale saber a diferença:

| Mecanismo                       | Pertence a | Dispara quando                                  |
| ------------------------------- | ---------- | ----------------------------------------------- |
| `@EntityListeners` (`@PostPersist`…) | JPA        | a entidade é persistida/alterada/removida, por qualquer caminho |
| `@RepositoryEventHandler` (`@HandleAfterCreate`…) | Spring Data REST | a operação entra **pela API REST** |

```java
// catalog — JPA: dispara em qualquer persistência da entidade
public class EventListener {
    @PostPersist
    public void onEventCreated(Event event) { logger.info("Event created via @PostPersist {}", event); }
}
```

```java
// registration — Spring Data REST: dispara só quando o POST chega pela API
@Component
@RepositoryEventHandler
public class CustomerEventHandler {

    @HandleAfterCreate
    public void handleAfterCreate(Customer customer) {
        publisher.publishEvent(new CustomerCreated(customer.getId().toString(), customer.getFirstName()));
    }
}
```

### 5. MongoDB — esquema flexível

Os metadados de um evento variam muito (um show tem requisitos técnicos diferentes de uma peça). Em vez de criar colunas ou tabelas para cada variação, o documento usa um `Map<String, Object>` e listas aninhadas. A auditoria preenche as datas automaticamente.

```java
@Data
@Document
public class EventMetadata {
    @Id
    private String id;

    private UUID eventId;                               // elo com o evento no MySQL
    private String eventDescription;
    private Map<String, Object> technicalRequirements;  // estrutura livre
    private List<Sector> sectors;
    private List<Seat> seats;

    @CreatedDate  private Instant createdOn;
    @LastModifiedDate private Instant updatedAt;
}
```

Um `AbstractMongoEventListener` observa o ciclo de vida do documento e, após salvar, publica o evento `EventUpdated` para o contexto de ticketing:

```java
@Override
public void onAfterSave(AfterSaveEvent<EventMetadata> event) {
    this.publisher.publishEvent(EventUpdated.from(event.getSource()));
}
```

### 6. Redis — cache e trava com TTL

O Redis aparece em **duas instâncias** com papéis diferentes.

**Cache da vitrine (catalog).** O `@Cacheable` guarda o resultado do método; o `unless` evita cachear resultado vazio.

```java
@Cacheable(value = "showcase", unless = "#result.isEmpty()")
public List<EventOutput> execute() { /* ... */ }
```

**Trava de assento (ticketing).** `@RedisHash` com `timeToLive = 30` faz o Redis **apagar a trava sozinho** após 30 segundos. Se o cliente desistir de comprar, o assento volta a ficar disponível sem nenhum job de limpeza.

```java
@RedisHash(value = "seat_locks", timeToLive = 30)
@Data
public class SeatLock {
    @Id
    private String id;           // "eventId:seatId"

    @Indexed
    private String customerId;

    private Instant createdAt;
}
```

> ⚠️ **Limitação conhecida.** `tryLockSeat` faz `existsById` e depois `save`, que são duas operações separadas. Duas requisições simultâneas para o mesmo assento podem passar pela verificação ao mesmo tempo, e ambas conseguem a trava. O padrão correto para um lock distribuído é uma operação **atômica** (`SET key value NX EX 30`, em Spring: `setIfAbsent` com expiração).

### 7. Adapters: o domínio não conhece o banco

Os casos de uso dependem de interfaces do domínio (`EventRepository`, `EventMetadataRepository`). Os **adapters** na infraestrutura as implementam usando o Spring Data e convertem entre entidade de persistência e objeto de domínio.

```java
// domain — contrato, sem tecnologia
public interface EventMetadataRepository {
    Optional<EventMetadata> findByEventId(EventId eventId);
}

// infrastructure — adapter para MongoDB
@Repository
public class MongoEventMetadataRepository implements EventMetadataRepository {
    @Override
    public Optional<EventMetadata> findByEventId(EventId eventId) {
        return eventMetadataEntityRepository.findByEventId(eventId.id())
                .map(MongoEventMetadataRepository::mapper);
    }
}
```

O `WorkOfUnitEventRepository` mostra o ganho: **um único repositório de domínio** (`EventRepository`) é implementado combinando JPA (PostgreSQL) para existência do assento e Redis para a trava. O caso de uso não percebe que são dois bancos.

```java
public void execute(EventId eventId, SeatId seatId, CustomerId customerId) {
    if (!eventRepository.existsSeat(eventId, seatId)) {
        throw new SeatNotFoundException(eventId, seatId);
    }
    if (!eventRepository.tryLockSeat(eventId, seatId, customerId)) {
        throw new SeatAlreadyReservedException();
    }
}
```

### 8. Assincronismo e virtual threads

A vitrine precisa buscar os metadados de **cada** evento no MongoDB. Em vez de fazer isso em sequência, cada busca roda em paralelo com `@Async`, e o caso de uso aguarda todas com `CompletableFuture`.

```java
// EventEnricher
@Async
public CompletableFuture<Event> enrich(Event event) {
    var metadata = eventMetadataRepository.findByEventId(event.getId());
    event.setMetadata(metadata);
    return CompletableFuture.completedFuture(event);
}

// BrowseShowcaseUseCase
var futures = eventRepository.findAll().stream().map(eventEnricher::enrich).toList();
var events = futures.stream().map(CompletableFuture::join).map(EventOutput::from).toList();
```

Com `spring.threads.virtual.enabled=true`, essas tarefas rodam em **virtual threads** (Java 21+): são baratas e não bloqueiam threads de plataforma enquanto esperam I/O de banco.

---

## `ENDPOINTS`

| Método | Endpoint                                      | Origem               | Descrição                                       |
| ------ | --------------------------------------------- | -------------------- | ----------------------------------------------- |
| `GET`  | `/showcase`                                   | Controller           | Vitrine de eventos com metadados (cacheada)     |
| `POST` | `/ticketing/events/{eventId}/seats/select`    | Controller           | Reserva um assento (header `X-CUSTOMER-ID`)     |
| `*`    | `/customers`                                  | Spring Data REST     | CRUD de clientes (sem `DELETE`), paginado       |
| `GET`  | `/customers/search/findByFirstNameStartingWithIgnoreCase?firstName=` | Spring Data REST | Busca por prefixo do nome |
| `*`    | `/events`                                     | Spring Data REST     | CRUD de eventos do catálogo                     |
| `GET`  | `/actuator/health`                            | Actuator             | Saúde da aplicação e dos bancos                 |

Exemplo de seleção de assento:

```bash
curl -X POST http://localhost:8080/ticketing/events/{eventId}/seats/select \
  -H "Content-Type: application/json" \
  -H "X-CUSTOMER-ID: {customerId}" \
  -d '{"id": "A1"}'
```

---

## `COMO EXECUTAR`

**Pré-requisitos:** JDK 25 (toolchain do `build.gradle`) e Docker.

```bash
./gradlew bootRun       # Linux/macOS
gradlew.bat bootRun     # Windows
```

O módulo usa `spring-boot-docker-compose`: ao iniciar a aplicação, o Spring sobe os containers do `compose.yml` automaticamente (`lifecycle-management=start-only`, ou seja, ele inicia mas não derruba ao parar). Para subir manualmente:

```bash
docker compose up -d
```

| Serviço                       | Imagem         | Porta  |
| ----------------------------- | -------------- | ------ |
| `registration-database`       | `mysql:9.6`    | `3307` |
| `catalog-database`            | `mysql:9.6`    | `3308` |
| `catalog-metadata-database`   | `mongo:8.2`    | `27018`|
| `catalog-cache`               | `redis:8.6`    | `6380` |
| `ticketing-database`          | `postgres:18.3`| `5433` |
| `ticketing-locking`           | `redis:8.6`    | `6381` |

> As credenciais (`app`/`app`) são **apenas para desenvolvimento local**. O esquema é gerado por `hbm2ddl.auto=update`.

**Testes:** o módulo possui hoje o teste de contexto (`MarketplaceApplicationTests`), que valida se os três contextos, os múltiplos DataSources e as conexões sobem corretamente. Requer os containers ativos.

---

## `TECNOLOGIAS`

```
[SYSTEM STATUS]

Language     : JAVA 25
Framework    : SPRING BOOT 4.0.5
Relational   : MYSQL 9.6 (x2) / POSTGRESQL 18.3 — SPRING DATA JPA
Document     : MONGODB 8.2 — SPRING DATA MONGODB
Key-Value    : REDIS 8.6 (x2) — SPRING DATA REDIS (JEDIS)
API          : SPRING DATA REST + HAL EXPLORER
Ops          : ACTUATOR + DOCKER COMPOSE
Concurrency  : @ASYNC + COMPLETABLEFUTURE + VIRTUAL THREADS
Build        : GRADLE
Status       : COMPLETED
```
