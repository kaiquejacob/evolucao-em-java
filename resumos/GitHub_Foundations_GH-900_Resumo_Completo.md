# GitHub Foundations (GH-900) — Resumo completo para o exame

Este material cobre os sete domínios oficiais do exame **GH-900: GitHub Foundations**, com foco em **o que cada recurso é** e **para que serve**. Ao final de cada domínio há 5 questões de múltipla escolha no nível do exame, com gabarito comentado.

**Formato do exame:** 100 minutos, nota mínima de 700 em escala de 1.000, questões de escolha única, múltipla escolha e cenários.

---

# 1. Git e GitHub básico

## Controle de versão

**O que é:** registro histórico das alterações feitas em arquivos de um projeto.

**Para que serve:** permitir recuperar versões anteriores, entender quem alterou o quê, trabalhar em paralelo e reverter erros sem perder o histórico.

Benefícios:

- Histórico completo de mudanças.
- Rastreabilidade de alterações.
- Trabalho simultâneo de várias pessoas.
- Possibilidade de voltar a uma versão anterior.
- Base para revisão de código e automação.

## Controle de versão centralizado x distribuído

| Modelo | O que é | Onde fica o histórico |
|---|---|---|
| Centralizado (CVCS) | Depende de um servidor central para armazenar código e histórico | No servidor central |
| Distribuído (DVCS) | Cada pessoa tem uma cópia completa do repositório, incluindo histórico | Em cada cópia local |

**Git é um DVCS.** Ele permite trabalhar localmente e sincronizar com repositórios remotos depois.

## Git

**O que é:** sistema de controle de versão distribuído.

**Para que serve:** versionar arquivos localmente, criar branches, registrar commits e sincronizar com repositórios remotos.

Características:

- Funciona no computador do desenvolvedor.
- Cada pessoa pode ter o repositório completo localmente.
- Permite trabalhar offline.
- Usa commits, branches, merges e remotes.

## GitHub

**O que é:** plataforma de hospedagem e colaboração baseada em Git.

**Para que serve:** armazenar repositórios remotamente e oferecer recursos como pull requests, issues, Actions, segurança, projetos e documentação.

| Git | GitHub |
|---|---|
| Ferramenta de controle de versão | Plataforma de colaboração |
| Roda localmente | Roda na nuvem |
| Registra commits e branches | Hospeda repositórios e adiciona recursos colaborativos |
| Exemplo: `git commit` | Exemplo: abrir pull request |

## Repositório

**O que é:** contêiner central de um projeto.

**Para que serve:** guardar arquivos, histórico, branches, issues, pull requests, configurações e documentação.

Tipos de repositório:

- **Local:** no computador.
- **Remoto:** hospedado no GitHub.
- **Público:** visível para qualquer pessoa.
- **Privado:** visível apenas para pessoas autorizadas.
- **Interno:** visível para membros da mesma organização/empresa.

## Remote

**O que é:** referência a outro repositório que contém o mesmo projeto, mas está em outro lugar.

**Para que serve:** sincronizar o repositório local com o repositório remoto.

O remote padrão geralmente se chama `origin`.

## Commit

**O que é:** registro permanente de alterações no histórico do repositório.

**Para que serve:** documentar o que mudou, quando mudou, quem mudou e permitir retornar àquele ponto.

Um commit contém:

- Alterações dos arquivos.
- Mensagem descritiva.
- Autor e data.
- Identificador único chamado SHA.

## Branch

**O que é:** linha do tempo separada de commits.

**Para que serve:** desenvolver uma funcionalidade, corrigir um bug ou testar uma ideia sem alterar diretamente a branch principal.

Conceitos:

- **Branch padrão:** geralmente `main`.
- **Merge:** integra alterações de uma branch em outra.
- **Conflito:** ocorre quando o Git não consegue combinar alterações automaticamente.
- **Remote:** referência a um repositório remoto.

## GitHub Flow

**O que é:** fluxo simples de trabalho baseado em branches e pull requests.

**Para que serve:** organizar contribuições, permitir revisão e evitar alterações diretas na branch principal.

Fluxo:

