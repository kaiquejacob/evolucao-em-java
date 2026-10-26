# Resumo — Building with the Claude API

> Material de estudo baseado nas transcrições do curso **Building with the Claude API**.  
> Foco: conceitos de LLMs, Claude API, prompts, tool use, RAG, MCP, agentes e workflows.  
> Este documento não inclui implementação em Java/Spring AI.

O curso ensina a construir aplicações com LLMs usando a Claude API da Anthropic: desde uma requisição básica até avaliação de prompts, tool use, RAG, recursos multimodais, cache, MCP, agentes e fluxos de trabalho.

## 1. Fundamentos da API

### Como funciona uma chamada

Uma interação típica passa por cinco fases:

1. Aplicativo envia a solicitação ao seu servidor.
2. Seu servidor chama a API da Anthropic.
3. O modelo processa a entrada.
4. A API devolve a resposta ao servidor.
5. O servidor repassa a resposta ao cliente.

A chamada à API deve ocorrer no **servidor**, nunca diretamente no frontend, porque a API key é secreta e ficaria exposta no cliente.

Dentro do modelo, o texto passa por:

- **Tokenização:** divisão do texto em tokens — palavras, partes de palavras, espaços ou símbolos.
- **Embeddings:** cada token é convertido em uma representação numérica que captura significado.
- **Contextualização:** o modelo ajusta essas representações conforme as palavras vizinhas.
- **Geração:** o modelo calcula probabilidades para o próximo token e o seleciona, repetindo o processo até parar.

A geração termina por três motivos principais:

- Atingir `max_tokens`.
- Produzir um token natural de fim de sequência.
- Encontrar uma `stop_sequence`.

A resposta da API traz:

- Texto gerado.
- Uso de tokens de entrada e saída.
- Motivo de parada (`stop_reason`).

### Requisição básica

Uma chamada essencial inclui:

- `model`: modelo Claude escolhido.
- `max_tokens`: limite máximo de tokens da resposta; é um teto de segurança, não uma meta de tamanho.
- `messages`: histórico da conversa, com mensagens `user` e `assistant`.

A resposta normalmente é lida a partir do primeiro bloco de conteúdo, por exemplo `message.content[0].text` no SDK Python.

### Chaves de API

- Criada no Anthropic Console.
- Exibida apenas uma vez.
- Deve ser armazenada fora do código, normalmente em `.env`.
- `.env` deve estar no `.gitignore`.

## 2. Conversas e contexto

### Claude não tem memória

A API é **stateless**: Claude não guarda automaticamente o histórico entre requisições.

Para simular uma conversa:

1. Enviar a mensagem do usuário.
2. Receber a resposta do assistente.
3. Adicionar essa resposta ao histórico como mensagem `assistant`.
4. Adicionar a próxima pergunta como mensagem `user`.
5. Enviar novamente toda a lista de mensagens.

Isso explica por que perguntas como “escreva outra frase” só funcionam se o contexto anterior for enviado junto.

### System prompts

O **system prompt** define papel, tom, estilo, restrições e comportamento geral do modelo.

Ele:

- Não é uma mensagem comum do usuário.
- É um parâmetro separado da requisição.
- Mantém o modelo focado na tarefa.
- Padroniza o comportamento entre interações.

Exemplo do curso: um system prompt de “tutor paciente de matemática” faz o modelo conduzir o aluno com perguntas e passos, em vez de entregar a solução imediatamente.

Na API Claude, `system=None` não é aceito; o parâmetro deve ser incluído apenas quando existir.

## 3. Temperatura e streaming

### Temperatura

A temperatura controla a aleatoriedade na escolha do próximo token.

| Faixa | Comportamento | Usos típicos |
|---|---|---|
| 0,0–0,3 | Mais previsível | Fatos, código, extração de dados, moderação |
| 0,4–0,7 | Equilíbrio | Resumos, conteúdo educacional, resolução de problemas |
| 0,8–1,0 | Mais variado | Brainstorming, escrita criativa, marketing, humor |

A temperatura **não garante** respostas diferentes; ela altera probabilidades. Mesmo com temperatura alta, o modelo pode repetir respostas semelhantes.

### Streaming

Sem streaming, o usuário espera a resposta completa. Com streaming, o servidor recebe eventos incrementais e pode exibir o texto conforme é gerado.

