# 🍃 Spring Security — Proposal Management

![Java](https://img.shields.io/badge/Java-25-0B0F14?style=for-the-badge&logo=openjdk&logoColor=2F81F7)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.0.5-0B0F14?style=for-the-badge&logo=springboot&logoColor=6DB33F)
![Spring Security](https://img.shields.io/badge/Spring%20Security-RBAC-0B0F14?style=for-the-badge&logo=springsecurity&logoColor=6DB33F)
![MySQL](https://img.shields.io/badge/MySQL-9.6-0B0F14?style=for-the-badge&logo=mysql&logoColor=4479A1)

---

## `SOBRE`

API de gestão de propostas entre **influenciadores** e **marcas**, usada para estudar autenticação e autorização com **Spring Security**. O módulo cobre:

- **Autenticação** com um filtro customizado que aceita login em **JSON** (em vez do formulário padrão).
- **Autorização** por papéis (*Role-Based Access Control*), na URL e no método.
- **Escopo de dados** por papel: o influenciador vê apenas as próprias propostas, e a marca vê todas.

> **Sobre o estado da sessão.** A autenticação é **baseada em sessão** (cookie `JSESSIONID`), e não em token. Após o login, o servidor guarda o contexto de segurança e o cliente reenvia o cookie nas próximas requisições.

---

## `ROADMAP`

```
SPRING SECURITY

[✓] 01. Autenticação
    ├── Filtro customizado (UsernamePasswordAuthenticationFilter) com login em JSON
    ├── UserDetails e UserDetailsService apoiados em JPA
    └── Senhas com BCrypt

[✓] 02. Autorização
    ├── SecurityFilterChain (regras por URL)
    ├── Segurança de método (@EnableMethodSecurity + @PreAuthorize)
    └── Papéis: INFLUENCER e BRAND

[✓] 03. Usuário autenticado no código
    └── @AuthenticationPrincipal

[✓] 04. Regras de visibilidade
    ├── Strategy Pattern (OwnStrategy / AllStrategy)
    └── Factory que seleciona a estratégia pelo escopo

[✓] 05. Arquitetura
    └── Contextos separados: auth/ e proposal/
```

---

## `ESTRUTURA`

```
03-spring-security/
├── src/main/java/dio/proposalmanagement/
│   ├── auth/                           # contexto de segurança
│   │   ├── domain/                     # UserRole
│   │   └── infrastructure/
│   │       ├── http/                   # Controller (endpoints de teste de acesso)
│   │       ├── persistence/            # User (entity), UserRepository
│   │       └── security/               # SecurityConfig, filtro, JpaUserDetailsService
│   ├── proposal/                       # contexto de negócio
│   │   ├── application/
│   │   │   ├── CreateProposalUseCase.java
│   │   │   ├── ListProposalsUseCase.java
│   │   │   ├── input/ output/
│   │   │   └── list/                   # Strategy, OwnStrategy, AllStrategy, Factory, AccessScope
│   │   ├── domain/                     # Proposal, Owner, OwnerId, ProposalRepository
│   │   └── infrastructure/
│   │       ├── http/                   # ProposalController + request/response
│   │       └── persistence/            # ProposalEntity, repositórios JPA
│   └── ProposalManagementApplication.java
├── compose.yml                         # MySQL
├── build.gradle
└── README.md
```

---

## `ARQUITETURA`

O contexto `auth` cuida de **quem é o usuário**. O contexto `proposal` cuida das **regras de proposta**. O `proposal` não conhece a entidade `User`: ele recebe apenas um `Owner` (id + nome) montado pelo controller.

```
   Requisição HTTP
         │
         ▼
┌──────────────────────────────────────────────┐
│ Cadeia de filtros do Spring Security         │
│  ├─ RestUsernamePasswordAuthenticationFilter │  ← login em /api/auth/login
│  ├─ ... (sessão, contexto de segurança)      │
│  └─ AuthorizationFilter (regras por URL)     │
└──────────────────────┬───────────────────────┘
                       ▼
              Controller + @PreAuthorize   ← regra por papel
                       ▼
                   Use Case
                       ▼
        Strategy (OWN | ALL) → Repositório
```

---

## `CONCEITOS E CÓDIGO`

### 1. Cadeia de filtros (`SecurityFilterChain`)

O Spring Security funciona como uma **sequência de filtros servlet** executados antes do controller. Cada filtro tem uma função (autenticar, carregar sessão, autorizar). Configurar a segurança é montar essa cadeia.

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http,
            RestUsernamePasswordAuthenticationFilter restFilter) throws Exception {
        http
            .csrf(AbstractHttpConfigurer::disable)
            .securityContext(context -> context.requireExplicitSave(false))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/login").permitAll()
                .anyRequest().authenticated())
            .addFilterAt(restFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

- `permitAll()` libera só o login; `anyRequest().authenticated()` exige autenticação em todo o resto (**negar por padrão**).
- `addFilterAt` **substitui** o filtro de formulário padrão pelo filtro JSON.
- `requireExplicitSave(false)` faz o contexto de segurança ser salvo na sessão automaticamente após o login.

> ⚠️ **CSRF desabilitado.** Isso é seguro em APIs *stateless* com token no cabeçalho. Como esta aplicação usa **cookie de sessão**, o navegador envia o cookie automaticamente e um site malicioso poderia disparar requisições em nome do usuário. Em produção, mantenha o CSRF ativo (com token) ou use cookie `SameSite=Strict`.

### 2. Filtro de autenticação em JSON

O `UsernamePasswordAuthenticationFilter` padrão lê o login de um **formulário**. Para uma API, estende-se o filtro e sobrescreve `attemptAuthentication` para ler o corpo JSON.

```java
@Component
public class RestUsernamePasswordAuthenticationFilter extends UsernamePasswordAuthenticationFilter {

    public RestUsernamePasswordAuthenticationFilter(AuthenticationConfiguration authConfig,
                                                    ObjectMapper objectMapper) {
        super(authConfig.getAuthenticationManager());
        this.objectMapper = objectMapper;
        setFilterProcessesUrl("/api/auth/login");
        setAuthenticationSuccessHandler((request, response, authentication) ->
                response.setStatus(HttpServletResponse.SC_OK));
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request,
                                                HttpServletResponse response) {
        var login = objectMapper.readValue(request.getInputStream(), LoginRequest.class);
        var token = UsernamePasswordAuthenticationToken.unauthenticated(login.username(), login.password());
        return getAuthenticationManager().authenticate(token);
    }

    public record LoginRequest(String username, String password) {}
}
```

O filtro **não** valida a senha. Ele monta um token "não autenticado" e entrega ao `AuthenticationManager`, que delega para um `DaoAuthenticationProvider`. Esse provider chama o `UserDetailsService` para buscar o usuário e o `PasswordEncoder` para comparar a senha.

```
filtro ─▶ AuthenticationManager ─▶ DaoAuthenticationProvider ─▶ UserDetailsService (busca o usuário)
                                                             └─▶ PasswordEncoder   (confere o hash)
```

### 3. `UserDetails` e `UserDetailsService`

A entidade `User` implementa `UserDetails`, o contrato que o Spring Security entende. O papel vira uma `GrantedAuthority`.

```java
@Entity
public class User implements UserDetails {
    @Id @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;

    @Column(unique = true, nullable = false)
    private String username;

    @Column(nullable = false)
    private String password;            // guarda o HASH, nunca a senha

    @Enumerated(EnumType.STRING)
    private UserRole role;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority(role.name()));
    }
}
```

```java
@Service
public class JpaUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException(username));
    }
}
```

### 4. Hash de senha com BCrypt

Senhas nunca são guardadas em texto puro. O **BCrypt** aplica um *salt* aleatório e é deliberadamente lento, o que torna ataques de força bruta caros.

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

Para facilitar o estudo, um `CommandLineRunner` cria **três usuários de teste** na primeira execução (tabela vazia):

| Usuário        | Papel            | Senha      |
| -------------- | ---------------- | ---------- |
| `fitness_vibe` | `ROLE_INFLUENCER`| `password` |
| `tech_guru`    | `ROLE_INFLUENCER`| `password` |
| `logistics`    | `ROLE_BRAND`     | `password` |

> Esses usuários existem **só para desenvolvimento**. Um seed com senha fixa não pode ir para produção.

### 5. Autorização: URL × método

Existem dois níveis, e eles se complementam:

| Nível       | Onde                          | Pergunta que responde                  |
| ----------- | ----------------------------- | -------------------------------------- |
| Por URL     | `authorizeHttpRequests`       | O usuário está autenticado?            |
| Por método  | `@PreAuthorize` (método)      | O usuário tem **este papel**?          |

```java
@GetMapping("/influencer")
@PreAuthorize("hasRole('INFLUENCER')")
public String influenceEndpoint() { return "Hello INFLUENCER"; }

@PostMapping
@PreAuthorize("hasRole('INFLUENCER')")          // só influenciador cria proposta
public ProposalResponse createProposal(...) { /* ... */ }

@GetMapping
@PreAuthorize("hasAnyRole('INFLUENCER','BRAND')")
public List<ProposalResponse> findAllProposals(...) { /* ... */ }
```

> `hasRole('INFLUENCER')` procura a authority `ROLE_INFLUENCER`. O Spring adiciona o prefixo `ROLE_` automaticamente, por isso os valores do enum já são `ROLE_INFLUENCER` e `ROLE_BRAND`.

Requisição de usuário autenticado **sem o papel exigido** recebe `403 Forbidden`. Para requisições **sem autenticação**, o status depende do `AuthenticationEntryPoint` configurado; sem formulário nem Basic habilitados, o padrão do Spring Security também responde `403`. Para devolver `401`, é preciso configurar um entry point.

### 6. Usuário logado dentro do controller

`@AuthenticationPrincipal` injeta o objeto `User` autenticado, então o controller sabe **quem** está chamando sem consultar o banco de novo.

```java
@PostMapping
@PreAuthorize("hasRole('INFLUENCER')")
public ProposalResponse createProposal(@RequestBody CreateProposalRequest request,
                                       @AuthenticationPrincipal User user) {
    var owner = new Owner(new OwnerId(user.getId()), user.getUsername());
    var output = createProposalUseCase.execute(request.toInput(), owner);
    return ProposalResponse.from(output);
}
```

### 7. Strategy Pattern para o escopo dos dados

A pergunta "quais propostas esse usuário pode ver?" não é uma regra de URL nem de método: é uma regra de **dados**. Em vez de `if/else` espalhado, cada escopo é uma **estratégia** com a mesma interface.

```java
public interface Strategy {
    List<Proposal> getProposals(OwnerId ownerId);
    AccessScope getScope();
}

@Service
public class OwnStrategy implements Strategy {          // influenciador: só as suas
    public List<Proposal> getProposals(OwnerId ownerId) {
        return proposalRepository.findAllByOwnerId(ownerId);
    }
    public AccessScope getScope() { return AccessScope.OWN; }
}

@Service
public class AllStrategy implements Strategy {          // marca: todas
    public List<Proposal> getProposals(OwnerId ownerId) {
        return proposalRepository.findAll();
    }
    public AccessScope getScope() { return AccessScope.ALL; }
}
```

A `Factory` recebe **todas** as implementações de `Strategy` por injeção (`List<Strategy>`) e monta um mapa por escopo. Para criar um novo escopo, basta criar uma nova classe `@Service`: nenhuma outra classe muda (**princípio aberto/fechado**).

```java
@Component
public class Factory {
    private final Map<AccessScope, Strategy> strategies;

    public Factory(List<Strategy> strategies) {
        this.strategies = strategies.stream()
                .collect(Collectors.toMap(Strategy::getScope, Function.identity()));
    }

    public Strategy getStrategy(AccessScope scope) { return strategies.get(scope); }
}
```

O controller traduz o papel em escopo com um `switch` exaustivo; se um novo papel for adicionado ao enum, o compilador exige tratá-lo.

```java
private static AccessScope getAccessScope(UserRole role) {
    return switch (role) {
        case ROLE_INFLUENCER -> AccessScope.OWN;
        case ROLE_BRAND      -> AccessScope.ALL;
    };
}
```

### 8. Separação entre domínio e persistência

`Proposal` (domínio) é uma classe sem anotações JPA. A `ProposalEntity` faz a conversão nos dois sentidos, mantendo o contexto `proposal` independente do banco.

```java
public static ProposalEntity from(Proposal proposal) { /* domínio → entidade */ }
public Proposal toDomain()                            { /* entidade → domínio */ }
```

---

## `ENDPOINTS`

| Método | Endpoint           | Acesso                    | Descrição                                          |
| ------ | ------------------ | ------------------------- | -------------------------------------------------- |
| `POST` | `/api/auth/login`  | Público                   | Login com JSON; cria a sessão (`200`)              |
| `GET`  | `/`                | Autenticado               | Retorna o id do usuário logado                     |
| `GET`  | `/influencer`      | `INFLUENCER`              | Teste de acesso por papel                          |
| `GET`  | `/brand`           | `BRAND`                   | Teste de acesso por papel                          |
| `POST` | `/proposals`       | `INFLUENCER`              | Cria proposta em nome do usuário logado            |
| `GET`  | `/proposals`       | `INFLUENCER` / `BRAND`    | Influenciador: as suas · Marca: todas              |

---

## `COMO EXECUTAR`

**Pré-requisitos:** JDK 25 (toolchain do `build.gradle`) e Docker.

```bash
# 1. Subir o MySQL (o auto-start do Docker Compose está desabilitado neste módulo)
docker compose up -d

# 2. Executar a aplicação
./gradlew bootRun            # Linux/macOS
gradlew.bat bootRun          # Windows
```

Fluxo completo de teste com `curl`, guardando o cookie de sessão:

```bash
# Login (grava o cookie JSESSIONID em cookies.txt)
curl -i -c cookies.txt -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "fitness_vibe", "password": "password"}'

# Criar uma proposta como influenciador
curl -b cookies.txt -X POST http://localhost:8080/proposals \
  -H "Content-Type: application/json" \
  -d '{"title": "Parceria de verão", "description": "Campanha de 30 dias"}'

# Listar (influenciador vê só as suas)
curl -b cookies.txt http://localhost:8080/proposals

# Tentar acessar rota de outro papel → 403
curl -b cookies.txt http://localhost:8080/brand
```

> O esquema do banco é recriado a cada execução (`ddl-auto=create`), então os dados não persistem entre reinícios. Isso é conveniente para estudo, mas não para produção.

**Testes:** o módulo possui hoje o teste de contexto (`ProposalManagementApplicationTests`). A cobertura dos fluxos de autorização (`401`/`403`, escopo `OWN` × `ALL`) ainda não foi escrita como teste automatizado.

---

## `TECNOLOGIAS`

```
[SYSTEM STATUS]

Language     : JAVA 25
Framework    : SPRING BOOT 4.0.5
Security     : SPRING SECURITY (SESSION + CUSTOM FILTER + METHOD SECURITY)
Hashing      : BCRYPT
Data         : SPRING DATA JPA / MYSQL 9.6
Patterns     : STRATEGY + FACTORY / ADAPTER
Build        : GRADLE
Status       : COMPLETED
```
