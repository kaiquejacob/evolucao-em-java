# 🍃 Spring Web REST API — Task Manager

![Java](https://img.shields.io/badge/Java-25-0B0F14?style=for-the-badge&logo=openjdk&logoColor=2F81F7)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.0.5-0B0F14?style=for-the-badge&logo=springboot&logoColor=6DB33F)
![Gradle](https://img.shields.io/badge/Gradle-Build-0B0F14?style=for-the-badge&logo=gradle&logoColor=02303A)
![Status](https://img.shields.io/badge/Status-Completed-0B0F14?style=for-the-badge&logo=github&logoColor=2F81F7)

---

## `SOBRE`

API REST de gerenciamento de tarefas construída com **Clean Architecture**. O foco do módulo não é o CRUD em si, mas **onde cada responsabilidade mora**: o domínio não conhece HTTP, os casos de uso não conhecem o framework web e a infraestrutura pode ser trocada sem tocar nas regras de negócio.

O armazenamento é **em memória** (`HashMap`). Isso é proposital: mantém o módulo concentrado em arquitetura e contratos, e a troca por um banco real só exigiria uma nova implementação de `TaskRepository`.

---

## `ROADMAP`

```
SPRING WEB REST API

[✓] 01. Arquitetura
    ├── Clean Architecture (domain / application / infrastructure)
    ├── Regra da dependência e inversão de dependência
    └── Value Objects com records (TaskId)

[✓] 02. API REST
    ├── Controller com injeção via construtor
    ├── DTOs de HTTP (Request/Response) separados dos DTOs de aplicação (Input/Output)
    └── Status HTTP com @ResponseStatus

[✓] 03. Qualidade
    ├── Bean Validation (@Valid, @NotBlank, @Size)
    └── Tratamento global de erros (@RestControllerAdvice)

[✓] 04. Testes e documentação
    ├── Teste unitário com Mockito
    ├── Teste de contrato do repositório (classe abstrata)
    └── Documentação gerada pelos testes (Spring REST Docs + Asciidoctor)
```

---

## `ESTRUTURA`

```
01-spring-web/
├── src/
│   ├── main/java/dio/taskmanager/
│   │   ├── domain/                 # regras de negócio e contratos
│   │   │   ├── Task.java
│   │   │   ├── TaskId.java
│   │   │   ├── TaskStatus.java
│   │   │   ├── TaskNotFoundException.java
│   │   │   └── TaskRepository.java
│   │   ├── application/            # casos de uso
│   │   │   ├── CreateTaskUseCase.java
│   │   │   ├── GetTasksUseCase.java
│   │   │   ├── GetTaskByIdUseCase.java
│   │   │   ├── UpdateTaskUseCase.java
│   │   │   ├── DeleteTaskUseCase.java
│   │   │   ├── input/              # CreateTaskInput, UpdateTaskInput
│   │   │   └── output/             # TaskOutput
│   │   ├── infrastructure/
│   │   │   ├── http/               # TaskController, GlobalExceptionHandler
│   │   │   │   ├── request/        # CreateTaskRequest, UpdateTaskRequest
│   │   │   │   └── response/       # TaskResponse
│   │   │   └── repository/         # InMemoryTaskRepository
│   │   └── TaskmanagerApplication.java
│   ├── docs/asciidoc/index.adoc    # documentação da API (Asciidoctor)
│   └── test/java/dio/taskmanager/
├── build.gradle
└── README.md
```

---

## `ARQUITETURA`

A regra central da Clean Architecture é a **regra da dependência**: o código-fonte só pode apontar para dentro. Camadas internas nunca importam camadas externas.

```
┌─────────────────────────────────────────────┐
│  infrastructure  (Spring MVC, repositórios) │
│   ┌─────────────────────────────────────┐   │
│   │  application  (casos de uso)        │   │
│   │   ┌─────────────────────────────┐   │   │
│   │   │  domain  (Task, regras)     │   │   │
│   │   └─────────────────────────────┘   │   │
│   └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
        dependências apontam para dentro
```

**Por que isso importa?** Se a regra de negócio importar `@RestController` ou `JpaRepository`, qualquer troca de tecnologia (outro framework web, outro banco) obriga a mexer nas regras. Aqui, o `domain` só depende do JDK e de um utilitário do Spring (`Assert`).

---

## `CONCEITOS E CÓDIGO`

### 1. Inversão de dependência

O `domain` declara **o que precisa** (uma interface), e a `infrastructure` fornece **como** (a implementação). O caso de uso depende da abstração, nunca da classe concreta.

```java
// domain/TaskRepository.java — contrato, sem tecnologia
public interface TaskRepository {
    Task save(Task task);
    List<Task> findAll();
    Optional<Task> findById(TaskId id);
    void delete(TaskId id);
}
```

```java
// infrastructure/repository/InMemoryTaskRepository.java — detalhe de implementação
@Repository
public class InMemoryTaskRepository implements TaskRepository {
    private final Map<TaskId, Task> storage = new HashMap<>();

    @Override
    public Task save(Task task) {
        storage.put(task.getId(), task);
        return task;
    }
    // ...
}
```

> O `HashMap` não é thread-safe. Para estudo é aceitável; com requisições concorrentes reais seria necessário `ConcurrentHashMap` ou um banco.

### 2. Domínio com comportamento e Value Objects

A entidade `Task` **protege as próprias regras**: não existe tarefa sem título, e toda tarefa nasce como `PENDING`. O `TaskId` é um **record** que valida no construtor compacto, então um `TaskId` inválido não consegue existir.

```java
public class Task {
    private TaskId id;
    private String title;
    private Optional<String> description;
    private TaskStatus status;

    public Task(String title, Optional<String> description) {
        Assert.notNull(title, "Title must not be null");

        this.id = new TaskId();
        this.title = title;
        this.description = description;
        this.status = TaskStatus.PENDING;
    }

    public void update(Optional<String> title, Optional<String> description, Optional<TaskStatus> status) {
        title.ifPresent(this::setTitle);
        description.ifPresent(d -> this.setDescription(Optional.of(d)));
        status.ifPresent(this::setStatus);
    }
}
```

```java
public record TaskId(UUID id) {
    public TaskId {                                   // construtor compacto
        Assert.notNull(id, "id must not be null");
    }

    public TaskId() {
        this(UUID.randomUUID());
    }
}
```

### 3. Casos de uso

Cada caso de uso é uma classe com **uma única ação** (`execute`), o que aplica o princípio da responsabilidade única. Eles orquestram o domínio e devolvem um DTO de saída, nunca a entidade.

```java
@Service
public class UpdateTaskUseCase {
    private final TaskRepository repository;

    public UpdateTaskUseCase(TaskRepository repository) {
        this.repository = repository;
    }

    public TaskOutput execute(TaskId id, UpdateTaskInput input) {
        var task = repository.findById(id).orElseThrow(() -> new TaskNotFoundException(id));
        task.update(input.title(), input.description(), input.status());
        var updated = repository.save(task);
        return TaskOutput.from(updated);
    }
}
```

### 4. Dois pares de DTOs: Request/Response e Input/Output

Parece redundante, mas cada par protege uma fronteira diferente:

| DTO                        | Camada           | Conhece                          |
| -------------------------- | ---------------- | -------------------------------- |
| `CreateTaskRequest`        | infrastructure   | JSON, anotações de validação     |
| `CreateTaskInput`          | application      | só dados do caso de uso          |
| `TaskOutput`               | application      | só dados do resultado            |
| `TaskResponse`             | infrastructure   | formato do JSON de resposta      |

O fluxo de uma criação atravessa todos eles:

```
CreateTaskRequest ──toInput()──▶ CreateTaskInput ──▶ UseCase ──▶ TaskOutput ──from()──▶ TaskResponse
     (HTTP)                       (application)                  (application)             (HTTP)
```

Assim, mudar o formato do JSON (renomear um campo, por exemplo) não afeta nenhum caso de uso.

### 5. Controller e injeção via construtor

O controller é fino: converte HTTP em chamada de caso de uso e devolve a resposta. A injeção é feita **pelo construtor**, o que deixa as dependências explícitas e a classe instanciável nos testes sem o Spring.

```java
@RestController
@RequestMapping("/tasks")
public class TaskController {
    private final CreateTaskUseCase createTaskUseCase;
    // ... demais casos de uso

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    TaskResponse create(@RequestBody @Valid CreateTaskRequest request) {
        var input = request.toInput();
        var output = createTaskUseCase.execute(input);
        return TaskResponse.from(output);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    void delete(@PathVariable UUID id) {
        deleteTaskUseCase.execute(new TaskId(id));
    }
}
```

### 6. Bean Validation

A validação acontece **na borda** (no DTO de Request), antes de qualquer caso de uso ser chamado. O `@Valid` no parâmetro dispara as regras. O campo opcional também pode ser validado, com a restrição aplicada ao tipo dentro do `Optional`.

```java
public record CreateTaskRequest(
        @NotBlank
        @Size(min = 3, max = 100)
        String title,
        Optional<@Size(max = 500) String> description) {

    public CreateTaskInput toInput() {
        return new CreateTaskInput(title, description);
    }
}
```

### 7. Tratamento global de exceções

Exceções de domínio (`TaskNotFoundException`) são traduzidas para HTTP **em um único lugar**, a camada de infraestrutura. O domínio lança o erro sem saber que existe um status `404`.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(TaskNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public String handleTaskNotFoundException(TaskNotFoundException ex) {
        return ex.getMessage();
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            errors.put(((FieldError) error).getField(), error.getDefaultMessage());
        });
        return errors;
    }
}
```

Exemplo de resposta para um título inválido (`400`):

```json
{ "title": "size must be between 3 and 100" }
```

### 8. Testes

**Teste unitário do caso de uso**, com o repositório substituído por um mock (Mockito):

```java
@ExtendWith(MockitoExtension.class)
class CreateTaskUseCaseTest {
    @Mock TaskRepository repository;
    @InjectMocks CreateTaskUseCase useCase;

    @Test
    void should_create_task_successfully() {
        var input = new CreateTaskInput("Estudar Java", Optional.of("Finalizar o módulo de Records"));
        when(repository.save(any(Task.class)))
                .thenAnswer(invocation -> invocation.getArgument(0));

        TaskOutput output = useCase.execute(input);

        assertNotNull(output.id());
        assertEquals("Estudar Java", output.title());
        verify(repository, times(1)).save(any(Task.class));
    }
}
```

**Teste de contrato do repositório.** A classe abstrata `TaskRepositoryTest` define o comportamento que **qualquer** implementação de `TaskRepository` deve cumprir. Cada implementação só precisa estender a classe e fornecer a instância. Quando existir uma implementação com banco, ela reaproveita os mesmos testes.

```java
public abstract class TaskRepositoryTest {
    TaskRepository repository;

    protected abstract TaskRepository createRepository();

    @BeforeEach
    public void setUp() { this.repository = createRepository(); }

    @Test
    void should_save_and_retrieve_task_by_id() { /* ... */ }
}

class InMemoryTaskRepositoryTest extends TaskRepositoryTest {
    @Override
    protected TaskRepository createRepository() {
        return new InMemoryTaskRepository();
    }
}
```

**Documentação gerada pelos testes.** O `TaskControllerTest` usa **MockMvc + Spring REST Docs**: ao executar as requisições, ele gera os snippets que o Asciidoctor monta em `index.adoc`. Se a API mudar e o teste não for atualizado, a build quebra, então a documentação não consegue ficar desatualizada.

```java
this.mockMvc.perform(post("/tasks")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(taskRequest)))
        .andExpect(status().isCreated())
        .andDo(document("create-task",
                requestFields(
                        fieldWithPath("title").description("Título da tarefa"),
                        fieldWithPath("description").description("Descrição detalhada").optional()),
                responseFields(
                        fieldWithPath("id").description("Identificador único da tarefa"),
                        fieldWithPath("title").description("Título da tarefa"),
                        fieldWithPath("description").description("Descrição detalhada").optional(),
                        fieldWithPath("status").description("Status da tarefa"))));
