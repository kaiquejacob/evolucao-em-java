# Guia de estudo — Building with the Claude API

> Material de estudo teórico baseado nas transcrições do curso **Building with the Claude API**.  
> Objetivo: compreender os conceitos de LLMs, Claude API, prompts, tool use, RAG, embeddings, busca híbrida, MCP, agentes e workflows.

> **Nota sobre os exemplos:** os exemplos de código do curso foram originalmente escritos em Python com o SDK da Anthropic. Neste guia, eles foram **adaptados para Java**, usando Java padrão e, quando adequado, Spring Boot/Spring AI.  
>
> **Diferenciação importante:**
>
> - **Código executável em Java:** exemplos de lógica Java que não dependem de uma integração específica com provedor de IA, como chunking, cálculo de similaridade e validação de JSON.
> - **Exemplo conceitual em Java:** exemplos que representam a intenção da chamada à API, mas dependem de configuração, versão do Spring AI ou recursos específicos do provedor. Neles, `chatClient` representa um bean Spring configurado/injetado; não é uma variável global.
> - Nomes exatos de classes e métodos do Spring AI podem variar conforme a versão. Confira sempre a documentação da versão utilizada no projeto.

**Nota sobre fontes e limitações:** este guia usa como referência principal as transcrições dos módulos disponíveis no projeto. As seções **Multi-Index RAG pipeline**, **Text Editor Tool**, **Web Search Tool**, **Code Execution** e **Files API** aparecem no índice do curso, mas as transcrições disponíveis não trazem seu detalhamento completo; onde isso ocorre, a limitação está indicada.

---

## 1. Fundamentos da API

### 1.1 O que é uma API de LLM

Uma API de modelo de linguagem (LLM) é uma interface HTTP pela qual sua aplicação envia texto — e, em alguns casos, imagens, PDFs ou definições de ferramentas — e recebe texto gerado pelo modelo.

O ponto central é: **o modelo não é um banco de dados nem um sistema com estado**. Ele recebe um contexto, processa esse contexto e gera uma continuação provável. Tudo que ele “sabe” sobre a conversa precisa estar presente na requisição.

**Problema que resolve:** permite incorporar capacidade de linguagem natural, raciocínio, resumo, classificação, geração de conteúdo e interação conversacional em aplicações, sem treinar um modelo próprio.

### 1.2 Ciclo de vida de uma requisição

O curso descreve cinco fases:

1. **Solicitação ao servidor:** o frontend envia a mensagem do usuário ao backend.
2. **Solicitação à API Anthropic:** o backend chama a API usando uma chave secreta.
3. **Processamento pelo modelo:** tokenização, embeddings, contextualização e geração.
4. **Resposta ao servidor:** a API devolve o texto, metadados de uso e motivo de parada.
5. **Resposta ao cliente:** o backend encaminha o resultado à interface.

A chamada à API deve acontecer no **servidor**, não no navegador ou aplicativo móvel. A API key é um segredo; se estiver no cliente, qualquer pessoa pode extraí-la e consumir a API em seu nome.

### 1.3 Como o modelo processa texto

O processamento interno tem quatro etapas conceituais:

| Etapa | O que acontece | Para que serve |
|---|---|---|
| Tokenização | O texto é dividido em tokens | Transforma linguagem natural em unidades processáveis |
| Embeddings | Cada token recebe uma representação numérica | Captura relações semânticas entre palavras |
| Contextualização | As representações são ajustadas conforme o contexto | Resolve ambiguidade de palavras com múltiplos sentidos |
| Geração | O modelo escolhe o próximo token repetidamente | Produz a resposta final |

**Exemplo:** na frase “O que é computação quântica?”, a palavra “quântico” pode remeter a física, mecânica quântica, algo muito pequeno ou computação. A contextualização usa as palavras vizinhas para escolher o sentido mais provável.

### 1.4 Quando a geração termina

A geração pode parar por três motivos:

- **Limite de tokens:** atingiu `max_tokens`.
- **Fim natural:** gerou um token de fim de sequência.
- **Stop sequence:** encontrou uma sequência de parada definida por você.

Isso importa porque `max_tokens` não é uma meta de tamanho: é um limite de segurança. Se você define 1000, o modelo para em 1000 tokens mesmo que a resposta esteja incompleta.

A resposta da API inclui:

- O texto gerado.
- Contagem de tokens de entrada e saída.
- O motivo de parada (`stop_reason`).

### 1.5 Requisição básica

No curso, a requisição é feita em Python com `client.messages.create()`. Em Java, a ideia é a mesma: enviar modelo, limite de tokens e lista de mensagens.

**Exemplo conceitual em Java — estrutura da requisição:**

```java
record ChatMessage(String role, String content) {}

var messages = List.of(
    new ChatMessage("user", "What is quantum computing? Answer in one sentence")
);

var requestBody = Map.of(
    "model", "claude-sonnet-4-5",
    "max_tokens", 1000,
    "messages", messages
);
```

**Explicação:**

- `model`: identifica o modelo Claude.
- `max_tokens`: limite máximo de tokens gerados.
- `messages`: histórico da conversa.
- `role`: quem “falou” a mensagem — `user` ou `assistant`.
- `content`: conteúdo da mensagem.

**Com Spring AI**, quando houver um `ChatModel` configurado, a intenção equivalente é enviar um prompt e receber o texto gerado:

```java
// Exemplo conceitual em Spring AI.
// Confira os nomes exatos de métodos na versão do Spring AI do seu projeto.
String resposta = chatClient.prompt()
    .user("What is quantum computing? Answer in one sentence")
    .call()
    .content();
```

**Resultado esperado:** uma resposta curta explicando computação quântica. O modelo não “lembra” dessa interação na próxima chamada, a menos que você envie o histórico novamente.

### 1.6 Chaves de API

A chave é criada no Anthropic Console, exibida apenas uma vez e deve ser guardada em variável de ambiente, normalmente em `.env`.

Em um projeto Spring Boot, o padrão equivalente é usar propriedades de configuração e variáveis de ambiente:

```properties
# application.properties
anthropic.api-key=${ANTHROPIC_API_KEY}
```

Boas práticas do curso:

- Nunca colocar a chave no código-fonte.
- Nunca expor a chave no frontend.
- Não versionar arquivos com credenciais.
- Se a chave for exposta, revogá-la e criar outra.

---

## 2. Conversas e contexto

### 2.1 O problema da ausência de memória

A API Claude é **stateless**: cada requisição é independente. O modelo não guarda automaticamente mensagens anteriores.

**Problema:** se você pergunta “O que é computação quântica?” e depois diz “Escreva outra frase”, o modelo não sabe a qual assunto a segunda mensagem se refere.

**Solução:** sua aplicação mantém o histórico e o envia integralmente a cada chamada.

### 2.2 Como funciona uma conversa multi-turn

Fluxo correto:

1. Usuário envia mensagem.
2. Modelo responde.
3. Aplicação adiciona a resposta como mensagem `assistant`.
4. Aplicação adiciona a nova pergunta como mensagem `user`.
5. Aplicação envia toda a lista novamente.

**Exemplo conceitual em Java:**

```java
record ChatMessage(String role, String content) {}

List<ChatMessage> messages = new ArrayList<>();

messages.add(new ChatMessage("user", "Define quantum computing in one sentence"));

String answer = callModel(messages);

messages.add(new ChatMessage("assistant", answer));

messages.add(new ChatMessage("user", "Write another sentence"));

String finalAnswer = callModel(messages);
```