1. Criar branch.
2. Fazer commits.
3. Abrir pull request.
4. Revisar e executar verificações.
5. Fazer merge.
6. Excluir a branch, se desejado.

## Contas no GitHub

| Tipo | O que é | Para que serve |
|---|---|---|
| Conta pessoal | Conta individual | Projetos próprios e contribuições pessoais |
| Organização | Conta compartilhada | Reunir pessoas, equipes, repositórios e permissões |
| Enterprise | Solução corporativa | Gerenciar várias organizações, usuários, políticas e segurança |

## Equipes

**O que é:** grupo de membros dentro de uma organização.

**Para que serve:** refletir a estrutura da empresa, conceder permissões em cascata e facilitar notificações por menções, como `@time-backend`.

## Markdown

**O que é:** linguagem de marcação leve usada para formatar texto.

**Para que serve:** escrever README, issues, pull requests, comentários, wikis e documentação.

Regras importantes:

- `#` cria títulos.
- `**texto**` cria negrito.
- `*texto*` cria itálico.
- Podem ser combinados: `*Hello, **world**!*`
- `\*` e `\_` exibem caracteres reservados literalmente.
- Markdown aceita HTML embutido quando necessário.
- O comando `/code` insere bloco de código em issues, PRs e comentários.

## GitHub Desktop e Mobile

| Ferramenta | O que é | Para que serve |
|---|---|---|
| GitHub Desktop | Aplicativo gráfico para Git e GitHub | Clonar, commitar, criar branches e sincronizar sem terminal |
| GitHub Mobile | Aplicativo móvel do GitHub | Receber notificações, revisar PRs e responder issues |

## Questões — Domínio 1

**1.** Qual é a principal diferença entre controle de versão centralizado e distribuído?  
A. O centralizado não usa servidor.  
B. O distribuído exige que todos trabalhem no mesmo servidor.  
C. O centralizado depende de um servidor central; o distribuído permite cópias locais completas do repositório.  
D. O Git é centralizado e o GitHub é distribuído.  

**2.** O que é um remote no Git?  
A. Uma cópia local do repositório.  
B. Uma referência a outro repositório do mesmo projeto, localizado em outro lugar.  
C. Um arquivo de configuração do GitHub.  
D. Uma branch protegida.  

**3.** Qual comando fornece informações sobre como usar o Git?  
A. `git init`  
B. `git status`  
C. `git help`  
D. `git clone`  

**4.** Qual é a função de uma branch?  
A. Publicar o site do projeto.  
B. Criar uma linha do tempo separada de commits para uma tarefa ou funcionalidade.  
C. Definir revisores obrigatórios.  
D. Enviar commits ao repositório remoto.  

**5.** Qual sequência representa o GitHub Flow?  
A. Criar branch → commits → pull request → revisão → merge.  
B. Merge → branch → commit → issue.  
C. Fork → excluir branch → push → merge.  
D. Criar issue → publicar Pages → criar tag → merge.  

**Gabarito:** 1-C; 2-B; 3-C; 4-B; 5-A.

**Comentários:** Git é DVCS; remote é referência a outro repositório; `git help` exibe ajuda; branch isola trabalho; GitHub Flow é baseado em branch, PR, revisão e merge.

---

# 2. Trabalhar com repositórios GitHub

## README

**O que é:** arquivo principal de documentação do repositório.

**Para que serve:** explicar o propósito do projeto, como instalá-lo, executá-lo, testá-lo e contribuir.

## LICENSE

**O que é:** arquivo que define a licença do projeto.

**Para que serve:** estabelecer como outras pessoas podem usar, modificar, distribuir e contribuir com o código.

## CONTRIBUTING.md

**O que é:** guia de contribuição.

**Para que serve:** orientar como abrir issues, criar branches, escrever commits, rodar testes e enviar pull requests.

## CODEOWNERS

**O que é:** arquivo que define responsáveis por arquivos ou diretórios.

**Para que serve:** solicitar automaticamente revisão das pessoas ou equipes responsáveis quando determinadas áreas forem alteradas.

Exemplo:

```text
/src/backend/ @time-backend
/docs/ @time-documentacao
```

