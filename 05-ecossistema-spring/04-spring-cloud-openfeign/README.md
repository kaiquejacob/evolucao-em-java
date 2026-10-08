# 🍃 Spring Cloud OpenFeign — Compliance

![Java](https://img.shields.io/badge/Java-25-0B0F14?style=for-the-badge&logo=openjdk&logoColor=2F81F7)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.0.5-0B0F14?style=for-the-badge&logo=springboot&logoColor=6DB33F)
![Spring Cloud](https://img.shields.io/badge/Spring%20Cloud-2025.1.1-0B0F14?style=for-the-badge&logo=spring&logoColor=6DB33F)
![Resilience4j](https://img.shields.io/badge/Resilience4j-Circuit%20Breaker-0B0F14?style=for-the-badge&logo=github&logoColor=2F81F7)

---

## `SOBRE`

Serviço de **análise de risco de empresas** (*compliance*). Quando uma empresa é cadastrada, a aplicação consulta duas APIs externas, uma de **sanções** e uma de **prevenção à lavagem de dinheiro (AML)**, aplica uma política de decisão e grava o resultado.

O módulo estuda como consumir APIs HTTP de forma **declarativa** com **Spring Cloud OpenFeign**, como protegê-las com **Circuit Breaker + fallback** e como **isolar o domínio** do formato das APIs externas.

As APIs externas são simuladas com **Mockoon**, então o módulo roda sem depender de serviços reais.

---

## `ROADMAP`

```
SPRING CLOUD OPENFEIGN

[✓] 01. Clientes HTTP declarativos
    ├── @FeignClient sobre interfaces
    ├── Configuração por propriedades (URL, headers, logs)
    └── @EnableFeignClients

[✓] 02. Resiliência
    ├── Circuit Breaker (Resilience4j)
    └── Fallback

[✓] 03. Integração segura com o domínio
    ├── DTOs externos com toDomain() (camada anticorrupção)
    └── Política de risco como regra de domínio pura

[✓] 04. Fluxo orientado a eventos
    ├── Spring Data REST + repositório em memória (KeyValue)
    └── @RepositoryEventHandler / @HandleAfterCreate

[✓] 05. Ambiente de desenvolvimento
    └── APIs simuladas com Mockoon
```

---

## `ESTRUTURA`

```
04-spring-cloud-openfeign/
├── src/main/java/dio/compliance/
│   ├── application/
│   │   └── AnalyzeCompanyRiskUseCase.java
│   ├── domain/
│   │   ├── Company.java, CompanyId.java, CompanyRepository.java
│   │   ├── CompliancePolicy.java          # regra de decisão
│   │   ├── ComplianceScreening.java       # resultado das triagens
│   │   ├── RiskAssessment.java            # score + nível + status
│   │   └── RiskAssessmentStatus.java, RiskLevel.java
│   ├── infrastructure/
│   │   ├── persistence/
│   │   │   ├── entity/                    # CompanyEntity
│   │   │   ├── event/                     # CompanyEventHandler
│   │   │   └── repository/               # CompanyEntityRepository, InMemoryCompanyRepository
│   │   └── rest/
│   │       ├── client/                    # SanctionClient, AntiMoneyLaunderingClient
│   │       └── dto/                       # SanctionResult, AmlResult
│   └── ComplianceApplication.java
├── src/main/resources/
│   ├── application.properties
│   ├── mockoon_kyc.json                   # API de sanções   (porta 3001)
│   └── mockoon_aml.json                   # API de AML       (porta 3002)
├── build.gradle
└── README.md
```

---

## `ARQUITETURA`

O cadastro de uma empresa dispara a análise **automaticamente**, por meio de um evento de repositório:

```
POST /companies
      │
      ▼
Spring Data REST salva a empresa
      │  @HandleAfterCreate
      ▼
CompanyEventHandler
      │
      ▼
AnalyzeCompanyRiskUseCase
      ├──▶ SanctionClient ───────▶ API de sanções   (:3001)  ─┐
      ├──▶ AntiMoneyLaunderingClient ▶ API de AML    (:3002)  ─┤  DTOs externos
      │                                                        ▼
      │                                   toDomain() → ComplianceScreening
      ▼
CompliancePolicy.evaluate(screening) → RiskAssessment
      │
      ▼
company.applyRiskAssessment(...) → CompanyRepository.save(...)
```

---

## `CONCEITOS E CÓDIGO`

### 1. Cliente HTTP declarativo

Sem o Feign, cada chamada exigiria montar a URL, serializar/deserializar JSON e tratar erros manualmente (`RestTemplate`/`WebClient`). Com o OpenFeign, você descreve **o contrato** em uma interface, e o Spring gera a implementação.

```java
@FeignClient("aml-client")
public interface AntiMoneyLaunderingClient {

    @GetMapping("/aml/v1/screening/{registrationNumber}")
    AmlResult screening(@PathVariable String registrationNumber);
}
```

As anotações são as mesmas do Spring MVC (`@GetMapping`, `@PathVariable`), então quem conhece controllers lê um client imediatamente. O `@EnableFeignClients` na classe principal ativa o escaneamento.

### 2. Configuração por propriedades

A URL e os headers **não ficam no código**. Cada client é configurado pelo nome (`sanction-client`, `aml-client`), o que permite trocar o ambiente sem recompilar.

```properties
spring.cloud.openfeign.client.config.sanction-client.url=http://localhost:3001
spring.cloud.openfeign.client.config.sanction-client.logger-level=full
spring.cloud.openfeign.client.config.sanction-client.default-request-headers.x-api-key=kyc-secret-123

spring.cloud.openfeign.client.config.aml-client.url=http://localhost:3002
spring.cloud.openfeign.client.config.aml-client.default-request-headers.authorization=Bearer xyz123

logging.level.dio.compliance.infrastructure.rest=DEBUG
```

O `logger-level=full` registra requisição e resposta completas, útil para depurar integrações. As chaves acima são **valores falsos para o Mockoon**; em integração real, credenciais vêm de variáveis de ambiente ou de um cofre de segredos.

### 3. Circuit Breaker e fallback

Quando uma API externa fica lenta ou cai, continuar chamando só piora: threads ficam presas e a falha se espalha. O **Circuit Breaker** monitora as chamadas e, depois de falhas repetidas, **abre o circuito** e passa a falhar imediatamente, sem tocar no serviço.

```
   CLOSED ──(muitas falhas)──▶ OPEN ──(tempo de espera)──▶ HALF-OPEN
      ▲                                                       │
      └──────────────(chamadas de teste com sucesso)──────────┘
```

Com `spring.cloud.openfeign.circuitbreaker.enabled=true` e o Resilience4j no classpath, o Feign passa a executar cada chamada dentro do circuit breaker. O **fallback** é a resposta alternativa usada quando a chamada falha ou o circuito está aberto:

```java
@FeignClient(name = "sanction-client", fallback = SanctionClient.Fallback.class)
public interface SanctionClient {

    @GetMapping("/sanctions/companies/{registrationNumber}")
    SanctionResult getCompanyRisk(@PathVariable String registrationNumber);

    @Component
    class Fallback implements SanctionClient {
        public SanctionResult getCompanyRisk(String registrationNumber) {
            return new SanctionResult(List.of());
        }
    }
}
```

> ⚠️ **Decisão de negócio em aberto.** O fallback devolve **lista vazia de sanções**, que o domínio interpreta como "nenhuma sanção encontrada". Em compliance, isso equivale a *fail-open*: se a API de sanções estiver fora do ar, a empresa pode ser aprovada sem ter sido verificada. O padrão mais seguro seria *fail-closed* (por exemplo, marcar a análise como `MANUAL_REVIEW`). Além disso, o `AntiMoneyLaunderingClient` **não** possui fallback, então uma falha ali interrompe a análise.

### 4. Camada anticorrupção (DTO externo → domínio)

O formato das APIs externas **não deve vazar** para o domínio. Se o fornecedor renomear um campo, só o DTO muda. Cada DTO externo converte a si mesmo para um tipo do domínio.

```java
public record SanctionResult(List<SanctionMatch> matches) {

    public List<ComplianceScreening.SanctionIdentity> toDomain() {
        if (matches() == null) {
            return List.of();
        }
        return matches().stream()
                .map(match -> new ComplianceScreening.SanctionIdentity(
                        match.entity(),
                        match.list(),
                        match.reason(),
                        match.confidenceScore() != null ? match.confidenceScore() : 0.0))
                .toList();
    }

    public record SanctionMatch(String entity, String list, String reason, Double confidenceScore) {}
}
```

Note o tratamento de `null`: dados externos são **não confiáveis** e a conversão os normaliza antes de entrarem no domínio.

### 5. Política de risco como regra de domínio pura

`CompliancePolicy.evaluate` é uma função **sem I/O e sem Spring**. Recebe o resultado das triagens e devolve a decisão, o que a torna trivial de testar.

```java
public static RiskAssessment evaluate(ComplianceScreening screening) {
    var status = RiskAssessmentStatus.APPROVED;

    boolean hasCriticalSanction = screening.sanctions().stream()
            .anyMatch(s -> s.confidence() > 0.8);

    if (hasCriticalSanction) {
        status = RiskAssessmentStatus.REJECTED;
    } else if (screening.amlProfile().isPepPresent()) {
        status = RiskAssessmentStatus.MANUAL_REVIEW;
    }

    int amlScore = screening.amlProfile().riskScore();

    if (status == RiskAssessmentStatus.APPROVED && amlScore > 70) {
        status = RiskAssessmentStatus.MANUAL_REVIEW;
    }

    return new RiskAssessment(amlScore, status);
}
```

| Condição                                    | Resultado         |
| ------------------------------------------- | ----------------- |
| Sanção com confiança > 0.8                  | `REJECTED`        |
| Pessoa politicamente exposta (PEP)          | `MANUAL_REVIEW`   |
| Score AML > 70 (e ainda aprovada)           | `MANUAL_REVIEW`   |
| Nenhuma das anteriores                      | `APPROVED`        |

O **nível de risco** é derivado no próprio `RiskAssessment`:

```java
private static RiskLevel determineRiskLevel(int score, RiskAssessmentStatus status) {
    if (status == RiskAssessmentStatus.REJECTED) return RiskLevel.CRITICAL;
    if (score > 70) return RiskLevel.HIGH;
    if (score > 30) return RiskLevel.MEDIUM;
    return RiskLevel.LOW;
}
```

### 6. Repositório em memória e evento de criação

O módulo usa **Spring Data KeyValue** (`@EnableMapRepositories`) para um repositório em `Map`, sem banco. O Spring Data REST o expõe como API e, após a criação, o `@RepositoryEventHandler` dispara a análise.

```java
@KeySpace("companies")
public class CompanyEntity {
    @Id
    private UUID id;
    private String name, registrationNumber;
    private RiskAssessment riskAssessment;
}

@RepositoryRestResource(path = "companies")
public interface CompanyEntityRepository extends CrudRepository<CompanyEntity, UUID> {}
```

```java
@Component
@RepositoryEventHandler
public class CompanyEventHandler {

    @HandleAfterCreate
    public void handleAfterCreateEvent(CompanyEntity entity) {
        this.analyzeCompanyRiskUseCase.execute(entity.toDomain());
    }
}
```

> Os dados vivem em memória e são perdidos ao reiniciar a aplicação.

### 7. Mocks de APIs externas com Mockoon

Os arquivos em `src/main/resources/` são ambientes do **Mockoon**, que sobe servidores HTTP falsos. As respostas usam templates (faker, `oneOf`) e variam a cada chamada, cobrindo cenários de risco `LOW`, `MEDIUM` e `HIGH` sem depender de um fornecedor real.

| Arquivo             | API simulada | Porta  | Rota                                          |
| ------------------- | ------------ | ------ | --------------------------------------------- |
| `mockoon_kyc.json`  | Sanções      | `3001` | `GET /sanctions/companies/:registrationNumber`|
| `mockoon_aml.json`  | AML          | `3002` | `GET /aml/v1/screening/:registrationNumber`   |

---

## `ENDPOINTS`

Expostos pelo Spring Data REST:

| Método | Endpoint            | Descrição                                                    |
| ------ | ------------------- | ------------------------------------------------------------ |
| `POST` | `/companies`        | Cadastra a empresa e **dispara a análise de risco**          |
| `GET`  | `/companies`        | Lista empresas com o `riskAssessment` calculado              |
| `GET`  | `/companies/{id}`   | Busca uma empresa                                            |
| `GET`  | `/actuator/health`  | Saúde da aplicação                                           |

```bash
curl -X POST http://localhost:8080/companies \
  -H "Content-Type: application/json" \
  -d '{"name": "Acme Ltda", "registrationNumber": "12345678000199"}'
```

---

## `COMO EXECUTAR`

**Pré-requisitos:** JDK 25 (toolchain do `build.gradle`) e o aplicativo **Mockoon**.

1. Abra o Mockoon, importe `mockoon_kyc.json` e `mockoon_aml.json` e **inicie os dois ambientes**.
2. Confira em `application.properties` se as URLs dos clients apontam para onde o Mockoon está rodando (`localhost`, se for na mesma máquina).
3. Execute a aplicação:

```bash
./gradlew bootRun            # Linux/macOS
gradlew.bat bootRun          # Windows
```

4. Cadastre uma empresa (comando `curl` acima) e consulte `GET /companies` para ver o `riskAssessment`.

**Testes:** o módulo possui hoje o teste de contexto (`ComplianceApplicationTests`), que valida se os clients Feign e o contexto Spring sobem corretamente. Testes da `CompliancePolicy` e do comportamento do fallback ainda não foram escritos.

---

## `TECNOLOGIAS`

```
[SYSTEM STATUS]

Language     : JAVA 25
Framework    : SPRING BOOT 4.0.5
Integration  : SPRING CLOUD OPENFEIGN (2025.1.1)
Resilience   : RESILIENCE4J CIRCUIT BREAKER + FALLBACK
Data         : SPRING DATA KEYVALUE (IN-MEMORY) + SPRING DATA REST
Mocks        : MOCKOON
Build        : GRADLE
Status       : COMPLETED
```