**Explicação:** na segunda chamada, o modelo recebe a pergunta original, a resposta anterior e o pedido de continuação. Por isso, “escreva outra frase” passa a significar “continue falando sobre computação quântica”.

**Ponto de atenção:** enviar todo o histórico aumenta o número de tokens de entrada. Conversas longas podem ficar caras, lentas e eventualmente exceder limites de contexto.

### 2.3 System prompts

Um **system prompt** é uma instrução de alto nível que define como o modelo deve se comportar durante toda a conversa.

Ele responde a perguntas como:

- Qual é o papel do assistente?
- Qual tom deve usar?
- Quais regras deve seguir?
- O que deve evitar?

**Exemplo conceitual em Java:**

```java
String systemPrompt = """
    You are a patient math tutor.
    Do not directly answer a student's questions.
    Guide them to a solution step by step.
    """;

// A intenção é enviar o system prompt separadamente das mensagens do usuário.
// Em Spring AI, isso normalmente é feito no prompt, antes das mensagens do usuário.
String resposta = chatClient.prompt()
    .system(systemPrompt)
    .user("How do I solve 5x + 2 = 3 for x?")
    .call()
    .content();
```

**Diferença entre system prompt e mensagem do usuário:**

| Aspecto | System prompt | Mensagem do usuário |
|---|---|---|
| Finalidade | Definir comportamento geral | Solicitar uma tarefa específica |
| Escopo | Toda a conversa | Aquela interação |
| Exemplo | “Aja como tutor paciente” | “Como resolvo 5x + 2 = 3?” |

No exemplo do curso, sem system prompt o modelo entrega a solução completa. Com o system prompt, ele faz perguntas orientadoras e conduz o aluno passo a passo.

**Limitação importante:** um system prompt orienta o comportamento, mas não é uma garantia absoluta. Instruções podem ser ignoradas, especialmente em prompts longos, conflitantes ou mal estruturados.

---

## 3. Temperatura e streaming

### 3.1 Temperatura

A **temperatura** controla o grau de aleatoriedade na escolha do próximo token.

- Temperatura próxima de **0**: o modelo tende a escolher o token mais provável.
- Temperatura próxima de **1**: o modelo distribui mais probabilidade entre opções menos óbvias.

| Faixa | Comportamento | Usos típicos |
|---|---|---|
| 0,0–0,3 | Mais determinístico | Fatos, código, extração de dados, moderação |
| 0,4–0,7 | Equilíbrio | Resumos, conteúdo educacional, resolução de problemas |
| 0,8–1,0 | Mais variado | Brainstorming, escrita criativa, marketing, humor |

**O que a temperatura não faz:** ela não garante respostas diferentes nem corrige erros factuais. Ela altera probabilidades de escolha, não a veracidade do conteúdo.

**Exemplo em Spring AI — sujeito à versão utilizada:**

```java
double temperatura = 0.2; // mais previsível

// ChatClient e ChatModel devem estar configurados/injetados na aplicação.
// A forma exata de construir options pode variar conforme a versão do Spring AI
// e o provedor configurado; confira a documentação da versão do seu projeto.
var resposta = chatClient.prompt()
    .user("Gere uma ideia de filme")
    .options(ChatOptions.builder().temperature(temperatura).build())
    .call()
    .content();
```

**Explicação:** `temperature=0.2` tende a produzir respostas mais parecidas entre execuções. Um valor como `1.0` tende a gerar temas, personagens e enredos mais variados.

### 3.2 Streaming

Sem streaming, o cliente espera a resposta completa. O curso cita que isso pode levar de 10 a 30 segundos, o que prejudica a experiência em chats.

Com **streaming**, a API envia eventos incrementais enquanto o texto é gerado.

Eventos principais:

| Evento | Significado |
|---|---|
| `message_start` | Uma nova mensagem começou |
| `content_block_start` | Um bloco de conteúdo começou |
| `content_block_delta` | Chegou um fragmento do conteúdo |
| `content_block_stop` | O bloco terminou |
| `message_delta` | Informações finais da mensagem |
| `message_stop` | A mensagem terminou |

**Exemplo conceitual em Java:**

```java
// A intenção é consumir a resposta em partes, em vez de esperar o texto completo.
chatClient.prompt()
    .user("Explique computação quântica em uma frase")
    .stream()
    .content()
    .doOnNext(fragmento -> System.out.print(fragmento))
    .blockLast();
```

**Como funciona:** cada fragmento recebido corresponde a uma parte do texto gerado. O backend pode repassar cada fragmento ao frontend, que exibe a resposta progressivamente.

**Resultado:** o usuário começa a ler antes de a resposta inteira estar pronta. A resposta final completa ainda pode ser acumulada para salvar no banco ou processar depois.

---

## 4. Saída estruturada

### 4.1 O problema

Ao pedir JSON, o modelo pode devolver:

- Explicações antes ou depois do JSON.
- Blocos de código Markdown.
- Comentários desnecessários.

Isso dificulta o parse automático.

### 4.2 Solução apresentada: prefill + stop sequences

O curso combina duas técnicas:

1. **Assistant message prefill:** você inicia artificialmente a resposta do assistente, por exemplo com ` ```json `.
2. **Stop sequence:** você define um marcador que interrompe a geração, por exemplo ` ``` `.

**Exemplo conceitual em Java:**

```java
List<ChatMessage> messages = new ArrayList<>();

messages.add(new ChatMessage(
    "user",
    "Generate a JSON rule for monitoring EC2 instances."
));

// Prefill: a resposta do assistente começa com o marcador JSON.
messages.add(new ChatMessage("assistant", "```json"));

var requestBody = Map.of(
    "model", "claude-sonnet-4-5",
    "max_tokens", 1000,
    "messages", messages,
    "stop_sequences", List.of("```")
);
```

**Como funciona:**

- O prefill induz o modelo a continuar exatamente de onde a resposta foi iniciada.
- A stop sequence corta a saída antes do fechamento do bloco Markdown.
- O texto restante tende a ser o JSON puro.

**Resultado esperado:** uma string mais fácil de converter para um objeto Java com uma biblioteca de serialização JSON, como Jackson.

**Atenção:** isso reduz a chance de texto extra, mas não é uma garantia absoluta. A aplicação ainda deve validar e tratar falhas de parse.

---

## 5. Avaliação de prompts

### 5.1 Por que avaliar

Escrever um prompt e testá-lo manualmente uma ou duas vezes é insuficiente. Usuários reais trazem entradas inesperadas, ambíguas, incompletas ou adversariais.

**Avaliação de prompt** é o processo de medir objetivamente a qualidade das respostas em um conjunto de casos de teste.

**Problema que resolve:** evita decidir se um prompt é bom com base em impressão subjetiva ou em poucos exemplos.

### 5.2 Fluxo de avaliação

O curso propõe cinco etapas:

1. **Escrever o prompt inicial.**
2. **Criar um dataset de entradas.**
3. **Executar cada entrada no modelo.**
4. **Avaliar cada resposta com um classificador.**
5. **Alterar o prompt e repetir.**

```text
Prompt v1 + casos de teste -> respostas -> avaliador -> pontuação média
Prompt v2 + mesmos casos    -> respostas -> avaliador -> nova pontuação
```

**Exemplo do curso:** um prompt básico atinge média 7,66. Ao acrescentar “Answer the question with ample detail”, a média sobe para 8,7. A melhoria é mensurável porque o mesmo dataset foi usado nas duas versões.

### 5.3 Dataset de avaliação

Um dataset contém entradas representativas do uso real.

Para um prompt que gera Python, JSON ou regex para tarefas AWS, cada caso pode conter:

```json
{
  "task": "Create a Python function to validate an AWS IAM username",
  "format": "python"
}
```

O campo `format` permite escolher o validador correto.

Em Java, esse dataset pode ser representado como:

```java
record TestCase(String task, String format) {}