Com revisões obrigatórias, impede que PRs de determinadas áreas sejam mesclados sem aprovação dos responsáveis.

## SECURITY.md

**O que é:** política de segurança do repositório.

**Para que serve:** informar como reportar vulnerabilidades de forma responsável.

## `.gitignore`

**O que é:** arquivo que lista arquivos e pastas que não devem ser incluídos em commits.

**Para que serve:** evitar o envio acidental de segredos, arquivos temporários, dependências e configurações locais.

Exemplo:

```gitignore
.env
.secrets/
node_modules/
target/
```

Importante: `.gitignore` é aplicado pelo cliente e não impede absolutamente o envio manual forçado de arquivos.

## Repository template

**O que é:** modelo de repositório reutilizável.

**Para que serve:** criar novos repositórios já com estrutura padrão, como README, licença, pastas e configurações iniciais.

## Insights

**O que é:** área de métricas e informações sobre o repositório.

**Para que serve:** acompanhar atividade, contribuições, tráfego, dependências e saúde do projeto.

Recursos:

- Repository insights.
- Dependency insights.
- Metrics dashboards.
- Feature previews.

## Stars, forks e clones

| Recurso | O que é | Para que serve |
|---|---|---|
| Star | Marcador de interesse | Indicar que você gostou ou acompanha um projeto |
| Fork | Cópia do repositório na sua conta | Contribuir sem alterar o original |
| Clone | Cópia local do repositório | Trabalhar no projeto no seu computador |

## Questões — Domínio 2

**1.** Qual arquivo informa como reportar vulnerabilidades de segurança?  
A. `README.md`  
B. `LICENSE`  
C. `SECURITY.md`  
D. `CODEOWNERS`  

**2.** Qual arquivo define revisores responsáveis por diretórios específicos?  
A. `CONTRIBUTING.md`  
B. `CODEOWNERS`  
C. Repository template  
D. Dependency insights  

**3.** Qual é a função de um repository template?  
A. Proteger a branch `main`.  
B. Criar novos repositórios com estrutura inicial padronizada.  
C. Publicar um site estático.  
D. Fechar issues automaticamente.  

**4.** Qual recurso indica interesse em um repositório?  
A. Watch  
B. Fork  
C. Star  
D. Clone  

**5.** Qual recurso ajuda a entender riscos nas bibliotecas usadas pelo projeto?  
A. GitHub Pages  
B. Dependency insights  
C. GitHub Sponsors  
D. Saved replies  

**Gabarito:** 1-C; 2-B; 3-B; 4-C; 5-B.

**Comentários:** `SECURITY.md` orienta reporte de vulnerabilidades; `CODEOWNERS` define revisores; templates padronizam repositórios; star indica interesse; dependency insights tratam de dependências.

---

# 3. Colaborar usando GitHub

## Issue

**O que é:** registro de um trabalho que precisa ser feito.

**Para que serve:** relatar bugs, sugerir melhorias, pedir funcionalidades e acompanhar tarefas.

Recursos:

- Título e descrição.
- Labels.
- Assignees.
- Milestones.
- Comentários.
- Templates e formulários.
- Filtros e busca.

## Discussion

**O que é:** espaço de conversa dentro de um repositório.

**Para que serve:** dúvidas, ideias, anúncios, votações e troca de conhecimento.

| Issue | Discussion |
|---|---|
| Trabalho a ser feito | Conversa ou troca de ideias |
| Bug, tarefa ou melhoria | Dúvida, anúncio ou proposta aberta |
| Pode ser fechada como concluída | Pode ser marcada como respondida |

## Pull request

**O que é:** proposta de integrar alterações de uma branch em outra.

**Para que serve:** permitir revisão de código, discussão, testes automatizados e merge controlado.

Abas principais:

| Aba | Para que serve |
|---|---|
| Conversation | Discussão, comentários e revisões |
| Commits | Histórico de commits do PR |
| Files changed | Alterações propostas |
| Checks | Resultado de verificações automatizadas |

## Base e compare

Em um pull request:

- **Base branch:** branch de destino, onde as alterações serão mescladas — normalmente `main`.
- **Compare branch:** branch de origem, que contém as alterações propostas.

**Para que serve a comparação:** mostrar as diferenças entre as duas branches.

Não existe “branch vencedora”.

## Vínculo entre issue e pull request

**O que é:** conexão entre um PR e uma issue.

**Para que serve:** dar contexto e, em alguns casos, fechar automaticamente a issue quando o PR for mesclado.

Palavras-chave:

```text
Closes #12
Fixes #12
Resolves #12
```

Também é possível mencionar o número do PR na descrição ou em comentários da issue.

## Issue templates e forms

**O que é:** estruturas predefinidas para criação de issues.

**Para que serve:** garantir que relatórios de bug e solicitações contenham informações consistentes, como passos de reprodução, ambiente e logs.

## Milestone

**O que é:** marco que agrupa issues e pull requests em torno de um objetivo.

**Para que serve:** acompanhar o progresso de uma versão, sprint ou entrega.

Para criar: aba **Issues** → **Milestones** → **New milestone**.

## Filtros e busca

**O que é:** forma de refinar listas de issues e pull requests.

**Para que serve:** encontrar rapidamente itens específicos.

Exemplo:

```text
is:issue is:open label:bug
```

## Notificações

**O que é:** alertas sobre atividades relevantes.

**Para que serve:** informar menções, atribuições, revisões solicitadas, comentários e atualizações de PRs.

## Gist

**O que é:** repositório simples para snippets, arquivos pequenos e texto em Markdown.

**Para que serve:** compartilhar trechos de código, scripts, notas ou exemplos.

Importante: Gists não têm exatamente os mesmos controles de acesso e recursos de um repositório regular.

## Wiki

**O que é:** espaço de documentação colaborativa separado do código versionado.

**Para que serve:** criar guias, tutoriais, documentação de arquitetura e informações extensas.

## GitHub Pages

**O que é:** serviço de hospedagem de sites estáticos.

**Para que serve:** publicar portfólios, documentação e sites simples diretamente de um repositório.

## Fork

**O que é:** cópia de um repositório na conta de outra pessoa.

**Para que serve:** permitir contribuições sem alterar diretamente o repositório original.

Fluxo de contribuição externa:

1. Fazer fork.
2. Clonar o fork.
3. Criar branch.
4. Fazer commits.
5. Enviar para o fork.
6. Abrir pull request para o repositório original.

## Revisão de pull request

**O que é:** processo de analisar alterações antes do merge.

**Para que serve:** garantir qualidade, segurança, consistência e entendimento do código.

Boa prática:

- Corrigir ou sugerir correções para erros óbvios.
- Comentar pontos que exigem discussão.
- Explicar o motivo das solicitações.
- Concluir com aprovação ou solicitação de alterações.

## Questões — Domínio 3

**1.** Qual recurso é usado para relatar bugs, sugerir alterações e acompanhar tarefas?  
A. Discussion  
B. Issue  
C. Gist  
D. Wiki  

**2.** Em um pull request, o que representa a branch base?  
A. A branch de origem das alterações.  
B. A branch de destino onde as alterações serão mescladas.  
C. A branch que será excluída.  
D. A branch que vence o conflito.  

**3.** Qual palavra-chave fecha automaticamente a issue #42 quando o PR é mesclado?  
A. `Links #42`  
B. `Closes #42`  
C. `Watches #42`  
D. `Follows #42`  

**4.** Qual recurso é indicado para documentação colaborativa extensa?  
A. Gist  
B. Wiki  
C. Issue  
D. Milestone  

**5.** Qual é a melhor prática ao revisar um PR com erros pequenos e pontos que exigem discussão?  
A. Rejeitar sem comentários.  
B. Fazer merge imediatamente.  
C. Iniciar revisão, sugerir correções óbvias, comentar pontos relevantes e concluir solicitando alterações.  
D. Excluir a branch do autor.  

**Gabarito:** 1-B; 2-B; 3-B; 4-B; 5-C.

**Comentários:** issues registram trabalho; base é destino e compare é origem; `Closes` fecha issue após merge; wiki é documentação extensa; revisão deve ser colaborativa.