Eventos principais:

- `message_start`: início da mensagem.
- `content_block_start`: início de um bloco de conteúdo.
- `content_block_delta`: partes do texto gerado.
- `content_block_stop`: fim do bloco.
- `message_delta` e `message_stop`: conclusão da mensagem.

O SDK também oferece uma interface simplificada que entrega apenas o texto, além de permitir recuperar a mensagem final completa após o streaming terminar.

## 4. Saída estruturada

Quando você pede JSON, código ou outro formato, o modelo pode acrescentar explicações ou cercar o conteúdo em blocos Markdown.

Para obter dados brutos, o curso combina:

- **Assistant message prefill:** pré-preenche o início da resposta do assistente, por exemplo com “```json”, induzindo o modelo a continuar dali.
- **Stop sequences:** define uma sequência que encerra a geração, por exemplo “```”, para cortar o fechamento do bloco de código.

O resultado é um JSON mais limpo, adequado para parse direto pelo aplicativo.

## 5. Avaliação de prompts

### Por que avaliar

Escrever um prompt e testá-lo uma ou duas vezes é insuficiente: usuários reais trazem entradas inesperadas.

O fluxo básico tem cinco etapas:

1. Escrever o prompt inicial.
2. Criar um dataset de entradas de teste.
3. Executar cada entrada no modelo.
4. Avaliar cada resposta com um classificador.
5. Ajustar o prompt e repetir.

O objetivo é comparar versões por métricas objetivas, não por impressão subjetiva.

### Geração de datasets

Os casos de teste podem ser criados manualmente ou gerados pelo próprio modelo.

Para gerar JSON confiável, o curso usa novamente:

- Prefill da mensagem do assistente.
- Stop sequence para interromper a geração no ponto desejado.

### Tipos de avaliadores

| Avaliador | Como funciona | Melhor para |
|---|---|---|
| Baseado em código | Regras programáticas e validação sintática | Formato, estrutura, sintaxe, presença de campos |
| Baseado em modelo | Outra chamada de LLM julga a resposta | Qualidade, completude, utilidade, segurança, aderência à tarefa |
| Humano | Pessoa revisa e pontua | Julgamento subjetivo, profundidade, relevância, tom |

Para código, o curso separa critérios:

- **Formato:** retornar apenas Python, JSON ou regex, sem comentários.
- **Sintaxe válida:** o código precisa ser parseável.
- **Aderência à tarefa:** a solução realmente resolve o pedido.

Os dois primeiros podem ser validados por código; o terceiro é mais adequado a um avaliador baseado em modelo.

O avaliador de modelo deve retornar não só a nota, mas também pontos fortes, fraquezas e raciocínio. Isso reduz a tendência de notas médias genéricas e torna o resultado mais interpretável.

## 6. Engenharia de prompts

### Processo iterativo

A engenharia de prompt é um ciclo:

1. Definir objetivo.
2. Escrever prompt inicial.
3. Avaliar.
4. Aplicar técnica de melhoria.
5. Reavaliar.
6. Repetir até atingir o resultado desejado.

### Ser claro e direto

A primeira linha do prompt é a mais importante.

Boas práticas:

- Usar linguagem simples.
- Dizer exatamente o que se espera.
- Começar com verbos de ação: “gere”, “crie”, “escreva”, “identifique”.
- Evitar perguntas vagas.

Exemplo:

- Ruim: “O que essa pessoa deveria comer?”
- Melhor: “Gere um plano alimentar de um dia para um atleta, respeitando as restrições alimentares.”

### Ser específico

Há dois tipos de especificidade:

- **Diretrizes de saída:** tamanho, formato, estrutura, elementos obrigatórios, tom e estilo.
- **Etapas de processo:** passos que o modelo deve seguir antes de responder.

Diretrizes de saída devem ser usadas na maioria dos prompts. Etapas de processo são úteis em análise, decisão e raciocínio complexo.

### Tags XML

Tags XML delimitam diferentes partes do prompt: instruções, dados, código, documentação e exemplos.

Exemplos de nomes descritivos:

- `<sales_records>`
- `<athlete_information>`
- `<my_code>`
- `<docs>`

Elas são especialmente úteis quando o prompt mistura:

- Grandes volumes de contexto.
- Código e documentação.
- Dados estruturados.
- Múltiplas variáveis interpoladas.

### Exemplos: one-shot e multi-shot

Exemplos mostram pares entrada → saída desejada.

- **One-shot:** um exemplo.
- **Multi-shot:** vários exemplos cobrindo casos diferentes.

Eles são eficazes para:

- Casos ambíguos, como sarcasmo.
- Formatos complexos, como JSON.
- Estilo e tom específicos.
- Casos de erro frequentes.

Boas práticas:

- Envolver exemplos em tags XML.
- Explicar por que a saída é ideal.
- Reaproveitar exemplos de alta nota encontrados nas avaliações.

## 7. Tool use

### Conceito

**Tool use** permite que o modelo solicite execução de funções externas quando precisa de dados ou ações fora do seu conhecimento de treinamento.

Exemplos:

- Consultar clima atual.
- Consultar banco de dados.
- Chamar APIs externas.
- Ler ou alterar arquivos.

### Fluxo básico

1. O aplicativo envia a pergunta e descreve as ferramentas disponíveis.
2. Claude decide que precisa de uma ferramenta e retorna uma solicitação de uso.
3. O servidor executa a função correspondente.
4. O resultado é devolvido ao modelo.
5. Claude gera a resposta final usando a pergunta original mais o dado recuperado.

### Elementos do mecanismo

O módulo cobre:

- **Tool functions:** implementações reais executadas pelo servidor.
- **Tool schemas:** descrição estruturada do nome, finalidade e parâmetros.
- **Message blocks:** blocos especiais para solicitação e resultado de ferramenta.
- **Tool results:** devolução do resultado ao modelo.
- **Múltiplas interações:** o modelo pode encadear várias chamadas.
- **Múltiplas ferramentas:** o modelo escolhe entre ferramentas disponíveis.
- **Fine-grained tool calling:** controle mais granular do comportamento de chamada.
- **Text editor tool** e **web search tool:** ferramentas específicas para edição de texto e busca na web.

A ideia central: o modelo **decide** quando chamar uma ferramenta, mas o **seu código** é quem executa a ação e controla o acesso aos sistemas externos.

## 8. RAG e busca agêntica

### O problema do RAG

**RAG** (*Retrieval Augmented Generation*) resolve o caso em que o documento é grande demais, confidencial ou externo ao treinamento do modelo.

Em vez de colocar tudo no prompt:

1. Divide-se o documento em partes.
2. Recupera-se apenas os trechos relevantes.
3. Envia-se o contexto recuperado ao modelo.

Colocar o documento inteiro no prompt tem limites:

- Tamanho máximo.
- Custo maior.
- Latência maior.
- Pior desempenho com instruções muito longas.

### Chunking

A segmentação (*chunking*) afeta diretamente a qualidade do RAG.

| Estratégia | Como funciona | Vantagem | Limitação |
|---|---|---|---|
| Por tamanho | Corta em blocos de N caracteres | Simples e universal | Pode cortar palavras/frases e separar cabeçalhos |
| Por estrutura | Divide por seções, cabeçalhos ou parágrafos | Blocos mais coerentes | Depende de documento bem estruturado |
| Semântica | Agrupa frases relacionadas por significado | Trechos mais relevantes | Mais custosa e complexa |
| Por frases | Agrupa N frases, com sobreposição opcional | Meio-termo prático | Pode perder contexto amplo |

O curso destaca o uso de **overlap**: repetir uma pequena parte do bloco anterior no próximo, preservando contexto e evitando cortes abruptos.

### Embeddings

Um **embedding** é um vetor numérico que representa o significado de um texto.

Textos semanticamente parecidos geram vetores próximos, mesmo sem compartilhar palavras idênticas.

No fluxo RAG:

1. Divide-se o documento em chunks.
2. Gera-se embedding de cada chunk.
3. Armazena-se vetor + texto original em um vector store.
4. Gera-se embedding da pergunta do usuário.
5. Buscam-se os vetores mais próximos.
6. Monta-se prompt com pergunta + trechos recuperados.
7. Envia-se ao Claude para gerar a resposta.

A similaridade é geralmente medida por **cosine similarity**:

- Próxima de 1: alta semelhança.
- Próxima de -1: alta dissimilaridade.
- 0: ausência de relação.