List<TestCase> casos = List.of(
    new TestCase(
        "Create a Python function to validate an AWS IAM username",
        "python"
    )
);
```

Os casos podem ser criados manualmente ou gerados pelo próprio modelo. Para gerar JSON confiável, o curso usa novamente prefill + stop sequence.

**Boa prática:** durante o desenvolvimento, use poucos casos — por exemplo, 2 ou 3 — para iterar rápido. Aumente o dataset na validação final.

### 5.4 Tipos de avaliadores

| Avaliador | Como funciona | Melhor para | Limitação |
|---|---|---|---|
| Baseado em código | Regras programáticas e validação sintática | Formato, estrutura, sintaxe, presença de campos | Não julga qualidade semântica |
| Baseado em modelo | Outra chamada de LLM julga a resposta | Qualidade, completude, utilidade, segurança, aderência à tarefa | Pode ser inconsistente ou “caprichoso” |
| Humano | Pessoa revisa e pontua | Julgamento subjetivo, profundidade, relevância, tom | Demorado e difícil de escalar |

### 5.5 Avaliação baseada em código

Para código, o curso separa critérios:

- **Formato:** retornar apenas Python, JSON ou regex, sem comentários.
- **Sintaxe válida:** o código precisa ser parseável.
- **Aderência à tarefa:** a solução realmente resolve o pedido.

**Exemplo executável em Java — validação de JSON com Jackson:**

```java
import com.fasterxml.jackson.databind.ObjectMapper;

class JsonValidator {

    private final ObjectMapper objectMapper = new ObjectMapper();

    int validate(String text) {
        try {
            objectMapper.readTree(text.strip());
            return 10;
        } catch (Exception e) {
            return 0;
        }
    }
}
```

**Como funciona:** `objectMapper.readTree()` tenta interpretar o texto como JSON. Se conseguir, o método retorna 10; se lançar exceção, retorna 0.

**Resultado:** uma nota objetiva para formato e sintaxe. Ela não diz se a solução está correta em relação ao problema.

### 5.6 Avaliação baseada em modelo

Um segundo LLM recebe a tarefa e a resposta gerada, e retorna uma avaliação estruturada:

```text
Forneça:
- strengths
- weaknesses
- reasoning
- score entre 1 e 10
```

**Exemplo conceitual em Java:**

```java
record ModelGrade(
    List<String> strengths,
    List<String> weaknesses,
    String reasoning,
    int score
) {}

String promptAvaliador = """
    You are an expert code reviewer.
    Evaluate this AI-generated solution.

    Task: %s
    Solution: %s

    Return JSON with:
    - strengths
    - weaknesses
    - reasoning
    - score between 1 and 10
    """.formatted(tarefa, solucao);
```

**Por que pedir raciocínio:** o curso observa que, sem pedir pontos fortes, fraquezas e justificativa, o avaliador tende a devolver notas médias genéricas.

**Limitação:** o avaliador baseado em modelo também é um LLM. Ele pode errar, ser influenciado pelo formato da resposta ou variar entre execuções. Por isso, é uma métrica útil, mas não uma verdade absoluta.

### 5.7 Combinação de notas

Uma abordagem simples é calcular a média:

```java
double score = (modelScore + syntaxScore) / 2.0;
```

**Como interpretar:** a nota em si não é “boa” ou “ruim”. O valor importante é a evolução entre versões do prompt usando o mesmo dataset e critérios.

---

## 6. Engenharia de prompts

### 6.1 O que é

**Prompt engineering** é o processo de melhorar iterativamente instruções para obter saídas mais consistentes, úteis e aderentes à tarefa.

Não é “escrever uma frase mágica”. É um ciclo de engenharia:

1. Definir objetivo.
2. Escrever prompt inicial.
3. Avaliar.
4. Aplicar técnica de melhoria.
5. Reavaliar.
6. Repetir.

**Problema que resolve:** reduz ambiguidade, melhora formato, aumenta aderência às restrições e torna o comportamento mais previsível — sem garantir previsibilidade absoluta.

### 6.2 Ser claro e direto

A primeira linha do prompt tem grande impacto. Ela deve dizer imediatamente qual é a tarefa.

Princípios:

- Use linguagem simples.
- Diga exatamente o que quer.
- Comece com verbos de ação: “gere”, “crie”, “escreva”, “identifique”.
- Prefira instruções a perguntas vagas.

| Versão | Exemplo |
|---|---|
| Vaga | “O que essa pessoa deveria comer?” |
| Clara e direta | “Gere um plano alimentar de um dia para um atleta, respeitando as restrições alimentares.” |

No exemplo do curso, essa mudança elevou a nota de 2,32 para 3,92.

**Por que funciona:** a instrução define ação, objeto, escopo e restrição. O modelo tem menos espaço para interpretar livremente.

### 6.3 Ser específico

Há dois tipos de especificidade:

| Tipo | O que controla | Exemplo |
|---|---|---|
| Diretrizes de saída | Formato, tamanho, estrutura, elementos obrigatórios, tom | “Inclua calorias, macros, horários e porções em gramas.” |
| Etapas de processo | Raciocínio e ordem de análise | “Primeiro identifique causas possíveis; depois compare evidências; por fim, conclua.” |

Diretrizes de saída devem estar presentes na maioria dos prompts. Etapas de processo são mais úteis em tarefas complexas, decisões e análises.

No exemplo do plano alimentar, diretrizes como calorias diárias, macros, horários, restrições e porções elevaram a pontuação de 3,92 para 7,86.

### 6.4 Tags XML

Tags XML delimitam diferentes partes do prompt: instruções, dados, código, documentação e exemplos.

```text
<athlete_information>
- Height: 180 cm
- Weight: 75 kg
- Goal: Build muscle
- Dietary restrictions: Vegetarian
</athlete_information>

Generate a meal plan based on the athlete information above.
```

**Para que servem:**

- Separar instruções de dados.
- Evitar que código, documentação e contexto se misturem.
- Deixar explícito o papel de cada bloco.
- Facilitar prompts com múltiplas variáveis.

**Ponto importante:** as tags não precisam ser XML oficial. Nomes descritivos, como `<sales_records>` ou `<my_code>`, são mais úteis que nomes genéricos como `<data>`.

### 6.5 Exemplos: one-shot e multi-shot

Exemplos mostram pares entrada → saída desejada.

| Técnica | Quantidade de exemplos | Quando usar |
|---|---|---|
| Zero-shot | Nenhum | Tarefa simples e bem definida |
| One-shot | Um exemplo | Estabelecer formato ou padrão básico |
| Multi-shot | Vários exemplos | Cobrir casos ambíguos, exceções e variações |

Exemplo de análise de sentimento:

```text
<examples>
  <sample_input>Great game tonight!</sample_input>
  <ideal_output>Positive</ideal_output>

  <sample_input>Oh yes, I really needed a flight delay tonight. Excellent!</sample_input>
  <ideal_output>Negative</ideal_output>