---

# 4. Práticas modernas de desenvolvimento

## GitHub Actions

**O que é:** plataforma de automação do GitHub.

**Para que serve:** executar tarefas automaticamente em resposta a eventos do repositório.

Casos de uso:

- Executar testes.
- Fazer build.
- Publicar artefatos.
- Verificar qualidade de código.
- Automatizar tarefas administrativas.
- Implementar CI/CD.
- Gerenciar infraestrutura como código.

Conceitos:

| Conceito | O que é |
|---|---|
| Workflow | Processo automatizado definido em YAML |
| Event | Gatilho, como `push` ou `pull_request` |
| Job | Conjunto de etapas executado em um runner |
| Step | Comando ou ação individual |
| Runner | Máquina que executa o job |
| Action | Componente reutilizável de automação |

## Gatilhos do Actions

Um workflow é disparado por eventos declarados em `on:`.

Exemplos válidos:

- Push em uma branch.
- Abertura de pull request.
- Merge de pull request.
- Criação de issue.
- Comentário em issue.
- Agendamento.

**Não é gatilho válido:** um usuário apenas visitar uma página específica do repositório.

## Tipos de Actions personalizadas

| Tipo | Característica |
|---|---|
| Container action | O ambiente faz parte do código da action |
| JavaScript action | O código roda diretamente no runner |
| Composite action | Combina várias etapas em uma action reutilizável |

## GitHub Copilot

**O que é:** assistente de programação com IA.

**Para que serve:** sugerir código, explicar trechos, ajudar em edições e acelerar o desenvolvimento.

Recursos:

- Sugestões enquanto você digita.
- Chat no IDE.
- Explicações de código.
- Edição baseada em prompt.
- Agentes e Agent Mode.
- Suporte a múltiplos modelos.

Para solicitar sugestão, uma forma comum é digitar um comentário ou texto, aguardar a sugestão e pressionar **Tab** para aceitar.

## IDEs compatíveis com Copilot

O Copilot tem suporte em vários ambientes, incluindo:

- Visual Studio Code
- Visual Studio
- Xcode
- IDEs JetBrains
- Neovim
- Eclipse
- Spyder

## GitHub Codespaces

**O que é:** ambiente de desenvolvimento completo na nuvem.

**Para que serve:** programar sem configurar manualmente o ambiente local, usando navegador ou VS Code.

Características:

- Criado a partir de um repositório ou branch.
- Personalizável com `.devcontainer`.
- Repositório clonado fica em `/workspaces`.
- Pode ser criado em número ilimitado, conforme armazenamento disponível.
- Exige internet para acesso e edição.
- Ciclo de vida manual: começa na criação e termina na exclusão.

## github.dev

**O que é:** editor de código no navegador.

**Para que serve:** fazer edições rápidas, ler código e realizar pequenas alterações sem configurar um ambiente completo.

| github.dev | Codespaces |
|---|---|
| Editor rápido no navegador | Ambiente completo na nuvem |
| Não é focado em execução | Permite executar e testar projetos |
| Uso leve | Personalizável com dev containers |

## Dev containers

**O que é:** configuração que define o ambiente de desenvolvimento de um Codespace.

**Para que serve:** garantir que todas as pessoas usem as mesmas ferramentas, extensões e dependências.

## Questões — Domínio 4

**1.** Qual evento pode disparar um workflow do GitHub Actions quando um pull request é aberto?  
A. `pull_request`  
B. `github_pages`  
C. `sponsorship`  
D. `wiki_update`  

**2.** Qual é a diferença entre container action e JavaScript action?  
A. Container action inclui o ambiente no código; JavaScript action roda diretamente no runner.  
B. JavaScript action exige Docker obrigatoriamente.  
C. Container action não pode usar dependências.  
D. Ambas exigem GitHub Enterprise.  

**3.** Qual recurso oferece ambiente de desenvolvimento completo na nuvem com dev containers?  
A. github.dev  
B. GitHub Codespaces  
C. GitHub Pages  
D. GitHub Sponsors  

