# 🍃 DevDojo Spring Boot 2 Essentials

![Java](https://img.shields.io/badge/Java-11%2B-0B0F14?style=for-the-badge&logo=openjdk&logoColor=2F81F7)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.x-0B0F14?style=for-the-badge&logo=springboot&logoColor=6DB33F)
![Maven](https://img.shields.io/badge/Maven-Build-0B0F14?style=for-the-badge&logo=apachemaven&logoColor=C71A36)
![Docker](https://img.shields.io/badge/Docker-Compose-0B0F14?style=for-the-badge&logo=docker&logoColor=2496ED)

---

## `SOBRE`

Projeto desenvolvido durante o curso **Spring Boot 2 Essentials**, da **DevDojo**. É uma API REST de gerenciamento de animes, construída em camadas, que serve de base prática para os conceitos do ecossistema Spring: injeção de dependência, persistência com JPA, validação, tratamento global de erros, paginação, segurança e testes.

> ⚠️ **Projeto guiado.** A estrutura e o domínio seguem o curso. O objetivo aqui é registrar o que foi aprendido, com a teoria de cada camada documentada abaixo.

> ⚠️ **Versão.** O projeto usa **Spring Boot 2.x** (pacotes `javax.*`). O restante do repositório mira Spring Boot 3+ (`jakarta.*`). As diferenças estão na seção [`PRÓXIMOS PASSOS`](#próximos-passos).

---

## `ROADMAP`

```
SPRING BOOT 2 ESSENTIALS

[✓] 01. Fundamentos Spring Boot
    ├── Auto-configuration e starters
    ├── Inversão de Controle (IoC) e Injeção de Dependência
    └── Configuração via application.properties

[✓] 02. API REST em camadas
    ├── Controller, Service, Repository
    ├── DTOs de entrada (requests) e Mapper
    └── ResponseEntity e status HTTP

[✓] 03. Persistência
    ├── Spring Data JPA (JpaRepository, query methods)
    ├── Transações (@Transactional)
    └── Banco via Docker Compose

[✓] 04. Qualidade e robustez
    ├── Bean Validation (@Valid)
    ├── Tratamento global de exceções (@ControllerAdvice)
    └── Paginação e ordenação (Pageable)

[✓] 05. Segurança e integração
    ├── Spring Security (HTTP Basic, UserDetailsService)
    └── Consumo de APIs com RestTemplate

[✓] 06. Testes
    ├── Testes unitários com JUnit 5 e Mockito
    └── Testes de repositório e integração
```

---

## `ESTRUTURA`

```
springboot2-essentials/
├── src/
│   ├── main/
│   │   ├── java/academy/devdojo/springboot2/
│   │   │   ├── client/        # consumo de APIs (RestTemplate)
│   │   │   ├── config/        # configurações (segurança)
│   │   │   ├── configurer/    # customizações do Spring
│   │   │   ├── controller/    # camada web (endpoints REST)
│   │   │   ├── domain/        # entidades JPA
│   │   │   ├── exception/     # exceções de negócio
│   │   │   ├── handler/       # tratamento global de exceções
│   │   │   ├── mapper/        # conversão DTO <-> entidade
│   │   │   ├── repository/    # acesso a dados
│   │   │   ├── requests/      # DTOs de entrada
│   │   │   ├── service/       # regras de negócio
│   │   │   ├── util/          # classes utilitárias
│   │   │   ├── wrapper/       # wrappers de resposta (paginação)
│   │   │   └── Springboot2EssentialsApplication.java
│   │   └── resources/         # application.properties
│   └── test/                  # testes automatizados
├── docker-compose.yml         # banco de dados local
├── pom.xml
├── mvnw / mvnw.cmd            # Maven Wrapper
└── README.md
```

---

## `ARQUITETURA EM CAMADAS`

Cada camada tem **uma única responsabilidade** e só conversa com a camada imediatamente abaixo. Isso é o princípio da responsabilidade única (SRP) do SOLID aplicado à arquitetura.

```
  Cliente HTTP
       │
       ▼
┌──────────────┐   recebe a requisição, valida (@Valid), devolve ResponseEntity
│  Controller  │
└──────┬───────┘
       ▼
┌──────────────┐   regras de negócio, transações, lança exceções de domínio
│   Service    │
└──────┬───────┘
       ▼
┌──────────────┐   abstrai o acesso ao banco (Spring Data JPA)
│  Repository  │
└──────┬───────┘
       ▼
   Banco de dados
```

**Por que separar?** Um `Controller` que acessa o banco diretamente mistura protocolo HTTP com persistência. Qualquer mudança em um lado quebra o outro, e a lógica fica impossível de testar isoladamente.

---

## `CONCEITOS E CÓDIGO`

### 1. Inversão de Controle e Injeção de Dependência

Em vez de a classe criar suas dependências com `new`, o **container do Spring** (o `ApplicationContext`) instancia os objetos (*beans*) e os entrega prontos. A classe só declara o que precisa.

Isso reduz o acoplamento: o `Service` depende de uma abstração (`AnimeRepository`) e não de uma implementação concreta, o que permite trocar a implementação por um *mock* nos testes.

```java
@Service
@RequiredArgsConstructor // Lombok gera o construtor com os campos final
public class AnimeService {

    private final AnimeRepository animeRepository; // injetado pelo Spring
}
```

> Prefira **injeção por construtor** em vez de `@Autowired` em campo. As dependências ficam explícitas, o campo pode ser `final` e a classe é instanciável nos testes sem o Spring.

### 2. Entidade JPA

Uma **entidade** é uma classe Java mapeada para uma tabela. O **JPA** é a especificação e o **Hibernate** é a implementação que gera o SQL.

```java
@Entity
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Anime {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotEmpty(message = "The anime name cannot be empty")
    private String name;
}
```

### 3. Repository com Spring Data JPA

Basta declarar uma **interface**. O Spring gera a implementação em tempo de execução. Os *query methods* transformam o nome do método em SQL.

```java
public interface AnimeRepository extends JpaRepository<Anime, Long> {

    // SELECT * FROM anime WHERE name = ?
    List<Anime> findByName(String name);
}
```

`JpaRepository` já entrega `save`, `findById`, `findAll(Pageable)`, `delete` e outros, sem escrever uma linha de SQL.

### 4. DTOs e validação

Expor a entidade direto na API acopla o contrato HTTP ao modelo do banco. Os **DTOs** (`requests/`) definem apenas o que o cliente pode enviar, e a **Bean Validation** rejeita dados inválidos antes de chegar ao `Service`.

```java
@Data
@Builder
public class AnimePostRequestBody {

    @NotEmpty(message = "The anime name cannot be empty")
    private String name;
}
```

```java
@PostMapping
public ResponseEntity<Anime> save(@RequestBody @Valid AnimePostRequestBody body) {
    return new ResponseEntity<>(animeService.save(body), HttpStatus.CREATED);
}
```

Se `name` vier vazio, o Spring lança `MethodArgumentNotValidException` e o método nem é executado.

### 5. Mapper (MapStruct)

O **MapStruct** gera, em tempo de compilação, o código de conversão entre DTO e entidade. É mais rápido que reflection e evita código manual repetitivo.

```java
@Mapper(componentModel = "spring")
public abstract class AnimeMapper {

    public static final AnimeMapper INSTANCE = Mappers.getMapper(AnimeMapper.class);

    public abstract Anime toAnime(AnimePostRequestBody body);

    public abstract Anime toAnime(AnimePutRequestBody body);
}
```

### 6. Service e transações

O `Service` concentra as regras de negócio. O `@Transactional` abre uma transação: se qualquer operação falhar, **todas** sofrem *rollback* (propriedade de atomicidade do ACID).

```java
@Transactional
public Anime save(AnimePostRequestBody body) {
    return animeRepository.save(AnimeMapper.INSTANCE.toAnime(body));
}

public Anime findByIdOrThrowBadRequestException(long id) {
    return animeRepository.findById(id)
            .orElseThrow(() -> new BadRequestException("Anime not found"));
}
```

> Por padrão, o Spring faz *rollback* apenas para exceções não verificadas (`RuntimeException`).

### 7. Controller REST

O `Controller` traduz HTTP para chamadas de `Service`. O `ResponseEntity` dá controle sobre o **status** e o corpo da resposta.

```java
@RestController
@RequestMapping("animes")
@RequiredArgsConstructor
public class AnimeController {

    private final AnimeService animeService;

    @GetMapping("/{id}")
    public ResponseEntity<Anime> findById(@PathVariable long id) {
        return ResponseEntity.ok(animeService.findByIdOrThrowBadRequestException(id));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable long id) {
        animeService.delete(id);
        return ResponseEntity.noContent().build(); // 204
    }
}
```

### 8. Tratamento global de exceções

Sem tratamento, um erro vira um *stack trace* e um `500`. O `@ControllerAdvice` intercepta as exceções de **todos** os controllers e devolve uma resposta padronizada.

```java
@ControllerAdvice
public class RestExceptionHandler {

    @ExceptionHandler(BadRequestException.class)
    public ResponseEntity<ExceptionDetails> handleBadRequest(BadRequestException ex) {
        ExceptionDetails details = ExceptionDetails.builder()
                .title("Bad Request Exception, Check the Documentation")
                .status(HttpStatus.BAD_REQUEST.value())
                .details(ex.getMessage())
                .timestamp(LocalDateTime.now())
                .build();
        return new ResponseEntity<>(details, HttpStatus.BAD_REQUEST);
    }
}
```

### 9. Paginação e ordenação

Devolver milhares de registros de uma vez derruba a aplicação e a rede. O Spring resolve o `Pageable` a partir dos parâmetros da URL e o repositório aplica `LIMIT`/`OFFSET` no SQL.

```java
@GetMapping
public ResponseEntity<Page<Anime>> list(Pageable pageable) {
    return ResponseEntity.ok(animeService.listAll(pageable));
}
```

```
GET /animes?page=0&size=5&sort=name,desc
```

### 10. Spring Security

A segurança é uma **cadeia de filtros** executada antes do `Controller`. Neste projeto: autenticação **HTTP Basic**, com usuários carregados do banco por um `UserDetailsService` e senhas armazenadas com hash (`PasswordEncoder`), nunca em texto puro.

```java
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final UserDetailsService userDetailsService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests()
                .anyRequest().authenticated()
            .and()
            .httpBasic();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
            .passwordEncoder(PasswordEncoderFactories.createDelegatingPasswordEncoder());
    }
}
```

> O CSRF só é desativado porque a API é *stateless* e não usa cookies de sessão. Em aplicações com sessão, ele deve continuar ativo.

### 11. Consumo de APIs com RestTemplate

O pacote `client/` demonstra como a aplicação consome outra API HTTP.

```java
ResponseEntity<Anime> entity = new RestTemplate()
        .getForEntity("http://localhost:8080/animes/{id}", Anime.class, 2);

log.info(entity.getBody());
```

### 12. Testes

Testes unitários isolam o `Service` com **Mockito**, trocando o repositório por um *mock*. Testes de repositório usam `@DataJpaTest`, que sobe apenas a camada JPA com um banco em memória.

```java
@ExtendWith(SpringExtension.class)
class AnimeServiceTest {

    @InjectMocks
    private AnimeService animeService;

    @Mock
    private AnimeRepository animeRepositoryMock;

    @Test
    void findByIdOrThrowBadRequestException_ThrowsException_WhenAnimeNotFound() {
        BDDMockito.when(animeRepositoryMock.findById(ArgumentMatchers.anyLong()))
                .thenReturn(Optional.empty());

        Assertions.assertThatExceptionOfType(BadRequestException.class)
                .isThrownBy(() -> animeService.findByIdOrThrowBadRequestException(1));
    }
}
```

---

## `ENDPOINTS`

| Método   | Endpoint        | Descrição                         | Status        |
| -------- | --------------- | --------------------------------- | ------------- |
| `GET`    | `/animes`       | Lista animes (paginado)           | `200`         |
| `GET`    | `/animes/{id}`  | Busca anime por ID                | `200` / `400` |
| `GET`    | `/animes/find`  | Busca anime por nome              | `200`         |
| `POST`   | `/animes`       | Cadastra um anime                 | `201` / `400` |
| `PUT`    | `/animes`       | Atualiza um anime                 | `204` / `400` |
| `DELETE` | `/animes/{id}`  | Remove um anime                   | `204` / `400` |

<!-- TODO: confirmar rotas, verbos e status exatos conforme o AnimeController -->

---

## `COMO EXECUTAR`

**Pré-requisitos:** JDK compatível com o `pom.xml` (`<java.version>`), Docker e Docker Compose.

```bash
# 1. Subir o banco de dados
docker compose up -d

# 2. Executar a aplicação (Maven Wrapper, sem precisar instalar o Maven)
./mvnw spring-boot:run          # Linux/macOS
mvnw.cmd spring-boot:run        # Windows

# 3. Rodar os testes
./mvnw test
```

A API sobe em `http://localhost:8080`.

```bash
curl -u usuario:senha http://localhost:8080/animes
```

<!-- TODO: confirmar porta, usuário/senha de exemplo e variáveis de ambiente do banco -->

---

## `TECNOLOGIAS`

```
[SYSTEM STATUS]

Language   : JAVA <!-- TODO: versão do pom.xml -->
Framework  : SPRING BOOT 2.x
Data       : SPRING DATA JPA / HIBERNATE
Security   : SPRING SECURITY (HTTP BASIC)
Mapping    : MAPSTRUCT + LOMBOK
Build      : MAVEN (WRAPPER)
Infra      : DOCKER COMPOSE
Tests      : JUNIT 5 + MOCKITO
Status     : COMPLETED
```

## `CRÉDITOS`

Conteúdo baseado no curso **Spring Boot 2 Essentials** da [DevDojo](https://github.com/devdojobr).