</examples>
```

**Como funciona:** o segundo exemplo ensina que sarcasmo deve ser classificado como negativo, mesmo quando as palavras parecem positivas.

**Boas práticas:**

- Envolver exemplos em tags XML.
- Explicar por que a saída é ideal.
- Incluir casos que o prompt costuma errar.
- Reaproveitar exemplos de alta nota encontrados nas avaliações.

**Limitação:** exemplos melhoram a aderência ao padrão mostrado, mas não garantem que o modelo siga o padrão em toda entrada.

---

## 7. Tool use

### 7.1 O que é

**Tool use** permite que o modelo solicite a execução de funções externas quando precisa de informações ou ações fora do seu conhecimento de treinamento.

Sem ferramentas, o modelo não pode acessar clima atual, banco de dados, arquivos locais ou APIs privadas. Ele só pode dizer que não tem acesso a esses dados.

**Problema que resolve:** transforma o modelo de uma base de conhecimento estática em um assistente capaz de trabalhar com dados dinâmicos e sistemas externos.

### 7.2 Quem decide e quem executa

A divisão de responsabilidades é essencial:

| Responsabilidade | Quem executa |
|---|---|
| Entender a pergunta | Modelo |
| Decidir se precisa de ferramenta | Modelo |
| Escolher a ferramenta e os argumentos | Modelo |
| Validar permissões e entradas | Sua aplicação |
| Executar a função | Sua aplicação/servidor |
| Devolver o resultado ao modelo | Sua aplicação |
| Gerar a resposta final | Modelo |

O modelo **não executa** a ferramenta diretamente. Ele emite uma solicitação estruturada; seu backend decide se aceita e executa.

### 7.3 Fluxo completo

1. A aplicação envia a pergunta e a lista de ferramentas disponíveis.
2. O modelo analisa a pergunta.
3. Se necessário, ele retorna um bloco `tool_use`.
4. O backend extrai nome e argumentos.
5. O backend executa a função real.
6. O backend devolve um bloco `tool_result`.
7. O modelo gera a resposta final.

```text
Usuário: "Qual é a hora atual?"
    ↓
Claude: tool_use -> get_current_datetime
    ↓
Servidor: executa a função e obtém "15:04:22"
    ↓
Servidor: tool_result -> "15:04:22"
    ↓
Claude: "Agora são 15:04."
```

### 7.4 Tool schema

Um **tool schema** descreve a ferramenta para o modelo:

- Nome.
- Descrição.
- Parâmetros.
- Tipos esperados.
- Descrição de cada parâmetro.

**Exemplo executável em Java — representação do schema:**

```java
// Records Java puros: representam a estrutura conceitual de um tool schema.
// A serialização exata para o formato exigido pelo provedor depende da integração.
record ToolParameter(
    String name,
    String type,
    String description,
    boolean required
) {}

record ToolSchema(
    String name,
    String description,
    List<ToolParameter> parameters
) {}

var getCurrentDatetime = new ToolSchema(
    "get_current_datetime",
    "Retorna a data e hora atuais.",
    List.of()
);
```

**Para que serve:** o modelo usa a descrição para decidir quando a ferramenta é relevante e como preencher os argumentos.

**Ponto crítico:** descrições vagas geram chamadas erradas. A qualidade do schema influencia diretamente a qualidade do tool use.

### 7.5 Blocos `tool_use` e `tool_result`

Quando o modelo decide usar uma ferramenta, a resposta pode conter múltiplos blocos:

- **Text block:** texto explicativo para o usuário.
- **ToolUse block:** instrução para o backend.

O `ToolUse block` contém:

- `id`: identificador da chamada.
- `name`: nome da ferramenta.
- `input`: argumentos.
- `type`: `tool_use`.

**Exemplo executável em Java — representação dos blocos:**

```java
// Records Java puros: representam os blocos conceituais trocados na conversa.
// A serialização exata depende do formato exigido pelo provedor.
record ToolUseBlock(
    String id,
    String name,
    Map<String, Object> input,
    String type
) {}

record ToolResultBlock(
    String toolUseId,
    String content,
    boolean isError,
    String type
) {}

ToolUseBlock toolUse = new ToolUseBlock(
    "call_123",
    "get_current_datetime",
    Map.of(),
    "tool_use"
);

// O backend executa a função real.
String resultado = LocalTime.now().toString();

ToolResultBlock toolResult = new ToolResultBlock(
    toolUse.id(),
    resultado,
    false,
    "tool_result"
);
```

Campos importantes:

- `toolUseId`: deve corresponder ao `id` do `tool_use`.
- `content`: resultado da execução, serializado como string.
- `isError`: indica se ocorreu erro.

**Por que o ID importa:** o modelo pode solicitar várias ferramentas na mesma resposta. O ID permite associar cada resultado à chamada correta, mesmo que os resultados voltem em ordem diferente.

### 7.6 Múltiplas interações e múltiplas ferramentas

O modelo pode:

- Chamar uma ferramenta, receber o resultado e depois chamar outra.
- Solicitar várias ferramentas em uma única resposta.
- Reconhecer que falta informação e perguntar ao usuário.

Exemplo: para “Que dia da semana será daqui a 11 dias?”, o modelo pode encadear:

1. `get_current_datetime`
2. `add_duration_to_datetime`

**Ponto de atenção:** cada etapa exige nova requisição com o histórico completo, incluindo os blocos de tool use e tool result. O curso destaca que o schema das ferramentas deve continuar sendo enviado, mesmo na chamada de acompanhamento, para o modelo entender as referências do histórico.

### 7.7 Fine-grained tool calling

No streaming normal de tool use, a API pode agrupar partes do JSON até validar um par chave-valor completo. Isso produz atrasos seguidos de “rajadas” de texto.

**Fine-grained tool calling** desativa essa validação no lado da API:

- Os fragmentos chegam mais cedo.
- Há menos buffering.
- O streaming fica mais granular.
- Porém, o JSON pode chegar inválido.

**Exemplo conceitual em Java:**

```java
// A intenção é habilitar o recebimento granular dos fragmentos do JSON
// durante o streaming da chamada de ferramenta.
//
// Como isso depende do provedor e da versão da integração,
// trate como configuração específica da API utilizada.
boolean fineGrained = true;
```

**Quando usar:**

- Você precisa mostrar progresso em tempo real.
- Quer processar resultados parciais rapidamente.
- O atraso de buffering prejudica a experiência.
- Seu código consegue tratar JSON inválido.

**Limitação:** para a maioria dos casos, o comportamento padrão com validação é mais seguro.

### 7.8 Text Editor Tool e Web Search Tool

O módulo inclui uma **ferramenta de edição de texto** e uma **ferramenta de busca na web**.

A transcrição disponível confirma que esses tópicos fazem parte do curso, mas não traz o detalhamento completo de seus schemas, parâmetros e comportamento. Portanto, não é possível descrever com precisão suas regras internas — nem propor uma conversão Java fiel — a partir das fontes fornecidas.

---

## 8. RAG e busca agêntica

### 8.1 O que é RAG

**RAG** (*Retrieval Augmented Generation*) é uma técnica para responder perguntas usando documentos externos sem colocar o documento inteiro no prompt.

O nome descreve o processo:

- **Retrieval:** recuperar trechos relevantes.
- **Augmented:** aumentar o prompt com esse contexto.
- **Generation:** gerar a resposta com base no contexto recuperado.

**Problema que resolve:** documentos grandes, confidenciais ou externos ao treinamento do modelo não cabem, não devem ser enviados integralmente ou não estão disponíveis para o modelo.

### 8.2 Por que não enviar tudo

Colocar um documento de 800 páginas inteiro no prompt tem problemas:

- Pode exceder o limite de contexto.
- Aumenta custo.
- Aumenta latência.
- Pode reduzir a eficácia do modelo diante de instruções muito longas.
- Inclui muito conteúdo irrelevante.

O RAG troca simplicidade por escalabilidade: exige pré-processamento e mecanismo de busca, mas permite trabalhar com documentos grandes e múltiplos documentos.

### 8.3 Chunking

**Chunking** é a divisão do documento em partes menores, chamadas *chunks*.

A qualidade do chunking afeta diretamente o RAG. Um chunk mal cortado pode separar um cabeçalho do seu conteúdo, cortar uma frase no meio ou misturar assuntos diferentes.

| Estratégia | Como funciona | Vantagem | Limitação |
|---|---|---|---|
| Por tamanho | Corta em blocos de N caracteres | Simples, universal | Pode cortar palavras/frases e separar cabeçalhos |
| Por estrutura | Divide por seções, cabeçalhos ou parágrafos | Blocos mais coerentes | Depende de documento bem estruturado |
| Semântica | Agrupa frases relacionadas por significado | Trechos mais relevantes | Mais custosa e complexa |
| Por frases | Agrupa N frases, com sobreposição opcional | Meio-termo prático | Pode perder contexto amplo |

**Overlap** é a repetição de uma pequena parte do chunk anterior no próximo. Ele preserva contexto e reduz o risco de cortar informações importantes.

**Exemplo executável em Java — chunking por tamanho com overlap:**

```java
import java.util.ArrayList;
import java.util.List;