Muitos bancos vetoriais usam **cosine distance**:

\[
cosine\ distance = 1 - cosine\ similarity
\]

Valores menores indicam maior semelhança.

### Busca híbrida e BM25

Busca semântica entende significado, mas pode falhar em identificadores exatos, como `INC-2023-Q4-011`.

A busca lexical **BM25** resolve isso dando peso maior a termos raros e específicos e penalizando palavras comuns.

O pipeline híbrido executa busca semântica e BM25 em paralelo e combina os resultados:

- Semântica ajuda com significado e contexto.
- BM25 garante correspondência exata de IDs, termos técnicos e frases específicas.

## 9. Recursos avançados do Claude

### Extended thinking

**Extended thinking** é o modo de raciocínio estendido: antes da resposta final, o modelo produz um bloco de raciocínio.

Benefícios:

- Melhor precisão em problemas complexos.
- Transparência do processo de raciocínio.

Custos:

- Mais tokens consumidos.
- Maior latência.
- Resposta mais complexa de tratar no código.

Pontos técnicos:

- A resposta contém um bloco de *thinking* e um bloco de texto final.
- Existe um `budget_tokens` mínimo de 1024 para o raciocínio.
- `max_tokens` deve ser maior que o orçamento de raciocínio.
- Há incompatibilidades, inclusive com prefill de mensagem do assistente.
- O recurso restringe o uso de temperatura.

O raciocínio pode vir **redigido** (*redacted*) quando sistemas de segurança sinalizam o conteúdo. Mesmo oculto, ele pode ser repassado em conversas futuras sem perder contexto.

A recomendação é: otimize primeiro o prompt e use avaliações; só ative extended thinking se a precisão continuar insuficiente.

### Imagens

Claude aceita imagens em base64 ou URL, junto com blocos de texto.

Limites citados:

- Até 100 imagens por requisição.
- 5 MB por imagem.
- 8000 px para uma imagem única.
- 2000 px ao enviar várias imagens.

Consumo estimado de tokens:

\[
tokens = rac{largura 	imes altura}{750}
\]

Para análise visual confiável:

- Use instruções passo a passo.
- Defina critérios claros.
- Use exemplos.
- Divida tarefas complexas em etapas menores.

### PDFs

Claude pode receber PDFs diretamente como blocos do tipo `document`, com `media_type: application/pdf`.

Ele consegue analisar:

- Texto.
- Imagens.
- Gráficos.
- Tabelas.
- Estrutura do documento.

### Citações

Com `citations: { "enabled": true }`, o modelo pode indicar a origem exata das informações usadas.

Para PDFs, a citação inclui:

- Texto citado.
- Índice do documento.
- Título do documento.
- Páginas inicial e final.

Para texto simples, inclui posições de caracteres.

Isso torna a resposta verificável: o usuário pode conferir a fonte original.

### Prompt caching

**Prompt caching** reutiliza o processamento de conteúdo repetido entre requisições, reduzindo latência e custo.

Regras principais:

- O cache não é automático: é preciso marcar **cache breakpoints** com `cache_control: { "type": "ephemeral" }`.
- O cache guarda tudo até o ponto de interrupção, inclusive.
- O conteúdo até o breakpoint precisa ser idêntico.
- Qualquer alteração invalida o cache.
- Duração padrão: 5 minutos.
- Cada reutilização renova o timer.
- Há opção de TTL de 1 hora, com custo maior de gravação.
- É possível criar até 4 breakpoints.
- O conteúdo mínimo para cache é 1024 tokens no total.

System prompts e definições de ferramentas são bons candidatos a cache porque raramente mudam.

A ordem interna de processamento é:

1. Ferramentas.
2. System prompt.
3. Mensagens.

### Execução de código e Files API

O módulo também lista **code execution** e **Files API**, mas a transcrição disponível não traz o detalhamento completo dessas seções.

## 10. Model Context Protocol (MCP)

### O que é MCP

O **Model Context Protocol** é uma camada padrão para conectar aplicações de IA a ferramentas, dados e prompts externos.

Em vez de implementar cada integração manualmente no seu servidor, você usa servidores MCP que já expõem essas capacidades.

A arquitetura tem três papéis:

- **Cliente MCP:** fica na sua aplicação e conversa com servidores MCP.
- **Servidor MCP:** expõe ferramentas, recursos e prompts.
- **Serviço externo:** GitHub, banco de dados, sistema de arquivos, API etc.

A diferença em relação a tool use comum:

- No tool use tradicional, você define e mantém os esquemas e funções.
- No MCP, um servidor especializado já fornece essas definições e execuções.

### Comunicação

O MCP é agnóstico quanto ao transporte:

- stdio local.
- HTTP.
- WebSockets.
- Outros protocolos de rede.

Mensagens centrais:

- `ListToolsRequest` / `ListToolsResult`: descobrir ferramentas disponíveis.
- `CallToolRequest` / `CallToolResult`: executar uma ferramenta com argumentos e receber o resultado.

Fluxo típico:

1. Usuário pergunta.
2. Aplicativo lista ferramentas.
3. Envia pergunta + ferramentas ao Claude.
4. Claude solicita uma ferramenta.
5. Cliente MCP chama o servidor MCP.
6. Servidor acessa o serviço externo.
7. Resultado volta ao Claude.
8. Claude responde ao usuário.

### Ferramentas, recursos e prompts

| Componente | Função | Exemplo |
|---|---|---|
| Tools | Executar ações | Ler ou editar documento |
| Resources | Expor dados para leitura | Listar documentos; obter conteúdo por ID |
| Prompts | Fornecer templates de prompt | Template para análise de documento |

No exemplo do curso, o servidor mantém documentos em memória e expõe:

- `read_doc_contents`: lê um documento por ID.
- `edit_document`: substitui um texto exato por outro.

O SDK gera automaticamente o schema JSON a partir de type hints e decoradores, reduzindo código repetitivo.

Erros como ID inexistente devem ser lançados com mensagens descritivas para o modelo entender.

### Recursos MCP

Recursos funcionam como leitura de dados, semelhante a `GET`:

- **Recurso direto:** URI fixo, como `docs://documents`.
- **Recurso com template:** URI parametrizado, como `docs://documents/{doc_id}`.

O cliente envia `ReadResourceRequest` com um URI e recebe os dados.

O `mime_type` informa o formato:

- `application/json`
- `text/plain`
- Outros tipos MIME válidos

### Desenvolvimento e testes

O **MCP Inspector** permite testar servidores sem integrá-los imediatamente a um app completo:

- Listar ferramentas.
- Executar chamadas.
- Testar recursos.
- Depurar isoladamente.

Prática de depuração recomendada:

- Ao adicionar uma nova camada — ferramenta, fonte de dados ou servidor MCP — faça primeiro uma pergunta cuja resposta você já conhece.
- Isso cria uma linha de base verificável antes de construir a próxima parte.

## 11. Claude Code e Computer Use

### Claude Code

**Claude Code** é um assistente de programação agêntico no terminal.

Ele pode:

- Ler, buscar e editar arquivos.
- Executar comandos.
- Acessar a web.
- Conectar servidores MCP.

Comandos importantes:

- `/init`: analisa o projeto e cria `CLAUDE.md`.
- `/clear`: limpa histórico e contexto da conversa.
- `#`: adiciona uma nota ao `CLAUDE.md`.

O `CLAUDE.md` funciona como contexto persistente do projeto, podendo existir em escopos:

- Projeto.
- Local.
- Usuário.

Ele pode conter:

- Comandos de build.
- Convenções de código.
- Padrões do repositório.

Fluxo recomendado:

1. Dar contexto: pedir que o Claude leia arquivos relevantes.
2. Pedir um plano, sem implementar ainda.
3. Somente depois pedir a implementação.

Variação eficaz: desenvolvimento orientado a testes.

1. Pedir casos de teste.
2. Implementar os testes.
3. Gerar código até passá-los.

Isso dá ao modelo critérios objetivos de sucesso.

### Computer Use

**Computer Use** é um conjunto de ferramentas que permite ao Claude interagir com um computador:

- Navegar na web.
- Usar aplicativos desktop.
- Executar tarefas que dependem de interface visual.

Ele é apresentado como exemplo de agente porque exige:

- Uso de ferramentas.
- Tarefas em várias etapas.
- Observação do ambiente.
- Resolução autônoma de problemas.

### MCP no Claude Code

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

Depois, o Claude Code pode usar as ferramentas expostas por ele.