**4.** Em qual diretório o repositório clonado fica após criar um Codespace?  
A. `/home`  
B. `/workspaces`  
C. `/github`  
D. `/repositories`  

**5.** O que acontece se a internet cair durante o uso de um Codespace?  
A. O Codespace é excluído imediatamente.  
B. O acesso de edição é perdido até a conexão ser restabelecida.  
C. Os arquivos são automaticamente enviados por push.  
D. O repositório é convertido em privado.  

**Gabarito:** 1-A; 2-A; 3-B; 4-B; 5-B.

**Comentários:** `pull_request` é evento válido; container action inclui ambiente; Codespaces é ambiente completo; o repositório fica em `/workspaces`; sem internet, perde-se acesso de edição.

---

# 5. Gerenciar projetos com GitHub

## GitHub Projects

**O que é:** ferramenta de planejamento e acompanhamento de trabalho.

**Para que serve:** organizar issues, pull requests e rascunhos em visualizações de tabela, quadro e linha do tempo.

## Layouts

| Layout | O que é | Para que serve |
|---|---|---|
| Table | Visualização em tabela | Filtrar, editar e visualizar campos |
| Board | Quadro com colunas | Acompanhar fluxo de trabalho |
| Roadmap | Linha do tempo | Planejar entregas por período |

## Campos

| Campo | O que é | Para que serve |
|---|---|---|
| Text | Campo de texto livre | Descrições ou observações |
| Number | Campo numérico | Pontos, estimativas ou métricas |
| Date | Campo de data | Prazos e datas |
| Single select | Lista de opções fixas | Prioridade Alta, Média e Baixa |
| Iteration | Fases ou blocos de tempo recorrentes | Sprints, semanas ou ciclos de entrega |

## Labels

**O que é:** etiqueta usada para categorizar issues e pull requests.

**Para que serve:** identificar tipo, prioridade, área ou status.

## Milestone

**O que é:** agrupamento de issues e pull requests em torno de um objetivo.

**Para que serve:** acompanhar o progresso de uma entrega, versão ou fase.

## Assignee

**O que é:** pessoa responsável por uma issue ou pull request.

**Para que serve:** definir quem deve executar ou acompanhar o trabalho.

## Saved replies

**O que é:** resposta pronta reutilizável.

**Para que serve:** acelerar comentários repetitivos em issues e pull requests.

## Built-in workflows

**O que é:** automação nativa do GitHub Projects.

**Para que serve:** executar ações simples automaticamente, como mover item para “Done” quando uma issue é fechada.

## Project insights

**O que é:** painel de acompanhamento do projeto.

**Para que serve:** visualizar progresso, status e produtividade.

## Danger zone

**O que é:** seção de configurações críticas do Project.

**Para que serve:** alterar visibilidade, fechar ou excluir o projeto.

## Questões — Domínio 5

**1.** Qual layout do GitHub Projects é mais adequado para visualizar trabalho em colunas?  
A. Table  
B. Board  
C. Roadmap  
D. Wiki  

**2.** Qual campo permite criar fases sequenciais ou blocos de tempo recorrentes?  
A. Single select  
B. Iteration  
C. Text  
D. Number  

**3.** Qual campo você usaria para criar prioridades Alta, Média e Baixa?  
A. Iteration  
B. Single select  
C. Milestone  
D. Label  

**4.** Qual é a maneira mais simples de adicionar automação a um Project?  
A. Criar um novo repositório.  
B. Usar a automação interna do Project.  
C. Criar um Gist.  
D. Publicar no GitHub Pages.  

**5.** Em qual seção é possível alterar visibilidade, fechar ou excluir um Project?  
A. Insights  
B. Danger zone  
C. Notifications  
D. Marketplace  

**Gabarito:** 1-B; 2-B; 3-B; 4-B; 5-B.

**Comentários:** Board usa colunas; Iteration cria fases; Single select cria prioridades; automação interna é a forma mais simples; Danger zone concentra ações críticas.

---

# 6. Privacidade, segurança e administração

## 2FA

**O que é:** autenticação de dois fatores.

**Para que serve:** exigir um segundo fator além da senha, reduzindo o risco de acesso não autorizado.