class TextChunker {

    List<String> chunkByChar(String text, int chunkSize, int overlap) {
        List<String> chunks = new ArrayList<>();
        int start = 0;

        while (start < text.length()) {
            int end = Math.min(start + chunkSize, text.length());
            chunks.add(text.substring(start, end));

            if (end == text.length()) {
                break;
            }

            start = end - overlap;
        }

        return chunks;
    }
}
```

**Explicação:**

- `chunkSize`: tamanho máximo de cada bloco.
- `overlap`: quantidade de caracteres repetidos entre blocos consecutivos.
- O método avança menos que o tamanho do chunk, preservando uma sobreposição.

**Resultado:** blocos menores, com contexto parcialmente compartilhado entre vizinhos.

**Escolha prática do curso:** chunking por tamanho com overlap é apresentado como opção simples e confiável para produção, especialmente quando o formato do documento não é garantido.

### 8.4 Embeddings

Um **embedding** é um vetor numérico que representa o significado de um texto.

Textos semanticamente parecidos produzem vetores próximos, mesmo sem compartilhar palavras idênticas.

**Exemplo:**

- “O departamento de engenharia corrigiu muitos bugs.”
- “A equipe de software resolveu diversos defeitos.”

Essas frases podem ter embeddings próximos, embora usem palavras diferentes.

**Limitação conceitual:** cada dimensão do vetor é aprendida durante o treinamento e não possui significado humano diretamente interpretável. Não é correto afirmar que uma dimensão significa exatamente “formalidade” ou “feliz”.

O curso usa VoyageAI para embeddings e observa que a Anthropic não oferecia geração de embeddings no momento do material.

**Exemplo executável em Java — representação de dados:**

```java
// Record Java puro: representa um chunk e seu vetor.
// Não depende de Spring AI nem de provedor específico.
record EmbeddedChunk(String content, double[] embedding) {}
```

**Explicação:** além do vetor, é essencial guardar o texto original. A busca retorna vetores, mas o modelo precisa do texto correspondente.

### 8.5 Fluxo RAG completo

Fase de preparação:

1. Dividir o documento em chunks.
2. Gerar embedding de cada chunk.
3. Armazenar vetor + texto original em um vector store.

Fase de consulta:

4. Receber a pergunta do usuário.
5. Gerar embedding da pergunta.
6. Buscar os vetores mais próximos.
7. Recuperar os textos correspondentes.
8. Montar prompt com pergunta + trechos recuperados.
9. Enviar ao modelo para gerar a resposta.

```text
Documento -> chunks -> embeddings -> vector store
Pergunta -> embedding -> busca por similaridade -> trechos relevantes
Pergunta + trechos -> modelo -> resposta
```

**Por que armazenar o texto original:** a busca retorna vetores, mas o modelo precisa de texto. O vector store deve guardar o chunk original ou uma referência capaz de recuperá-lo.

### 8.6 Similaridade de cosseno

A **cosine similarity** mede o cosseno do ângulo entre dois vetores.

- Próxima de **1**: alta semelhança.
- Próxima de **0**: pouca relação.
- Próxima de **-1**: alta dissimilaridade.

Muitos bancos vetoriais usam **cosine distance**:

\[
cosine\ distance = 1 - cosine\ similarity
\]

Com distance:

- Valor próximo de **0**: alta similaridade.
- Valor maior: menor similaridade.

**Exemplo executável em Java — cálculo de similaridade de cosseno:**

```java
static double cosineSimilarity(double[] a, double[] b) {
    double dot = 0.0;
    double normA = 0.0;
    double normB = 0.0;

    for (int i = 0; i < a.length; i++) {
        dot += a[i] * b[i];
        normA += Math.pow(a[i], 2);
        normB += Math.pow(b[i], 2);
    }

    return dot / (Math.sqrt(normA) * Math.sqrt(normB));
}
```

**Explicação:** o método calcula o produto interno entre os vetores e divide pela multiplicação das normas. O resultado mede o ângulo entre eles.

**Exemplo do curso:** uma pergunta sobre engenharia de software teve similaridade 0,983 com o chunk de engenharia e 0,398 com o chunk de pesquisa médica. O sistema recupera o primeiro.

### 8.7 Busca híbrida e BM25

Busca semântica é forte para significado, mas fraca para identificadores exatos.

**Exemplo:** ao procurar `INC-2023-Q4-011`, a busca semântica pode retornar seções relacionadas a incidentes, mas não necessariamente o trecho que contém o ID exato.

**BM25** (*Best Match 25*) é um algoritmo de busca lexical. Ele funciona assim:

1. Tokeniza a consulta.
2. Conta a frequência dos termos.
3. Dá mais peso a termos raros.
4. Retorna documentos com maior pontuação para os termos relevantes.

| Tipo de busca | Ponta forte | Fraqueza |
|---|---|---|
| Semântica | Significado, contexto, sinônimos | Pode perder correspondência exata |
| Lexical BM25 | IDs, termos técnicos, correspondência exata | Não entende sinônimos ou contexto |
| Híbrida | Combina as duas | Exige combinar e ordenar resultados |

**Pipeline híbrido:**

```text
Pergunta -> busca semântica -> resultados A
Pergunta -> BM25           -> resultados B
Resultados A + B -> combinação/reranking -> contexto final
```

### 8.8 Multi-Index RAG pipeline

O curso lista um **pipeline RAG de múltiplos índices**, além de reclassificação (*reranking*) e recuperação contextual em sua descrição geral.

Porém, a transcrição disponível não detalha como esse pipeline é construído, quais índices são usados ou como o reranking é implementado. Essa parte não pode ser confirmada em profundidade pelas fontes fornecidas.

---

## 9. Recursos avançados do Claude

### 9.1 Extended thinking

**Extended thinking** é o modo de raciocínio estendido. Antes de gerar a resposta final, o modelo produz um bloco de raciocínio.

**Para que serve:**

- Problemas complexos.
- Tarefas que exigem análise em várias etapas.
- Casos em que prompts normais não atingem a precisão necessária.

**Como funciona:** a resposta passa a conter dois blocos:

1. Bloco de raciocínio (*thinking*).
2. Bloco de texto com a resposta final.

**Custos:**

- Mais tokens consumidos.
- Maior latência.
- Resposta mais complexa de tratar no código.

**Regras apresentadas:**

- `budget_tokens` mínimo: 1024.
- `max_tokens` deve ser maior que o orçamento de raciocínio.
- Há incompatibilidades, inclusive com prefill de mensagem do assistente.
- O recurso restringe o uso de temperatura.

O raciocínio pode vir **redigido** (*redacted*) quando sistemas de segurança sinalizam o conteúdo. Mesmo oculto, ele pode ser repassado em conversas futuras sem perder contexto.

**Recomendação do curso:** otimize primeiro o prompt e use avaliações. Ative extended thinking somente se a precisão continuar insuficiente.

### 9.2 Imagens

Claude aceita imagens em base64 ou URL, junto com blocos de texto.

Limites citados:

- Até 100 imagens por requisição.
- 5 MB por imagem.
- 8000 px para uma imagem única.
- 2000 px ao enviar várias imagens.

Consumo estimado de tokens:

\[
tokens = \frac{largura \times altura}{750}
\]

**Para que serve:** descrever imagens, comparar imagens, contar objetos, extrair informações visuais e apoiar análises estruturadas.

**Boa prática:** instruções visuais simples produzem resultados menos confiáveis. Para contagem ou avaliação, forneça metodologia:

```text
1. Identifique cada objeto individualmente e numere-o.
2. Conte novamente de outro ponto de partida.
3. Informe o total verificado.
```

**Resultado esperado:** maior consistência do que uma pergunta solta como “quantos objetos há?”.

### 9.3 PDFs

Claude pode receber PDFs diretamente como blocos do tipo `document`, com `media_type: application/pdf`.

Ele consegue analisar:

- Texto.
- Imagens.
- Gráficos.
- Tabelas.
- Estrutura do documento.

**Diferença em relação ao RAG:** enviar um PDF diretamente é útil quando o documento cabe no contexto. RAG é mais adequado quando o material é grande demais ou quando você precisa buscar apenas trechos relevantes.

### 9.4 Citações

Com `citations: { "enabled": true }`, o modelo pode indicar a origem exata das informações usadas.

Para PDFs, a citação inclui:

- Texto citado.
- Índice do documento.
- Título do documento.
- Páginas inicial e final.

Para texto simples, inclui posições de caracteres.

**Problema que resolve:** reduz a opacidade da resposta. O usuário pode verificar se a afirmação veio do documento fornecido e em qual parte.

**Quando é útil:**

- Documentos oficiais.
- Contratos, relatórios e normas.
- Respostas que exigem auditoria.
- Interfaces em que o usuário precisa conferir a fonte.

### 9.5 Prompt caching

**Prompt caching** reutiliza o processamento de conteúdo repetido entre requisições, reduzindo latência e custo.

Sem cache, o modelo reprocessa todo o contexto a cada chamada. Com cache, o trabalho de pré-processamento de um prefixo idêntico pode ser reaproveitado.

**Quando ajuda:**

- Várias perguntas sobre o mesmo documento longo.
- System prompt fixo e grande.
- Definições de ferramentas estáveis.
- Conversas em que o histórico anterior permanece idêntico.

**Quando não ajuda:**

- Prompts curtos.
- Conteúdo que muda a cada requisição.
- Requisições esparsas, fora da janela de validade do cache.

**Regras principais:**

- O cache não é automático: é preciso marcar **cache breakpoints** com `cache_control: { "type": "ephemeral" }`.
- O cache guarda tudo até o ponto de interrupção, inclusive.
- O conteúdo até o breakpoint precisa ser idêntico.
- Qualquer alteração invalida o cache.
- Duração padrão: 5 minutos.
- Cada reutilização renova o timer.
- Há opção de TTL de 1 hora, com custo maior de gravação.
- É possível criar até 4 breakpoints.
- O conteúdo mínimo para cache é 1024 tokens no total.

**Ordem interna de processamento:**

1. Ferramentas.
2. System prompt.
3. Mensagens.

**Exemplo:** se você envia um documento de 20 mil tokens e faz dez perguntas sobre ele, o ideal é colocar o breakpoint após o documento. Assim, apenas a pergunta muda a cada requisição.

### 9.6 Code execution e Files API

O módulo lista **code execution** e **Files API**, mas a transcrição disponível não traz o detalhamento dessas seções. Não é possível confirmar, a partir das fontes fornecidas, como esses recursos foram apresentados em detalhe — nem propor uma conversão Java fiel.

---

## 10. Model Context Protocol (MCP)

### 10.1 O que é MCP

O **Model Context Protocol** é uma camada padrão para conectar aplicações de IA a ferramentas, dados e prompts externos.

Em vez de implementar cada integração manualmente no seu servidor, você usa servidores MCP que já expõem essas capacidades.

**Problema que resolve:** integrar um serviço como GitHub exigiria criar, testar e manter muitos schemas e funções. Um servidor MCP centraliza essa integração e a torna reutilizável.

### 10.2 Arquitetura

| Componente | Papel |
|---|---|
| Cliente MCP | Fica na sua aplicação e conversa com servidores MCP |
| Servidor MCP | Expõe ferramentas, recursos e prompts |
| Serviço externo | GitHub, banco de dados, sistema de arquivos, API etc. |

**Diferença em relação a tool use comum:**

| Aspecto | Tool use tradicional | MCP |
|---|---|---|
| Definição das ferramentas | Feita por você | Fornecida pelo servidor MCP |
| Implementação das funções | Feita por você | Implementada/mantida no servidor MCP |
| Reuso | Específico da sua aplicação | Pode ser reutilizado por clientes compatíveis |
| Foco | Mecanismo de chamada | Padronização e distribuição de integrações |

MCP e tool use não são concorrentes: MCP é uma forma padronizada de fornecer ferramentas que o modelo pode usar.

### 10.3 Comunicação

O MCP é agnóstico quanto ao transporte:

- stdio local.
- HTTP.
- WebSockets.
- Outros protocolos de rede.

Mensagens centrais:

- `ListToolsRequest` / `ListToolsResult`: descobrir ferramentas disponíveis.
- `CallToolRequest` / `CallToolResult`: executar uma ferramenta com argumentos e receber o resultado.

**Fluxo completo:**

```text
Usuário pergunta
    ↓