```

---

## `ENDPOINTS`

| Método   | Endpoint       | Descrição                                   | Sucesso | Erros        |
| -------- | -------------- | ------------------------------------------- | ------- | ------------ |
| `POST`   | `/tasks`       | Cria uma tarefa (nasce como `PENDING`)      | `201`   | `400`        |
| `GET`    | `/tasks`       | Lista todas as tarefas                      | `200`   | —            |
| `GET`    | `/tasks/{id}`  | Busca uma tarefa por UUID                   | `200`   | `404`        |
| `PATCH`  | `/tasks/{id}`  | Atualiza título, descrição ou status        | `200`   | `404`        |
| `DELETE` | `/tasks/{id}`  | Remove uma tarefa                           | `204`   | —            |

Status possíveis: `PENDING`, `IN_PROGRESS`, `COMPLETED`.

---

## `COMO EXECUTAR`

**Pré-requisito:** o `build.gradle` define a toolchain **Java 25**. O Gradle a localiza (ou baixa) automaticamente quando há suporte configurado.

```bash
# Executar a aplicação
./gradlew bootRun            # Linux/macOS
gradlew.bat bootRun          # Windows

# Rodar os testes
./gradlew test

# Gerar a documentação da API (executa os testes antes)
./gradlew asciidoctor        # saída em build/docs/asciidoc/
```

Exemplo de uso:

```bash
curl -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Estudar Spring", "description": "Clean Architecture"}'
```

---

## `TECNOLOGIAS`

```
[SYSTEM STATUS]

Language     : JAVA 25
Framework    : SPRING BOOT 4.0.5 (WEB, VALIDATION)
Architecture : CLEAN ARCHITECTURE
Storage      : IN-MEMORY
Docs         : SPRING REST DOCS + ASCIIDOCTOR
Build        : GRADLE
Tests        : JUNIT 5 + MOCKITO + MOCKMVC
Status       : COMPLETED
```