## Passkeys

**O que é:** credencial de autenticação baseada em criptografia.

**Para que serve:** substituir ou complementar senhas com um método mais resistente a phishing.

## SAML SSO

**O que é:** autenticação única baseada no padrão SAML.

**Para que serve:** validar a identidade do usuário em um provedor de identidade corporativo.

Fluxo:

1. Usuário tenta acessar recursos da organização.
2. GitHub redireciona para o IdP.
3. O IdP valida a identidade.
4. O usuário retorna autenticado ao GitHub.

## Team synchronization

**O que é:** sincronização entre grupos do provedor de identidade e equipes do GitHub.

**Para que serve:** manter automaticamente a composição das equipes conforme grupos corporativos.

## Enterprise Managed Users

**O que é:** modelo de gestão de usuários controlado pela empresa.

**Para que serve:** centralizar identidades, restringir acesso público e administrar contas por meio do provedor corporativo.

## Permissões de repositório

| Permissão | O que permite |
|---|---|
| Read | Visualizar o repositório |
| Triage | Gerenciar issues e PRs |
| Write | Criar branches, fazer push e abrir PRs |
| Maintain | Gerenciar configurações sem acesso administrativo total |
| Admin | Controle administrativo completo |

## Permissões de organização

| Papel | Para que serve |
|---|---|
| Owner | Administração completa da organização |
| Member | Participação como membro da organização |
| Team maintainer | Gerenciar membros e configurações de uma equipe |

## Branch protection

**O que é:** conjunto de regras que protege branches importantes.

**Para que serve:** impedir merges não revisados ou não testados na branch principal.

Pode exigir:

- Pull request antes do merge.
- Aprovações de revisão.
- Checks aprovados.
- Conversas resolvidas.
- Restrições de push.

## Segurança de repositório

A guia **Security** reúne recursos como:

| Recurso | O que é | Para que serve |
|---|---|---|
| Security policy | Política de segurança | Orientar como reportar vulnerabilidades |
| Security advisories | Avisos de segurança | Documentar e comunicar vulnerabilidades |
| Dependabot alerts | Alertas de dependências vulneráveis | Avisar sobre pacotes inseguros |
| Code scanning | Análise automatizada de código | Detectar vulnerabilidades e erros no código |
| Secret scanning | Detecção de segredos expostos | Identificar chaves, tokens e senhas em código |

## Dependabot

**O que é:** ferramenta que monitora dependências do projeto.

**Para que serve:** identificar dependências vulneráveis, alertar sobre riscos e, quando configurado, criar PRs de atualização de segurança.

## Secret scanning

**O que é:** recurso que detecta segredos expostos no código.

**Para que serve:** alertar sobre exposição de chaves de API, tokens e credenciais.

Os alertas abertos ficam na aba **Security** do repositório.

## Questões — Domínio 6

**1.** Qual recurso protege a conta mesmo se a senha for comprometida?  
A. 2FA ou passkeys  
B. GitHub Pages  
C. Repository template  
D. Star  

**2.** Qual configuração pode exigir aprovação e checks antes do merge em `main`?  
A. Branch protection rules  
B. GitHub Sponsors  
C. Saved replies  
D. Gist  

**3.** Onde o usuário se autentica após habilitar SAML SSO?  
A. No GitHub diretamente.  
B. No IdP, o provedor de identidade.  
C. No GitHub Pages.  
D. No GitHub Marketplace.  

**4.** Qual permissão permite adicionar e remover membros de uma equipe?  
A. Read  
B. Write  
C. Team maintainer  
D. Triage  

**5.** Onde ficam os alertas abertos de segredos expostos?  
A. Na aba Security do repositório.  
B. Na aba Insights.  
C. No GitHub Marketplace.  
D. No perfil do usuário.  

**Gabarito:** 1-A; 2-A; 3-B; 4-C; 5-A.

**Comentários:** 2FA e passkeys protegem identidade; branch protection controla merges; SAML SSO autentica no IdP; team maintainer gerencia membros; alertas de segredos ficam na aba Security.

---

# 7. Comunidade GitHub

## Open source