Aplicação pede lista de ferramentas ao cliente MCP
    ↓
Cliente MCP -> ListToolsRequest -> Servidor MCP
    ↓
Servidor MCP -> ListToolsResult -> Cliente MCP
    ↓
Aplicação envia pergunta + ferramentas ao Claude
    ↓
Claude solicita tool_use
    ↓
Aplicação pede execução ao cliente MCP
    ↓
Cliente MCP -> CallToolRequest -> Servidor MCP
    ↓
Servidor MCP acessa serviço externo
    ↓
Resultado volta como CallToolResult
    ↓
Aplicação envia tool_result ao Claude
    ↓
Claude responde ao usuário
```

### 10.4 Ferramentas, recursos e prompts

| Componente | Função | Exemplo |
|---|---|---|
| Tools | Executar ações | Ler ou editar documento |
| Resources | Expor dados para leitura | Listar documentos; obter conteúdo por ID |
| Prompts | Fornecer templates de prompt | Template para análise de documento |

No exemplo do curso, o servidor mantém documentos em memória e expõe:

- `read_doc_contents`: lê um documento por ID.
- `edit_document`: substitui um texto exato por outro.

**Exemplo executável em Java — lógica equivalente da ferramenta:**

```java
import java.util.HashMap;
import java.util.Map;