## 12. Agentes e fluxos de trabalho

### Workflow vs. agente

| Abordagem | Definição | Quando usar | Vantagens | Desvantagens |
|---|---|---|---|---|
| **Workflow** | Sequência pré-definida de chamadas ao modelo | Você conhece as etapas exatas | Previsível, testável, mais preciso por etapa | Pouco flexível |
| **Agente** | Modelo recebe objetivo + ferramentas e decide o caminho | Tarefas variáveis ou imprevisíveis | Flexível, adaptável, lida com casos não previstos | Menos previsível, mais difícil/caro de controlar |

Um agente não é apenas “um LLM com ferramentas”: é um sistema em que o modelo planeja e combina ferramentas para atingir um objetivo.

### Padrão avaliador-otimizador

No exemplo de imagem → modelo CAD:

1. Descrever a imagem.
2. Gerar o modelo 3D.
3. Criar renderização.
4. Comparar renderização com a imagem original.
5. Corrigir e repetir até aprovar.

Esse é o padrão **avaliador-otimizador**:

- Um produtor gera saída.
- Um avaliador verifica critérios.
- Se necessário, devolve feedback para nova iteração.

### Paralelização

Na paralelização, uma tarefa complexa é dividida em subtarefas independentes executadas ao mesmo tempo; depois, um passo final agrega os resultados.

Exemplo do curso: escolha de material.

Em vez de um único prompt analisando metal, polímero, cerâmica e compósito juntos, cada material é avaliado em uma chamada separada com critérios próprios. Os resultados são então combinados em uma recomendação final.

Benefícios:

- Foco em um critério por chamada.
- Facilidade para melhorar um prompt sem afetar os outros.
- Escalabilidade ao adicionar novas opções.
- Menor carga cognitiva por requisição.

### Encadeamento

**Chaining** divide uma tarefa em etapas sequenciais, onde a saída de uma etapa alimenta a seguinte.

É útil quando uma instrução única longa faz o modelo ignorar restrições.

Exemplo:

1. Gerar um artigo.
2. Enviar esse artigo para uma segunda chamada focada exclusivamente em revisão.
3. Remover emojis, menções de autoria por IA e linguagem inadequada.

A revisão separada tende a funcionar melhor do que exigir tudo em um prompt só.

### Roteamento

No **routing**, uma primeira chamada classifica a entrada e o sistema direciona ela para um pipeline especializado.

Cada categoria pode ter:

- Prompt próprio.
- Ferramentas próprias.
- Fluxo próprio.

Exemplo: classificar um tópico como:

- Educacional.
- Entretenimento.
- Comédia.
- Vlog pessoal.
- Review.
- Storytelling.

Depois, usar o template de prompt correspondente.

É indicado quando:

- Há tipos distintos de solicitação.
- As categorias são bem definidas.
- A classificação é confiável.
- O ganho de especialização compensa a chamada extra de roteamento.

### Ferramentas para agentes

Agentes funcionam melhor com ferramentas **abstratas e combináveis**, não hiperespecializadas.

O Claude Code usa ferramentas genéricas como:

- Ler arquivo.
- Escrever arquivo.
- Editar arquivo.
- Buscar arquivos.
- Executar comandos.
- Pesquisar conteúdo.

O modelo combina essas primitivas para realizar tarefas complexas.

Com ferramentas simples de data e hora, ele pode encadeá-las:

1. Obter data atual.
2. Somar duração.
3. Criar lembrete.

Ele também pode reconhecer informação faltante e perguntar ao usuário antes de agir.

### Inspeção ambiental

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

Princípio central:

**Ler antes de escrever.**

Sempre permitir que o agente verifique se a ação teve o efeito esperado.

## Pontos transversais do curso

- **Prompts são software:** devem ter objetivo, testes, métricas e iteração.
- **Contexto é responsabilidade do aplicativo:** a API não mantém memória entre chamadas.
- **Ferramentas ampliam o modelo, mas o backend executa e controla as ações.**
- **RAG reduz custo e latência ao enviar apenas contexto relevante.**
- **Cache só ajuda quando há conteúdo grande e repetido.**
- **MCP padroniza a integração com ferramentas e fontes de dados.**
- **Prefira workflows quando o processo é conhecido; use agentes quando a flexibilidade for necessária.**