**O que é:** software com código publicamente disponível.

**Para que serve:** permitir que qualquer pessoa veja, use, estude, modifique e distribua o código conforme a licença.

## InnerSource

**O que é:** aplicação de princípios de open source dentro de uma organização.

**Para que serve:** incentivar reuso, colaboração e contribuições entre equipes internas.

| Open source | InnerSource |
|---|---|
| Qualquer pessoa no mundo pode contribuir | Pessoas da organização contribuem |
| Código geralmente público | Código geralmente interno |
| Comunidade externa | Comunidade interna |

Uma métrica positiva de InnerSource é o aumento de pull requests que resolvem bugs.

## GitHub Sponsors

**O que é:** plataforma de apoio financeiro a desenvolvedores e projetos.

**Para que serve:** permitir patrocínios pontuais ou recorrentes a mantenedores open source.

## GitHub Marketplace

**O que é:** catálogo de ferramentas e integrações.

**Para que serve:** descobrir e instalar apps, Actions e extensões que adicionam funcionalidades ao GitHub.

## Follow

**O que é:** ação de acompanhar um usuário ou organização.

**Para que serve:** receber atualizações sobre a atividade pública dessa pessoa ou organização.

## Discoverable repositories

**O que é:** repositório fácil de encontrar e entender.

**Para que serve:** aumentar a visibilidade, reuso e contribuição no projeto.

## Questões — Domínio 7

**1.** O que é InnerSource?  
A. Publicação obrigatória de todo código da empresa na internet.  
B. Aplicação de princípios de open source dentro de uma organização.  
C. Uma licença proprietária.  
D. Um tipo de repositório privado.  

**2.** Qual recurso permite apoiar financeiramente mantenedores de projetos open source?  
A. GitHub Sponsors  
B. GitHub Marketplace  
C. GitHub Pages  
D. GitHub Codespaces  

**3.** Qual recurso reúne integrações e Actions de terceiros?  
A. GitHub Marketplace  
B. GitHub Discussions  
C. GitHub Insights  
D. GitHub Mobile  

**4.** Qual é a principal finalidade de um fork?  
A. Proteger a branch principal.  
B. Criar uma cópia de um repositório para contribuir ou reutilizar.  
C. Agrupar issues em uma entrega.  
D. Definir revisores automáticos.  

**5.** Qual métrica indica melhor que um programa InnerSource está funcionando?  
A. Queda constante de novas issues.  
B. Aumento de pull requests que resolvem bugs.  
C. Aumento do número de stars.  
D. Redução do número de forks.  

**Gabarito:** 1-B; 2-A; 3-A; 4-B; 5-B.

**Comentários:** InnerSource é colaboração open source interna; Sponsors permite patrocínio; Marketplace reúne integrações; fork cria cópia; PRs que resolvem bugs mostram contribuição real.

---

# Revisão final rápida

- Git é DVCS local; GitHub é plataforma remota e colaborativa.
- Commit registra alterações; push envia ao remoto.
- Branch isola trabalho; merge integra.
- Issue é tarefa; discussion é conversa; PR é revisão e merge.
- Base é destino; compare é origem.
- `Closes #n` fecha issue após merge.
- README documenta; LICENSE licencia; CODEOWNERS define revisores; SECURITY.md orienta vulnerabilidades.
- `.gitignore` evita commit de arquivos sensíveis.
- Actions automatiza por eventos; runner executa jobs.
- Container action inclui ambiente; JavaScript action roda no runner.
- Codespaces fica em `/workspaces` e usa `.devcontainer`.
- Iteration organiza sprints; single select organiza prioridades.
- Built-in workflow é automação nativa do Project.
- Danger zone altera visibilidade, fecha ou exclui Project.
- Write permite push; Triage gerencia issues/PRs; Team maintainer gerencia equipe.
- SAML SSO autentica via IdP; EMU gerencia usuários corporativos.
- Dependabot alerta sobre dependências vulneráveis.
- Secret scanning detecta segredos expostos; alertas ficam na aba Security.
- Sponsors apoia projetos; Marketplace reúne integrações; InnerSource internaliza colaboração open source.