class DocumentService {

    private final Map<String, String> documents = new HashMap<>();

    String readDocContents(String docId) {
        String document = documents.get(docId);

        if (document == null) {
            throw new IllegalArgumentException(
                "Doc with id " + docId + " not found"
            );
        }

        return document;
    }

    void editDocument(String docId, String oldStr, String newStr) {
        String document = documents.get(docId);

        if (document == null) {
            throw new IllegalArgumentException(
                "Doc with id " + docId + " not found"
            );
        }

        documents.put(docId, document.replace(oldStr, newStr));
    }
}
```

**Como funciona:**

- `readDocContents` recebe um ID e retorna o conteúdo.
- `editDocument` recebe ID, texto antigo e texto novo.
- IDs inválidos geram erro descritivo.
- Em um servidor MCP real, essas operações seriam expostas conforme o protocolo e o SDK utilizados.

### 10.5 Recursos MCP

Recursos funcionam como leitura de dados, semelhante a `GET`.

| Tipo | URI | Uso |
|---|---|---|
| Direto | `docs://documents` | Listar todos os documentos |
| Com template | `docs://documents/{doc_id}` | Obter um documento específico |

O cliente envia `ReadResourceRequest` com um URI e recebe os dados.

O `mime_type` informa o formato:

- `application/json`
- `text/plain`
- Outros tipos MIME válidos

**Diferença entre tool e resource:**

- **Tool:** o modelo pode solicitar uma ação.
- **Resource:** a aplicação recupera dados, normalmente para incluir no contexto.

### 10.6 Desenvolvimento e testes

O **MCP Inspector** permite testar servidores sem integrá-los imediatamente a um app completo:

- Listar ferramentas.
- Executar chamadas.
- Testar recursos.
- Depurar isoladamente.

Prática de depuração recomendada:

- Ao adicionar uma nova camada — ferramenta, fonte de dados ou servidor MCP — faça primeiro uma pergunta cuja resposta você já conhece.
- Isso cria uma linha de base verificável antes de construir a próxima parte.

**Exemplo:** antes de integrar ferramentas de documentos, pergunte “quanto é 1 + 1?”. Se a resposta for 2, a configuração básica do chat está funcionando.

---

## 11. Claude Code e Computer Use

### 11.1 Claude Code

**Claude Code** é um assistente de programação agêntico no terminal.

Ele pode:

- Ler, buscar e editar arquivos.
- Executar comandos.
- Acessar a web.
- Conectar servidores MCP.

Comandos importantes:

| Comando | Função |
|---|---|
| `/init` | Analisa o projeto e cria `CLAUDE.md` |
| `/clear` | Limpa histórico e contexto da conversa |
| `#` | Adiciona uma nota ao `CLAUDE.md` |

O `CLAUDE.md` funciona como contexto persistente do projeto, podendo existir em escopos:

- Projeto.
- Local.
- Usuário.

Ele pode conter:

- Comandos de build.
- Convenções de código.
- Padrões do repositório.

**Fluxo recomendado:**

1. Dar contexto: pedir que o Claude leia arquivos relevantes.
2. Pedir um plano, sem implementar ainda.
3. Somente depois pedir a implementação.

**Exemplo:**

```text
1. Read the math.py and document.py files.
2. Plan how to implement document_path_to_markdown. Do not write code yet.
3. Implement the plan.
```

**Resultado esperado:** o modelo primeiro entende o código existente, depois propõe uma abordagem e só então altera arquivos. Isso reduz o risco de implementações incoerentes com o projeto.

### 11.2 Desenvolvimento orientado a testes

Variação recomendada pelo curso:

1. Fornecer contexto.
2. Pedir casos de teste.
3. Implementar os testes.
4. Gerar código até passar nos testes.

**Por que funciona:** os testes se tornam critérios objetivos de sucesso. O modelo não precisa “adivinhar” quando a tarefa está concluída.

### 11.3 Computer Use

**Computer Use** é um conjunto de ferramentas que permite ao Claude interagir com um computador:

- Navegar na web.
- Usar aplicativos desktop.
- Executar tarefas que dependem de interface visual.

Ele é apresentado como exemplo de agente porque exige:

- Uso de ferramentas.
- Tarefas em várias etapas.
- Observação do ambiente.
- Resolução autônoma de problemas.

### 11.4 MCP no Claude Code

O Claude Code possui cliente MCP integrado.

Servidores MCP podem adicionar ferramentas para serviços como:

- Sentry.
- Playwright.
- Figma.
- Jira/Confluence.
- Web scraping.
- Slack.

Um servidor é registrado com um comando como:

```bash
claude mcp add [nome] [comando]
```

**Exemplo do curso:**

```bash
claude mcp add documents uv run main.py
```

Depois, o Claude Code pode usar as ferramentas expostas por esse servidor.

---

## 12. Agentes e fluxos de trabalho

### 12.1 Workflow vs. agente

| Abordagem | Definição | Quando usar | Vantagens | Desvantagens |
|---|---|---|---|---|
| **Workflow** | Sequência pré-definida de chamadas ao modelo | Você conhece as etapas exatas | Previsível, testável, mais preciso por etapa | Pouco flexível |
| **Agente** | Modelo recebe objetivo + ferramentas e decide o caminho | Tarefas variáveis ou imprevisíveis | Flexível, adaptável, lida com casos não previstos | Menos previsível, mais difícil/caro de controlar |

**Regra prática do curso:**

- Use **workflow** quando você consegue desenhar o fluxo de etapas antecipadamente.
- Use **agente** quando não sabe exatamente quais etapas, ferramentas ou parâmetros serão necessários.

Um agente não é apenas “um LLM com ferramentas”: é um sistema em que o modelo planeja e combina ferramentas para atingir um objetivo.

### 12.2 Padrão avaliador-otimizador

No exemplo de imagem → modelo CAD:

1. Descrever a imagem.
2. Gerar o modelo 3D.
3. Criar renderização.
4. Comparar renderização com a imagem original.
5. Corrigir e repetir até aprovar.

Esse é o padrão **avaliador-otimizador**:

```text
Entrada -> Produtor -> Saída provisória
                         ↓
                    Avaliador
                         ↓
              Aprovada? -> Sim: resultado final
                         ↓
                        Não -> feedback -> Produtor
```

**Para que serve:** melhora iterativamente uma saída com base em critérios explícitos.

**Cuidado:** é preciso definir critérios de parada. Sem isso, o ciclo pode se repetir indefinidamente.

### 12.3 Paralelização

Na paralelização, uma tarefa complexa é dividida em subtarefas independentes executadas ao mesmo tempo; depois, um passo final agrega os resultados.

```text
            -> Avaliação de metal     ->
Entrada     -> Avaliação de polímero  -> Agregador -> Recomendação final
            -> Avaliação de cerâmica  ->
```

**Quando usar:**

- Há critérios independentes.
- Cada análise pode ser feita separadamente.
- Você quer foco em um critério por chamada.
- Adicionar novas opções não deve exigir reescrever os prompts existentes.

**Benefícios:**

- Foco em um critério por chamada.
- Facilidade para melhorar um prompt sem afetar os outros.
- Escalabilidade ao adicionar novas opções.
- Menor carga cognitiva por requisição.

### 12.4 Encadeamento

**Chaining** divide uma tarefa em etapas sequenciais, onde a saída de uma etapa alimenta a seguinte.

```text
Entrada -> Etapa 1 -> Etapa 2 -> Etapa 3 -> Saída
```

É útil quando uma instrução única longa faz o modelo ignorar restrições.

**Exemplo:**

1. Gerar um artigo.
2. Enviar esse artigo para uma segunda chamada focada exclusivamente em revisão.
3. Remover emojis, menções de autoria por IA e linguagem inadequada.

**Por que funciona:** a segunda chamada tem uma tarefa estreita: revisar. Ela não precisa simultaneamente criar conteúdo e cumprir todas as restrições.

### 12.5 Roteamento

No **routing**, uma primeira chamada classifica a entrada e o sistema direciona ela para um pipeline especializado.

```text
Entrada -> Classificador -> Categoria A -> Pipeline A
                        -> Categoria B -> Pipeline B
                        -> Categoria C -> Pipeline C
```

Cada categoria pode ter:

- Prompt próprio.
- Ferramentas próprias.
- Fluxo próprio.

**Exemplo do curso:** classificar um tópico como educacional, entretenimento, comédia, vlog, review ou storytelling; depois usar o template de prompt correspondente.

**Quando usar:**

- Há tipos distintos de solicitação.
- As categorias são bem definidas.
- A classificação é confiável.
- O ganho de especialização compensa a chamada extra de roteamento.

### 12.6 Ferramentas para agentes

Agentes funcionam melhor com ferramentas **abstratas e combináveis**, não hiperespecializadas.

O Claude Code usa ferramentas genéricas como:

- Ler arquivo.
- Escrever arquivo.
- Editar arquivo.
- Buscar arquivos.
- Executar comandos.
- Pesquisar conteúdo.

O modelo combina essas primitivas para realizar tarefas complexas.

**Exemplo:** com ferramentas de data e hora, o modelo pode encadear:

1. `get_current_datetime`
2. `add_duration_to_datetime`
3. `set_reminder`

Para “Que dia da semana será daqui a 11 dias?”, ele usa as duas primeiras. Para “lembre-me de ir à academia na próxima quarta-feira”, pode usar as três.

Ele também pode reconhecer informação faltante e perguntar ao usuário antes de agir.

### 12.7 Inspeção ambiental

Um agente precisa observar o resultado das próprias ações.

Sem isso, ele não sabe se:

- Um clique funcionou.
- Um arquivo foi alterado corretamente.
- Uma API retornou o esperado.

Práticas apresentadas:

- Ler arquivos antes de editá-los.
- Capturar tela após interações com interface.
- Verificar respostas de API.
- Validar saída gerada contra requisitos.
- Usar system prompts para instruir o agente a inspecionar o ambiente.

**Princípio central:**

**Ler antes de escrever.**

Sempre permitir que o agente verifique se a ação teve o efeito esperado.

**Exemplo:** antes de adicionar uma rota a um arquivo, o agente lê o arquivo existente. Depois, edita com base na estrutura real, em vez de presumir seu conteúdo.

---

## Pontos transversais do curso

- **Prompts são software:** devem ter objetivo, testes, métricas e iteração.
- **Contexto é responsabilidade do aplicativo:** a API não mantém memória entre chamadas.
- **Ferramentas ampliam o modelo, mas o backend executa e controla as ações.**
- **RAG reduz custo e latência ao enviar apenas contexto relevante.**
- **Cache só ajuda quando há conteúdo grande e repetido.**
- **MCP padroniza a integração com ferramentas e fontes de dados.**
- **Prefira workflows quando o processo é conhecido; use agentes quando a flexibilidade for necessária.**
- **Nenhuma técnica de prompting é garantia absoluta:** instruções, exemplos, schemas e system prompts aumentam a probabilidade de um bom resultado, mas exigem validação.
- **Modelos podem errar:** cálculos, fatos, classificações e decisões devem ser verificados pela aplicação quando tiverem consequências reais.

---

## Adaptação dos exemplos para Java

- Os exemplos originais em Python foram adaptados para **Java**.
- Quando a conversão é direta, foi usado **Java padrão**: `record`, `List`, `Map`, `LocalTime`, `HashMap` e estruturas equivalentes.
- Quando o exemplo depende de uma chamada a LLM, foi usada a **intenção equivalente em Spring AI**, com aviso de que nomes exatos de métodos podem variar conforme a versão.
- Exemplos que dependem de recursos específicos da Claude API — como `cache_control`, `fine_grained`, tool blocks e schemas — foram mantidos como **conceituais em Java**, para não inventar APIs ou métodos inexistentes.
- A teoria, a estrutura dos 12 capítulos, as tabelas e as explicações do resumo original foram preservadas.

## Correções técnicas desta revisão

- **Corrigido o exemplo de temperatura:** a forma `.options(options -> options.temperature(...))` não corresponde à API fluente do Spring AI. Foi substituída por `ChatOptions.builder().temperature(...).build()`, com ressalva explícita de que a construção exata de options pode variar conforme a versão do Spring AI e o provedor.
- **Clarificada a natureza dos exemplos:** o guia agora distingue explicitamente **código executável em Java** — lógica independente de provedor, como chunking, cosine similarity e validação JSON — de **exemplos conceituais**, que representam a intenção da chamada à API.
- **Clarificado o `chatClient`:** nos exemplos Spring AI, `chatClient` representa um bean configurado/injetado na aplicação, não uma variável global ou classe estática.
- **Corrigida a classificação do `EmbeddedChunk`:** agora está identificado como exemplo executável em Java puro, pois é apenas um `record` de representação de dados, sem dependência de Spring AI ou provedor.
- **Corrigida a nomenclatura do tool schema:** a variável Java foi renomeada de `get_current_datetime` para `getCurrentDatetime`, seguindo a convenção Java; o nome da ferramenta exposto ao modelo continua `get_current_datetime`.
- **Corrigido erro de texto:** “Explicai melhor a diferença” foi corrigido para “Explicitei melhor a diferença”.
- **Reforçada a ressalva de compatibilidade:** os exemplos Spring AI devem ser conferidos contra a versão utilizada no projeto, pois nomes de classes, builders e opções podem mudar entre versões.
- **Mantidas as ressalvas sobre fontes incompletas:** Multi-Index RAG pipeline, Text Editor Tool, Web Search Tool, Code Execution e Files API continuam marcados como não detalhados nas transcrições disponíveis.

## Pontos que ainda precisam ser verificados

- **Versão do Spring AI no seu projeto:** confirme se `ChatClient`, `ChatOptions.builder()`, `.stream().content()` e a forma de configurar o provedor correspondem à versão usada.
- **Provedor Groq via Spring AI:** verifique quais opções de chat — como temperatura, `maxTokens` e tool calling — são efetivamente suportadas pela integração/configuração utilizada.
- **Tool calling no Spring AI:** a forma de declarar ferramentas, expor schemas e tratar `tool_use`/`tool_result` varia entre versões e provedores; os exemplos deste guia são conceituais e não devem ser copiados sem conferência.
- **Recursos específicos da Claude API:** `cache_control`, `fine_grained`, thinking blocks, citations e tool blocks são recursos da API Anthropic; não assuma equivalência direta na Groq ou no Spring AI.
- **Tópicos sem detalhe nas transcrições:** Multi-Index RAG pipeline, Text Editor Tool, Web Search Tool, Code Execution e Files API permanecem sem detalhamento suficiente nas fontes fornecidas.
