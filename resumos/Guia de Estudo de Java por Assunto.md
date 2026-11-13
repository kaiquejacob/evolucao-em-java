# Guia de Estudo de Java por Assunto

Material de consulta teórico, do básico ao avançado. O repositório *Evolução em Java* serviu apenas de **parâmetro** para definir o que cobrir; a organização aqui é por assunto. Atualizado em 24/10/2026.

## Como ler

Cada tópico segue o mesmo molde:

- **O que é**: a definição e como funciona.
- **Para que serve**: quando e por que usar (e quando evitar).
- **Exemplo**: um código curto e simples no final.

## Sumário

1. **Fundamentos da linguagem** (plataforma, tipos, operadores, controle de fluxo, arrays, métodos, strings)
2. **Orientação a objetos** (classes, herança, polimorfismo, interfaces, enums, records, exceções...)
3. **API padrão do Java** (coleções, generics, datas, regex, I/O, serialização, números, reflection, annotations)
4. **Java avançado** (lambdas, Optional, Streams, concorrência, memória)
5. **Persistência e qualidade** (JDBC, JPA, build, testes)
6. **Arquitetura e boas práticas** (SOLID, Clean Code, padrões de projeto)
7. **Ecossistema Spring** (IoC/DI, Boot, Web, Data, Security, Cloud, testes)

---

# Parte 1: Fundamentos da linguagem

## 1.1 A plataforma Java (JDK, JRE, JVM)

**O que é**: Java é, ao mesmo tempo, uma linguagem e uma plataforma. O código-fonte (`.java`) não é traduzido para instruções de um processador específico, e sim para **bytecode** (`.class`), o conjunto de instruções de uma máquina abstrata. Esse bytecode é executado pela **JVM** (*Java Virtual Machine*), que existe para cada sistema operacional e arquitetura. Por isso o mesmo `.class` roda em Windows, Linux ou macOS ("escreva uma vez, rode em qualquer lugar"), desde que haja uma JVM compatível.

**Para que serve**: dar **portabilidade** ao código, **gerenciar a memória automaticamente** (você não faz `free`), **verificar a segurança** do bytecode antes de executá-lo e **otimizar o desempenho** em tempo de execução. Isso explica por que Java é tão usado em sistemas corporativos, back-ends, Android e big data.

**Como funciona**: o caminho de um programa, do arquivo ao resultado:

1. O compilador **`javac`** transforma `.java` em `.class` (bytecode). Erros de sintaxe e de tipos são detectados aqui.
2. Ao executar, o **class loader** carrega as classes sob demanda.
3. O **verificador** confere se o bytecode é válido e seguro.
4. A JVM começa **interpretando** o bytecode. O **JIT** (*Just-In-Time*, no HotSpot os compiladores C1 e C2) observa quais métodos são executados com muita frequência (*hot spots*) e os **compila para código nativo otimizado** durante a execução. Por isso um programa Java "esquenta": fica mais rápido depois de rodar um tempo.
5. O **Garbage Collector** (GC) libera objetos que não podem mais ser alcançados.

Os três nomes que mais confundem:

| Sigla | O que contém | Para quê |
| --- | --- | --- |
| **JVM** | a máquina virtual (class loader, interpretador, JIT, GC) | executar bytecode |
| **JRE** | JVM + bibliotecas padrão (`java.base`, `java.sql`...) | executar aplicações |
| **JDK** | JRE + ferramentas (`javac`, `jar`, `javadoc`, `jshell`, `jlink`, `jcmd`) | desenvolver |

Hoje se instala o **JDK**; a distribuição separada de JRE deixou de ser comum a partir do Java 11, e `jlink` monta um runtime enxuto sob medida.

**Memória da JVM** (modelo conceitual):

- **Stack (pilha)**: uma por *thread*. Cada chamada de método cria um *frame* com variáveis locais e referências. É liberada sozinha quando o método termina.
- **Heap**: área compartilhada onde vivem os **objetos** e os arrays. É a área gerenciada pelo GC.
- **Metaspace**: metadados das classes (usa memória nativa; substituiu a antiga *PermGen* no Java 8).

O GC considera "vivo" tudo que é alcançável a partir das **raízes** (variáveis locais nas pilhas, campos `static`). O resto é lixo. Coletores disponíveis: **G1** (padrão em máquinas com recursos de servidor desde o Java 9), **ZGC** e **Shenandoah** (pausas muito curtas), **Parallel** (foco em vazão) e **Serial** (programas pequenos). O tópico 4.8 aprofunda o assunto.

**Pontos importantes**:

- Java é **compilado e interpretado/JIT**: não é nem "só interpretado" nem compilado direto para nativo (a menos que se use *native image* do GraalVM, que troca flexibilidade por inicialização rápida).
- **Versões**: sai uma versão a cada 6 meses; as de **suporte longo (LTS)** são 8, 11, 17, 21 e 25. Recursos citados neste guia indicam a versão em que se tornaram finais.
- "Rode em qualquer lugar" tem limites: bibliotecas nativas (JNI), caminhos de arquivo e *charset* padrão podem variar por sistema.
- O código **não** vira nativo antes de rodar: o `.class` é independente de CPU.

**Exemplo**:

```java
// Ola.java: o arquivo tem o mesmo nome da classe pública
public class Ola {
    public static void main(String[] args) {          // ponto de entrada
        System.out.println("Olá, JVM!");               // escreve no console
    }
}
```

```bash
javac Ola.java     # compila: gera Ola.class (bytecode)
java Ola           # a JVM carrega Ola.class e executa main
java Ola.java      # Java 11+: compila em memória e executa em um passo
```

**Explicação do exemplo**: `javac` produz o arquivo `Ola.class`, sem executar nada. O comando `java Ola` inicia a JVM, que carrega a classe `Ola` e invoca o método `main`; o resultado é a linha `Olá, JVM!` no console. A terceira forma é um atalho para testes rápidos: compila em memória e executa, sem deixar `.class` no disco.

## 1.2 Estrutura de um programa Java

**O que é**: todo código Java vive dentro de **tipos** (classes, interfaces, enums, records). A execução começa em um método especial, o `main`. Os tipos são agrupados em **pacotes** (`package`) e usam tipos de outros pacotes por meio de `import`.

**Para que serve**: organizar o código em unidades coesas, evitar **conflito de nomes** (duas classes `Data` em pacotes diferentes convivem) e definir onde o programa começa.

**Como funciona**: a ordem em um arquivo é fixa: (1) `package`, (2) `import`s, (3) declarações de tipos.

- **Pacote**: `package br.com.exemplo;` declara a que pacote a classe pertence. A estrutura de pastas **precisa espelhar** o nome (`br/com/exemplo/`). Por convenção, o nome é o domínio da empresa invertido, em minúsculas.
- **Import**: `import java.util.List;` traz uma classe; `import java.util.*;` traz todas as classes **daquele pacote** (não dos subpacotes); `import static java.lang.Math.*;` traz membros estáticos. Tudo de `java.lang` (`String`, `Math`, `System`...) é importado automaticamente. Importar não copia código: só permite usar o nome curto.
- **Arquivo e classe pública**: um arquivo pode ter vários tipos, mas **no máximo um `public`**, e o arquivo deve ter o nome dele.
- **Identificadores** (nomes): podem ter letras, dígitos, `_` e `$`, **não** começam com dígito, diferenciam maiúsculas de minúsculas (`idade` ≠ `Idade`) e não podem ser palavras reservadas (`class`, `int`, `new`...). A partir do Java 22, `_` sozinho é usado para variáveis "sem nome" (que não serão lidas).
- **Comentários**: `//` (linha), `/* ... */` (bloco) e `/** ... */` (**Javadoc**, usado pela ferramenta `javadoc` para gerar documentação, com tags como `@param` e `@return`).
- **Convenções**: classes em `PascalCase`, métodos e variáveis em `camelCase`, constantes em `MAIUSCULAS_COM_UNDERSCORE`, pacotes em minúsculas.
- **Ponto de entrada**: no Java clássico, `public static void main(String[] args)`. Cada parte importa: `public` (a JVM precisa acessar), `static` (roda sem criar objeto), `void` (não devolve valor) e `String[] args` (argumentos da linha de comando). Desde o **Java 25** (JEP 512, recurso final), a JVM também aceita `static void main(String[] args)` sem `public`, `main()` sem parâmetros e até `main` de instância (nesse caso a classe precisa de um construtor sem argumentos acessível). A classe `java.io.IO` (também do Java 25) oferece `IO.println`, `IO.print` e `IO.readln`; ela é importada automaticamente só em arquivos "compactos" (sem declaração de classe); em uma classe comum é preciso `import java.io.IO;`.

**Pontos importantes**: o estilo novo de `main` é ótimo para aprender, mas, em projetos que serão executados por Maven, Gradle, IDEs ou servidores com JDKs variados, o `public static void main` clássico continua sendo o mais compatível. `import java.util.*;` não é pecado, mas importar classes explicitamente deixa claro de onde cada nome vem e evita ambiguidade (`java.util.Date` e `java.sql.Date`).

**Exemplo**:

```java
// arquivo: src/br/com/exemplo/Saudacao.java
package br.com.exemplo;                 // 1. pacote

import java.util.List;                  // 2. imports

/** Mostra uma saudação para cada nome da lista. */
public class Saudacao {                 // 3. tipo (nome = nome do arquivo)
    public static void main(String[] args) {
        List<String> nomes = List.of("Ana", "Bia");
        for (String nome : nomes) {
            System.out.println("Olá, " + nome);
        }
    }
}
```

```bash
javac -d out src/br/com/exemplo/Saudacao.java   # gera out/br/com/exemplo/Saudacao.class
java -cp out br.com.exemplo.Saudacao           # usa o nome COMPLETO (pacote + classe)
```

**Explicação do exemplo**: o compilador cria, dentro de `out`, as pastas do pacote e coloca o `.class` nelas. Para executar, informa-se o **nome totalmente qualificado** (`br.com.exemplo.Saudacao`) e onde procurar as classes (`-cp`, o *classpath*). A saída é `Olá, Ana` e `Olá, Bia`, uma por linha.

## 1.3 Variáveis e tipos de dados

**O que é**: uma **variável** é um espaço de memória com **nome** e **tipo**. Java é **estaticamente tipado**: o tipo é conhecido em tempo de compilação e não muda. Há dois grupos de tipos:

- **Primitivos** (8): a variável guarda **o próprio valor**.
- **De referência** (classes, interfaces, arrays, enums, records): a variável guarda uma **referência** (endereço) para um objeto no heap, ou `null` quando não aponta para nada.

**Para que serve**: o tipo define quanta memória é usada, que valores são aceitos e que operações são permitidas. Como a checagem acontece antes da execução, o compilador pega muitos erros cedo (atribuir texto a um `int`, por exemplo).

**Como funciona**: os 8 tipos primitivos:

| Tipo | Tamanho | Faixa / característica | Padrão (atributo) |
| --- | --- | --- | --- |
| `byte` | 8 bits | -128 a 127 | 0 |
| `short` | 16 bits | -32.768 a 32.767 | 0 |
| `int` | 32 bits | -2.147.483.648 a 2.147.483.647 (padrão para inteiros) | 0 |
| `long` | 64 bits | ≈ ±9,22 × 10^18 | 0L |
| `float` | 32 bits | IEEE 754, \~6 a 7 dígitos significativos | 0.0f |
| `double` | 64 bits | IEEE 754, \~15 a 16 dígitos (padrão para decimais) | 0.0 |
| `char` | 16 bits | unidade de código UTF-16, de 0 a 65.535 (sem sinal) | `'\u0000'` |
| `boolean` | (não especificado) | `true` ou `false`; não é convertível para número | false |

- **Declaração e inicialização**: `int idade;` declara; `idade = 25;` atribui; `int idade = 25;` faz os dois.
- **Valor padrão**: só **atributos** de classe e **elementos de array** recebem valor padrão (0, `false`, `null`). **Variáveis locais não**: usar antes de atribuir é erro de compilação (*definite assignment*).
- **Escopo**: a variável existe apenas no bloco `{ }` onde foi declarada (e nos blocos internos). Não se pode redeclarar, num bloco interno, uma variável local de mesmo nome.
- **Literais**: inteiros são `int` e decimais são `double`. Use sufixos `L` (`long`) e `F` (`float`); `_` separa dígitos (`1_000_000`); prefixos `0x` (hexadecimal) e `0b` (binário); `'A'` é `char`, `"A"` é `String`.
- **`final`**: a variável só pode receber valor uma vez. Em tipos de referência, o que fica fixo é a **referência**, não o conteúdo do objeto.
- **`var`** (Java 10): o compilador **infere** o tipo de uma variável **local** a partir do inicializador (`var n = 10;` continua sendo `int`). Não é tipagem dinâmica. Não vale para atributos, parâmetros de método, nem sem inicializador (`var x;`) ou com `null`.
- **Primitivo x referência**: `int a = 5; int b = a;` copia o valor (mudar `b` não afeta `a`). Com objetos, `Pessoa q = p;` copia a **referência**: os dois nomes apontam para o mesmo objeto. Acessar membro por uma referência `null` lança `NullPointerException`.

**Pontos importantes**:

- `float` e `double` guardam números em **base 2**, e muitos decimais (como 0,1) não têm representação exata: `0.1 + 0.2` dá `0.30000000000000004`. Para dinheiro e cálculos exatos, use `BigDecimal` (tópico 3.7), nunca `double`.
- Um `char` é uma **unidade de código UTF-16**, não necessariamente um caractere completo: símbolos fora do plano básico (muitos emojis) ocupam **dois** `char`.
- O tamanho do `boolean` na memória não é definido pela linguagem.

**Exemplo**:

```java
int idade = 25;                    // primitivo: guarda o valor 25
long populacao = 8_000_000_000L;   // sufixo L: não cabe em int
double preco = 19.90;
char letra = 'A';
boolean ativo = true;
final double PI = 3.14159;         // constante: não pode ser reatribuída
var nome = "Ka";                   // var infere String

String texto = null;               // referência que não aponta para nenhum objeto
// texto.length();                 // lançaria NullPointerException

int x = 5;
int y = x;                         // cópia do valor
y = 99;
System.out.println(x + " " + y);   // 5 99
```

**Explicação do exemplo**: cada variável primitiva guarda seu valor diretamente. `1_000_000` é só legibilidade; o `L` é obrigatório porque `8_000_000_000` estoura o `int`. `PI` não pode mudar. `var nome` torna-se `String`. A última parte mostra que primitivos são **independentes**: alterar `y` não muda `x`, e a saída é `5 99`.

## 1.4 Conversão de tipos (casting)

**O que é**: transformar um valor de um tipo em outro. A conversão pode ser **implícita** (feita pelo compilador) ou **explícita** (você escreve o *cast*, `(tipo) valor`).

**Para que serve**: misturar tipos em cálculos e adaptar valores a métodos que exigem um tipo específico, sabendo exatamente onde pode haver perda de informação.

**Como funciona**:

- **Ampliação (*widening*)**: de um tipo "menor" para um "maior", automática: `byte → short → int → long → float → double` (e `char → int`). Não há erro, mas **pode haver perda de precisão** em `int → float`, `long → float` e `long → double`, porque `float` e `double` têm menos dígitos significativos do que `int` e `long` têm bits.
- **Estreitamento (*narrowing*)**: de um tipo maior para um menor, **exige cast** e pode perder dados:
  - `double → int`: descarta a parte fracionária (**trunca em direção a zero**: `(int) 3.99` é 3 e `(int) -3.99` é -3). Valores fora da faixa viram o menor/maior `int`, e `NaN` vira 0.
  - `int → byte`: mantém só os 8 bits de baixo, então `(byte) 200` vira **-56** (estouro).
- **Promoção em expressões**: operandos `byte`, `short` e `char` são promovidos a `int` antes de uma operação aritmética; se houver `long`, `float` ou `double`, o resultado sobe para o maior tipo presente. Por isso `byte c = a + b;` não compila (o resultado é `int`).
- **Atribuição composta** (`+=`, `-=`...) faz o cast implícito: `byte b = 10; b += 5;` compila e funciona.
- **Overflow** de inteiros é **silencioso** (dá a volta): `Integer.MAX_VALUE + 1` resulta em `Integer.MIN_VALUE`. `Math.addExact` lança `ArithmeticException` em vez de estourar.
- **Texto ↔ número**: não é *cast*, é conversão por método: `Integer.parseInt("42")`, `Double.parseDouble("3.5")`, `String.valueOf(42)`. Texto inválido lança `NumberFormatException`.
- **Referências**: *upcast* (para o tipo pai) é automático; *downcast* (para o filho) exige cast e pode lançar `ClassCastException` (tópico 2.8).

**Pontos importantes**: `boolean` não converte para número nem o contrário. `(char)` sobre `int` grava o código do caractere. Cast **não arredonda**; para arredondar use `Math.round`.

**Exemplo**:

```java
int a = 5, b = 2;
System.out.println(a / b);              // 2   (int / int = divisão inteira)
System.out.println((double) a / b);     // 2.5 (cast do a ANTES de dividir)
long grande = a;                        // widening automático
int trunc = (int) 3.99;                 // 3   (trunca)
byte pequeno = (byte) 200;              // -56 (estouro)
byte x = 10;
x += 5;                                 // ok: cast implícito -> 15
System.out.println(Integer.MAX_VALUE + 1);   // -2147483648
System.out.println(Math.round(3.5));    // 4
```

**Explicação do exemplo**: na primeira linha, os dois operandos são `int`, então a divisão é inteira e o resto é descartado. No cast `(double) a / b`, o `a` vira `double` **antes** da divisão, e o `b` é promovido, dando 2.5. `(byte) 200` mantém só os 8 bits de baixo (11001000), que, em complemento de dois, valem -56. O `x += 5` compila porque o operador composto inclui o cast. A soma `MAX_VALUE + 1` estoura e dá a volta para o menor `int`.

## 1.5 Operadores

**O que é**: símbolos que combinam valores para produzir um resultado. O **operando** é o valor e o **operador** é a operação.

**Para que serve**: construir expressões de cálculo, comparação e lógica, que alimentam todas as decisões e repetições do programa.

**Como funciona**: por grupo:

| Grupo | Operadores | Observações |
| --- | --- | --- |
| Aritméticos | `+ - * / %` | `/` entre inteiros é inteira; `%` é o resto |
| Relacionais | `== != > < >= <=` | resultado `boolean` |
| Lógicos | `&& \|\| !` e `& \| ^` | `&&`/`\|\|` fazem curto-circuito |
| Atribuição | `= += -= *= /= %= &= \|= ^= <<= >>= >>>=` | compostos fazem cast implícito |
| Incremento | `++ --` | prefixo ou sufixo |
| Ternário | `cond ? a : b` | é uma **expressão** |
| Bit a bit / deslocamento | `& \| ^ ~ << >> >>>` | `>>` mantém o sinal; `>>>` preenche com zeros |
| Tipo | `instanceof` | testa o tipo do objeto |

- **Divisão por zero**: com **inteiros**, lança `ArithmeticException`; com `double`/`float`, resulta em `Infinity` ou `NaN`, sem erro. `%` aceita decimais e, com negativos, o sinal do resultado segue o dividendo (`-7 % 3` é `-1`).
- **`+` com `String`**: se um operando é `String`, concatena. A avaliação é da **esquerda para a direita**: `1 + 2 + "a"` é `"3a"` e `"a" + 1 + 2` é `"a12"`.
- **`==` com objetos** compara **referências** (se são o mesmo objeto); para comparar conteúdo, use `equals`. **Com decimais**, evite `==` (`0.1 + 0.2 == 0.3` é `false`); compare com tolerância ou use `BigDecimal`. `NaN` nunca é igual a nada, nem a ele mesmo.
- **Curto-circuito**: em `a && b`, se `a` é `false`, `b` **não é avaliado**; em `a || b`, se `a` é `true`, `b` não é avaliado. `&` e `|` (sobre booleanos) avaliam sempre os dois lados. Isso é usado de propósito: `if (lista != null && !lista.isEmpty())` não lança `NullPointerException`.
- **`i++` x `++i`**: ambos somam 1; `i++` devolve o valor **antes** de somar, `++i` devolve o valor **depois**. Evite usá-los dentro de expressões maiores (`x = i++ + ++i` é ilegível).
- **`instanceof`**: `obj instanceof String` é `true` se `obj` é uma `String` (e `false` para `null`). Com *pattern matching* (Java 16), `if (obj instanceof String s)` já cria a variável `s` com o cast feito.
- **Precedência** (da maior para a menor, simplificada): `++ --` e `!` → `* / %` → `+ -` → `<< >>` → `< > <= >=` → `== !=` → `&` → `^` → `|` → `&&` → `||` → `?:` → atribuição. Na dúvida, use parênteses.

**Pontos importantes**: operadores bit a bit têm uso específico (máscaras, flags, protocolos); no dia a dia dominam os aritméticos, relacionais e lógicos. O ternário não substitui `if/else` quando há efeitos colaterais; use-o para escolher um valor.

**Exemplo**:

```java
int x = 10;
boolean grandePar = x > 5 && x % 2 == 0;      // true && true
x += 5;                                        // x = 15
String tipo = x > 10 ? "grande" : "pequeno";   // "grande"

int i = 5;
int a = i++;    // a = 5, depois i = 6
int b = ++i;    // i = 7, depois b = 7

System.out.println(1 + 2 + "a");   // 3a
System.out.println("a" + 1 + 2);   // a12
System.out.println(5 / 2);         // 2
System.out.println(5.0 / 0);       // Infinity
// System.out.println(5 / 0);      // ArithmeticException
```

**Explicação do exemplo**: em `grandePar`, as duas condições são verdadeiras. `x += 5` é `x = x + 5`. No ternário, a condição `15 > 10` escolhe `"grande"`. Em `i++`, o valor antigo (5) vai para `a`; em `++i`, o `i` é incrementado antes e `b` recebe 7. A concatenação é da esquerda para a direita: `1 + 2` ainda é soma (3) e só depois vira texto; em `"a" + 1 + 2`, o primeiro `+` já concatena, resultando em `a12`. Dividir `double` por zero dá `Infinity`; dividir `int` por zero lança exceção.

## 1.6 Estruturas condicionais

**O que é**: comandos que escolhem qual trecho de código executar conforme uma condição ou um valor.

**Para que serve**: ramificar a lógica do programa de acordo com dados e estados (validar entrada, classificar valores, escolher um caminho).

**Como funciona**:

- **`if / else if / else`**: avalia as condições `boolean` **em ordem** e executa o **primeiro** bloco verdadeiro; os demais são ignorados. A condição deve ser um `boolean` (não vale `if (n)` com `int`). Sem chaves, o `if` controla só a próxima instrução (fonte de bugs: use sempre `{ }`).
- **`switch` tradicional (instrução)**: compara **uma** variável com constantes `case`. Aceita `byte`, `short`, `int`, `char`, seus *wrappers*, `String` e `enum`. **Sem `break`**, a execução "cai" nos próximos `case` (*fall-through*), às vezes usado de propósito para agrupar casos. Não aceita `long`, `float`, `double` nem `boolean`.
- **`switch` moderno (Java 14, final)**: com `->` não há *fall-through* e não precisa de `break`; vários valores no mesmo `case` (`case 1, 7 ->`); pode ser uma **expressão** que **devolve valor**. Em um bloco, `yield valor;` devolve o valor. Uma *switch expression* **precisa ser exaustiva**: cobrir todos os valores possíveis (todas as constantes de um `enum`, todos os subtipos de uma interface `sealed`, ou ter `default`).
- **Pattern matching em `switch` (Java 21, final)**: `case Circulo c ->` testa o tipo e já declara a variável; `case Circulo c when c.raio() > 10 ->` adiciona uma **guarda**; `case null ->` trata `null` explicitamente (sem isso, `switch` com `null` lança `NullPointerException`). O compilador rejeita um `case` inalcançável ("dominado" por um mais genérico anterior). *Pattern matching* com tipos primitivos ainda é recurso em **pré-visualização** nas versões recentes (verifique a versão do seu JDK).
- **Ternário** (`cond ? a : b`): a versão compacta do `if/else` que produz valor.

**Pontos importantes**: `if` serve para faixas e combinações de condições; `switch` para comparar **uma** variável com valores discretos. O `switch` sobre `String` usa `equals` (diferencia maiúsculas). Prefira a forma com `->`: é mais curta e elimina o erro do `break` esquecido.

**Exemplo**:

```java
int nota = 85;
if (nota >= 90) {
    System.out.println("A");
} else if (nota >= 70) {            // só testa se a anterior foi falsa
    System.out.println("B");         // este bloco executa
} else {
    System.out.println("C");
}

int dia = 3;
String tipo = switch (dia) {            // switch como expressão
    case 1, 7 -> "Fim de semana";
    case 2, 3, 4, 5, 6 -> "Dia útil";
    default -> "Inválido";              // obrigatório aqui: int não tem todos os casos cobertos
};
System.out.println(tipo);               // Dia útil
```

**Explicação do exemplo**: com `nota = 85`, a primeira condição (`>= 90`) é falsa, a segunda (`>= 70`) é verdadeira e imprime `B`; o `else` final é ignorado. No `switch`, `dia = 3` casa com o segundo `case`, e a expressão devolve `"Dia útil"`, que é atribuído a `tipo`. O `default` é exigido porque um `int` tem valores demais para serem listados; com um `enum`, listar todas as constantes bastaria.

## 1.7 Estruturas de repetição

**O que é**: comandos que executam um bloco de código **várias vezes**, enquanto uma condição for verdadeira ou para cada elemento de uma sequência.

**Para que serve**: processar coleções e arrays, repetir tentativas, ler dados até acabarem e acumular resultados, sem escrever o mesmo código várias vezes.

**Como funciona**:

| Laço | Estrutura | Quando usar |
| --- | --- | --- |
| `for` | `for (inicialização; condição; passo)` | o número de voltas é conhecido ou controlado por um contador |
| `while` | `while (condição)` | a parada depende de algo que só se resolve durante a execução; pode **nunca** executar |
| `do-while` | `do { ... } while (condição);` | o corpo deve executar **pelo menos uma vez** (a condição é testada no fim) |
| *for-each* | `for (T x : fonte)` | percorrer todos os elementos de um array ou `Iterable` sem precisar do índice |

- No `for`, a inicialização roda **uma vez**; depois, a cada volta: testa a condição, executa o corpo, executa o passo. A variável declarada na inicialização só existe dentro do laço. Qualquer das três partes pode ser omitida (`for (;;)` é um laço infinito).
- No *for-each*, a variável recebe **uma cópia** do valor (ou da referência) de cada elemento: reatribuí-la **não** altera o array ou a coleção. Para alterar posições, use o `for` com índice.
- **`break`** encerra o laço mais interno imediatamente. **`continue`** abandona só a volta atual e segue para a próxima (no `for`, vai para o passo). Com **rótulo** (`externo:`), `break externo;` e `continue externo;` atuam sobre um laço de fora.
- **Cuidados**: **laço infinito** (a condição nunca muda), **erro de uma unidade** (`<` x `<=`, índice de 0 a `length - 1`) e **alterar a coleção durante um *for-each***: remover/adicionar elementos costuma lançar `ConcurrentModificationException` (a detecção é de melhor esforço, não garantida). Use `Iterator.remove()` ou `removeIf`.

**Pontos importantes**: escolha o laço pela intenção: contador → `for`; "até que algo aconteça" → `while`; "faça uma vez e repita se preciso" → `do-while`; "para cada item" → *for-each*. Para transformar ou filtrar coleções, as *Streams* (tópico 4.4) costumam ser mais declarativas, mas um laço simples é melhor quando precisa de índice, `break` ou estado mutável.

**Exemplo**:

```java
for (int i = 1; i <= 5; i++) {
    if (i == 3) continue;          // pula só a volta do 3
    System.out.print(i + " ");     // 1 2 4 5
}

int n = 0;
do {
    n++;                           // executa ao menos uma vez
} while (n < 3);                   // n termina valendo 3

int[][] m = {{1, 2}, {3, 4}};
busca:
for (int[] linha : m) {
    for (int v : linha) {
        if (v == 3) {
            System.out.println("achei o 3");
            break busca;           // sai dos DOIS laços
        }
    }
}
```

**Explicação do exemplo**: no primeiro laço, quando `i` vale 3 o `continue` pula o `print` e o laço segue para `i = 4`, então a saída é `1 2 4 5`. O `do-while` incrementa `n` antes de testar a condição, de modo que o corpo roda ao menos uma vez e para quando `n` chega a 3. No terceiro trecho, o *for-each* externo percorre as linhas e o interno os valores; ao achar o 3, `break busca` encerra os dois laços de uma vez (um `break` simples sairia só do interno).

## 1.8 Arrays

**O que é**: estrutura de **tamanho fixo** que guarda elementos do **mesmo tipo** em posições numeradas a partir de 0. Em Java, o array é um **objeto** (vive no heap), mesmo quando guarda primitivos, e a variável guarda a **referência** para ele.

**Para que serve**: armazenar uma sequência de valores de quantidade conhecida, com **acesso direto e rápido** por índice. Quando a quantidade varia, use `List` (tópico 3.2), que por baixo usa arrays redimensionáveis.

**Como funciona**:

- **Criar**: `int[] a = new int[3];` (3 posições, todas com o padrão do tipo: 0, `false`, `null`...). Ou com valores: `int[] b = {1, 2, 3};` / `new int[]{1, 2, 3}` (a forma com `new` é necessária ao passar um array "na hora" para um método).
- **Tamanho**: `a.length` é um **atributo** (sem parênteses, ao contrário de `String.length()` e `List.size()`) e **nunca muda** depois da criação.
- **Acesso**: `a[0]` até `a[a.length - 1]`. Um índice fora da faixa lança `ArrayIndexOutOfBoundsException`; tamanho negativo, `NegativeArraySizeException`.
- **Array de objetos** guarda **referências**: `new String[3]` cria 3 posições `null`, não 3 Strings.
- **Multidimensional** é array de arrays. `new int[2][3]` cria uma matriz regular; `new int[2][]` cria só as linhas, e você define cada uma depois, podendo ter **tamanhos diferentes** (*jagged*).
- **Cópia**: `int[] c = a;` **não copia** (as duas variáveis apontam para o mesmo array). Para copiar: `a.clone()`, `Arrays.copyOf(a, n)` ou `System.arraycopy`. A cópia é **rasa**: em arrays de objetos, os objetos continuam compartilhados.
- **Comparação**: `a == b` compara referências; `a.equals(b)` também (arrays não sobrescrevem `equals`). Use `Arrays.equals(a, b)` (e `Arrays.deepEquals` para matrizes).
- **`java.util.Arrays`**: `toString` / `deepToString` (para imprimir; `println(array)` mostra algo como `[I@6d06d69c`), `sort` (primitivos: *dual-pivot quicksort*; objetos: *TimSort*, **estável**), `binarySearch` (exige array **ordenado**), `fill`, `copyOfRange`, `asList`, `stream`.
- **Covariância**: `Object[] o = new String[1]; o[0] = 1;` compila, mas lança `ArrayStoreException` na execução. (É uma fragilidade dos arrays que os *generics* evitam.)

**Pontos importantes**: arrays dão acesso O(1) por índice, mas inserir/remover no meio exige deslocar elementos. Elementos de tipos primitivos ficam contíguos, o que é eficiente em memória e cache. *Varargs* (`int... nums`) é um array disfarçado.

**Exemplo**:

```java
int[] notas = new int[3];          // {0, 0, 0}: valor padrão do tipo int
notas[0] = 8;
notas[1] = 6;
notas[2] = 9;

int[] mesmo = notas;               // NÃO copia: as duas variáveis apontam para o mesmo array
mesmo[0] = 10;
System.out.println(notas[0]);      // 10

int[] copia = Arrays.copyOf(notas, notas.length);   // cópia de verdade
copia[1] = 0;
System.out.println(Arrays.toString(notas));         // [10, 6, 9]
System.out.println(Arrays.toString(copia));         // [10, 0, 9]

int[][] irregular = new int[2][];  // 2 linhas, ainda sem conteúdo (null)
irregular[0] = new int[]{1};
irregular[1] = new int[]{2, 3, 4};
System.out.println(irregular[1].length);            // 3
```

**Explicação do exemplo**: o array nasce com zeros e é preenchido posição a posição. Em `mesmo = notas` **não há cópia**: as duas variáveis referenciam o mesmo objeto, então alterar `mesmo[0]` muda também `notas[0]` (a saída é `10`). `Arrays.copyOf` cria um array **novo**, e por isso mudar `copia[1]` não afeta `notas`; as duas impressões mostram a diferença. O array irregular tem duas linhas de tamanhos diferentes (1 e 3), o que só é possível porque uma matriz em Java é um array de arrays.

## 1.9 Métodos

**O que é**: bloco de código **com nome** que executa uma tarefa, pode receber **parâmetros** (entradas) e **devolver** um resultado (saída). Forma geral: `modificadores tipoDeRetorno nome(tipo param, ...) { corpo }`. O nome mais os **tipos dos parâmetros** formam a **assinatura**; o tipo de retorno **não** faz parte dela.

**Para que serve**: evitar repetição (princípio DRY), dividir um problema grande em partes pequenas e testáveis, dar um **nome** à intenção do código e reutilizá-lo.

**Como funciona**:

- **Chamada**: os argumentos são avaliados e **copiados** para os parâmetros; a JVM cria um novo *frame* na pilha; quando o método termina (`return` ou fim do corpo), o frame é descartado e a execução volta para quem chamou.
- **Retorno**: `void` não devolve nada; qualquer outro tipo exige `return valor;` em **todos os caminhos** possíveis (senão o compilador acusa *missing return statement*). `return;` em um método `void` encerra a execução mais cedo.
- **Passagem por valor**: Java **sempre copia** o argumento; não existe passagem por referência. Com primitivos, copia-se o dado: reatribuir o parâmetro não afeta o chamador. Com objetos, copia-se a **referência**: o método consegue **alterar o estado** do objeto apontado (a mudança é visível fora), mas reatribuir o parâmetro para outro objeto afeta só a cópia local.
- **Sobrecarga (*overload*)**: vários métodos com o **mesmo nome** e parâmetros diferentes (quantidade, tipos ou ordem). O compilador escolhe pela chamada, nesta ordem de tentativas: (1) correspondência exata ou **ampliação** (`int` → `double`), (2) *boxing/unboxing*, (3) *varargs*. Se duas candidatas empatam, há erro de ambiguidade. Mudar só o tipo de retorno **não** é sobrecarga.
- **Varargs** (`int... nums`): aceita zero ou mais argumentos (ou um array pronto); dentro do método é um array. Só um por método e sempre o **último** parâmetro. Por ser a última opção na escolha da sobrecarga, `soma(1, 2)` prefere `soma(double, double)` a `soma(int...)`.
- **Recursão**: método que chama a si mesmo; exige um **caso base** que encerre as chamadas. Cada chamada ocupa um frame, então recursão muito profunda causa `StackOverflowError` (Java não otimiza recursão de cauda).
- **`static` x de instância**: método `static` pertence à classe (`Math.max`), não tem `this` e não acessa membros de instância diretamente; método de instância opera sobre um objeto.
- Não se declara um método **dentro** de outro (use lambdas ou classes locais).

**Pontos importantes**: nomes são verbos em `camelCase`; métodos curtos, com poucos parâmetros (mais de três sugere agrupar em um objeto) e uma única responsabilidade são mais fáceis de testar e entender.

**Exemplo**:

```java
static int soma(int... nums) {            // varargs: nums é um int[]
    int total = 0;
    for (int n : nums) total += n;
    return total;
}
static double soma(double a, double b) {  // sobrecarga
    return a + b;
}
static int fatorial(int n) {
    if (n <= 1) return 1;                 // caso base
    return n * fatorial(n - 1);           // chamada recursiva
}
static void alterar(int x, int[] v) {
    x = 0;                                // altera só a cópia local
    v[0] = 0;                             // altera o array original (mesma referência)
    v = new int[]{9};                     // reatribui só a cópia da referência
}

System.out.println(soma(1, 2, 3));        // 6   (só o varargs aceita 3 argumentos)
System.out.println(soma(1, 2));           // 3.0 (sobrecarga com double vence o varargs)
System.out.println(fatorial(5));          // 120
int a = 5;
int[] arr = {5};
alterar(a, arr);
System.out.println(a + " " + arr[0]);     // 5 0
```

**Explicação do exemplo**: `soma(1, 2, 3)` só casa com a versão de varargs. Em `soma(1, 2)`, os dois `int` se ampliam para `double`, e essa chamada é escolhida **antes** de se considerar varargs, por isso o resultado é `3.0`. `fatorial(5)` empilha cinco chamadas até o caso base e multiplica na volta (5 × 4 × 3 × 2 × 1 = 120). Em `alterar`, `x = 0` muda apenas a cópia local (o `a` continua 5); `v[0] = 0` altera o **mesmo array** que o chamador tem (por isso `arr[0]` vira 0); `v = new int[]{9}` só aponta a cópia local da referência para outro array, sem efeito fora do método.

## 1.10 Strings

**O que é**: `String` é a classe que representa **texto**, uma sequência de `char`. É **imutável**: depois de criada, o conteúdo nunca muda; todo método que "altera" (`toUpperCase`, `replace`, `concat`) devolve **uma nova** String e deixa a original intacta.

**Para que serve**: representar e manipular qualquer informação textual (nomes, mensagens, JSON, SQL, caminhos).

**Como funciona**:

- **String pool**: literais (`"java"`) ficam em uma área compartilhada; dois literais iguais apontam para o **mesmo** objeto. `new String("java")` força um objeto **novo**, fora do pool, e `intern()` devolve a versão do pool. A imutabilidade é o que torna esse compartilhamento seguro (e também a String segura como chave de `HashMap` e em segurança de *threads*).
- **`==` x `equals`**: `==` compara **referências**; `equals` compara **conteúdo**. Use sempre `equals` (ou `equalsIgnoreCase`). Com literal à esquerda evita `NullPointerException`: `"admin".equals(usuario)`.
- **Codificação**: internamente é UTF-16. `length()` conta **unidades de código** (`char`), então um emoji pode contar 2. Para contar caracteres reais, use `codePointCount`.
- **Métodos comuns**: `length`, `charAt(i)`, `substring(ini, fim)` (**fim exclusivo**), `indexOf`, `contains`, `startsWith`/`endsWith`, `replace`, `split(regex)`, `toUpperCase`/`toLowerCase`, `trim` e `strip` (Java 11; `strip` reconhece espaços Unicode), `isEmpty` (tamanho 0) e `isBlank` (Java 11; vazio ou só espaços), `repeat` (11), `join`, `compareTo` (ordem lexicográfica; negativo, 0, positivo), `format` / `formatted` (15).
- **Concatenação**: `+` funciona, mas em **laços** cria uma String nova a cada volta (custo O(n²)). Para montar texto aos poucos, use **`StringBuilder`**: buffer **mutável** (`append`, `insert`, `delete`, `reverse`, `toString`). **`StringBuffer`** é o equivalente **sincronizado** (seguro entre threads, porém mais lento); em código de uma só thread use `StringBuilder`.
- **Text blocks** (Java 15): `""" ... """` para texto multilinha (JSON, SQL). O compilador remove a **indentação comum** (a posição do delimitador de fechamento ajuda a controlá-la) e dispensa o escape de aspas.
- `split` trata o argumento como **regex** e descarta strings vazias no **final** do resultado. `switch` sobre String usa `equals`.

**Pontos importantes**: como `String` é imutável, nunca confie em `texto.trim();` sozinho (o resultado é descartado): atribua, `texto = texto.trim();`. Evite guardar senhas em `String` quando possível (ela fica no heap até o GC); prefira `char[]` que pode ser zerado.

**Exemplo**:

```java
String a = "java";
String b = "java";
String c = new String("java");
System.out.println(a == b);              // true  (mesmo literal no pool)
System.out.println(a == c);              // false (objeto novo fora do pool)
System.out.println(a.equals(c));         // true  (mesmo conteúdo)
System.out.println(a == c.intern());     // true  (intern devolve o do pool)

String nome = "  Maria  ";
nome.toUpperCase();                      // resultado descartado: nome não muda
System.out.println(nome.trim().toUpperCase());   // MARIA

StringBuilder sb = new StringBuilder();
for (int i = 0; i < 3; i++) sb.append(i).append(' ');
System.out.println(sb.toString().trim());        // 0 1 2

String json = """
        {"nome": "Ana"}
        """;
```

**Explicação do exemplo**: `a` e `b` referenciam o mesmo objeto do pool; `c` é outro objeto, por isso `==` dá `false` embora `equals` dê `true`. A chamada `nome.toUpperCase()` sem atribuição não faz nada útil, porque devolve uma nova String que é jogada fora; já `nome.trim().toUpperCase()` encadeia dois métodos e imprime `MARIA`. O `StringBuilder` altera o **mesmo** buffer a cada `append`, sem criar Strings intermediárias, e o resultado final é `0 1 2`. O text block guarda o JSON sem precisar escapar as aspas.

## 1.11 Classes wrapper, autoboxing e unboxing

**O que é**: cada tipo primitivo tem uma classe correspondente que o "embrulha" em um objeto: `byte→Byte`, `short→Short`, `int→Integer`, `long→Long`, `float→Float`, `double→Double`, `char→Character`, `boolean→Boolean`. **Autoboxing** é a conversão automática de primitivo para wrapper (`Integer x = 5;`); **unboxing** é o caminho inverso (`int y = x;`).

**Para que serve**: coleções e *generics* só trabalham com **objetos** (`List<Integer>`, não `List<int>`); wrappers também aceitam `null` ("ausência de valor") e trazem métodos e constantes úteis (`Integer.parseInt`, `Integer.MAX_VALUE`, `Character.isDigit`).

**Como funciona**:

- **Boxing** chama `Integer.valueOf(5)`; **unboxing** chama `x.intValue()`. O compilador insere essas chamadas.
- **Cache**: `valueOf` reaproveita objetos para valores pequenos: `Integer`, `Short`, `Long` e `Byte` de **-128 a 127**; `Character` de 0 a 127; `Boolean` sempre. Por isso `==` entre dois `Integer` pode dar `true` para 127 e `false` para 128. **Compare wrappers com `equals`** (ou unboxing: `a.intValue() == b.intValue()`).
- **Unboxing de `null`** lança `NullPointerException`: `Integer x = null; int y = x;`.
- **`equals` entre wrappers diferentes é `false`**: `Integer.valueOf(1).equals(1L)` é `false` (um é `Integer`, o outro `Long`).
- **Conversão de texto**: `Integer.parseInt("42")` devolve `int`; `Integer.valueOf("42")` devolve `Integer`. Texto inválido lança `NumberFormatException`. Idem `Double.parseDouble`, `Boolean.parseBoolean` (qualquer coisa diferente de "true", ignorando caixa, vira `false`: não valida).
- **Úteis**: `Integer.MAX_VALUE`/`MIN_VALUE`, `Integer.compare(a, b)`, `Integer.toBinaryString`, `Double.isNaN`, `Character.isDigit/isLetter/isUpperCase/toUpperCase`. Todos os wrappers numéricos estendem `Number`.
- **Construtores** (`new Integer(5)`) estão **obsoletos e marcados para remoção**: use `valueOf` ou autoboxing.

**Pontos importantes**: wrappers são **imutáveis**. Boxing em laços pesados aloca objetos e deixa o programa mais lento: para cálculo numérico intenso, prefira primitivos e *streams* primitivas (`IntStream`). Cuidado com expressões que misturam wrapper e primitivo: o unboxing pode ocorrer sem você perceber (e lançar NPE).

**Exemplo**:

```java
Integer a = 127, b = 127;
System.out.println(a == b);              // true  (valores do cache)
Integer c = 128, d = 128;
System.out.println(c == d);              // false (objetos diferentes)
System.out.println(c.equals(d));         // true  (compara o valor)

int n = Integer.parseInt("42");          // texto -> int
List<Integer> lista = new ArrayList<>();
lista.add(5);                            // autoboxing: int -> Integer
int primeiro = lista.get(0);             // unboxing: Integer -> int
System.out.println(Character.isDigit('7'));   // true
```

**Explicação do exemplo**: de -128 a 127, `valueOf` devolve sempre o mesmo objeto do cache, então `a == b` é `true`; para 128 são criados dois objetos distintos e o `==` (que compara referências) dá `false`, enquanto `equals` compara os valores e dá `true`. Esse comportamento é a razão da regra "compare wrappers com `equals`". Em `lista.add(5)`, o `int` é convertido em `Integer` automaticamente, e `lista.get(0)` faz o caminho inverso.

## 1.12 Entrada e saída no console

**O que é**: o Java expõe três fluxos padrão: **`System.out`** (saída), **`System.err`** (saída de erros) e **`System.in`** (entrada, em bytes). Para ler texto de forma prática, usa-se `Scanner` ou `BufferedReader`.

**Para que serve**: interagir com o usuário em programas de console, exibir resultados e mensagens de diagnóstico e receber dados em ferramentas de linha de comando.

**Como funciona**:

- **Saída**: `println` (com quebra de linha), `print` (sem), `printf`/`format` (texto formatado). Principais marcadores de formato: `%d` (inteiro), `%s` (texto), `%f` (decimal; `%.2f` = 2 casas), `%n` (quebra de linha da plataforma), `%,d` (separador de milhar), `%5d` (largura 5), `%-10s` (texto alinhado à esquerda). **Atenção**: `%.2f` usa o **`Locale` padrão** da JVM (em pt-BR sai `1234,50`); para forçar o formato, use `String.format(Locale.US, ...)`.
- **`Scanner`**: divide a entrada em *tokens* separados por espaços e quebras de linha. `nextLine()` lê a **linha inteira**; `nextInt()`, `nextDouble()` e `next()` leem o **próximo token**; `hasNext()`, `hasNextInt()` testam antes de ler (evita `InputMismatchException`).
- **Pegadinha do buffer**: `nextInt()` lê o número, mas **deixa a quebra de linha** no buffer. O `nextLine()` seguinte lê o que restou da linha (vazio) em vez de esperar nova digitação. Soluções: chamar um `nextLine()` extra para descartar a sobra ou, mais simples, ler tudo com `nextLine()` e converter com `Integer.parseInt`.
- `nextDouble()` também depende do `Locale`: em pt-BR espera vírgula decimal (`3,5`).
- **Fechar**: `Scanner` sobre arquivo deve usar *try-with-resources*; fechar o `Scanner(System.in)` **fecha o `System.in`** para o resto do programa.
- **`BufferedReader(new InputStreamReader(System.in))`** com `readLine()` é uma alternativa mais rápida para muitos dados (não faz conversão por tipo).
- **Argumentos de linha de comando**: o array `args` de `main` (`java Programa a b` → `args[0]` é `"a"`).
- **Java 25**: `IO.println(...)` e `IO.readln("prompt")` (classe `java.io.IO`) simplificam o console; em classes comuns exigem `import java.io.IO;`.

**Exemplo**:

```java
Scanner sc = new Scanner(System.in);
System.out.print("Nome: ");
String nome = sc.nextLine();                       // lê a linha inteira
System.out.print("Idade: ");
int idade = Integer.parseInt(sc.nextLine());       // lê linha e converte (evita a pegadinha)
System.out.printf("%s tem %d anos e %,.2f reais%n", nome, idade, 1234.5);
System.err.println("isto vai para o fluxo de erro");
```

**Explicação do exemplo**: se o usuário digitar `Ana` e `30`, o programa imprime `Ana tem 30 anos e 1.234,50 reais` em um computador configurado para pt-BR (ou `1,234.50` em en-US, porque `%,.2f` segue o `Locale` padrão). Ler a idade com `nextLine()` + `parseInt` evita deixar uma quebra de linha no buffer. A mensagem em `System.err` é separada da saída normal: no terminal aparece junto, mas pode ser redirecionada para outro destino (`2> erros.txt`).

---

# Parte 2: Orientação a objetos

## 2.1 Classes e objetos

**O que é**: **classe** é o molde (o *tipo*) que descreve como são e o que fazem as coisas de uma categoria: declara os **atributos** (dados, o estado) e os **métodos** (ações, o comportamento). **Objeto** é uma **instância** desse molde, criada com `new`, com espaço próprio no heap e **valores próprios** para os atributos. A classe `Carro` é a ideia de carro; cada `new Carro()` é um carro concreto, com seu nome, seu ano, sua velocidade. Todo objeto tem **identidade** (é distinto dos outros, mesmo que os dados sejam iguais), **estado** (os valores dos atributos) e **comportamento** (os métodos).

**Para que serve**: modelar o domínio do problema agrupando, no mesmo lugar, os dados e as regras que operam sobre eles, em vez de variáveis e funções soltas. Esse é o ponto de partida dos quatro pilares da orientação a objetos: **abstração** (representar só o que importa), **encapsulamento** (proteger o estado), **herança** (reaproveitar e especializar) e **polimorfismo** (tratar tipos diferentes pela mesma interface).

**Como funciona**:

- **Criação com `new`**: (1) reserva-se memória no heap para o objeto; (2) todos os atributos recebem o valor padrão do tipo (`0`, `false`, `null`); (3) executam-se os inicializadores e o **construtor**; (4) `new` devolve a **referência** (o endereço) do objeto, que fica guardada na variável.
- **Variável de tipo objeto guarda uma referência, não o objeto.** `Carro b = a;` copia a **referência**: as duas variáveis apontam para o **mesmo** objeto, e uma alteração feita por uma aparece pela outra. Só `new` cria outro objeto.
- **`null`** significa "esta referência não aponta para objeto nenhum". Acessar atributo ou método por uma referência `null` lança `NullPointerException`.
- **Membros**: *atributos* (campos), *métodos*, *construtores* e tipos aninhados. Membros declarados sem modificador de acesso são visíveis só no mesmo pacote (tópico 2.4).
- **Objeto sem referência** (ninguém mais aponta para ele) torna-se elegível para o *garbage collector*.
- Um arquivo pode conter várias classes, mas só uma `public`, com o nome do arquivo.

**Pontos importantes**:

- **Classe x objeto**: a classe é única e existe em tempo de compilação; objetos são muitos e existem em tempo de execução.
- **Identidade x igualdade**: `a == b` pergunta "é o *mesmo* objeto?"; `a.equals(b)` pergunta "têm o *mesmo conteúdo*?" (depende de a classe sobrescrever `equals`, tópico 2.16).
- Atributos `public` (como neste primeiro exemplo) permitem estado inválido (`carro.ano = -500`); o encapsulamento (2.4) resolve isso.

**Exemplo**:

```java
class Carro {
    String nome;                         // atributo (estado)
    int ano;
    void ligar() {                       // método (comportamento)
        System.out.println(nome + " ligado");
    }
}

Carro a = new Carro();                   // cria um objeto; 'a' guarda a referência
a.nome = "Fusca";
Carro b = a;                             // copia a REFERÊNCIA (não o objeto)
b.nome = "Gol";
System.out.println(a.nome);              // Gol  (mesmo objeto)
Carro c = new Carro();                   // outro objeto, independente
System.out.println(c.nome);              // null (valor padrão)
System.out.println(a == b);              // true  (mesma identidade)
System.out.println(a == c);              // false
```

**Explicação do exemplo**: `new Carro()` cria o primeiro objeto e `a` recebe seu endereço. Em `Carro b = a`, **não nasce um segundo carro**: `a` e `b` são dois nomes para o mesmo objeto, então mudar `b.nome` muda o que `a.nome` mostra (`Gol`). `c` é um objeto novo, com `nome` ainda `null`, por isso `a == c` é `false` e `a == b` é `true`.

## 2.2 Construtores

**O que é**: método especial executado na criação do objeto (`new`), com o **mesmo nome da classe** e **sem tipo de retorno** (nem `void`). Sua função é deixar o objeto **em um estado válido** desde o primeiro instante.

**Para que serve**: garantir que os dados obrigatórios sejam informados, validar valores iniciais e evitar objetos "pela metade" (sem nome, sem identificador).

**Como funciona**:

- **Construtor padrão**: se a classe **não declara nenhum** construtor, o compilador cria um **sem parâmetros** e sem corpo. Ao declarar **qualquer** construtor, o padrão **deixa de existir**: se ainda quiser criar sem argumentos, declare-o você mesmo (erro comum ao adicionar um construtor e quebrar `new Classe()` em outros pontos).
- **Sobrecarga**: vários construtores com parâmetros diferentes; o compilador escolhe pelos argumentos do `new`.
- **Encadeamento com `this(...)`**: um construtor chama outro da **mesma** classe, evitando repetir a inicialização. O construtor mais completo costuma ser o "principal" e os demais delegam a ele.
- **Toda criação passa pela superclasse**: a primeira ação de um construtor é chamar `super(...)`; se você não escrever, o compilador insere `super()` (e falha se o pai não tiver construtor sem argumentos). Por isso um objeto é construído **de cima para baixo** na hierarquia.
- **Ordem dentro de um objeto**: valores padrão → inicializadores de campo e blocos de instância (na ordem em que aparecem) → corpo do construtor.
- Atributos **`final`** devem receber valor até o fim do construtor.
- Construtores **não são herdados**. Um construtor `private` impede criar objetos de fora (usado em classes utilitárias, *singletons* e fábricas).
- Quando há muitos parâmetros opcionais, o padrão **Builder** (tópico 6.3) é mais legível que muitos construtores.

**Pontos importantes**: um construtor pode lançar exceção para impedir a criação de objeto inválido. Evite chamar métodos sobrescritíveis dentro do construtor (a versão da subclasse pode rodar antes de os campos dela serem inicializados).

**Exemplo**:

```java
class Pessoa {
    private final String nome;
    private final int idade;

    Pessoa(String nome, int idade) {            // construtor principal
        if (nome == null || nome.isBlank()) throw new IllegalArgumentException("nome obrigatório");
        if (idade < 0) throw new IllegalArgumentException("idade inválida");
        this.nome = nome;
        this.idade = idade;
    }
    Pessoa(String nome) {                       // sobrecarga: delega ao principal
        this(nome, 0);
    }
}

Pessoa p = new Pessoa("Ana", 30);
Pessoa recemNascida = new Pessoa("Bia");        // idade 0
// new Pessoa();                                // não compila: não existe construtor sem argumentos
```

**Explicação do exemplo**: o construtor principal valida os dados antes de atribuí-los, então é impossível existir uma `Pessoa` sem nome ou com idade negativa. O segundo construtor não repete a lógica: `this(nome, 0)` redireciona para o principal. A linha comentada não compila porque, ao declarar construtores, perdemos o padrão sem argumentos. Como os campos são `final`, ficam definidos para sempre no construtor.

## 2.3 As palavras `this` e `super`

**O que é**: `this` é a **referência ao objeto atual** (aquele em cujo método ou construtor o código está executando). `super` é a forma de acessar a **parte da superclasse** do objeto atual.

**Para que serve**: `this` resolve ambiguidades (atributo x parâmetro de mesmo nome), permite passar o próprio objeto adiante, devolvê-lo (`return this`, base dos *builders* fluentes) e encadear construtores. `super` reaproveita a inicialização e o comportamento do pai.

**Como funciona**:

- **`this.campo`**: em `this.nome = nome;`, o da esquerda é o atributo e o da direita é o parâmetro (que "sombreia" o atributo). Sem `this`, a atribuição seria do parâmetro para ele mesmo.
- **`this(...)`**: chama outro construtor da mesma classe (tópico 2.2). **`super(...)`**: chama um construtor da superclasse. Não podem aparecer os dois no mesmo construtor. Antes do Java 25, tinham de ser a **primeira instrução**; o Java 25 (JEP 513, *flexible constructor bodies*) permite código que **não use `this`** antes dessa chamada, por exemplo validar argumentos.
- **`super.metodo()`**: executa a versão do **pai** de um método sobrescrito (acrescentar ao comportamento em vez de substituí-lo).
- **`super.campo`**: acessa um atributo do pai que foi "escondido" por um atributo de mesmo nome na subclasse (situação a evitar).
- **Não existe `this` em contexto `static`** (não há objeto atual).

**Pontos importantes**: `this` e `super` não são variáveis que você declara; são palavras reservadas. Dentro de uma classe interna, `Externa.this` identifica o objeto da classe externa (tópico 2.14).

**Exemplo**:

```java
class Animal {
    protected String nome;
    Animal(String nome) { this.nome = nome; }
    String som() { return "..."; }
}
class Cao extends Animal {
    Cao(String nome) {
        super(nome);                         // inicializa a parte Animal do objeto
    }
    @Override
    String som() {
        return super.som() + " Au!";         // reaproveita o do pai e acrescenta
    }
}
System.out.println(new Cao("Rex").som());    // ... Au!
```

**Explicação do exemplo**: ao criar `new Cao("Rex")`, o construtor de `Cao` chama `super(nome)`, que guarda o nome na parte `Animal` do objeto. Em `som()`, `super.som()` executa a versão do pai (`"..."`) e o filho concatena `  Au! `, produzindo `... Au!`. Sem `super.`, a chamada `som()` dentro de `som()` seria recursiva.

## 2.4 Encapsulamento e modificadores de acesso

**O que é**: esconder o **estado interno** do objeto e permitir o acesso somente por métodos controlados. Os atributos ficam `private`; a leitura e a escrita passam por **getters** (devolvem o valor) e **setters** (alteram), que podem **validar** o que entra.

**Para que serve**: proteger as **invariantes** da classe (regras que devem valer sempre, como "saldo nunca negativo"), reduzir o acoplamento (quem usa a classe depende do que ela *faz*, não de como guarda os dados) e permitir mudar a implementação interna sem quebrar quem a usa.

**Como funciona**: os quatro níveis de acesso, do mais restrito ao mais aberto:

| Modificador | Mesma classe | Mesmo pacote | Subclasse (outro pacote) | Qualquer lugar |
| --- | --- | --- | --- | --- |
| `private` | sim | não | não | não |
| (nenhum, *package-private*) | sim | sim | não | não |
| `protected` | sim | sim | sim | não |
| `public` | sim | sim | sim | sim |

- Uma classe de **nível superior** só pode ser `public` ou *package-private*; `private`/`protected` valem para membros e classes internas.
- `private` é por **classe**, não por objeto: um método da classe `Conta` pode acessar o `saldo` privado de **outra** `Conta`.
- **Convenção de nomes**: `getNome()`/`setNome(...)`; para `boolean`, `isAtivo()`. (Frameworks como Spring, JPA e Jackson contam com esse padrão *JavaBeans*.)
- **Regra prática**: comece com tudo `private` e abra só o que for necessário. Nem todo atributo precisa de setter; **imutabilidade** (campos `final`, sem setters) é o nível mais forte de proteção e é naturalmente segura entre *threads*.
- **Cópia defensiva**: um getter que devolve um array ou uma coleção **mutável** entrega a referência ao estado interno, e quem receber pode alterá-lo. Devolva uma cópia ou uma visão imutável (`List.copyOf`, `Collections.unmodifiableList`).

**Pontos importantes**: encapsulamento não é "colocar getter e setter em tudo" (isso apenas reexpõe os dados). O objetivo é oferecer **operações com significado** (`depositar`, `sacar`) em vez de expor o campo `saldo`. Em equipes, o Lombok (`@Getter`, `@Setter`, `@Value`) gera o código repetitivo, e `record` (2.13) resolve o caso de classes só de dados.

**Exemplo**:

```java
class Conta {
    private double saldo;                         // ninguém de fora mexe direto
    private final List<String> historico = new ArrayList<>();

    public void depositar(double valor) {
        if (valor <= 0) throw new IllegalArgumentException("Valor inválido");
        saldo += valor;
        historico.add("depósito " + valor);
    }
    public double getSaldo() { return saldo; }
    public List<String> getHistorico() {
        return List.copyOf(historico);            // cópia imutável: protege o estado interno
    }
}

Conta c = new Conta();
c.depositar(100);
// c.saldo = 1_000_000;                           // erro de compilação: saldo é private
// c.depositar(-5);                               // lançaria IllegalArgumentException
System.out.println(c.getSaldo());                 // 100.0
```

**Explicação do exemplo**: `saldo` só muda por `depositar`, que impõe a regra "valor positivo"; não há como colocar a conta em estado inválido. O getter do histórico devolve `List.copyOf`, então quem recebe a lista não consegue adicionar ou remover itens do histórico real (tentar lançaria `UnsupportedOperationException`).

## 2.5 Membros `static` e blocos de inicialização

**O que é**: um membro `static` pertence à **classe**, não a cada objeto: existe **uma única cópia**, compartilhada por todas as instâncias, acessada por `Classe.membro`. Um método `static` não tem `this` e, por isso, não acessa diretamente membros de instância.

**Para que serve**: constantes (`static final`), utilitários sem estado (`Math.sqrt`), contadores e *caches* compartilhados, métodos de fábrica (`List.of`) e inicialização única de recursos.

**Como funciona**:

- **Atributo `static`**: criado quando a classe é carregada e compartilhado. Alterá-lo por qualquer objeto afeta todos. Acesse sempre pelo nome da classe (acessar por uma instância compila, mas confunde).
- **Método `static`**: pode ser chamado sem criar objeto; só enxerga outros membros `static` (ou precisa receber um objeto como parâmetro). **Não** é polimórfico: não sofre sobrescrita, apenas *ocultação* (*hiding*).
- **Constante**: `static final` com nome em MAIÚSCULAS; se for primitivo/`String` com valor conhecido em compilação, o valor é **copiado** para quem a usa.
- **Bloco de instância** `{ ... }`: roda a **cada** `new`, antes do corpo do construtor. **Bloco estático** `static { ... }`: roda **uma vez**, quando a classe é carregada pela primeira vez (ao criar o primeiro objeto, chamar um método estático ou acessar um atributo estático não constante).
- **Ordem ao criar um objeto de subclasse** (a primeira vez): estáticos da superclasse → estáticos da subclasse → (campos, blocos de instância, construtor) da superclasse → (campos, blocos de instância, construtor) da subclasse. Nas criações seguintes, os estáticos **não** rodam de novo.
- **Classe utilitária**: só métodos estáticos e construtor `private` (impede instanciar).

**Pontos importantes**: estado estático **mutável** é estado **global**: dificulta testes (um teste afeta o outro) e é perigoso com várias *threads* (exige sincronização). Quando o dado varia de objeto para objeto, **não** deve ser `static`. Importação estática (`import static java.lang.Math.*;`) permite usar membros sem o nome da classe.

**Exemplo**:

```java
class Pai {
    static { System.out.println("1. static Pai"); }
    { System.out.println("3. bloco Pai"); }
    Pai() { System.out.println("4. construtor Pai"); }
}
class Filho extends Pai {
    static int criados = 0;                       // compartilhado por todos os Filho
    static { System.out.println("2. static Filho"); }
    { System.out.println("5. bloco Filho"); }
    Filho() { criados++; System.out.println("6. construtor Filho"); }
}

new Filho();
new Filho();
System.out.println(Filho.criados);               // 2
```

**Explicação do exemplo**: na primeira criação, a saída segue a ordem 1, 2, 3, 4, 5, 6: primeiro os blocos estáticos (pai, depois filho), depois, de cima para baixo, o bloco de instância e o construtor do pai e, por fim, os do filho. Na **segunda** criação aparecem só 3, 4, 5 e 6, pois os estáticos já foram executados. `criados` é compartilhado, então termina em 2.

## 2.6 Herança

**O que é**: mecanismo pelo qual uma classe (**subclasse**) herda atributos e métodos de outra (**superclasse**) por meio de `extends`, podendo acrescentar membros novos e **especializar** o comportamento herdado. Em Java, uma classe tem **uma única** superclasse direta (herança simples), e toda classe herda, direta ou indiretamente, de `Object`.

**Para que serve**: reaproveitar código comum e modelar hierarquias de especialização (`Funcionario` → `Gerente`), de modo que a subclasse possa ser usada em qualquer lugar onde a superclasse é esperada (base do polimorfismo, tópico 2.8).

**Como funciona**:

- A subclasse **herda** os membros acessíveis da superclasse (`public`, `protected` e, no mesmo pacote, *package-private*). Os `private` existem no objeto, mas não são acessíveis diretamente (use getters).
- **Construtores não são herdados**: a subclasse chama o do pai com `super(...)` (tópicos 2.2 e 2.3).
- A subclasse pode **sobrescrever** métodos (tópico 2.7) e adicionar os seus.
- A relação deve ser **"é um"** (*is-a*): um `Gerente` **é um** `Funcionario`. Vale o **Princípio de Substituição de Liskov**: onde se espera o pai, a subclasse deve funcionar **sem surpresas**.
- **Herança x composição**: herança cria acoplamento forte (a subclasse depende de detalhes do pai; mudar o pai pode quebrar os filhos, o *fragile base class problem*). Para apenas reaproveitar código, prefira **composição** ("tem um", tópico 2.15).
- `final` em uma classe impede estendê-la; `abstract` obriga a estendê-la (tópicos 2.9 e 2.11).
- Atributos de mesmo nome na subclasse **escondem** (não sobrescrevem) os do pai: evite.

**Pontos importantes**: herança múltipla de **classes** não existe em Java (evita o problema do diamante); a solução é implementar várias **interfaces** (2.10). Hierarquias profundas são difíceis de entender: prefira poucos níveis.

**Exemplo**:

```java
class Funcionario {
    protected double salario;                     // visível para subclasses
    Funcionario(double salario) { this.salario = salario; }
    double bonus() { return salario * 0.05; }
}
class Gerente extends Funcionario {
    Gerente(double salario) { super(salario); }
    @Override
    double bonus() { return salario * 0.20; }     // especializa o comportamento
}

Funcionario f = new Gerente(10_000);               // um Gerente É UM Funcionario
System.out.println(f.bonus());                     // 2000.0
System.out.println(f instanceof Funcionario);      // true
```

**Explicação do exemplo**: `Gerente` herda o atributo `salario` e o construtor do pai é chamado por `super(salario)`. A variável `f` tem o tipo do pai, mas aponta para um `Gerente`; ao chamar `bonus()`, a JVM executa a versão do **objeto real** (a do `Gerente`), e o resultado é 20% de 10.000, ou seja, `2000.0`. Isso funciona porque um `Gerente` pode ser usado em qualquer lugar em que um `Funcionario` é esperado.

## 2.7 Sobrescrita (*override*) e sobrecarga (*overload*)

**O que é**: dois mecanismos que muita gente confunde, porque os nomes são parecidos e ambos envolvem "métodos de mesmo nome".

- **Sobrecarga (*overload*)**: **na mesma classe** (ou herdada), métodos com o **mesmo nome** e **parâmetros diferentes** (quantidade, tipos ou ordem). É resolvida em **tempo de compilação**, pelo **tipo declarado** dos argumentos.
- **Sobrescrita (*override*)**: a **subclasse** redefine um método herdado, com a **mesma assinatura**. É resolvida em **tempo de execução**, pelo **tipo real** do objeto.

**Para que serve**: a sobrecarga oferece várias formas convenientes de chamar a mesma operação (`println(int)`, `println(String)`...). A sobrescrita permite que cada subclasse tenha seu próprio comportamento para a mesma operação, o que sustenta o polimorfismo (tópico 2.8).

**Como funciona**:

|  | Sobrecarga | Sobrescrita |
| --- | --- | --- |
| Onde ocorre | mesma classe/hierarquia | subclasse redefine método do pai |
| Assinatura | **diferente** (parâmetros) | **igual** |
| Tipo de retorno | pode variar (não conta como diferença) | igual ou **covariante** (um subtipo) |
| Escolha feita | compilação (tipo **declarado**) | execução (tipo **real**) |

Regras da **sobrescrita**:

- Mesmo nome e mesmos tipos de parâmetros.
- **Retorno** igual ou um **subtipo** (covariância).
- **Acesso igual ou mais amplo** (não dá para reduzir `public` para `protected`).
- **Exceções *checked*** iguais, subtipos ou nenhuma (nunca novas ou mais amplas); exceções *unchecked* são livres.
- Não se sobrescrevem métodos `private` (não são visíveis), `static` (apenas *ocultação*, sem polimorfismo) nem `final`.
- Use sempre **`@Override`**: o compilador acusa erro se a assinatura não bater com nada herdado (protege contra erro de digitação que criaria, sem querer, uma *sobrecarga* em vez de uma sobrescrita).

**Pontos importantes**: a armadilha clássica é misturar os dois: com métodos sobrecarregados, a versão escolhida depende do **tipo da variável**, não do objeto. Veja o exemplo.

**Exemplo**:

```java
class Animal { String som() { return "..."; } }
class Gato extends Animal {
    @Override String som() { return "Miau"; }      // SOBRESCRITA
}

class Veterinario {
    String examinar(Animal a) { return "animal genérico"; }   // SOBRECARGA
    String examinar(Gato g)   { return "gato"; }
}

Animal a = new Gato();
System.out.println(a.som());                     // Miau  (tipo REAL decide: sobrescrita)
Veterinario v = new Veterinario();
System.out.println(v.examinar(a));               // animal genérico  (tipo DECLARADO decide: sobrecarga)
System.out.println(v.examinar(new Gato()));      // gato
```

**Explicação do exemplo**: `a.som()` devolve `Miau` porque a sobrescrita é decidida na execução, olhando o objeto real (um `Gato`). Já `v.examinar(a)` chama a versão para `Animal`: como `examinar` é **sobrecarregado**, o compilador escolhe pela variável `a`, cujo tipo declarado é `Animal`, mesmo que o objeto seja um `Gato`. Só na última linha o argumento tem tipo `Gato`.

## 2.8 Polimorfismo

**O que é**: capacidade de tratar objetos de **tipos diferentes pela mesma interface**. Uma variável do tipo pai (classe ou interface) pode apontar para um objeto de **qualquer subtipo**, e a versão do método executada é a do **objeto real**, escolhida em **tempo de execução** (*dynamic dispatch*). "Polimorfismo" significa "muitas formas": uma chamada, vários comportamentos.

**Para que serve**: escrever código **genérico** que funciona para qualquer subtipo, **sem `if/else` por tipo**, e estender o sistema com novos tipos **sem alterar** o código que já existe (princípio Aberto/Fechado). É a base de *Strategy*, injeção de dependência e da maior parte dos frameworks.

**Como funciona**:

- **Upcast** (filho → pai) é automático e seguro: `Animal a = new Gato();`. Com o tipo `Animal`, só se enxergam os métodos declarados em `Animal`.
- **Despacho dinâmico**: ao chamar `a.som()`, a JVM procura a implementação na classe real do objeto (`Gato`), não na do tipo da variável. Internamente isso usa uma tabela de métodos virtuais (*vtable*).
- **Downcast** (pai → filho) exige cast e pode lançar `ClassCastException`; proteja com `instanceof` (com *pattern matching*: `if (a instanceof Gato g) { g.miar(); }`).
- **O que NÃO é polimórfico**: atributos (o acesso depende do tipo da variável), métodos `static` e `private`, construtores, e a escolha entre **sobrecargas** (tópico 2.7).
- **Tipos de polimorfismo**: de **subtipo** (este tópico, por herança ou interface), **paramétrico** (generics, tópico 3.8) e **ad hoc** (sobrecarga).
- **Programe para a interface, não para a implementação**: declare `List<String> l = new ArrayList<>();`, `Repositorio r = new RepositorioJpa();`. Assim, trocar a implementação não altera o código que a usa.

**Pontos importantes**: se você escreve `if (obj instanceof A) ... else if (obj instanceof B) ...` repetidamente, provavelmente falta um método polimórfico nesses tipos. Já para tipos fechados (`sealed`), o `switch` com *pattern matching* (2.19) é uma alternativa legítima.

**Exemplo**:

```java
interface Pagamento { double calcularTaxa(double valor); }

class Pix     implements Pagamento { public double calcularTaxa(double v) { return 0; } }
class Cartao  implements Pagamento { public double calcularTaxa(double v) { return v * 0.03; } }
class Boleto  implements Pagamento { public double calcularTaxa(double v) { return 3.50; } }

List<Pagamento> formas = List.of(new Pix(), new Cartao(), new Boleto());
for (Pagamento p : formas) {
    System.out.println(p.getClass().getSimpleName() + ": " + p.calcularTaxa(200));
}
```

**Explicação do exemplo**: o laço trata todos como `Pagamento` e chama `calcularTaxa`, sem saber qual é a classe concreta. A saída é `Pix: 0.0`, `Cartao: 6.0` e `Boleto: 3.5`: cada objeto executa a **sua** versão. Para adicionar `Cripto`, basta criar uma nova classe que implemente `Pagamento`; o laço (e qualquer código que use `Pagamento`) permanece intacto.

## 2.9 Classes abstratas

**O que é**: classe declarada `abstract`, que **não pode ser instanciada** com `new`. Serve de **base incompleta** para outras classes. Pode ter atributos, construtores, métodos concretos e **métodos abstratos**: declarados sem corpo (`abstract double area();`), que toda subclasse **concreta** é obrigada a implementar.

**Para que serve**: representar um **conceito genérico** que só faz sentido por meio de suas versões concretas ("Forma" não existe sozinha, só Círculo, Quadrado), **compartilhando estado e código comum** e impondo às subclasses o que elas precisam implementar. É a base do padrão **Template Method**: a classe abstrata define o esqueleto de um algoritmo e deixa alguns passos para as subclasses.

**Como funciona**:

- Se uma classe tem **pelo menos um** método abstrato, ela **deve** ser `abstract`. O inverso não vale: uma classe abstrata pode não ter nenhum método abstrato.
- A subclasse concreta deve implementar **todos** os abstratos herdados; senão, também precisa ser `abstract`.
- Métodos abstratos não podem ser `private`, `static` nem `final` (contradizem a ideia de serem implementados pela subclasse).
- Tem **construtor**, chamado por `super(...)` pelas subclasses, e pode ter atributos `protected`/`private`.
- Pode-se declarar variáveis do tipo abstrato (`Forma f = new Circulo(2);`), só não instanciá-lo.
- Cuidado: chamar um método abstrato **dentro do construtor** da classe abstrata executa a versão da subclasse **antes** de os campos dela serem inicializados.

**Quando usar / diferenças**: classe abstrata x interface:

|  | Classe abstrata | Interface |
| --- | --- | --- |
| Estado (atributos de instância) | sim | não (só constantes) |
| Construtor | sim | não |
| Herança | uma só (`extends`) | várias (`implements`) |
| Ideia | "é um" com base comum | "sabe fazer" (capacidade/contrato) |

Use abstrata quando há **estado e código reaproveitável** entre classes muito relacionadas; use interface para definir **contratos** que classes não relacionadas possam cumprir.

**Exemplo**:

```java
abstract class Relatorio {
    // Template Method: define o esqueleto; não pode ser sobrescrito
    public final void gerar() {
        System.out.println(cabecalho());
        System.out.println(corpo());          // passo que varia
        System.out.println("-- fim --");
    }
    protected String cabecalho() { return "RELATÓRIO"; }   // padrão reaproveitável
    protected abstract String corpo();                     // cada subclasse implementa
}
class RelatorioVendas extends Relatorio {
    @Override protected String corpo() { return "Vendas: 10"; }
}

// new Relatorio();                           // erro: classe abstrata não instancia
new RelatorioVendas().gerar();
```

**Explicação do exemplo**: `gerar()` fixa a **ordem** dos passos (cabeçalho, corpo, rodapé) e é `final` para que ninguém quebre o algoritmo. O cabeçalho tem implementação padrão, e o `corpo()` é abstrato: `RelatorioVendas` é obrigada a fornecê-lo. A saída é `RELATÓRIO`, `Vendas: 10` e `-- fim --`.

## 2.10 Interfaces

**O que é**: um **contrato** que define **o que** uma classe deve fazer (os métodos), sem dizer **como**. Uma classe **implementa** (`implements`) uma ou **várias** interfaces, e uma interface pode **estender** várias outras (`extends A, B`). É a forma de Java obter polimorfismo sem herança múltipla de classes.

**Para que serve**: **desacoplar**: quem usa a interface não conhece a implementação (injeção de dependência, testes com *mocks*, trocar banco ou serviço sem alterar o resto). Também define **papéis** que classes sem relação de herança podem cumprir (`Comparable`, `Runnable`, `Closeable`).

**Como funciona**:

- **Métodos abstratos**: são `public abstract` por padrão (escrever os modificadores é redundante).
- **Constantes**: campos de interface são implicitamente `public static final`.
- **`default` (Java 8)**: método **com implementação**, herdado pelos implementadores (que podem sobrescrever). Foi criado para **evoluir** interfaces existentes sem quebrar todas as classes que já as implementam.
- **`static` (Java 8)**: método chamado pela própria interface (`Pagamento.padrao()`); **não é herdado** pelas classes.
- **`private` (Java 9)**: reaproveita código entre métodos `default`/`static`.
- **Conflito de `default`** (problema do diamante): se uma classe herda dois `default` de mesma assinatura, é **obrigada a sobrescrever**, podendo escolher uma versão com `Interface.super.metodo()`. Regras de resolução: a **classe vence** a interface; a interface **mais específica** vence a mais genérica.
- **Interface funcional**: **exatamente um** método abstrato; `@FunctionalInterface` faz o compilador garantir; pode ser implementada por **lambda** (tópico 4.1). **Interface marcadora**: sem métodos (`Serializable`).
- **`sealed interface`** restringe quem pode implementá-la (tópico 2.19).
- Interfaces **não têm construtor nem atributos de instância**.

**Pontos importantes**: métodos `default` não devem guardar estado nem virar "classe abstrata disfarçada". Uma interface muito grande obriga implementadores a criar métodos vazios (violação do ISP, tópico 6.1): prefira várias interfaces pequenas.

**Exemplo**:

```java
interface Notificador {
    void enviar(String mensagem);                   // abstrato
    default void enviarUrgente(String m) {          // default: reaproveita enviar
        enviar("[URGENTE] " + m);
    }
}
interface Auditavel {
    default String origem() { return "auditavel"; }
}
interface Rastreavel {
    default String origem() { return "rastreavel"; }
}

class Email implements Notificador, Auditavel, Rastreavel {
    public void enviar(String m) { System.out.println("Email: " + m); }
    @Override public String origem() {              // conflito: obrigado a resolver
        return Auditavel.super.origem();
    }
}

Notificador n = new Email();
n.enviarUrgente("servidor fora do ar");             // Email: [URGENTE] servidor fora do ar
System.out.println(new Email().origem());           // auditavel
```

**Explicação do exemplo**: `Email` só precisou implementar `enviar`; `enviarUrgente` vem pronto da interface e chama `enviar`. As duas interfaces `Auditavel` e `Rastreavel` oferecem `origem()` com a mesma assinatura, o que gera ambiguidade, e o compilador **exige** que `Email` a resolva: aqui, escolhendo a versão de `Auditavel`.

## 2.11 O modificador `final`

**O que é**: `final` significa "não pode mudar depois de definido". O efeito depende de onde é aplicado.

**Para que serve**: criar **constantes**, garantir **imutabilidade** (que torna objetos seguros para compartilhar entre *threads*), impedir que um comportamento seja alterado por subclasses e documentar a intenção do código.

**Como funciona**:

| Onde | Efeito |
| --- | --- |
| **Variável local / parâmetro** | recebe valor uma só vez; não pode ser reatribuída |
| **Atributo** | deve ser atribuído **uma vez**: na declaração, em bloco de inicialização ou até o fim do construtor (*blank final*) |
| **`static final`** | constante da classe (por convenção, `MAIUSCULAS`) |
| **Método** | **não pode ser sobrescrito** |
| **Classe** | **não pode ser estendida** (`String`, `Integer`, `Math`) |

- **`final` trava a *referência*, não o objeto**: `final List<String> l = new ArrayList<>();` impede `l = outra`, mas `l.add("x")` continua permitido. Imutabilidade real exige campos `final` **e** tipos imutáveis (ou cópias defensivas) e nenhum setter.
- **Efetivamente final**: uma variável local que **nunca é reatribuída**, mesmo sem a palavra `final`. É a condição para ser **capturada** por lambdas e classes anônimas/locais.
- Constantes `static final` de tipo primitivo/`String` com valor conhecido na compilação são **copiadas** nos pontos de uso: mudar o valor exige recompilar quem a usa.

**Pontos importantes**: `final` em parâmetros e variáveis locais é opcional e questão de estilo; já `final` em campos é uma ótima prática sempre que possível. Não confunda com `finally` (bloco de exceções) nem com `finalize` (método obsoleto).

**Exemplo**:

```java
class Config {
    static final int MAX_TENTATIVAS = 3;                // constante da classe
    private final List<String> itens = new ArrayList<>();
    private final String nome;

    Config(String nome) { this.nome = nome; }           // atribuído uma única vez

    void teste() {
        itens.add("x");                                 // OK: o OBJETO continua mutável
        // itens = new ArrayList<>();                   // erro: a REFERÊNCIA é final
        // nome = "outro";                              // erro
        int contador = 0;                                // efetivamente final (nunca muda)
        Runnable r = () -> System.out.println(contador); // pode ser capturada
    }
}
```

**Explicação do exemplo**: `MAX_TENTATIVAS` é uma constante. O campo `nome` só recebe valor no construtor. O campo `itens` é `final`, mas a lista que ele referencia pode receber elementos; `final` não torna o conteúdo imutável. `contador` nunca é reatribuída, portanto é *efetivamente final* e a lambda pode usá-la; se você fizesse `contador++` depois, a lambda deixaria de compilar.

## 2.12 Enumerações (`enum`)

**O que é**: tipo cujos valores formam um **conjunto fixo e conhecido de constantes**. Por baixo, um `enum` é uma **classe** especial (implicitamente `final`, que estende `java.lang.Enum`) em que cada constante é **uma instância única**, criada uma vez quando a classe é carregada.

**Para que serve**: representar categorias e estados (status do pedido, dias da semana, tipos de pagamento) com **segurança de tipos**. Em vez de `int` (`1`, `2`) ou `String` (`"ABERTO"`) soltos, que aceitam qualquer valor e geram erros de digitação, o compilador só aceita as constantes declaradas.

**Como funciona**:

- **Declaração**: `enum Status { ABERTO, FECHADO }`. Pode ter **atributos, construtor (sempre privado) e métodos**, e implementar interfaces. Não pode estender outra classe nem ser estendido.
- **Métodos prontos**: `values()` (array com todas as constantes), `valueOf(String)` (por nome; lança `IllegalArgumentException` se não existir), `name()`, `ordinal()` (posição, a partir de 0; **evite persistir**, pois muda se a ordem mudar), `compareTo` (pela ordem de declaração) e `toString()` (devolve o nome, mas pode ser sobrescrito).
- **Comparação**: use `==` (cada constante é única); é seguro até com `null`.
- **Método abstrato por constante**: o enum declara `abstract` um método e cada constante o implementa em um corpo próprio `{ ... }`, formando uma espécie de *Strategy* embutida que dispensa `switch`.
- **Em `switch`**: vale `case ABERTO ->`; em *switch expression*, cobrir todas as constantes dispensa o `default`.
- **Coleções especializadas**: `EnumMap<K, V>` e `EnumSet<E>` são muito eficientes (usam arrays/bits internamente).
- **Persistência**: em JPA use `@Enumerated(EnumType.STRING)`; o padrão (`ORDINAL`) quebra se a ordem do enum mudar.
- É a forma recomendada de implementar um **Singleton** à prova de reflexão e serialização.

**Pontos importantes**: ao adicionar uma constante, `switch` exaustivos deixam de compilar e apontam todos os lugares a tratar (uma vantagem sobre `String`). Enums servem a conjuntos **fixos**; se os valores mudam em tempo de execução (dados do banco), use uma classe ou tabela.

**Exemplo**:

```java
enum TipoPagamento {
    DEBITO("Débito") {
        @Override double desconto(double valor) { return valor * 0.10; }
    },
    CREDITO("Crédito") {
        @Override double desconto(double valor) { return valor * 0.05; }
    };

    private final String rotulo;
    TipoPagamento(String rotulo) { this.rotulo = rotulo; }   // construtor privado implícito
    String getRotulo() { return rotulo; }
    abstract double desconto(double valor);                  // cada constante implementa
}

TipoPagamento t = TipoPagamento.valueOf("DEBITO");
System.out.println(t + " " + t.getRotulo() + " " + t.desconto(100));  // DEBITO Débito 10.0
for (TipoPagamento x : TipoPagamento.values())
    System.out.println(x.ordinal() + " " + x.name());                 // 0 DEBITO / 1 CREDITO
```

**Explicação do exemplo**: cada constante é um objeto com seu próprio `rotulo` (passado ao construtor) e sua própria implementação de `desconto`, então não há `if` ou `switch` espalhado pelo código para decidir o desconto. `valueOf("DEBITO")` devolve a constante correspondente (um nome inexistente lançaria `IllegalArgumentException`). O laço com `values()` mostra a ordem de declaração: `ordinal()` 0 e 1.

## 2.13 Records

**O que é**: forma enxuta de declarar uma **classe de dados imutável** (Java 16, recurso final). Você lista os **componentes** entre parênteses, e o compilador gera: os campos `private final`, o **construtor canônico**, os métodos de acesso (`nome()`, **sem** prefixo `get`), `equals`, `hashCode` e `toString` baseados nos componentes.

**Para que serve**: representar dados que apenas **carregam informação**: DTOs, *value objects* (identificadores, coordenadas, dinheiro), chaves de mapa, retornos com vários valores. Elimina dezenas de linhas de código repetitivo (e os bugs de `equals`/`hashCode` mal escritos).

**Como funciona**:

- `record Ponto(int x, int y) {}` equivale a uma classe `final` com campos `x` e `y`, construtor `Ponto(int x, int y)`, `x()`, `y()` e os três métodos de `Object`.
- **Construtor compacto**: sem lista de parâmetros, usado para **validar ou normalizar** antes da atribuição (que o compilador faz no fim): `record Faixa(int ini, int fim) { Faixa { if (ini > fim) throw new IllegalArgumentException(); } }`.
- Pode ter **métodos de instância, métodos e atributos `static`** e construtores adicionais (que devem delegar ao canônico). **Não** pode declarar atributos de instância extras.
- Um record é **`final`**, **não estende** outras classes (já estende `java.lang.Record`), mas **pode implementar interfaces**.
- **Imutabilidade rasa**: os campos não mudam, mas, se um componente for um objeto mutável (uma `List`), o conteúdo dele pode mudar. Faça cópia defensiva no construtor compacto (`lista = List.copyOf(lista)`).
- **Record patterns** (Java 21): permitem desestruturar em `instanceof` e `switch` (tópico 2.19).
- Frameworks modernos (Jackson 2.12+, Spring) suportam records.

**Quando usar / diferenças**: records x classes comuns: use record quando a classe é **só dados** e a igualdade é **por valor**. Não use quando precisa de estado mutável, de herança ou de um ciclo de vida complexo (por isso entidades JPA, que exigem classe não final com construtor sem argumentos, normalmente **não** são records). Lombok `@Value` faz algo parecido, mas record é recurso da linguagem.

**Exemplo**:

```java
record Produto(String nome, double preco) {
    Produto {                                   // construtor compacto: validação
        if (preco < 0) throw new IllegalArgumentException("preço negativo");
    }
    String rotulo() { return nome + " (R$ " + preco + ")"; }   // método comum
}

Produto a = new Produto("Caneta", 2.5);
Produto b = new Produto("Caneta", 2.5);
System.out.println(a);                 // Produto[nome=Caneta, preco=2.5]
System.out.println(a.nome());          // Caneta  (accessor sem 'get')
System.out.println(a.equals(b));       // true   (igualdade por valor)
System.out.println(a == b);            // false  (objetos distintos)
```

**Explicação do exemplo**: sem escrever nenhum método além da validação, o record já oferece `toString`, `equals` por **conteúdo** e os acessores. O construtor compacto garante que nenhum `Produto` tenha preço negativo. `a.equals(b)` é `true` porque os componentes coincidem, enquanto `a == b` continua falso, pois são dois objetos.

## 2.14 Classes internas, locais e anônimas

**O que é**: classes declaradas **dentro** de outras (ou de métodos). Existem quatro formas:

| Tipo | Onde é declarada | Precisa de instância externa? | Acessa membros da externa? |
| --- | --- | --- | --- |
| **Interna (*inner*)** | dentro da classe, **sem** `static` | **sim** | sim, até os `private` |
| **Aninhada estática (*static nested*)** | dentro da classe, **com** `static` | não | só os `static` (ou via referência) |
| **Local** | dentro de um método/bloco | depende do contexto | sim + variáveis locais *efetivamente finais* |
| **Anônima** | expressão `new Tipo() { ... }` | depende do contexto | idem local |

**Para que serve**: agrupar uma classe auxiliar que só faz sentido junto de outra (o `Builder` de uma classe, um nó de lista ligada), esconder detalhes de implementação, e criar implementações **de uso único** (callbacks) sem arquivo separado.

**Como funciona**:

- **Interna**: cada objeto da interna guarda uma **referência implícita** ao objeto da externa que a criou. Cria-se com `externa.new Interna()`; dentro dela, `Externa.this` é o objeto externo (e `this`, o interno). Isso permite acessar até campos `private` da externa.
- **Aninhada estática**: comporta-se como uma classe independente apenas "guardada" no *namespace* da externa; instancia-se com `new Externa.Aninhada()`. **É a opção padrão** quando a classe não precisa do objeto externo.
- **Local**: existe só no método onde foi declarada; pode usar parâmetros e variáveis locais, desde que **efetivamente finais**.
- **Anônima**: define e instancia uma classe sem nome que estende uma classe ou implementa uma interface na hora. Para **interface funcional**, a **lambda** é a alternativa moderna e mais enxuta. Diferenças: dentro da anônima `this` é a própria instância anônima (na lambda é a classe que a contém), a anônima pode ter estado e vários métodos, e gera um arquivo `.class` próprio.
- **Risco de *memory leak***: como a *inner* mantém a referência à externa, um objeto interno de vida longa impede a coleta da externa. Se a classe não usa a externa, torne-a `static`.
- Até o Java 15, classes internas não podiam ter membros `static`; a partir do 16 podem.

**Pontos importantes**: regra prática: **aninhada estática** como padrão; **interna** só quando precisa mesmo do estado da externa; **local** raramente; **anônima** para implementações descartáveis (e prefira lambda quando possível).

**Exemplo**:

```java
class Externa {
    private String nome = "externa";

    class Interna {                                   // precisa de um objeto Externa
        void mostrar() { System.out.println(nome + " / " + Externa.this.nome); }
    }
    static class Aninhada {                           // independente
        void oi() { System.out.println("oi"); }
    }
}

Externa e = new Externa();
Externa.Interna i = e.new Interna();                  // sintaxe da inner
i.mostrar();                                          // externa / externa
new Externa.Aninhada().oi();                          // oi

Runnable r = new Runnable() {                         // anônima
    @Override public void run() { System.out.println("rodando"); }
};
Runnable lambda = () -> System.out.println("rodando"); // mesma ideia, mais curta
```

**Explicação do exemplo**: a `Interna` só existe ligada a um objeto `Externa` (por isso `e.new Interna()`), e por isso lê o campo `nome` da externa diretamente. A `Aninhada` é criada sem nenhum objeto externo. O `Runnable` anônimo e a lambda fazem o mesmo; a lambda é preferida porque `Runnable` tem um único método abstrato.

## 2.15 Associação, agregação e composição

**O que é**: formas de relacionar objetos, em geral lidas como **"tem um"** (*has-a*), em contraste com a herança, que é **"é um"**.

- **Dependência**: uma classe apenas **usa** outra de forma passageira (como parâmetro ou variável local de um método).
- **Associação**: uma classe **guarda uma referência** para outra como atributo. Pode ser **unidirecional** (só A conhece B) ou **bidirecional** (A conhece B e B conhece A). Tem **cardinalidade**: 1-1, 1-N (um time tem vários jogadores), N-N (alunos e disciplinas).
- **Agregação**: associação do tipo "todo-parte" em que a parte **existe de forma independente** do todo (uma escola tem professores, mas o professor existe sem a escola).
- **Composição**: "todo-parte" em que a parte **depende** do todo e **morre com ele** (um pedido e seus itens: sem o pedido, o item não faz sentido).

**Para que serve**: modelar as relações reais do domínio e montar comportamentos complexos **combinando** objetos simples. É o fundamento do princípio **"favoreça composição em vez de herança"**: compor dá flexibilidade (trocar uma parte em tempo de execução) sem o forte acoplamento da herança.

**Como funciona**:

- Em código, todas aparecem como **atributos** do tipo da outra classe (ou coleções dele). A diferença entre agregação e composição está no **ciclo de vida**: quem cria e quem controla o fim da parte.
- **Bidirecional**: exige manter **os dois lados sincronizados**. Centralize isso em um método (`time.adicionar(jogador)` que atualiza ambos) para não ficar inconsistente. Cuidado com `toString`/`equals`/`hashCode` que percorrem os dois lados (recursão infinita, `StackOverflowError`).
- **Composição + delegação**: a classe externa **delega** a trabalho a seus componentes (`carro.ligar()` chama `motor.ligar()`), expondo só o que interessa.

**Pontos importantes**: antes de usar `extends`, pergunte "é um"? Se a resposta for "tem um" ou "usa um", é composição. Em JPA (`@OneToMany`, `@ManyToOne`) essas relações viram mapeamentos de banco.

**Exemplo**:

```java
class Motor { void ligar() { System.out.println("motor ligado"); } }

class Carro {
    private final Motor motor = new Motor();     // COMPOSIÇÃO: o carro cria e controla o motor
    void ligar() { motor.ligar(); }               // delegação
}

class Jogador {
    final String nome;
    Time time;                                    // lado N da relação
    Jogador(String nome) { this.nome = nome; }
}
class Time {
    private final List<Jogador> jogadores = new ArrayList<>();   // associação 1-N
    void adicionar(Jogador j) {                   // mantém os dois lados sincronizados
        jogadores.add(j);
        j.time = this;
    }
}

Time t = new Time();
Jogador j = new Jogador("Cafu");
t.adicionar(j);
System.out.println(j.time == t);                  // true
```

**Explicação do exemplo**: `Carro` cria seu próprio `Motor` e o esconde, expondo apenas `ligar()` (composição com delegação). Já `Time` e `Jogador` formam uma associação bidirecional: o método `adicionar` atualiza `jogadores` e também `j.time`, de modo que os dois lados nunca ficam desencontrados.

## 2.16 A classe `Object`: `equals`, `hashCode` e `toString`

**O que é**: `java.lang.Object` é a **raiz** de toda hierarquia: qualquer classe herda, direta ou indiretamente, seus métodos.

**Para que serve**: fornecem o contrato básico de **identidade, igualdade e representação** que a biblioteca inteira (coleções, logs, frameworks) assume que todo objeto tem.

**Como funciona**:

- **`toString()`**: texto que representa o objeto, chamado automaticamente por `println` e pela concatenação com `String`. O padrão (`Classe@hash`) é pouco útil; sobrescreva incluindo os atributos relevantes e **sem dados sensíveis** (senhas). IDEs e Lombok (`@ToString`) geram.
- **`equals(Object)`**: define a **igualdade lógica**. O padrão compara **referências** (`==`). Contrato: **reflexivo** (`x.equals(x)`), **simétrico** (`x.equals(y)` ⇔ `y.equals(x)`), **transitivo**, **consistente** (mesmo resultado enquanto nada mudar) e `x.equals(null)` é `false`.
- **`hashCode()`**: inteiro usado por `HashMap`/`HashSet` para decidir em qual "balde" guardar o objeto. **Regra de ouro**: se `a.equals(b)`, então `a.hashCode() == b.hashCode()`. O inverso não é exigido (colisões são normais). Por isso se sobrescrevem **sempre os dois juntos**, com os **mesmos campos** (`Objects.equals`, `Objects.hash`). Sem isso, dois objetos "iguais" caem em baldes diferentes e a coleção os trata como distintos.
- **Armadilha dos campos mutáveis**: se um campo usado no `hashCode` muda depois de o objeto entrar em um `HashSet`/chave de `HashMap`, o objeto "some" da coleção. Use campos imutáveis (ou `record`, que gera tudo corretamente).
- **`getClass() != o.getClass()` x `instanceof`** no `equals`: `getClass()` preserva a simetria em hierarquias; `instanceof` aceita subclasses (cuidado com simetria).
- **`getClass()`** devolve a classe real. **`clone()`** (protegido, cópia rasa, exige `Cloneable`) é frágil: prefira construtor de cópia. **`finalize()`** foi **descontinuado para remoção** (Java 18): não use; para liberar recursos, use `AutoCloseable`.

**Exemplo**:

```java
class Produto {
    private final String codigo;
    Produto(String codigo) { this.codigo = codigo; }

    @Override public boolean equals(Object o) {
        if (this == o) return true;                       // mesmo objeto
        if (o == null || getClass() != o.getClass()) return false;
        return Objects.equals(codigo, ((Produto) o).codigo);
    }
    @Override public int hashCode() { return Objects.hash(codigo); }   // mesmos campos
    @Override public String toString() { return "Produto{codigo=" + codigo + "}"; }
}

Set<Produto> set = new HashSet<>();
set.add(new Produto("A1"));
set.add(new Produto("A1"));                                // duplicado lógico
System.out.println(set.size());                           // 1
System.out.println(new Produto("A1"));                    // Produto{codigo=A1}
```

**Explicação do exemplo**: os dois `Produto("A1")` são objetos diferentes, mas `equals` os considera iguais e `hashCode` produz o **mesmo** valor, então o `HashSet` guarda só um (`size()` = 1). Se o `hashCode` não tivesse sido sobrescrito, os dois provavelmente cairiam em baldes diferentes e o conjunto teria **2** elementos, mesmo com `equals` correto.

## 2.17 Tratamento de exceções

**O que é**: mecanismo para **sinalizar e tratar situações anormais** sem misturá-las ao fluxo normal. Quando algo dá errado, cria-se um objeto **exceção** (um `Throwable`) e **lança-se** (`throw`). A execução do método atual é **interrompida**, e a JVM sobe pela pilha de chamadas procurando um `catch` compatível; se ninguém trata, a *thread* termina e imprime o *stack trace*.

**Para que serve**: separar o "caminho feliz" do tratamento de erros, **propagar** a falha até quem sabe resolvê-la e **garantir a liberação de recursos** (arquivos, conexões) mesmo quando algo falha.

**Como funciona**: a hierarquia:

```
Throwable
├── Error                (problemas da JVM: OutOfMemoryError, StackOverflowError; não se trata)
└── Exception
    ├── (checked)  IOException, SQLException, exceções suas...
    └── RuntimeException (unchecked)  NullPointerException, IllegalArgumentException, ...
```

- **Checked** (as `Exception` que não são `RuntimeException`): o compilador **exige** tratar (`try/catch`) ou declarar (`throws`). Representam condições externas previsíveis (arquivo ausente, rede, banco).
- **Unchecked** (`RuntimeException` e subclasses, mais os `Error`): não exigem tratamento; em geral indicam **bug** (índice inválido, `null`, argumento ilegal) ou violação de contrato. Frameworks modernos (Spring) preferem exceções unchecked.
- **`try / catch / finally`**: os `catch` são avaliados **em ordem**, então as **específicas** vêm antes das genéricas (o contrário não compila). **Multi-catch** (`catch (A | B e)`) para tratamento idêntico, com tipos sem relação de herança entre si. O **`finally`** executa **sempre** (com ou sem exceção, mesmo com `return` no `try`); só não roda se a JVM terminar (`System.exit`, queda). **Nunca coloque `return` ou `throw` no `finally`**: ele engole a exceção original.
- **`throw`** lança uma instância; **`throws`** na assinatura **declara** que o método pode lançar aquela exceção *checked*, repassando o tratamento a quem chama.
- **try-with-resources**: recursos declarados em `try (...)` que implementam `AutoCloseable` são **fechados automaticamente, na ordem inversa**, antes dos `catch`/`finally`. Se o corpo e o `close()` lançarem, a exceção do corpo prevalece e a do `close` fica em `getSuppressed()`.
- **Encadeamento (*cause*)**: preserve a causa original: `throw new ServicoException("falha ao salvar", e);`.
- **Exceções customizadas**: estenda `Exception` (obriga a tratar) ou `RuntimeException` (não obriga); ofereça sempre um construtor com `cause`.
- **Sobrescrita**: o método filho não pode declarar exceção *checked* **nova ou mais ampla** que a do pai (tópico 2.7).

**Boas práticas**: **nunca** deixe um `catch` vazio; não capture `Exception`/`Throwable` genéricas fora da borda da aplicação; **falhe cedo** (`Objects.requireNonNull`, validação no início do método); **log OU relance**, não os dois, em cada camada; mensagens com contexto, sem dados sensíveis; não use exceções para controle de fluxo normal; não devolva `null` para indicar erro (lance exceção ou devolva `Optional`).

**Exemplo**:

```java
class Recurso implements AutoCloseable {
    private final String nome;
    Recurso(String nome) { this.nome = nome; System.out.println("abre " + nome); }
    @Override public void close() { System.out.println("fecha " + nome); }
}

static void processar() {
    try (Recurso a = new Recurso("A"); Recurso b = new Recurso("B")) {
        System.out.println("usa");
        throw new IllegalStateException("falhou");
    } catch (IllegalStateException e) {
        System.out.println("catch: " + e.getMessage());
    } finally {
        System.out.println("finally");
    }
}

class SaldoInsuficienteException extends Exception {          // checked
    SaldoInsuficienteException(String m) { super(m); }
}
```

**Explicação do exemplo**: ao executar `processar()`, a saída é, em ordem: `abre A`, `abre B`, `usa`, `fecha B`, `fecha A`, `catch: falhou`, `finally`. Note que os recursos são **fechados antes** do `catch` e na ordem **inversa** da abertura, mesmo com a exceção. O `finally` roda por último. A `SaldoInsuficienteException` estende `Exception`, então qualquer método que a lance precisa declará-la com `throws` ou tratá-la.

## 2.18 Pacotes e módulos

**O que é**: **pacotes** (`package`) agrupam classes relacionadas e funcionam como *namespaces*; a estrutura de pastas espelha o nome (`br.com.app.servico` → `br/com/app/servico/`). **Módulos** (JPMS, Java 9) agrupam **vários pacotes** em uma unidade descrita por um arquivo `module-info.java`, que declara de que o módulo depende e o que ele expõe.

**Para que serve**: organizar projetos grandes, evitar colisão de nomes e **controlar o que é API pública**. Os pacotes dão encapsulamento em nível de pacote (`protected` e *package-private*); os módulos adicionam encapsulamento **forte**: classes `public` de pacotes **não exportados** ficam inacessíveis fora do módulo. Também permitem montar runtimes enxutos (`jlink`) e foram usados para modularizar o próprio JDK.

**Como funciona**:

- **Pacote**: convenção de nome = domínio invertido em minúsculas. O *classpath* é onde a JVM procura as classes (pastas e JARs). Pacotes **não são hierárquicos de verdade**: `br.com` e `br.com.app` são pacotes independentes (`import br.com.*` não inclui subpacotes).
- **`module-info.java`** (na raiz do código): `requires java.sql;` (dependência), `requires transitive` (repassa a quem depende de você), `exports br.com.app.api;` (pacote público), `opens br.com.app.modelo;` (permite *reflection* em execução, necessário para frameworks), `uses`/`provides` (serviços).
- **Sem `module-info`**, o código roda no *classpath* tradicional, e quase todos os projetos de aplicação (Spring Boot, Maven e Gradle comuns) seguem assim. Módulos aparecem mais em bibliotecas e no próprio JDK.
- Em Java 17+, acessar internos do JDK por *reflection* é bloqueado por padrão (`--add-opens` libera).

**Pontos importantes**: não confunda **módulo Java** (JPMS) com **módulo Maven/Gradle** (um subprojeto de build); são conceitos diferentes.

**Exemplo**:

```java
// module-info.java
module meu.app {
    requires java.sql;                  // usa o módulo java.sql do JDK
    exports br.com.app.api;             // só este pacote é visível para quem usar o módulo
    opens br.com.app.modelo to com.fasterxml.jackson.databind;   // reflection só para o Jackson
}
```

**Explicação do exemplo**: `requires` declara a dependência; `exports` torna `br.com.app.api` acessível por outros módulos, e tudo que não for exportado fica escondido, mesmo que as classes sejam `public`; `opens ... to` permite que **só** o Jackson use *reflection* nos campos do pacote `modelo`, por exemplo, para serializar JSON.

## 2.19 Classes seladas e pattern matching

**O que é**: **`sealed`** (Java 17, recurso final) permite que uma classe ou interface declare **exatamente quais tipos podem estendê-la ou implementá-la**, com `permits`. Cada subtipo permitido deve ser declarado `final`, `sealed` (restringe os seus descendentes) ou `non-sealed` (reabre a hierarquia). Em conjunto com *pattern matching* em `switch` (Java 21) e *record patterns* (Java 21), o compilador consegue verificar se **todos os casos** foram cobertos (**exaustividade**).

**Para que serve**: modelar **tipos algébricos**, isto é, valores que são **uma entre poucas alternativas conhecidas** (resultado `Sucesso` ou `Falha`; formas geométricas; eventos de um sistema), com a garantia de que, ao incluir uma alternativa nova, todos os `switch` que a ignoram **deixam de compilar**. Evita cadeias de `instanceof` e o `default` que esconde casos esquecidos.

**Como funciona**:

- `sealed interface Forma permits Circulo, Retangulo {}`: apenas esses dois tipos podem implementar `Forma`. Se os subtipos estão no **mesmo arquivo**, o `permits` pode ser omitido.
- Os subtipos costumam ser **records** (que são implicitamente `final`).
- **`switch` com padrões**: `case Circulo c -> ...` testa o tipo e já declara a variável. **Record patterns** desestruturam o record na própria `case`: `case Retangulo(double l, double a) -> l * a`.
- Em um *switch expression* sobre um tipo `sealed`, cobrir todos os subtipos **dispensa o `default`** (e é melhor assim: um subtipo novo gera erro de compilação).
- Subtipos `non-sealed` ou classes não permitidas **não** podem ser adicionados fora da lista.

**Pontos importantes**: *sealed* limita a **extensibilidade** de propósito (diferente das interfaces comuns, abertas). Use quando o conjunto de variantes é **fechado e conhecido**; para pontos de extensão abertos (plugins), prefira interfaces normais e polimorfismo.

**Exemplo**:

```java
sealed interface Resultado<T> permits Sucesso, Falha {}
record Sucesso<T>(T valor) implements Resultado<T> {}
record Falha<T>(String erro) implements Resultado<T> {}

static String descrever(Resultado<Integer> r) {
    return switch (r) {                                   // sem default: todos cobertos
        case Sucesso<Integer>(var v) -> "ok: " + v;       // record pattern
        case Falha<Integer>(var e)   -> "erro: " + e;
    };
}

System.out.println(descrever(new Sucesso<>(42)));         // ok: 42
System.out.println(descrever(new Falha<>("timeout")));    // erro: timeout
```

**Explicação do exemplo**: `Resultado` só pode ser `Sucesso` ou `Falha`. O `switch` cobre exatamente essas duas alternativas, então o compilador aceita sem `default`; se um dia for criada uma terceira (`Pendente`), este `switch` **passa a falhar na compilação**, apontando o lugar a corrigir. Os *record patterns* `Sucesso<Integer>(var v)` extraem o componente do record diretamente.

---

# Parte 3: API padrão do Java

## 3.1 Visão geral do Collections Framework

**O que é**: conjunto de interfaces e classes do pacote `java.util` para **guardar e manipular grupos de objetos** cujo tamanho pode variar (diferente dos arrays, que têm tamanho fixo). Cada **interface** define um *contrato* (o que a estrutura faz) e cada **implementação** escolhe *como* (a estrutura de dados por trás), com custos diferentes. A hierarquia:

```
Iterable                         (permite usar o for-each)
└── Collection
    ├── List    (ordenada, aceita duplicados, acesso por índice)
    ├── Set     (sem duplicados)
    └── Queue   (fila / prioridade)  →  Deque (fila de duas pontas)
Map  (chave → valor; chaves únicas; NÃO é uma Collection)
```

**Para que serve**: oferecer estruturas prontas, testadas e otimizadas (busca, ordenação, unicidade, filas) que você combina conforme a necessidade, sem reimplementá-las. Programar contra a **interface** (`List`, `Map`) deixa trocar a implementação sem mexer no resto do código.

**Como funciona**:

| Implementação | Estrutura interna | Ordem de iteração | Custo típico |
| --- | --- | --- | --- |
| `ArrayList` | array redimensionável | de inserção | `get(i)` O(1); `add` no fim O(1) amortizado; inserir/remover no meio O(n) |
| `LinkedList` | lista duplamente ligada | de inserção | `get(i)` O(n); inserir/remover nas pontas O(1) |
| `HashSet` / `HashMap` | tabela hash | **sem garantia** | O(1) em média |
| `LinkedHashSet` / `LinkedHashMap` | hash + lista ligada | de inserção | O(1) |
| `TreeSet` / `TreeMap` | árvore rubro-negra | **ordenada** | O(log n) |
| `PriorityQueue` | heap binário | por prioridade (só no `poll`) | `offer`/`poll` O(log n) |
| `ArrayDeque` | array circular | de inserção | pontas O(1) |

- **Só armazenam objetos**: `List<int>` não compila; use `List<Integer>` (autoboxing). O diamante `<>` infere o tipo: `List<String> l = new ArrayList<>();`.
- **Métodos comuns de `Collection`**: `add`, `remove`, `contains`, `size`, `isEmpty`, `clear`, `addAll`, `removeAll`, `retainAll`, `removeIf`, `stream`, `iterator`. `contains` e `remove(Object)` usam **`equals`**; estruturas baseadas em hash também dependem de **`hashCode`** (tópico 2.16).
- **Iteradores *fail-fast***: se a coleção sofre modificação estrutural fora do próprio iterador durante a iteração, ele tende a lançar `ConcurrentModificationException` (detecção de melhor esforço, não garantida).
- **Não são *thread-safe***: para acesso concorrente, use as coleções de `java.util.concurrent` (tópico 4.5). `Vector`, `Hashtable` e `Stack` são legados sincronizados: evite.
- **Não confunda** `Collection` (interface) com `Collections` (classe utilitária com métodos estáticos), nem `Arrays` (utilitário de arrays).
- **Java 21** adicionou `SequencedCollection` (`getFirst()`, `getLast()`, `reversed()`, `addFirst`...), implementada por `List`, `Deque`, `LinkedHashSet`, `SortedSet`; e `SequencedMap` para `LinkedHashMap`/`TreeMap`.

**Como escolher**: lista com índice → `ArrayList`; sem duplicados → `HashSet` (mantendo inserção `LinkedHashSet`; ordenado `TreeSet`); chave→valor → `HashMap` (`LinkedHashMap`/`TreeMap` conforme a ordem); fila de prioridade → `PriorityQueue`; pilha ou fila simples → `ArrayDeque`; chaves enum → `EnumMap`/`EnumSet`.

**Exemplo**:

```java
List<String> nomes = new ArrayList<>();           // programa para a interface List
nomes.add("Ana");
Set<Integer> unicos = new HashSet<>(List.of(1, 2, 3));
Map<String, Integer> idades = new HashMap<>();
idades.put("Ana", 30);
System.out.println(nomes + " " + unicos.contains(2) + " " + idades.get("Ana"));
```

**Explicação do exemplo**: as três variáveis são declaradas pelo **tipo da interface** (`List`, `Set`, `Map`) e criadas por uma implementação concreta. Isso significa que trocar `ArrayList` por `LinkedList` mudaria só a linha do `new`. A saída é `[Ana] true 30`.

## 3.2 List

**O que é**: coleção **ordenada** (mantém a **ordem de inserção**), que **aceita elementos duplicados** e permite **acesso direto por índice** (a partir de 0). `ArrayList` (array interno que cresce sozinho) é a escolha padrão; `LinkedList` só compensa em uso intenso de inserção/remoção nas pontas (e para isso `ArrayDeque` costuma ser melhor).

**Para que serve**: sequências em que a **ordem importa** e repetições são válidas: itens de um pedido, linhas de um arquivo, resultados de uma consulta.

**Como funciona**:

- **Operações**: `add(e)` (no fim), `add(i, e)`, `get(i)`, `set(i, e)`, `size()`, `contains`, `indexOf`/`lastIndexOf` (devolvem -1 se não achar), `addAll`, `sort`, `subList`, `removeIf`.
- **Pegadinha do `remove`**: `remove(int)` remove por **índice**; `remove(Object)` remove por **objeto**. Em `List<Integer>`, `remove(1)` remove a posição 1; para remover o valor 1, use `remove(Integer.valueOf(1))`.
- **`ArrayList` por dentro**: array que, ao encher, é copiado para outro maior (cerca de 1,5×); por isso `add` é O(1) amortizado, mas inserir/remover no meio é O(n) (desloca elementos). A capacidade inicial (`new ArrayList<>(100)`) só evita recópias, **não limita** o tamanho.
- **Listas imutáveis e visões**: `List.of(...)` e `List.copyOf` são **imutáveis**, **proíbem `null`** (e `contains(null)` lança NPE); alterar lança `UnsupportedOperationException`. `Arrays.asList(array)` é uma **visão de tamanho fixo** sobre o array (`set` funciona e altera o array; `add`/`remove` lançam exceção). `subList(a, b)` é uma **visão** da lista original (não uma cópia). Para uma lista mutável independente: `new ArrayList<>(outra)`.
- **Remover durante a iteração**: dentro de um *for-each*, lança `ConcurrentModificationException`; use `Iterator.remove()` ou, mais simples, **`list.removeIf(predicado)`**.
- **Ordenar**: `list.sort(comparator)` ou `Collections.sort(list)` (algoritmo estável, TimSort).
- **Java 21**: `getFirst()`, `getLast()` e `reversed()` (lançam `NoSuchElementException` se a lista estiver vazia).

**Pontos importantes**: `contains`/`indexOf` usam `equals`, então a classe dos elementos deve implementá-lo corretamente. Em `LinkedList`, `get(i)` percorre a lista (O(n)); percorrer com `for (int i...)` nela é muito lento.

**Exemplo**:

```java
List<String> l = new ArrayList<>(List.of("c", "a", "b"));
l.add("d");                          // [c, a, b, d]
Collections.sort(l);                 // [a, b, c, d]
l.removeIf(s -> s.equals("b"));      // [a, c, d]
System.out.println(l.get(0) + " " + l.size() + " " + l.indexOf("d"));   // a 3 2

List<Integer> nums = new ArrayList<>(List.of(10, 20, 30));
nums.remove(1);                      // remove o ÍNDICE 1  -> [10, 30]
nums.remove(Integer.valueOf(10));    // remove o VALOR 10   -> [30]
```

**Explicação do exemplo**: a lista é criada a partir de `List.of` (que é imutável) dentro de `new ArrayList<>(...)`, produzindo uma cópia **mutável**. Depois de adicionar e ordenar, `removeIf` remove o `"b"` com segurança. Em `nums`, `remove(1)` apaga o elemento da posição 1 (o 20) porque o argumento é `int`; para apagar o *valor* 10 é preciso passar um `Integer`, usando `Integer.valueOf`.

## 3.3 Set

**O que é**: coleção **sem elementos duplicados** e sem acesso por índice (modela o conjunto matemático). O que conta como "duplicado" depende da implementação: `HashSet` usa `equals`/`hashCode`; `TreeSet` usa `compareTo`/`Comparator`.

**Para que serve**: eliminar repetidos, testar **pertencimento** rapidamente (`contains` em O(1) no `HashSet`) e fazer operações de conjuntos (união, interseção, diferença).

**Como funciona**:

- **`HashSet`**: o mais rápido; usa um `HashMap` por baixo; a ordem de iteração **não é garantida** e pode mudar. **`LinkedHashSet`**: mantém a **ordem de inserção**. **`TreeSet`** (`NavigableSet`): sempre **ordenado**, exige `Comparable` ou um `Comparator`.
- `add(e)` devolve `boolean`: `false` se o elemento já existia (e o conjunto não muda).
- **Navegação** (`NavigableSet`): `lower(x)` (< x), `floor(x)` (<= x), `higher(x)` (> x), `ceiling(x)` (>= x), `first`, `last`, `pollFirst`, `pollLast`, `headSet`, `tailSet`, `subSet`, `descendingSet`.
- **Armadilha do `TreeSet`**: quem decide a unicidade é o **comparator**, **não o `equals`**. Dois objetos que "empatam" na comparação (mesmo preço, por exemplo) são tratados como o **mesmo** elemento, e o segundo **não é inserido**. Desempate com `thenComparing`. `TreeSet` não aceita `null` com ordem natural.
- **Operações de conjunto**: `addAll` (união), `retainAll` (interseção), `removeAll` (diferença), `containsAll` (subconjunto).
- `Set.of(...)` é imutável e proíbe duplicados no argumento (lança `IllegalArgumentException`); a ordem de iteração de `Set.of` é **imprevisível** (varia entre execuções).
- Elementos de `HashSet` devem ter `equals`/`hashCode` corretos e **não mudar** os campos usados nele depois de inseridos.

**Exemplo**:

```java
Set<String> tags = new TreeSet<>(List.of("java", "spring", "java"));
System.out.println(tags);                    // [java, spring]  (duplicado descartado, ordenado)
System.out.println(tags.add("java"));        // false

NavigableSet<Integer> n = new TreeSet<>(List.of(10, 20, 30));
System.out.println(n.floor(25) + " " + n.ceiling(25));   // 20 30

Set<Integer> a = new HashSet<>(List.of(1, 2, 3));
Set<Integer> b = Set.of(2, 3, 4);
a.retainAll(b);                              // interseção
System.out.println(a);                       // [2, 3]
```

**Explicação do exemplo**: o `TreeSet` descarta o `"java"` repetido e ordena alfabeticamente. `add` devolve `false` ao tentar inserir de novo. Com os números `[10, 20, 30]`, `floor(25)` busca o maior elemento **menor ou igual** a 25 (20) e `ceiling(25)` o menor **maior ou igual** (30). `retainAll` mantém em `a` só o que também está em `b`.

## 3.4 Map

**O que é**: estrutura que guarda pares **chave → valor**. A **chave é única**: inserir uma chave que já existe **substitui** o valor (e devolve o antigo). `Map` tem hierarquia própria, **não** estende `Collection`.

**Para que serve**: buscas por identificador (id → cliente), contagem de ocorrências, agrupamento, *caches*, tabelas de configuração, índices.

**Como funciona**:

- **`HashMap`**: tabela hash; O(1) em média; sem ordem garantida; aceita **uma** chave `null` e valores `null`. Por baixo, a chave é transformada por `hashCode` num "balde"; dentro do balde, `equals` encontra a entrada exata. Quando a ocupação passa de 75% a tabela dobra (*rehash*); baldes muito cheios viram árvore.
- **`LinkedHashMap`**: mantém a **ordem de inserção** (ou de **acesso**, em um modo útil para implementar *cache LRU* sobrescrevendo `removeEldestEntry`). **`TreeMap`** (`NavigableMap`): chaves **ordenadas** (O(log n)); `firstKey`, `lastKey`, `floorKey`, `ceilingKey`, `headMap`, `tailMap`; não aceita chave `null` com ordem natural.
- **Operações**: `put`, `get` (devolve `null` se a chave não existe), `getOrDefault`, `putIfAbsent`, `containsKey`, `containsValue`, `remove`, `computeIfAbsent(k, f)` (cria o valor se faltar; ótimo para agrupar), `merge(k, v, f)` (ótimo para contar), `forEach`, `replaceAll`.
- **Três visões**: `keySet()`, `values()` e `entrySet()` (pares `Map.Entry`). **Use `entrySet()`** quando precisar de chave e valor ao mesmo tempo (evita um `get` por chave). As visões refletem o mapa; remover por elas remove do mapa.
- **Chaves objeto exigem `equals`/`hashCode` corretos e, de preferência, imutáveis**; senão `get` com um objeto "igual" não encontra a entrada.
- **Imutáveis**: `Map.of(k1, v1, ...)` (até 10 pares), `Map.ofEntries`, `Map.copyOf`; proíbem `null`; ordem imprevisível.
- **Não é *thread-safe***: use `ConcurrentHashMap` (tópico 4.5).

**Exemplo**:

```java
Map<String, Integer> cont = new HashMap<>();
for (String p : "a b a c a b".split(" ")) {
    cont.merge(p, 1, Integer::sum);          // se não existe põe 1; se existe soma
}
System.out.println(cont);                    // {a=3, b=2, c=1}

for (Map.Entry<String, Integer> e : cont.entrySet()) {
    System.out.println(e.getKey() + " -> " + e.getValue());
}

Map<String, List<String>> grupos = new HashMap<>();
grupos.computeIfAbsent("frutas", k -> new ArrayList<>()).add("uva");
grupos.computeIfAbsent("frutas", k -> new ArrayList<>()).add("pera");
System.out.println(grupos);                  // {frutas=[uva, pera]}
```

**Explicação do exemplo**: `merge` resolve o padrão "contar ocorrências" sem `if`: na primeira vez grava 1; nas seguintes aplica `Integer::sum` ao valor atual e ao novo. A ordem de impressão de um `HashMap` não é garantida (aqui `a`, `b`, `c` é só um resultado típico). `computeIfAbsent` cria a lista na primeira vez que a chave aparece e devolve a lista (existente ou nova), na qual se faz o `add`.

## 3.5 Queue e Deque

**O que é**: **`Queue`** é uma **fila**, normalmente **FIFO** (*first in, first out*: o primeiro a entrar é o primeiro a sair). **`Deque`** (*double-ended queue*) permite inserir e remover **nas duas pontas**, servindo como fila **e como pilha** (LIFO, *last in, first out*).

**Para que serve**: processar elementos **por ordem de chegada ou por prioridade** (tarefas, eventos, impressão), pilhas de "desfazer", percursos em grafos e árvores (busca em largura usa fila; em profundidade usa pilha).

**Como funciona**: cada operação existe em **duas versões**: uma que lança exceção e outra que devolve valor especial.

| Operação | Lança exceção | Devolve valor especial |
| --- | --- | --- |
| Inserir | `add(e)` | `offer(e)` (`false` se não couber) |
| Remover a cabeça | `remove()` | `poll()` (`null` se vazia) |
| Ver a cabeça | `element()` | `peek()` (`null` se vazia) |

- **`PriorityQueue`**: **não é FIFO**. `poll` devolve o elemento de **maior prioridade** (o menor, pela ordem natural ou pelo `Comparator`). Só o `poll` garante a ordem; **iterar ou imprimir** a fila **não** mostra os elementos ordenados. Não aceita `null`. Inserção/remoção O(log n); `remove(Object)` é O(n). Não é thread-safe (`PriorityBlockingQueue` é).
- **`ArrayDeque`**: implementação recomendada de **pilha** (`push`, `pop`, `peek`) e de **fila** (`offer`, `poll`); não aceita `null`; mais rápida que `Stack` (legada) e que `LinkedList` nesses usos.
- **`LinkedList`** implementa `Queue`/`Deque` e aceita `null`.
- Filas **bloqueantes** para trabalho entre *threads* (`BlockingQueue`): tópico 4.5.

**Exemplo**:

```java
Queue<String> fila = new ArrayDeque<>();
fila.offer("a"); fila.offer("b");
System.out.println(fila.poll() + " " + fila.peek());   // a b   (FIFO)

Queue<Integer> pq = new PriorityQueue<>(Comparator.reverseOrder());
pq.addAll(List.of(3, 9, 1));
System.out.println(pq.poll());                          // 9  (maior primeiro)

Deque<Integer> pilha = new ArrayDeque<>();
pilha.push(1); pilha.push(2);
System.out.println(pilha.pop());                        // 2  (LIFO)
```

**Explicação do exemplo**: na fila comum, `poll` remove `"a"` (o primeiro que entrou) e `peek` mostra o próximo (`"b"`) sem removê-lo. A `PriorityQueue` com `reverseOrder()` trata o **maior** valor como mais prioritário, então `poll` devolve 9, independentemente da ordem de inserção. No `Deque` usado como pilha, `push` coloca na frente e `pop` retira da frente, por isso sai o último inserido (2).

## 3.6 `Comparable`, `Comparator`, `Iterator` e utilitários

**O que é**: as peças que definem **como ordenar, comparar e percorrer** coleções.

- **`Comparable<T>`**: a própria classe define sua **ordem natural** por meio de `int compareTo(T outro)`, que devolve **negativo** (this vem antes), **zero** (equivalentes) ou **positivo** (this vem depois). Usado por `Collections.sort`, `TreeSet`, `TreeMap`, `PriorityQueue`.
- **`Comparator<T>`**: um critério de ordenação **externo** (`int compare(a, b)`), usado quando a ordem natural não serve ou a classe não é sua. Permite vários critérios para a mesma classe.
- **`Iterator<T>`**: percorre uma coleção (`hasNext()`, `next()`, `remove()`); o *for-each* usa um por baixo dos panos. **`ListIterator`** anda nos dois sentidos e permite `set`/`add`.
- **`Collections`** e **`Arrays`**: utilitários (`sort`, `reverse`, `shuffle`, `max`, `min`, `frequency`, `binarySearch`, `unmodifiableList`, `emptyList`, `swap`...).

**Para que serve**: ordenar objetos por diferentes critérios, buscar com rapidez em listas ordenadas e remover itens com segurança durante a iteração.

**Como funciona**:

- **Contrato de `compareTo`**: antissimétrico (`a.compareTo(b)` e `b.compareTo(a)` têm sinais opostos), transitivo, e é **fortemente recomendado** ser consistente com `equals` (`compareTo == 0` ⇔ `equals`). Em `TreeSet`/`TreeMap`, a inconsistência causa elementos "sumindo" (tópico 3.3).
- **`Comparator` moderno**: `Comparator.comparing(Pessoa::idade).thenComparing(Pessoa::nome).reversed()`; `Comparator.naturalOrder()`/`reverseOrder()`; `nullsFirst`/`nullsLast`. Para primitivos, use `Integer.compare(a, b)`/`Double.compare(a, b)`: **subtrair** (`a - b`) pode estourar.
- **`Collections.binarySearch(lista, chave[, comparator])`**: exige lista **já ordenada com o mesmo critério**; se não encontra, devolve `-(ponto de inserção) - 1` (um número negativo, não necessariamente `-1`).
- **Imutável x visão**: `Collections.unmodifiableList(l)` é só uma **visão** (acompanha mudanças em `l`); `List.copyOf(l)` é uma **cópia** imutável.
- **Ordenação estável**: `List.sort` e `Collections.sort` preservam a ordem relativa de elementos equivalentes.

**Exemplo**:

```java
record Pessoa(String nome, int idade) {}
List<Pessoa> ps = new ArrayList<>(List.of(
        new Pessoa("Bia", 30), new Pessoa("Ana", 30), new Pessoa("Caio", 25)));

ps.sort(Comparator.comparing(Pessoa::idade).thenComparing(Pessoa::nome));
System.out.println(ps.get(0).nome() + " " + ps.get(1).nome() + " " + ps.get(2).nome());   // Caio Ana Bia

Iterator<Pessoa> it = ps.iterator();
while (it.hasNext()) {
    if (it.next().idade() > 28) it.remove();      // remoção segura durante a iteração
}
System.out.println(ps.size());                    // 1
```

**Explicação do exemplo**: o `Comparator` ordena primeiro por idade (25 antes de 30) e, em caso de empate, por nome (`Ana` antes de `Bia`), resultando em `Caio Ana Bia`. No laço com `Iterator`, `it.remove()` apaga o último elemento devolvido por `next()` sem causar `ConcurrentModificationException`; restam só as pessoas com idade até 28, ou seja, 1.

## 3.7 Números precisos: `BigDecimal`, `BigInteger` e `Math`

**O que é**: `float` e `double` guardam números em **base 2** (ponto flutuante binário, padrão IEEE 754). Muitos decimais comuns, como 0,1, **não têm representação exata** em base 2, então `0.1 + 0.2` resulta em `0.30000000000000004`. **`BigDecimal`** representa números decimais com **precisão arbitrária** e **escala controlada**; **`BigInteger`** representa **inteiros sem limite de tamanho**. Ambos são **imutáveis** (toda operação devolve um novo objeto).

**Para que serve**: **dinheiro**, impostos, taxas e qualquer cálculo decimal em que o resultado precisa ser **exato** e arredondado de forma **definida** (regra de negócio). `BigInteger` serve para fatoriais, criptografia e números gigantes.

**Como funciona**:

- **`BigDecimal` = valor sem escala × 10^(−escala)**: `19.90` guarda o inteiro 1990 e a **escala** 2 (casas decimais). Em uma multiplicação as escalas se somam (`19.90 × 3` dá `59.70`).
- **Como criar**: use **String** (`new BigDecimal("0.1")`) ou `BigDecimal.valueOf(0.1)` (converte via `Double.toString`). **Nunca** `new BigDecimal(0.1)`: isso captura o erro binário do `double` (`0.1000000000000000055511...`).
- **Operações**: `add`, `subtract`, `multiply`, `divide`, `negate`, `abs`, `pow`, `compareTo`, `signum`. Não se usam operadores (`+`, `*`).
- **`equals` x `compareTo`**: `equals` compara **valor e escala** (`2.0` ≠ `2.00`); `compareTo` compara apenas o **valor** (devolve 0). Para igualdade numérica use `compareTo(...) == 0`. Isso também afeta `HashSet`/`HashMap`.
- **Divisão**: `a.divide(b)` lança `ArithmeticException` quando o resultado é uma dízima infinita (ex.: 1/3). Informe a escala e o arredondamento: `a.divide(b, 2, RoundingMode.HALF_UP)`, ou um `MathContext`.
- **Arredondamento (`RoundingMode`)**: `HALF_UP` (o "arredondamento escolar": 0,5 sobe), `HALF_EVEN` ("do banqueiro": empate vai para o par; reduz viés estatístico), `DOWN` (trunca), `UP`, `FLOOR`, `CEILING`. `setScale(2, RoundingMode.HALF_UP)` ajusta as casas.
- **`BigInteger`**: `add`, `multiply`, `pow`, `mod`, `gcd`, `isProbablePrime`; `BigInteger.valueOf(long)`.
- **`Math`**: `abs`, `max`, `min`, `pow`, `sqrt`, `floor`, `ceil`, `round` (devolve `long` para `double`; empates sobem), `floorDiv`/`floorMod` (divisão e resto "matemáticos" com negativos) e as versões `*Exact` (`addExact`, `multiplyExact`, `toIntExact`) que lançam `ArithmeticException` em estouro. Curiosidade: `Math.abs(Integer.MIN_VALUE)` continua negativo (estouro).

**Pontos importantes**: `BigDecimal` é mais lento e verboso que `double`; use onde a exatidão importa (dinheiro), não em cálculo científico. Em bancos de dados, mapeie para `DECIMAL`/`NUMERIC`.

**Exemplo**:

```java
System.out.println(0.1 + 0.2);                            // 0.30000000000000004

BigDecimal preco = new BigDecimal("19.90");
BigDecimal total = preco.multiply(BigDecimal.valueOf(3));  // escalas: 2 + 0
System.out.println(total);                                 // 59.70

BigDecimal a = new BigDecimal("2.0");
BigDecimal b = new BigDecimal("2.00");
System.out.println(a.equals(b) + " " + (a.compareTo(b) == 0));   // false true

BigDecimal terco = BigDecimal.ONE.divide(new BigDecimal("3"), 4, RoundingMode.HALF_UP);
System.out.println(terco);                                 // 0.3333
```

**Explicação do exemplo**: o `double` mostra o erro clássico. Com `BigDecimal` criado a partir de String, `19.90 × 3` dá exatamente `59.70`. `2.0` e `2.00` são **numericamente iguais** (`compareTo` = 0), mas `equals` é `false` porque as escalas diferem. A divisão `1/3` exige escala e modo de arredondamento (sem eles lançaria exceção); com 4 casas e `HALF_UP` o resultado é `0.3333`.

## 3.8 Generics

**O que é**: recurso que permite **parametrizar** classes, interfaces e métodos com **tipos**. Em vez de uma lista de `Object`, você declara `List<String>`, e o compilador **garante** que só entram `String` e que a leitura devolve `String`, **sem *cast***. Antes do Java 5, coleções guardavam `Object`, o erro só aparecia em execução (`ClassCastException`).

**Para que serve**: mover a verificação de tipos para a **compilação** (segurança), eliminar *casts* (legibilidade) e **reutilizar** o mesmo código para vários tipos (coleções, repositórios genéricos, utilitários como `filter` e `map`).

**Como funciona**:

- **Classe genérica**: `class Caixa<T> { T item; }`; `T` é o **parâmetro de tipo**, substituído por um tipo real ao usar (`Caixa<String>`). Convenções: `T` (type), `E` (element), `K`/`V` (key/value), `R` (return).
- **Método genérico**: `static <T> List<T> listaDe(T x)`; o `<T>` **antes do tipo de retorno** declara o parâmetro do método, e o compilador **infere** `T` pelos argumentos. Independe de a classe ser genérica.
- **Tipos limitados**: `<T extends Comparable<T>>` exige que `T` ofereça `compareTo`; `extends` vale para classes **e interfaces**; vários limites com `&` (a classe primeiro). Isso permite chamar métodos do limite dentro do código genérico.
- **Invariância**: `List<Cachorro>` **não é** um `List<Animal>`, mesmo que `Cachorro extends Animal`. Se fosse, daria para inserir um `Gato` em uma lista de cachorros. (Arrays são **covariantes** e por isso inseguros: `Animal[] a = new Cachorro[1]; a[0] = new Gato();` compila e falha com `ArrayStoreException`.)
- **Curingas (*wildcards*)**: `?` representa um tipo desconhecido. **`? extends T`** (limite superior): aceita `T` e subtipos; você só consegue **ler** (como `T`), não inserir. **`? super T`** (limite inferior): aceita `T` e supertipos; você consegue **inserir** `T`, mas ao ler só recebe `Object`. Regra **PECS**: *Producer Extends, Consumer Super* (quem **produz** dados para você usa `extends`; quem **consome** os seus usa `super`).
- **Apagamento de tipos (*type erasure*)**: os parâmetros de tipo existem só na **compilação**; em execução são apagados (viram `Object` ou o limite). Consequências: não existem `new T()`, `new T[10]`, `T.class` nem `instanceof List<String>` (só `instanceof List<?>`); `List<String>` e `List<Integer>` têm o **mesmo tipo em execução** (sobrecargas que só diferem nisso não compilam); campos `static` não usam o `T` da classe; **tipos primitivos não podem ser argumento** (`List<int>` → `List<Integer>`, com *boxing*). Para saber o tipo em execução, passe um `Class<T>` por parâmetro.
- ***Raw types*** (`List` sem `<>`) desligam a checagem e existem só por compatibilidade: **evite**. O diamante `<>` evita repetir o tipo na criação.

**Pontos importantes**: generics e a **herança** não se combinam como em arrays: use curingas para expressar "uma lista de qualquer subtipo de Animal". Declare métodos com `List<? extends Animal>` quando só vai **ler**, e `List<? super Animal>` quando só vai **escrever**.

**Exemplo**:

```java
class Caixa<T> {                                   // classe genérica
    private T item;
    void guardar(T item) { this.item = item; }
    T pegar() { return item; }
}

static <T extends Comparable<T>> T maior(List<T> lista) {   // método genérico limitado
    T m = lista.get(0);
    for (T t : lista) if (t.compareTo(m) > 0) m = t;
    return m;
}
static double somar(List<? extends Number> nums) {           // só lê (produtor)
    double s = 0;
    for (Number n : nums) s += n.doubleValue();
    return s;
}
static void preencher(List<? super Integer> destino) {       // só escreve (consumidor)
    destino.add(1);
}

Caixa<String> c = new Caixa<>();
c.guardar("oi");
String s = c.pegar();                                // sem cast
System.out.println(maior(List.of(3, 9, 4)));         // 9
System.out.println(somar(List.of(1, 2.5, 3L)));      // 6.5
```

**Explicação do exemplo**: `Caixa<String>` só aceita `String`, e `pegar()` já devolve `String`. `maior` funciona para **qualquer** tipo que seja comparável com ele mesmo (`Integer`, `String`...), porque o limite `Comparable<T>` garante o `compareTo`. `somar` aceita listas de `Integer`, `Double` ou `Long` (`? extends Number`) e apenas **lê** os valores (1 + 2,5 + 3 = 6,5). `preencher` aceita `List<Integer>`, `List<Number>` ou `List<Object>` e só **escreve** `Integer`.

## 3.9 Datas e horas (`java.time`)

**O que é**: API moderna (Java 8) para datas e horas, com classes **imutáveis** e *thread-safe*: toda operação (`plusDays`, `withDayOfMonth`) devolve **um novo objeto**. Substitui `Date` e `Calendar`, a API legada, que é **mutável**, tem meses de `Calendar` começando em **0** e nomes enganosos (`Date` guarda também a hora).

**Para que serve**: aniversários, vencimentos, agendamentos, **medição de intervalos**, *timestamps* de auditoria e **conversões entre fusos horários**, sem os bugs clássicos da API antiga.

**Como funciona**: a escolha da classe depende do que você precisa representar:

| Classe | Representa | Quando usar |
| --- | --- | --- |
| `LocalDate` | data, sem hora e sem fuso | aniversário, vencimento |
| `LocalTime` | hora, sem data e sem fuso | horário de abertura |
| `LocalDateTime` | data + hora, **sem fuso** | agendamento "local" |
| `Instant` | ponto exato na linha do tempo (UTC), contado desde a *epoch* 1970-01-01T00:00Z | registrar **quando algo aconteceu** |
| `ZonedDateTime` | data-hora + **fuso** (`ZoneId`, com regras de horário de verão) | exibir ao usuário em seu fuso |
| `OffsetDateTime` | data-hora + **deslocamento fixo** (`-03:00`) | APIs/banco que guardam offset |
| `Duration` | quantidade de tempo em horas/min/s/ns | tempo de relógio (duração de tarefa) |
| `Period` | quantidade em anos/meses/dias | tempo de calendário (idade, contrato) |
| `YearMonth`, `Year`, `DayOfWeek`, `Month` | tipos auxiliares | competência mensal, dia da semana |

- **`Duration` x `Period`**: `Duration` é exata (segundos); `Period` é de calendário (um mês tem tamanho variável). `Duration.between` com `LocalDate` lança `UnsupportedTemporalTypeException`, pois não há horas.
- **`ZoneId` x `ZoneOffset`**: `ZoneId` (`America/Sao_Paulo`) conhece as **regras de horário de verão**; `ZoneOffset` é um desvio fixo (`-03:00`).
- **Datas inválidas lançam `DateTimeException`** (`LocalDate.of(2026, 2, 30)`). Somar meses **ajusta ao fim do mês** (31/jan + 1 mês = 28/fev).
- **`ChronoUnit.X.between(a, b)`** devolve a diferença **inteira e truncada** na unidade (`DAYS`, `MONTHS`...). **`TemporalAdjusters`** (`next(DayOfWeek.MONDAY)`, `firstDayOfMonth()`, `lastDayOfMonth()`) e `TemporalAdjuster` customizado (`data.with(adjuster)`).
- **Boas práticas**: grave instantes em **UTC** (`Instant`) e converta para o fuso do usuário só na **borda** (exibição); injete um **`Clock`** em vez de chamar `now()` diretamente (testes determinísticos); nunca monte datas por concatenação de Strings; use ISO-8601 em APIs.
- **Legado**: `Date.from(instant)`/`date.toInstant()`, `java.sql.Date.valueOf(localDate)` fazem a ponte.

**Exemplo**:

```java
LocalDate d = LocalDate.of(2026, 1, 31);
System.out.println(d.plusMonths(1));                         // 2026-02-28  (ajusta ao fim do mês)
System.out.println(ChronoUnit.DAYS.between(LocalDate.of(2026, 1, 1), d));   // 30

Period p = Period.between(LocalDate.of(2020, 5, 10), LocalDate.of(2026, 10, 24));
System.out.println(p);                                       // P6Y5M14D

Duration dur = Duration.ofMinutes(90);
System.out.println(dur + " " + dur.toHours());               // PT1H30M 1

ZonedDateTime sp = ZonedDateTime.of(2026, 10, 24, 12, 0, 0, 0, ZoneId.of("America/Sao_Paulo"));
System.out.println(sp.withZoneSameInstant(ZoneId.of("Asia/Tokyo")));   // 2026-10-25T00:00+09:00[Asia/Tokyo]
```

**Explicação do exemplo**: somar um mês a 31/jan não existe em fevereiro, então a API ajusta para o último dia (28/fev). `ChronoUnit.DAYS.between` conta 30 dias inteiros. O `Period` expressa a diferença em anos, meses e dias de calendário (`P6Y5M14D`), enquanto `Duration` expressa 90 minutos como `PT1H30M`. Na conversão de fuso, o **instante é o mesmo**: meio-dia em São Paulo (UTC-3) é 15:00 UTC, que em Tóquio (UTC+9) é **meia-noite do dia seguinte**.

## 3.10 Formatação e internacionalização

**O que é**: adaptar a **exibição e a leitura** de números, moedas, datas e textos às convenções de um idioma/região. O mesmo valor se escreve `1.234,56` no Brasil e `1,234.56` nos EUA.

**Para que serve**: apresentar informações no formato esperado pelo usuário e **traduzir** mensagens de uma aplicação para vários idiomas.

**Como funciona**:

- **`Locale`**: idioma + país (`pt-BR`, `en-US`). O construtor `new Locale("pt", "BR")` está **depreciado** (Java 19+): use `Locale.of("pt", "BR")` ou `Locale.forLanguageTag("pt-BR")`. Há constantes (`Locale.US`, `Locale.JAPAN`). Sempre informe o `Locale` explicitamente: o **padrão da JVM varia** por máquina, e é fonte comum de bugs entre ambientes (por exemplo, `String.format("%.2f")` usar vírgula em um servidor e ponto em outro).
- **`NumberFormat`**: `getInstance`, `getCurrencyInstance`, `getPercentInstance` (cada uma recebe um `Locale`); `setMaximumFractionDigits`. `DecimalFormat` aceita um padrão (`"#,##0.00"`). Em versões recentes do JDK, o espaço entre `R$` e o número pode ser um espaço **não separável** (U+00A0).
- **`DateTimeFormatter`** (moderno, imutável, *thread-safe*): constantes ISO (`ISO_LOCAL_DATE`, `BASIC_ISO_DATE`), `ofPattern("dd/MM/yyyy", locale)` e estilos localizados (`ofLocalizedDate(FormatStyle.LONG)`). Serve para **formatar** (`data.format(f)`) e **ler** (`LocalDate.parse(texto, f)`; texto inválido lança `DateTimeParseException`).
- **Letras do padrão (e armadilhas)**:

| Letra | Significa | Armadilha |
| --- | --- | --- |
| `yyyy` | ano | **`YYYY` é o ano da semana ISO**: na virada do ano mostra o ano errado |
| `MM` | mês numérico | **`mm` é minuto** |
| `dd` | dia do mês | `DD` é o dia do ano |
| `HH` | hora 0 a 23 | `hh` é 1 a 12 (exige `a` para AM/PM) |
| `MMMM` | nome do mês | depende do `Locale` |

- **Legado**: `SimpleDateFormat`/`DateFormat` **não são *thread-safe*** (jamais em campo `static` compartilhado).
- **`String.format`/`printf`**, **`MessageFormat`** (`"Olá, {0}"`) e **`ResourceBundle`** (arquivos `.properties` por idioma, como `mensagens_pt_BR.properties`) para textos traduzidos.

**Exemplo**:

```java
Locale br = Locale.of("pt", "BR");
System.out.println(NumberFormat.getCurrencyInstance(br).format(1500.5));   // R$ 1.500,50
System.out.println(String.format(Locale.US, "%,.2f", 1234.5));            // 1,234.50

DateTimeFormatter f = DateTimeFormatter.ofPattern("dd/MM/yyyy");
System.out.println(LocalDate.of(2026, 10, 24).format(f));                 // 24/10/2026
LocalDate d = LocalDate.parse("18/08/2026", f);                           // texto -> data
System.out.println(d.getMonth());                                         // AUGUST
```

**Explicação do exemplo**: o formatador de moeda do `pt-BR` usa o símbolo `R$`, ponto de milhar e vírgula decimal. No segundo exemplo, o `Locale.US` **força** o formato americano, independentemente da configuração da máquina. O mesmo `DateTimeFormatter` serve para escrever uma data (`24/10/2026`) e para **ler** o texto `18/08/2026` de volta como `LocalDate`.

## 3.11 Expressões regulares (regex)

**O que é**: linguagem para descrever **padrões de texto**. No pacote `java.util.regex`: **`Pattern`** é a expressão **compilada** (operação cara: guarde em `static final`); **`Matcher`** aplica o padrão a um texto.

**Para que serve**: **validar** formatos (CPF, CEP, telefone, placa), **extrair** partes de um texto (datas, números), **substituir** conteúdo e **dividir** strings com separadores complexos.

**Como funciona**:

- **`matches()`**: o texto **inteiro** precisa casar. **`find()`**: procura a **próxima ocorrência** em qualquer lugar do texto (pegadinha: um texto pode falhar em `matches` e ser encontrado por `find`). **`lookingAt()`**: casa no **início**. Depois de casar: `group()`, `group(n)`, `start()`, `end()`; `replaceAll`.
- **Escape duplo**: a regex `\d` (dígito) precisa ser escrita `"\\d"` em uma String Java (uma barra para o Java, outra para a regex). **Text blocks** ajudam a legibilidade.

| Elemento | Significado |
| --- | --- |
| `.` | qualquer caractere (exceto quebra de linha) |
| `\d` / `\D` | dígito / não dígito |
| `\w` / `\W` | `[a-zA-Z0-9_]` / o restante |
| `\s` / `\S` | espaço em branco / não branco |
| `[abc]`, `[a-z]`, `[^a-z]` | conjunto, intervalo, negação |
| `?` `*` `+` | 0 ou 1 / 0 ou mais / 1 ou mais |
| `{n}`, `{n,m}` | exatamente n / de n a m |
| `( )` e `\|` | grupo (captura) / alternância (ou) |
| `^` `$` | início / fim |

- **Gananciosos (*greedy*)** por padrão (`.*` pega o máximo); `?` depois (`.*?`) torna-os **mínimos** (*lazy*).
- **Grupos nomeados**: `(?<ano>\\d{4})` e `matcher.group("ano")`. **Flags**: `Pattern.CASE_INSENSITIVE`.
- `String.matches`, `replaceAll` e `split` aceitam regex, mas **recompilam** o padrão a cada chamada.
- **Cuidados**: **ReDoS** (padrões com repetições aninhadas, como `(a+)+`, podem travar a CPU com certas entradas); regex não serve para formatos recursivos (HTML, JSON) e é pouco legível: para validar e-mail, prefira Bean Validation (`@Email`) a uma regex caseira.

**Exemplo**:

```java
Pattern p = Pattern.compile("(\\d{3})-(\\d{4})");
Matcher m = p.matcher("Ligue 123-4567 agora");
System.out.println(m.matches());            // false (o texto inteiro não é só o padrão)
m.reset();
if (m.find()) {
    System.out.println(m.group());          // 123-4567
    System.out.println(m.group(2));         // 4567
}
System.out.println("a1b22c".replaceAll("\\d+", "#"));   // a#b#c
System.out.println("12345".matches("\\d+"));            // true
```

**Explicação do exemplo**: `matches()` é `false` porque exige que o texto **todo** seja `ddd-dddd`; `find()` procura **dentro** do texto e acha `123-4567` (o `reset()` reinicia o *matcher*). `group(2)` é o segundo par de parênteses (os quatro dígitos finais). `\d+` casa uma sequência de dígitos, e `replaceAll` troca cada sequência por `#`.

## 3.12 Arquivos e I/O clássico (`java.io`)

**O que é**: leitura e escrita de dados por meio de **fluxos** (*streams*): um canal por onde os dados passam em sequência, sem se carregar tudo em memória. Há dois grupos: **fluxos de bytes** (`InputStream`/`OutputStream`, para dados binários como imagens e ZIPs) e **fluxos de caracteres** (`Reader`/`Writer`, para texto, que fazem a conversão entre bytes e caracteres usando um *charset*).

**Para que serve**: ler e gravar arquivos, importar/exportar dados, escrever logs, trocar dados por rede (sockets também usam streams).

**Como funciona**:

- **`File`** representa um **caminho** e seus metadados, **não o conteúdo**: `exists`, `isFile`, `isDirectory`, `length`, `lastModified`, `delete`, `createNewFile` (devolve `boolean`; lança `IOException` se a pasta não existir), `mkdir` (um nível) e `mkdirs` (cria os pais), `renameTo` (devolve `boolean`), `list`.
- **`FileReader`/`FileWriter`**: texto, caractere a caractere. `new FileWriter(arquivo, true)`: o segundo argumento ativa o ***append*** (sem ele, o arquivo é **sobrescrito**). `read()` devolve um `int` (`-1` no fim).
- **Buffer**: `BufferedReader`/`BufferedWriter` guardam dados em memória e reduzem os acessos a disco; **`readLine()`** lê uma linha (devolve `null` no fim) e `newLine()` escreve o separador da plataforma. `flush()` força a gravação (o `close()` já faz).
- **Padrão *Decorator***: streams se **encadeiam** como camadas (`new BufferedReader(new FileReader(f))`). Outros: `PrintWriter`, `InputStreamReader`, `DataInputStream`.
- **Recursos**: sempre **try-with-resources** (fecha os streams mesmo com exceção).
- **Charset**: informe explicitamente (`StandardCharsets.UTF_8`). Desde o Java 18 (JEP 400) o padrão é UTF-8 em todas as plataformas; antes dependia do sistema.
- **Portabilidade**: não use `\\` em caminhos (só funciona no Windows); use `/`, `File.separator` ou `Path.of(...)`.
- Para código novo, prefira **NIO.2** (`Path`/`Files`, tópico 3.13), mais concisa; o I/O clássico ainda é muito encontrado.

**Exemplo**:

```java
try (BufferedWriter bw = new BufferedWriter(new FileWriter("saida.txt", true))) {   // append
    bw.write("linha 1");
    bw.newLine();
    bw.write("linha 2");
    bw.newLine();
} catch (IOException e) {
    e.printStackTrace();
}

try (BufferedReader br = new BufferedReader(new FileReader("saida.txt"))) {
    String linha;
    while ((linha = br.readLine()) != null) {      // null = fim do arquivo
        System.out.println(linha);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

**Explicação do exemplo**: o primeiro bloco abre `saida.txt` em modo *append*, escreve duas linhas e fecha o arquivo automaticamente (*try-with-resources* também descarrega o *buffer*). O segundo bloco lê o arquivo linha a linha: `readLine()` devolve `null` ao chegar ao fim, o que encerra o `while`. A saída é `linha 1` e `linha 2`. Executar o programa mais uma vez acrescentaria **mais** duas linhas, por causa do `true` do *append*.

## 3.13 NIO.2 (`java.nio.file`)

**O que é**: API moderna de arquivos (Java 7). **`Path`** descreve um **caminho** (criado por `Path.of(...)`, Java 11, ou `Paths.get(...)`) e **`Files`** executa as **operações** (criar, copiar, ler, listar). A velha `File` misturava as duas coisas e tratava mal erros e atributos.

**Para que serve**: manipular arquivos e pastas de forma **concisa, portável e com erros claros** (cada falha lança uma exceção específica, enquanto `File` apenas devolvia `false`).

**Como funciona**:

- **`Path`** (imutável): `getFileName`, `getParent`, `toAbsolutePath`, `startsWith`. Operações **puramente textuais** (não tocam o disco): **`normalize()`** remove `.` e `..` redundantes; **`resolve(outro)`** concatena (se `outro` é **absoluto**, devolve o próprio `outro`); **`relativize(outro)`** calcula o caminho de um até o outro (ambos devem ser absolutos, ou ambos relativos, senão `IllegalArgumentException`).
- **`Files`**: `createFile`, `createDirectory`, `createDirectories` (cria os pais), `copy` e `move` (**não sobrescrevem** por padrão: lançam `FileAlreadyExistsException`; use `StandardCopyOption.REPLACE_EXISTING`), `delete` / `deleteIfExists`, `exists` / `notExists` (os dois podem ser `false` se o acesso for impossível), `size`, `readAllLines`, `readString`, `writeString` (por padrão cria e **trunca**; `StandardOpenOption.APPEND` para acrescentar), `newBufferedReader/Writer`.
- **Leitura em *stream***: `Files.lines(path)`, `Files.list(dir)` e `Files.walk(dir)` devolvem **`Stream`** preguiçosa que **mantém o arquivo/diretório aberto**: use *try-with-resources*. `readAllLines` carrega **tudo** na memória (cuidado com arquivos grandes).
- **Atributos**: `BasicFileAttributes` (tamanho, datas de criação/modificação; vale em qualquer SO), **`DosFileAttributes`** (só Windows: oculto, somente leitura) e **`PosixFileAttributes`** (só Unix/Linux/macOS: dono, grupo, permissões `rwxr-xr-x`, com `PosixFilePermissions.fromString`). `Files.readAttributes` lê; `Files.getFileAttributeView` também **altera**.
- **Percorrer diretórios**: **`DirectoryStream`** lista **um único nível** (aceita um filtro *glob*); **`Files.walkFileTree`** com **`SimpleFileVisitor`** percorre **recursivamente**, chamando `preVisitDirectory`, `visitFile`, `visitFileFailed`, `postVisitDirectory`, e o retorno `FileVisitResult` (`CONTINUE`, `SKIP_SUBTREE`, `TERMINATE`) controla o percurso.
- **`PathMatcher`** com sintaxe **glob**: `*.java` (um nível; `*` não cruza `/`), `**/*.txt` (qualquer profundidade), `*.{bkp,txt}`, `?` (um caractere).
- **ZIP**: `ZipOutputStream` com `ZipEntry` (`putNextEntry`, copiar, `closeEntry`); ou tratar o `.zip` como sistema de arquivos com `FileSystems.newFileSystem`.
- `WatchService` observa criação/alteração de arquivos em uma pasta.

**Exemplo**:

```java
Path dir = Path.of("dados", "sub");
Files.createDirectories(dir);                          // cria dados/sub (e os pais)
Path arq = dir.resolve("a.txt");                        // dados/sub/a.txt
Files.writeString(arq, "olá" + System.lineSeparator() + "mundo");
List<String> linhas = Files.readAllLines(arq);          // [olá, mundo]
System.out.println(linhas.size());                      // 2

try (Stream<Path> s = Files.walk(Path.of("dados"))) {   // recursivo; fechar o stream
    s.filter(p -> p.toString().endsWith(".txt")).forEach(System.out::println);
}
Files.copy(arq, dir.resolve("b.txt"), StandardCopyOption.REPLACE_EXISTING);
```

**Explicação do exemplo**: `createDirectories` monta a pasta e os pais de uma vez. `resolve` concatena sem tocar o disco. `writeString` cria o arquivo e `readAllLines` o lê de volta (2 linhas). O `Files.walk` percorre `dados` recursivamente; o `Stream` está em *try-with-resources* para liberar a pasta. O `copy` só sobrescreve `b.txt` se já existir graças ao `REPLACE_EXISTING`.

## 3.14 Serialização

**O que é**: transformar o **estado de um objeto em bytes** (`ObjectOutputStream.writeObject`) e **reconstruí-lo** depois (`ObjectInputStream.readObject`). Para ser serializável, a classe implementa a interface marcadora **`Serializable`** (sem métodos). Todo o **grafo** de objetos referenciados é gravado, e todos os campos (exceto `transient` e `static`) também precisam ser serializáveis, senão ocorre `NotSerializableException`.

**Para que serve**: salvar um objeto em disco, enviá-lo pela rede (RMI), guardar sessões e caches distribuídos. **Não** é um bom formato de armazenamento de longo prazo nem de integração entre sistemas (use JSON, Protobuf, etc.).

**Como funciona**:

- **`serialVersionUID`**: número de versão da classe, gravado junto com os dados. Se não for declarado, é **calculado** a partir da estrutura da classe, e qualquer mudança (um campo novo) faz a leitura de dados antigos falhar com `InvalidClassException`. Declare explicitamente (`@Serial private static final long serialVersionUID = 1L;`) e mude-o **de propósito** quando quiser quebrar a compatibilidade.
- **`transient`**: o campo **não é gravado** e, ao ler, volta com o valor **padrão** (`null`, `0`). Use em senhas, caches, conexões. Campos **`static`** pertencem à classe e também não são gravados.
- **O construtor da classe serializável não é chamado ao ler**; os campos são restaurados diretamente. Se a **superclasse não é serializável**, o construtor **sem argumentos** dela é executado, e os campos dela **não** são restaurados.
- **Serialização customizada**: métodos privados `writeObject(ObjectOutputStream)` e `readObject(ObjectInputStream)` (chamando `defaultWriteObject()`/`defaultReadObject()` e gravando extras); `readResolve()` mantém *singletons*; **`Externalizable`** dá controle total.
- **Enums** são serializados pelo nome (seguros); **records** serializados via construtor canônico.
- **Segurança**: desserializar **dados não confiáveis** é um vetor clássico de ataque (execução de código). Use `ObjectInputFilter` para restringir as classes aceitas, ou evite a serialização Java nativa.

**Exemplo**:

```java
class Aluno implements Serializable {
    @Serial private static final long serialVersionUID = 1L;
    String nome;
    transient String senha;                          // não será gravada
    Aluno(String nome, String senha) { this.nome = nome; this.senha = senha; }
}

Path arq = Path.of("aluno.ser");
try (ObjectOutputStream out = new ObjectOutputStream(Files.newOutputStream(arq))) {
    out.writeObject(new Aluno("Ana", "segredo"));
}
try (ObjectInputStream in = new ObjectInputStream(Files.newInputStream(arq))) {
    Aluno lido = (Aluno) in.readObject();
    System.out.println(lido.nome + " " + lido.senha);   // Ana null
}
```

**Explicação do exemplo**: o objeto é gravado em bytes e relido como uma **cópia**. Como `senha` é `transient`, não foi gravada e volta `null`, então a saída é `Ana null`. O construtor de `Aluno` **não** é executado na leitura: o estado vem dos bytes.

## 3.15 Reflection

**O que é**: capacidade de **inspecionar e manipular classes, atributos, métodos e construtores em tempo de execução**, mesmo sem conhecê-los em tempo de compilação. O ponto de partida é um objeto **`Class<?>`**, obtido por `objeto.getClass()`, `Pessoa.class` ou `Class.forName("br.com.Pessoa")`.

**Para que serve**: é o **alicerce dos frameworks**: o Spring descobre e instancia seus beans e injeta dependências; o JPA/Hibernate lê campos para mapear tabelas; o Jackson converte JSON em objetos; o JUnit encontra métodos `@Test`. No dia a dia de aplicação você raramente escreve reflection; é importante **entender** o que os frameworks fazem.

**Como funciona**:

- **Inspeção**: `getDeclaredFields()`, `getDeclaredMethods()`, `getDeclaredConstructors()` (todos os membros da própria classe, inclusive `private`) e `getFields()`/`getMethods()` (só os `public`, inclusive herdados); `getModifiers()`, `getSuperclass()`, `getInterfaces()`, `getAnnotation(...)`, `getRecordComponents()`.
- **Uso**: `constructor.newInstance(args)`, `method.invoke(objeto, args)` (exceções do método chegam embrulhadas em `InvocationTargetException`), `field.get(objeto)` / `field.set(objeto, valor)`. **`setAccessible(true)`** ignora `private`, mas, desde o Java 9/17, o sistema de módulos **bloqueia** o acesso a membros internos do JDK (`InaccessibleObjectException`; flag `--add-opens` libera).
- **Generics** são apagados em execução, mas informações em assinaturas (por exemplo, o tipo de um campo `List<String>`) ficam disponíveis por `getGenericType()`.
- **`java.lang.reflect.Proxy`** cria, em execução, objetos que implementam **interfaces**, delegando as chamadas a um `InvocationHandler`. É a base de AOP, `@Transactional`, clientes Feign e repositórios Spring Data.
- **Custos**: mais lenta que a chamada direta, **sem checagem em compilação** (erros viram exceções em execução), **quebra o encapsulamento** e exige configuração extra em *native images* (GraalVM). Para desempenho, existem `MethodHandle` e geração de código em compilação (*annotation processing*).

**Exemplo**:

```java
class Pessoa {
    private String nome;
    public Pessoa(String nome) { this.nome = nome; }
    public String getNome() { return nome; }
}

Class<?> c = Pessoa.class;
for (Field f : c.getDeclaredFields()) {
    System.out.println(f.getName() + " : " + f.getType().getSimpleName());   // nome : String
}
Object p = c.getDeclaredConstructor(String.class).newInstance("Ana");     // new Pessoa("Ana")
Method m = c.getMethod("getNome");
System.out.println(m.invoke(p));                                           // Ana
```

**Explicação do exemplo**: sem escrever `new Pessoa(...)` nem `p.getNome()` no código, o programa descobre o campo `nome` pelo nome e pelo tipo, **cria** um objeto procurando o construtor que recebe `String`, e **invoca** `getNome` pelo nome. É exatamente assim que um framework consegue trabalhar com classes que nunca viu em compilação.

## 3.16 Annotations e Annotation Processing

**O que é**: **anotações** são **metadados** colocados no código (`@Nome`). Elas não executam lógica por si só: são **lidas** pelo compilador, por ferramentas ou por frameworks (em compilação ou em execução). São declaradas com `@interface` e podem ter **elementos** (`String motivo() default "";`).

**Para que serve**: declarar intenção de forma concisa (`@Entity`, `@Test`, `@GetMapping`), validar regras (`@NotNull`), orientar o compilador (`@Override`) e **gerar código repetitivo** (builders, getters, mappers).

**Como funciona**:

- **Embutidas**: `@Override`, `@Deprecated(since = "...", forRemoval = true)`, `@SuppressWarnings`, `@FunctionalInterface`, `@SafeVarargs`.
- **Meta-anotações** (anotam a anotação): `@Retention` define até quando ela vive: `SOURCE` (só no fonte, descartada pelo compilador), `CLASS` (gravada no `.class`, mas indisponível em execução; **padrão**) e `RUNTIME` (acessível por reflection); `@Target` define onde pode ser aplicada (`TYPE`, `FIELD`, `METHOD`, `PARAMETER`, `CONSTRUCTOR`, `TYPE_USE`...); `@Documented`; `@Inherited` (classes filhas "herdam" a anotação da classe); `@Repeatable`.
- **Lidas em execução**: por reflection, só as `RUNTIME` (`metodo.getAnnotation(X.class)`).
- **Annotation Processing (JSR 269)**: o `javac` invoca **processadores** (subclasses de `AbstractProcessor`) **durante a compilação**, que leem os elementos anotados e **geram novos arquivos** (normalmente `.java`) por meio do `Filer`. Eles trabalham em **rodadas**: se uma rodada gera arquivos novos, outra rodada os processa. Um processador **não altera** arquivos existentes (o Lombok usa APIs internas não oficiais para isso). Erros são reportados por `Messager`. O processador é descoberto por `META-INF/services/javax.annotation.processing.Processor` (gerado por `@AutoService`); a geração de código costuma usar **JavaPoet**. Em build: Gradle `annotationProcessor(...)`, Maven `annotationProcessorPaths`.
- **Reflection x Annotation Processing**: reflection age em **execução** (custo, sem checagem prévia); o processamento age em **compilação** (custo zero em execução, erros detectados cedo, bom para *native image*). Exemplos: **Lombok**, **MapStruct**, Dagger.
- Nas versões recentes do JDK (a partir do **23**), o `javac` não executa processadores encontrados no *classpath* por padrão; é preciso declará-los (`-processorpath`, `annotationProcessor`) ou usar `-proc:full` (verifique a versão do seu JDK e ferramenta de build).

**Exemplo**:

```java
@Retention(RetentionPolicy.RUNTIME)                   // visível em execução
@Target(ElementType.METHOD)                           // só em métodos
@interface Auditar { String motivo() default ""; }

class Servico {
    @Auditar(motivo = "saque")
    void sacar() { }
}

Method m = Servico.class.getDeclaredMethod("sacar");
if (m.isAnnotationPresent(Auditar.class)) {
    System.out.println(m.getAnnotation(Auditar.class).motivo());   // saque
}
```

**Explicação do exemplo**: `@Retention(RUNTIME)` mantém a anotação disponível quando o programa roda; sem ela (padrão `CLASS`), a leitura por reflection devolveria `null`. `@Target(METHOD)` impede usá-la em outros lugares. O código lê, por reflection, o elemento `motivo` e imprime `saque`: é o mecanismo pelo qual frameworks decidem o que fazer com `@Transactional`, `@GetMapping` etc.

## 3.17 Logging

**O que é**: **registrar eventos** da aplicação (diagnóstico, auditoria, erros) em um destino (console, arquivo, serviço de logs), com **níveis de severidade** e **formato** configuráveis, no lugar de `System.out.println` e `e.printStackTrace()`.

**Para que serve**: entender o que aconteceu em produção, investigar falhas, auditar ações e alimentar ferramentas de monitoramento. Os níveis permitem **filtrar** o volume sem alterar o código.

**Como funciona**:

- **Níveis** (do mais verboso ao mais grave): `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`. Em produção costuma-se `INFO` (ou `WARN`); `DEBUG`/`TRACE` para investigar.
- **Fachada e implementações**: o código usa uma **API** (**SLF4J** é o padrão; também `System.Logger`, do JDK 9, e `java.util.logging`) e uma **implementação** faz o trabalho (**Logback**, padrão do Spring Boot, ou **Log4j2**). Trocar a implementação não exige mudar o código que registra logs. O Lombok oferece `@Slf4j`/`@Log4j2`.
- **Mensagem parametrizada**: `log.info("Usuário {} logou", id)` evita concatenar a String quando o nível está desligado. Para registrar o ***stack trace***, passe a exceção como **último** argumento: `log.error("Falha ao salvar {}", id, e)`.
- **Appenders** definem o destino (console, arquivo com **rotação**, rede); o **layout/*pattern*** define o formato (data, thread, nível, classe, mensagem). **MDC** acrescenta contexto a cada linha (id da requisição, usuário). **Logs estruturados** (JSON) facilitam a busca em ferramentas.
- **Configuração**: `logging.level.br.com.app=DEBUG` no Spring Boot; arquivos `logback.xml`/`log4j2.xml` para o resto.
- **Boas práticas**: **não registre dados sensíveis** (senhas, tokens, cartões); **log OU relance** a exceção, não os dois, em cada camada (senão o mesmo erro aparece várias vezes); use o nível adequado (`ERROR` só para falhas reais); evite logar dentro de laços muito quentes; mantenha as bibliotecas de log atualizadas (a falha **Log4Shell**, de 2021, mostrou o risco de vulnerabilidades).

**Exemplo**:

```java
private static final Logger log = LoggerFactory.getLogger(PedidoService.class);

void salvar(Pedido p) {
    log.info("Salvando pedido {}", p.getId());              // INFO: marco normal
    try {
        repositorio.save(p);
    } catch (Exception e) {
        log.error("Erro ao salvar pedido {}", p.getId(), e); // ERROR + stack trace
        throw e;                                             // (ver a ressalva: log OU relance)
    }
}
```

**Explicação do exemplo**: o `Logger` é um campo `static final`, um por classe, e carrega o nome da classe nas mensagens. `{}` é preenchido apenas se o nível `INFO` estiver ativo. No `catch`, passar `e` como último argumento registra o *stack trace* completo. **Atenção**: logar **e** relançar a mesma exceção em várias camadas duplica o erro no log; escolha a camada responsável por registrá-lo (normalmente a mais externa, como um tratador global).

## 3.18 Outras classes utilitárias importantes

**O que é**: classes pequenas e muito usadas da biblioteca padrão.

**Para que serve / como funciona**:

- **`Objects`**: `requireNonNull(x, "msg")` (**falhar cedo** com mensagem clara), `equals(a, b)` e `hash(...)` (seguros contra `null`), `toString(x, padrão)`, `isNull`/`nonNull`, `requireNonNullElse`.
- **`UUID`**: identificador universal de 128 bits; `UUID.randomUUID()` (versão 4, aleatória), `UUID.fromString(...)`. Usado como chave primária e identificador de correlação, evita colisão sem coordenação central.
- **Números aleatórios**: `java.util.Random` (rápido, previsível: **não** use para segurança), `ThreadLocalRandom.current().nextInt(1, 7)` (melhor em ambiente com várias *threads*) e **`SecureRandom`** (criptograficamente seguro: tokens, senhas). O Java 17 trouxe a interface `RandomGenerator`.
- **`System`**: `currentTimeMillis()` (relógio de parede), **`nanoTime()`** (para medir **intervalos**; valor sem relação com data), `getenv` (variáveis de ambiente), `getProperty`, `lineSeparator`, `exit`, `arraycopy`. **`Runtime`**: `availableProcessors()`, `totalMemory()`, `freeMemory()`, `maxMemory()`.
- **`Properties`**: pares `chave=valor` lidos de arquivos `.properties` (`load`, `getProperty(chave, padrão)`).
- **`StringJoiner`** (montar texto com separador), **`TimeUnit`** (`TimeUnit.SECONDS.sleep(2)`, conversões de unidades), **`Timer`** (legado: prefira `ScheduledExecutorService`), **`Optional`** (tópico 4.3).

**Exemplo**:

```java
Objects.requireNonNull(nome, "nome é obrigatório");              // NPE com mensagem útil
String id = UUID.randomUUID().toString();                        // ex.: 3f2b...-...
int dado = ThreadLocalRandom.current().nextInt(1, 7);           // 1 a 6
String ambiente = System.getenv().getOrDefault("AMBIENTE", "dev");

long inicio = System.nanoTime();
// ... trabalho ...
long ms = (System.nanoTime() - inicio) / 1_000_000;             // duração em ms

Properties props = new Properties();
try (InputStream in = Files.newInputStream(Path.of("app.properties"))) {
    props.load(in);
}
System.out.println(props.getProperty("porta", "8080"));          // valor ou padrão
```

**Explicação do exemplo**: `requireNonNull` interrompe cedo, com uma mensagem que diz **qual** argumento era nulo. O `nanoTime` é usado para medir a duração (subtração de dois instantes), nunca para "que horas são". `getProperty("porta", "8080")` devolve o valor do arquivo ou o padrão se a chave não existir.

## 3.13 NIO.2 (`java.nio.file`)

**O que é**: API moderna de arquivos (Java 7) que separa as responsabilidades que `File` misturava: **`Path`** descreve **caminhos** e **`Files`** executa as **operações**. (Não confundir com o *NIO* original, `java.nio`, de *buffers*, *channels* e *selectors*, usado em I/O não bloqueante de alto desempenho.)

**Para que serve**: ler, escrever, copiar, mover, apagar e percorrer arquivos e pastas de forma concisa, portátil e com **exceções específicas** (`NoSuchFileException`, `FileAlreadyExistsException`, `DirectoryNotEmptyException`), em vez dos `boolean` ambíguos do `File`.

**Como funciona**:

- **`Path`**: criado por `Path.of("pasta", "a.txt")` (Java 11; `Paths.get` é a forma antiga). Operações **só sobre o caminho** (não tocam o disco): `getFileName`, `getParent`, `toAbsolutePath`, `normalize()` (remove `.` e `..`), `resolve(outro)` (concatena; se `outro` for **absoluto**, devolve o próprio `outro`), `relativize(outro)` (caminho de um até o outro; ambos devem ser absolutos ou ambos relativos).
- **`Files`**: `createFile`, `createDirectory`/`createDirectories`, `copy`/`move` (**falham** se o destino existe, a menos que se passe `StandardCopyOption.REPLACE_EXISTING`), `delete` (lança exceção se a pasta não estiver vazia) e `deleteIfExists`, `exists`/`notExists` (os dois podem ser `false` se não for possível determinar), `size`, `isDirectory`.
- **Leitura/escrita**: `readString`, `writeString`, `readAllLines` (carrega **tudo** na memória), **`lines`** (devolve uma `Stream`, **lazy**, que mantém o arquivo aberto: feche com *try-with-resources*), `newBufferedReader`/`newBufferedWriter` (UTF-8 por padrão). Opções como `StandardOpenOption.APPEND`.
- **Percorrer diretórios**: `Files.list` (um nível), `Files.walk` (recursivo, devolve `Stream`; feche), `DirectoryStream` (um nível, eficiente) e **`Files.walkFileTree` + `SimpleFileVisitor`** (recursivo, com *callbacks* `preVisitDirectory`, `visitFile`, `visitFileFailed`, `postVisitDirectory`, e `FileVisitResult.CONTINUE/SKIP_SUBTREE/TERMINATE`).
- **Atributos**: `BasicFileAttributes` (tamanho, datas de criação/modificação; todos os sistemas), `DosFileAttributes` (**só Windows**: oculto, somente leitura) e `PosixFileAttributes` (**só Unix/Linux/macOS**: dono, grupo, permissões `rwxr-xr-x`). `Files.readAttributes` lê; `Files.getFileAttributeView` também **altera**.
- **`PathMatcher`** com sintaxe **glob**: `*.java` (no nome), `**/*.java` (em qualquer subpasta), `*.{bkp,txt}` (alternativas), `?` (um caractere).
- **Outros**: `WatchService` (observa mudanças em pastas), links simbólicos, sistema de arquivos ZIP (`FileSystems.newFileSystem`), `ZipOutputStream`/`ZipFile` para criar e ler `.zip`.

**Pontos importantes**: para arquivos **grandes**, evite `readAllLines`/`readString`; use `lines` ou `newBufferedReader` (processamento por partes).

**Exemplo**:

```java
Path dir = Path.of("dados", "sub");
Files.createDirectories(dir);                          // cria a árvore (não falha se já existe)
Path arq = dir.resolve("a.txt");                        // dados/sub/a.txt
Files.writeString(arq, "olá\nmundo");                   // grava (UTF-8)
List<String> linhas = Files.readAllLines(arq);
System.out.println(linhas);                             // [olá, mundo]

Files.copy(arq, dir.resolve("b.txt"), StandardCopyOption.REPLACE_EXISTING);
try (Stream<Path> s = Files.walk(Path.of("dados"))) {   // recursivo; fecha o recurso
    s.filter(p -> p.toString().endsWith(".txt")).forEach(System.out::println);
}
```

**Explicação do exemplo**: `resolve` monta o caminho sem tocar o disco; `writeString` cria o arquivo e `readAllLines` o lê de volta, separando por linha. `copy` precisa de `REPLACE_EXISTING` para não lançar `FileAlreadyExistsException` se `b.txt` já existir. `Files.walk` percorre `dados` recursivamente e devolve uma `Stream`, que é filtrada pelos `.txt`; como a stream mantém recursos abertos, ela é fechada no *try-with-resources*.

## 3.14 Serialização

**O que é**: transformar o **estado de um objeto** em uma sequência de **bytes** (`ObjectOutputStream.writeObject`) e reconstruí-lo depois (`ObjectInputStream.readObject`), podendo gravá-lo em disco, enviá-lo pela rede ou guardá-lo em cache. A classe precisa implementar a interface marcadora **`Serializable`** (sem métodos: só autoriza).

**Para que serve**: persistir ou transmitir grafos de objetos (cache em disco, sessões distribuídas, RMI). **Não é adequada como formato de armazenamento de longo prazo nem para troca entre sistemas**: para isso, prefira JSON (Jackson), Protobuf e similares, que são explícitos, legíveis e independentes de linguagem.

**Como funciona**:

- **Grafo de objetos**: serializar um objeto grava também os objetos que ele referencia. **Todos** os campos (exceto `transient` e `static`) devem ser serializáveis, senão ocorre `NotSerializableException`.
- **`serialVersionUID`**: número de versão da classe, conferido na leitura. Se não for declarado, é **calculado** a partir da estrutura, e qualquer alteração (adicionar um campo) muda o valor e causa `InvalidClassException` ao ler dados antigos. Declare explicitamente (`@Serial private static final long serialVersionUID = 1L;`); `@Serial` é uma anotação de verificação (Java 14).
- **`transient`**: o campo é **ignorado** e volta com o valor padrão (`null`, `0`) na leitura; use para senhas, conexões, *caches*. Campos **`static`** pertencem à classe e também não são gravados.
- **A desserialização não chama o construtor** da classe serializável: o objeto é reconstruído a partir dos bytes, sem passar pelas validações do construtor (por isso, desserializar dado não confiável é perigoso). Exceção: **records** são reconstruídos pelo construtor canônico.
- **Herança**: se a superclasse **não** for `Serializable`, ela precisa de um **construtor sem argumentos** acessível (executado na leitura) e seus campos **não** são gravados.
- **Personalização**: métodos `private void writeObject(ObjectOutputStream)` e `private void readObject(ObjectInputStream)` (usam `defaultWriteObject()`/`defaultReadObject()` e acrescentam dados extras); `readResolve()` (garantir *singleton*/enum); `Externalizable` dá controle total (exige construtor público sem argumentos).
- **Segurança**: desserializar bytes **não confiáveis** é um vetor clássico de ataque (execução de código por *gadget chains*). Use **`ObjectInputFilter`** (Java 9+), nunca desserialize entradas externas sem filtro e prefira formatos de dados puros.

**Exemplo**:

```java
class Aluno implements Serializable {
    @Serial private static final long serialVersionUID = 1L;
    String nome;
    transient String senha;                 // não será gravada
    Aluno(String nome, String senha) { this.nome = nome; this.senha = senha; }
}

Path arq = Path.of("aluno.ser");
try (ObjectOutputStream out = new ObjectOutputStream(Files.newOutputStream(arq))) {
    out.writeObject(new Aluno("Ana", "segredo"));
}
try (ObjectInputStream in = new ObjectInputStream(Files.newInputStream(arq))) {
    Aluno lido = (Aluno) in.readObject();
    System.out.println(lido.nome + " " + lido.senha);     // Ana null
}
```

**Explicação do exemplo**: o objeto `Aluno` é gravado em `aluno.ser` e relido. Como `senha` é `transient`, **não foi gravada**, e o objeto lido tem `senha == null`; só `nome` sobreviveu. O `serialVersionUID` fixo garante que a classe continue lendo arquivos antigos mesmo após mudanças compatíveis. Note que, na leitura, o construtor `Aluno(...)` **não** é executado.

## 3.15 Reflection (reflexão)

**O que é**: capacidade de **inspecionar e manipular classes, métodos, campos e construtores em tempo de execução**, mesmo sem conhecê-los na compilação. O ponto de entrada é a classe `Class<?>`, obtida por `objeto.getClass()`, `Pessoa.class` ou `Class.forName("br.com.Pessoa")`.

**Para que serve**: é o **alicerce dos frameworks**: o Spring descobre e instancia seus beans e injeta dependências; JPA/Hibernate lê as anotações e preenche os campos; Jackson converte JSON em objetos; JUnit encontra métodos `@Test`. No código de aplicação do dia a dia, você raramente precisa escrevê-la.

**Como funciona**:

- **Inspeção**: `getDeclaredFields()`, `getDeclaredMethods()`, `getDeclaredConstructors()` (os declarados na classe, inclusive `private`), `getFields()`/`getMethods()` (só os `public`, inclusive herdados), `getModifiers()`, `getSuperclass()`, `getInterfaces()`, `getAnnotation(...)`, `getRecordComponents()`.
- **Uso**: `constructor.newInstance(args)`, `method.invoke(objeto, args)`, `field.get(objeto)`/`field.set(objeto, valor)`. `setAccessible(true)` permite acessar membros `private`, mas, a partir do **Java 9 (módulos)** e **17 (encapsulamento forte)**, só funciona se o módulo "abrir" o pacote (`opens`, ou `--add-opens`).
- **`java.lang.reflect.Proxy`** cria, em execução, objetos que **implementam interfaces** e interceptam as chamadas: é a base de AOP, `@Transactional`, clientes Feign e repositórios Spring Data.
- **Custos e riscos**: mais lenta que a chamada direta, **sem verificação em compilação** (erros viram exceções em execução, como `NoSuchMethodException`), **quebra o encapsulamento** e exige configuração extra em *GraalVM native image*.

**Exemplo**:

```java
class Pessoa {
    private String nome;
    public Pessoa(String nome) { this.nome = nome; }
    public String getNome() { return nome; }
}

Class<?> c = Pessoa.class;
for (Field f : c.getDeclaredFields()) {
    System.out.println(f.getName() + " : " + f.getType().getSimpleName());   // nome : String
}
Object p = c.getDeclaredConstructor(String.class).newInstance("Ana");      // cria sem usar 'new Pessoa'
Method m = c.getMethod("getNome");
System.out.println(m.invoke(p));                                           // Ana
```

**Explicação do exemplo**: o programa descobre os campos de `Pessoa` em execução e imprime `nome : String`. Depois obtém o construtor que recebe uma `String`, cria uma instância **sem escrever `new Pessoa(...)`**, localiza o método `getNome` pelo nome e o invoca sobre o objeto criado, imprimindo `Ana`. É exatamente o que um framework faz para criar seus objetos a partir de configuração.

## 3.16 Annotations e *annotation processing*

**O que é**: **anotações** são **metadados** colocados no código (`@Nome`) que não alteram diretamente a lógica, mas podem ser lidos por compilador, ferramentas e frameworks. Declaram-se com `@interface`.

**Para que serve**: declarar **intenção e configuração** de forma concisa (`@Entity`, `@Test`, `@GetMapping`, `@NotNull`), orientar o compilador (`@Override`, `@FunctionalInterface`) e, com **processadores**, **gerar código** repetitivo (Lombok, MapStruct).

**Como funciona**:

- **Embutidas**: `@Override`, `@Deprecated`, `@SuppressWarnings`, `@FunctionalInterface`, `@SafeVarargs`.
- **Meta-anotações** (anotam a própria anotação): **`@Retention`**: `SOURCE` (só no fonte; descartada pelo compilador), `CLASS` (vai para o `.class`, mas **não** fica disponível em execução; o **padrão**) e `RUNTIME` (acessível por *reflection*); **`@Target`**: onde pode ser aplicada (`TYPE`, `FIELD`, `METHOD`, `PARAMETER`, `CONSTRUCTOR`...); `@Documented`, `@Inherited`, `@Repeatable`.
- **Elementos**: `String valor() default "";`. Se há um único elemento chamado `value`, pode-se escrever só `@Nome("x")`.
- **Leitura em execução** (anotações `RUNTIME`): `metodo.getAnnotation(Auditar.class)` (tópico 3.15).
- **Annotation Processing (JSR 269)**: durante a **compilação**, o `javac` invoca **processadores** (classes que estendem `AbstractProcessor`, com `@SupportedAnnotationTypes`) que recebem os elementos anotados e **geram novos arquivos** `.java` (via `Filer`; costuma-se usar a biblioteca **JavaPoet**). O processador **não consegue alterar** um arquivo existente (o Lombok usa APIs internas não oficiais para isso). O registro é por `META-INF/services/javax.annotation.processing.Processor` (ou `@AutoService`). Em Maven/Gradle, aparece como `annotationProcessor`.
- **Reflection x annotation processing**: reflection age em **execução** (custo em tempo, erros tardios); o processador age em **compilação** (custo zero em execução, erros cedo, compatível com GraalVM).

**Exemplo**:

```java
@Retention(RetentionPolicy.RUNTIME)         // visível em execução
@Target(ElementType.METHOD)                 // só em métodos
@interface Auditar { String motivo() default ""; }

class Servico {
    @Auditar(motivo = "saque")
    void sacar() { }
}

Method m = Servico.class.getDeclaredMethod("sacar");
if (m.isAnnotationPresent(Auditar.class)) {
    System.out.println(m.getAnnotation(Auditar.class).motivo());   // saque
}
```

**Explicação do exemplo**: `@Retention(RUNTIME)` mantém a anotação disponível depois da compilação; sem isso, `isAnnotationPresent` devolveria `false`. `@Target(METHOD)` faz o compilador recusar `@Auditar` em uma classe. O código usa *reflection* para ler o elemento `motivo` e imprime `saque`: é o mesmo mecanismo pelo qual frameworks leem `@GetMapping("/x")`.

## 3.17 Logging

**O que é**: registrar eventos da aplicação (diagnóstico, auditoria, erros) em **níveis** de severidade e para **destinos** configuráveis (console, arquivo, serviço de logs), em vez de usar `System.out.println` e `printStackTrace()`.

**Para que serve**: entender o que a aplicação fez em produção, investigar falhas e incidentes, auditar ações e medir comportamento, **sem recompilar** para ligar ou desligar detalhes (muda-se a configuração).

**Como funciona**:

- **Níveis** (do mais detalhado ao mais grave): `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`. A configuração define o **nível mínimo** mostrado (em produção, geralmente `INFO`; `DEBUG` para investigar).
- **Bibliotecas**: `java.util.logging` (JDK), `System.Logger` (Java 9), **Logback** (padrão do Spring Boot) e **Log4j2**. O **SLF4J** é uma **fachada**: o código usa a API do SLF4J e a implementação pode ser trocada sem alterar o código. O Lombok gera o *logger* com `@Slf4j`/`@Log4j2`.
- **Mensagem parametrizada**: `log.info("Usuário {} entrou", id)`: a mensagem só é montada se o nível estiver ativo (evita concatenação à toa). Para registrar o ***stack trace***, passe a exceção como **último argumento**: `log.error("Falha ao salvar {}", id, e)`.
- **Appenders** definem o destino (console, arquivo com **rotação**, rede); **layout/pattern** define o formato (data, nível, *thread*, classe, mensagem). **MDC** (*Mapped Diagnostic Context*) anexa contexto a cada linha (id da requisição, usuário). Há também logs **estruturados** em JSON para ferramentas de análise.
- **Boas práticas**: nunca registre **dados sensíveis** (senhas, tokens, cartão, dados pessoais); escolha o nível correto; **não registre e relance** a mesma exceção em cada camada (o erro aparece repetido); evite logar em laços muito quentes; um *logger* por classe (`static final`).

**Exemplo**:

```java
private static final Logger log = LoggerFactory.getLogger(PedidoService.class);

void salvar(Pedido p) {
    log.info("Salvando pedido {}", p.getId());              // INFO: fluxo normal
    try {
        repo.save(p);
    } catch (RuntimeException e) {
        log.error("Erro ao salvar pedido {}", p.getId(), e);  // ERROR + stack trace
        throw e;                                              // aqui só relança; quem trata, loga
    }
}
```

**Explicação do exemplo**: o `{}` é substituído pelo valor, sem concatenar texto manualmente. No `catch`, passar `e` como último argumento faz o *stack trace* completo aparecer no log. Aqui ocorre log **e** relançamento apenas para fins de demonstração; em um sistema real, escolha **um** ponto (normalmente a camada que trata a exceção, como um `@RestControllerAdvice`) para registrar.

## 3.18 Classes utilitárias úteis

**O que é**: classes pequenas e muito usadas da biblioteca padrão, que resolvem tarefas recorrentes.

**Para que serve**: validações rápidas, geração de identificadores e números aleatórios, leitura de configuração e medição de desempenho.

**Como funciona**:

- **`Objects`**: `requireNonNull(x, "mensagem")` (falha cedo com NPE e mensagem), `equals(a, b)` e `hash(...)` (seguros com `null`), `hashCode`, `toString(x, padrão)`, `isNull`/`nonNull`, `requireNonNullElse`.
- **`UUID`**: identificador de 128 bits; `UUID.randomUUID()` (versão 4, aleatória) é muito usado como chave primária e id de correlação.
- **Aleatoriedade**: `java.util.Random` (previsível, **não** use para segurança); `ThreadLocalRandom.current().nextInt(1, 100)` (melhor em ambientes multi-thread); **`SecureRandom`** (criptograficamente forte: senhas, *tokens*); `RandomGenerator` (Java 17) é a interface comum dos geradores.
- **`System`**: `currentTimeMillis()` (relógio de parede), **`nanoTime()`** (para **medir intervalos**), `getProperty`, `getenv` (variáveis de ambiente), `exit`, `lineSeparator`, `arraycopy`. **`Runtime`**: `availableProcessors()`, memória (`totalMemory`, `freeMemory`).
- **`Properties`**: carrega pares `chave=valor` de arquivos `.properties`. **`StringJoiner`** monta textos com separador. **`TimeUnit`**: unidades e pausas (`TimeUnit.SECONDS.sleep(2)`). **`Timer`**: legado; use `ScheduledExecutorService`.
- (`Optional`: tópico 4.3.)

**Exemplo**:

```java
Objects.requireNonNull(nome, "nome é obrigatório");        // NPE com mensagem se for null
String id = UUID.randomUUID().toString();                   // ex.: 3f1c...-...
int dado = ThreadLocalRandom.current().nextInt(1, 7);       // 1 a 6
String ambiente = System.getenv().getOrDefault("AMBIENTE", "dev");

Properties props = new Properties();
try (InputStream in = Files.newInputStream(Path.of("app.properties"))) {
    props.load(in);                                         // lê chave=valor
}
long t0 = System.nanoTime();
/* ... trabalho ... */
long ms = (System.nanoTime() - t0) / 1_000_000;             // duração em milissegundos
```

**Explicação do exemplo**: `requireNonNull` impede que um valor nulo "passe" silenciosamente, falhando perto da origem. O `UUID` aleatório é praticamente único. `nextInt(1, 7)` sorteia de 1 a 6 (o limite superior é exclusivo). Para medir tempo de execução, usa-se `nanoTime`, que é monotônico; `currentTimeMillis` pode andar para trás em ajustes de relógio.

---

# Parte 4: Java avançado

## 4.1 Lambdas e interfaces funcionais

**O que é**: uma **interface funcional** é uma interface com **exatamente um método abstrato** (métodos `default`, `static` e os herdados de `Object`, como `equals`, não contam). A anotação `@FunctionalInterface` é opcional, mas faz o compilador garantir essa regra. Uma **expressão lambda** é uma forma compacta de **implementar** essa interface sem escrever uma classe: `(parâmetros) -> corpo`. O compilador descobre qual interface está sendo implementada pelo **contexto** (o tipo esperado onde a lambda aparece) e **infere** os tipos dos parâmetros.

**Para que serve**: **passar comportamento como argumento** (um filtro, uma ordenação, um *callback*), escrever código mais curto e declarativo, e sustentar as *Streams*, o `CompletableFuture` e boa parte dos frameworks. Em vez de um método por critério (`filtrarVerdes`, `filtrarVermelhos`), você escreve um único `filtrar(lista, regra)` e passa a regra.

**Como funciona**:

```
(a, b) -> a + b                          // expressão: o resultado é devolvido sem 'return'
s -> s.length()                          // um parâmetro: parênteses opcionais
() -> new ArrayList<String>()            // sem parâmetros
(x, y) -> { int s = x + y; return s; }   // bloco: 'return' explícito
```

- **Tipagem pelo contexto**: `Predicate<String> p = s -> s.isEmpty();` só compila porque `Predicate<String>` tem um único método `boolean test(String)`; o mesmo texto de lambda pode implementar interfaces diferentes conforme o contexto.
- **Captura de variáveis**: a lambda pode usar variáveis locais do método **somente se forem *final* ou *efetivamente finais*** (nunca reatribuídas). Os valores são **copiados** para a lambda, e variáveis locais mudando por baixo causariam inconsistências (e problemas com *threads*). Atributos da classe podem ser lidos e alterados (cuidado com concorrência).
- **`this`** dentro de uma lambda é a **classe que a contém** (diferente de uma classe anônima, em que `this` é a própria instância anônima). Lambdas não são, por baixo, classes anônimas: o compilador usa `invokedynamic`.
- **Exceções *checked***: a lambda só pode lançá-las se o método da interface funcional as declarar (`Callable.call` declara `Exception`; `Function.apply` não).

As principais interfaces prontas de `java.util.function`:

| Interface | Método | Entrada → Saída | Uso típico |
| --- | --- | --- | --- |
| `Predicate<T>` | `boolean test(T)` | T → boolean | filtros (`filter`, `removeIf`) |
| `Function<T,R>` | `R apply(T)` | T → R | transformar (`map`) |
| `Consumer<T>` | `void accept(T)` | T → nada | efeito colateral (`forEach`) |
| `Supplier<T>` | `T get()` | nada → T | produzir/adiar (`orElseGet`, `generate`) |
| `BiFunction<T,U,R>` | `R apply(T,U)` | (T,U) → R | combinar dois valores |
| `BiPredicate<T,U>` | `boolean test(T,U)` | (T,U) → boolean | testes com dois argumentos |
| `UnaryOperator<T>` | `T apply(T)` | T → T | `iterate`, `replaceAll` |
| `BinaryOperator<T>` | `T apply(T,T)` | (T,T) → T | `reduce` |
| `Runnable` / `Callable<V>` | `run()` / `call()` | sem argumentos | tarefas concorrentes |
| `Comparator<T>` | `int compare(T,T)` | (T,T) → int | ordenação |

- **Versões primitivas** evitam *boxing*: `IntPredicate`, `IntFunction`, `ToIntFunction`, `IntBinaryOperator`...
- **Composição**: `p1.and(p2)`, `p1.or(p2)`, `p.negate()`, `Predicate.not(p)`; para funções, **`f.andThen(g)`** executa f e **depois** g (g(f(x))), e **`f.compose(g)`** executa g e **depois** f (f(g(x))); `Function.identity()`.
- Passar **comportamento como dado** (funções de ordem superior) é o princípio por trás do padrão *Strategy* e de todo o `java.util.stream`.

**Pontos importantes**: mantenha lambdas **curtas** (uma a três linhas); se crescer, extraia para um método e use *method reference* (4.2). Evite efeitos colaterais em lambdas de `filter`/`map`. `Supplier` serve para **adiar** a criação de um valor até ser realmente necessário.

**Exemplo**:

```java
static <T> List<T> filtrar(List<T> lista, Predicate<T> regra) {   // recebe comportamento
    List<T> resultado = new ArrayList<>();
    for (T e : lista) {
        if (regra.test(e)) resultado.add(e);          // aplica a regra recebida
    }
    return resultado;
}

List<Integer> pares = filtrar(List.of(1, 2, 3, 4), n -> n % 2 == 0);
System.out.println(pares);                               // [2, 4]

Function<Integer, Integer> dobro = x -> x * 2;
Function<Integer, Integer> maisUm = x -> x + 1;
System.out.println(dobro.andThen(maisUm).apply(5));       // 11  (5*2 e depois +1)
System.out.println(dobro.compose(maisUm).apply(5));       // 12  (5+1 e depois *2)

Predicate<String> longo = s -> s.length() > 3;
Predicate<String> comecaComJ = s -> s.startsWith("j");
System.out.println(longo.and(comecaComJ).test("java"));   // true
```

**Explicação do exemplo**: `filtrar` não sabe **o que** filtrar: a regra chega como um `Predicate`, e a lambda `n -> n % 2 == 0` mantém só os pares. O mesmo método serviria para Strings, objetos etc. (por ser genérico). Em `andThen`, o resultado do primeiro (10) alimenta o segundo (11); em `compose`, a ordem se inverte (primeiro `maisUm`, depois `dobro`, dando 12). `and` combina dois predicados: `"java"` tem mais de 3 letras **e** começa com "j".

## 4.2 Method references

**O que é**: um atalho com `::` para uma lambda que **apenas chama um método já existente**. Não muda o comportamento, só remove o ruído de "declarar um parâmetro para repassá-lo". `s -> s.length()` vira `String::length`.

**Para que serve**: tornar *pipelines*, ordenações e *callbacks* mais legíveis (`.map(Pessoa::getNome)`, `.forEach(System.out::println)`), dando ênfase ao **nome do método** em vez da mecânica de repasse.

**Como funciona**: quatro categorias, diferenciadas pelo **de onde vem o método**:

| Categoria | Sintaxe | Equivale a |
| --- | --- | --- |
| 1. Método **estático** | `Integer::parseInt` | `s -> Integer.parseInt(s)` |
| 2. Método de um **objeto específico** (já existente) | `System.out::println`, `comparador::comparar` | `x -> System.out.println(x)` |
| 3. Método de instância de um **objeto arbitrário de um tipo** | `String::toUpperCase`, `String::compareTo`, `List::contains` | `s -> s.toUpperCase()`; `(a, b) -> a.compareTo(b)` (o **1º parâmetro vira o receptor** da chamada) |
| 4. **Construtor** | `ArrayList::new`, `Anime::new`, `int[]::new` | `() -> new ArrayList<>()`; `(t, e) -> new Anime(t, e)` |

- As categorias 1 e 3 têm a **mesma sintaxe** (`Classe::metodo`); o compilador decide pela assinatura (o método é `static` ou de instância?). Se for ambíguo, dá erro.
- Nas referências a construtor, é a **interface funcional de destino** que decide qual construtor será usado: `Supplier<X> s = X::new` (sem argumentos) e `BiFunction<String,Integer,X> f = X::new` (dois argumentos).
- Se a lambda tem **lógica extra** (uma condição, uma conversão antes de chamar), ela **precisa** continuar sendo lambda.

**Exemplo**:

```java
List<String> nomes = new ArrayList<>(List.of("rui", "ana", "bia"));
nomes.sort(String::compareTo);                         // 3: objeto arbitrário -> [ana, bia, rui]
nomes.forEach(System.out::println);                    // 2: objeto específico (System.out)
Function<String, Integer> converter = Integer::parseInt;   // 1: método estático
Supplier<List<String>> novaLista = ArrayList::new;     // 4: construtor
System.out.println(converter.apply("42") + novaLista.get().size());   // 42 + 0 = 42
```

**Explicação do exemplo**: `String::compareTo` equivale a `(a, b) -> a.compareTo(b)`: o primeiro argumento é quem recebe a chamada, e o segundo é o parâmetro do método. `System.out::println` captura o objeto `System.out` e chama `println` para cada elemento. `Integer::parseInt` é estático, e `ArrayList::new` cria uma lista nova a cada `get()`. A soma final (`42 + 0`) mostra que `novaLista.get().size()` é 0.

## 4.3 Optional

**O que é**: um **contêiner** que **pode ou não** conter um valor. Seu propósito é tornar a **ausência de valor explícita no tipo de retorno**, em vez de devolver `null`, que é invisível na assinatura do método e fonte de `NullPointerException` quando quem chama esquece de verificar.

**Para que serve**: métodos de **busca** que podem não encontrar resultado (`findById`, `findByEmail`), obrigando o chamador a decidir o que fazer na ausência (valor padrão, exceção, ignorar), sem cadeias de `if (x != null)`.

**Como funciona**:

- **Criar**: `Optional.of(x)` (lança NPE se `x` for `null`), `Optional.ofNullable(x)` (vazio se `x` for `null`), `Optional.empty()`.
- **Consumir**: `ifPresent(consumer)`; `ifPresentOrElse(consumer, runnable)` (Java 9); `orElse(valor)`; `orElseGet(supplier)`; `orElseThrow(...)`; `isPresent()` / `isEmpty()` (Java 11).
- **`orElse` x `orElseGet`**: `orElse(valor)` **sempre avalia o argumento**, mesmo se o `Optional` tiver valor; use para constantes. `orElseGet(supplier)` só executa a lambda **se estiver vazio**; use quando criar o padrão é custoso ou tem efeito colateral.
- **Transformar**: `map(f)` (aplica `f` ao valor e reembrulha), `flatMap(f)` (quando `f` **já devolve** um `Optional`; evita `Optional<Optional<T>>`), `filter(p)`, `or(supplier)` (Java 9), `stream()` (Java 9).
- **`get()` sem verificar** lança `NoSuchElementException`: é o `null` disfarçado. Prefira `orElseThrow()` (sem argumento, Java 10), que declara a intenção.
- **Quando NÃO usar** (recomendação do próprio JDK): campos de classe, parâmetros de método, elementos de coleção, e no lugar de coleções (devolva lista vazia, não `Optional<List<T>>`). Foi desenhado para **tipo de retorno**. Também não é serializável por padrão.
- Variantes primitivas: `OptionalInt`, `OptionalLong`, `OptionalDouble`.

**Exemplo**:

```java
static Optional<String> buscarNome(int id) {
    return id == 1 ? Optional.of("Ana") : Optional.empty();
}

System.out.println(buscarNome(1).map(String::toUpperCase).orElse("DESCONHECIDO"));   // ANA
System.out.println(buscarNome(2).map(String::toUpperCase).orElse("DESCONHECIDO"));   // DESCONHECIDO
buscarNome(1).ifPresent(n -> System.out.println("achei " + n));                      // achei Ana

try {
    buscarNome(2).orElseThrow(() -> new IllegalStateException("não achei"));
} catch (IllegalStateException e) {
    System.out.println(e.getMessage());                                              // não achei
}
```

**Explicação do exemplo**: `buscarNome` deixa claro, **no tipo de retorno**, que pode não haver resultado. Com id 1, o `map` transforma o valor e `orElse` é ignorado; com id 2, o `Optional` está vazio, o `map` não faz nada e `orElse` entrega `"DESCONHECIDO"`. `ifPresent` só executa quando há valor. `orElseThrow` converte a ausência em exceção, para os casos em que não achar é um erro.

## 4.4 Streams

**O que é**: um ***pipeline*** declarativo para processar sequências de elementos. A **Stream não guarda dados** nem **altera a fonte**; descreve uma cadeia de operações. É **preguiçosa** (*lazy*) e de **uso único**. Estrutura em três partes: **fonte** → **operações intermediárias** (devolvem outra Stream) → **operação terminal** (dispara o processamento e produz um resultado).

**Para que serve**: filtrar, transformar, agrupar e agregar dados de coleções de forma **legível e concisa**, descrevendo **o que** se quer em vez de **como** iterar.

**Como funciona**:

- **Fontes**: `colecao.stream()`, `Arrays.stream(arr)`, `Stream.of(...)`, `IntStream.range`/`rangeClosed`, `Files.lines(path)` (mantém o arquivo aberto: feche), `Stream.iterate`, `Stream.generate`, `mapa.entrySet().stream()`.
- **Preguiçosa**: as operações intermediárias **só executam quando existe uma terminal**. Cada elemento percorre o *pipeline* **inteiro** (um por vez, "na vertical"), e operações de curto-circuito (`limit`, `findFirst`, `anyMatch`) encerram o processamento assim que possível.
- **Uso único**: depois de uma operação terminal, a Stream está consumida; reutilizá-la lança `IllegalStateException`. Crie outra com `.stream()`.
- **Intermediárias**: `filter`, `map`, `flatMap` (1:N e **achata**: `Stream<List<X>>` vira `Stream<X>`), `distinct` (usa `equals`/`hashCode`), `sorted`, `limit`, `skip`, `peek` (depuração), `takeWhile`/`dropWhile`, `mapToInt/Long/Double`, `boxed`. `sorted` e `distinct` têm **estado** (precisam ver todos os elementos), o que reduz a preguiça.
- **Terminais**: `forEach` (em paralelo a ordem não é garantida; use `forEachOrdered`), `collect`, `toList()` (Java 16; **imutável**), `toArray`, `count`, `min`/`max`, `reduce`, `anyMatch`/`allMatch`/`noneMatch` (booleanos, com curto-circuito; `allMatch` em stream vazia é `true`), `findFirst`/`findAny` (devolvem `Optional`).
- **`reduce`**: combina todos em um valor. O operador deve ser **associativo**. `reduce(0, Integer::sum)` (com identidade) devolve o valor direto; sem identidade devolve `Optional` (a stream pode ser vazia).
- **Streams primitivas** (`IntStream`, `LongStream`, `DoubleStream`): evitam *boxing*; oferecem `sum`, `average` (`OptionalDouble`), `min`, `max`, `summaryStatistics`. `range(1, 5)` exclui o fim; `rangeClosed(1, 5)` inclui.
- **Infinitas**: `Stream.iterate(1, n -> n * 2)` e `Stream.generate(supplier)`; exigem `limit`/`takeWhile`, senão rodam para sempre. `iterate(seed, hasNext, next)` (Java 9) tem condição de parada.
- **`Collectors`**: `toList`, `toSet`, `toMap(chave, valor[, merge])` (**chave repetida sem `merge` lança `IllegalStateException`**), `joining`, `counting`, `summingX`, `averagingX`, `summarizingX`, `maxBy`/`minBy`, `mapping`, `collectingAndThen`, `teeing`, `toCollection`.
- **Agrupamento**: **`groupingBy(classificador)`** devolve `Map<Chave, List<T>>`; aceita um **coletor *downstream*** (`groupingBy(f, counting())`) e pode ser aninhado (multinível). **`partitioningBy(predicado)`** separa em `true`/`false`.
- **Paralelas**: `.parallel()` divide o trabalho no *ForkJoin common pool*. Só compensa com **muitos dados**, **custo alto por elemento**, fonte fácil de dividir (`ArrayList`, `range`; **não** `iterate` nem `LinkedList`) e operações **sem estado, sem interferência e sem efeitos colaterais**. Em geral **meça** antes (o custo de dividir e juntar pode superar o ganho).
- **Boas práticas**: não altere a fonte nem variáveis externas dentro do *pipeline*; `filter` antes de `map`/`sorted` caros; não use stream quando um laço simples for mais claro (precisa de índice, `break` complexo ou estado mutável).

**Exemplo**:

```java
List<String> r = Stream.of("a", "bb", "ccc", "dd")
        .peek(s -> System.out.println("filter " + s))
        .filter(s -> s.length() > 1)
        .peek(s -> System.out.println("map " + s))
        .map(String::toUpperCase)
        .limit(2)
        .toList();
System.out.println(r);                            // [BB, CCC]

record Livro(String titulo, double preco, String categoria) {}
List<Livro> livros = List.of(new Livro("A", 10, "X"), new Livro("B", 30, "X"), new Livro("C", 20, "Y"));

Map<String, Long> porCategoria = livros.stream()
        .collect(Collectors.groupingBy(Livro::categoria, Collectors.counting()));   // {X=2, Y=1}
double media = livros.stream().mapToDouble(Livro::preco).average().orElse(0);       // 20.0
```

**Explicação do exemplo**: a primeira stream mostra a **preguiça**. As saídas intermediárias são `filter a`, `filter bb`, `map bb`, `filter ccc`, `map ccc`: cada elemento passa por todo o *pipeline* antes do próximo; `"a"` é barrado no filtro; assim que `limit(2)` recebe `"BB"` e `"CCC"`, a stream **para**, e `"dd"` nem é visitado (curto-circuito). O resultado é `[BB, CCC]`. No segundo trecho, `groupingBy` agrupa os livros pela categoria e `counting()` conta cada grupo; `mapToDouble(...).average()` calcula a média dos preços (10 + 30 + 20) / 3 = 20.0, e `orElse(0)` cobre o caso de lista vazia.

## 4.5 Concorrência: threads, sincronização e coleções concorrentes

**O que é**: executar várias tarefas **ao mesmo tempo** (ou de forma intercalada). Uma **thread** é uma linha de execução independente dentro de um processo: todas as threads **compartilham o heap** (daí os riscos), mas cada uma tem sua **própria pilha**. Todo programa tem ao menos a thread `main`. *Concorrência* é lidar com várias tarefas em andamento; *paralelismo* é executá-las **simultaneamente** em vários núcleos.

**Para que serve**: aproveitar vários núcleos de CPU, manter uma aplicação responsiva (não travar a interface/atendimento enquanto espera I/O) e atender muitas requisições ao mesmo tempo.

**Como funciona**:

**Criação e ciclo de vida**

- Implemente **`Runnable`** (preferido: separa a tarefa de quem a executa) e passe a uma `Thread`, ou estenda `Thread` (gasta a única herança da classe). `new Thread(tarefa, "nome").start();`.
- **`start()` x `run()`**: `start()` cria uma **nova thread** que executa `run()`. Chamar `run()` diretamente executa na **thread atual**, **sem concorrência nenhuma**.
- **Estados** (`Thread.State`): `NEW` → `RUNNABLE` → `BLOCKED` (esperando um *monitor*), `WAITING`/`TIMED_WAITING` (`wait`, `sleep`, `join`) → `TERMINATED`.
- `Thread.sleep(ms)` pausa a thread atual (lança `InterruptedException`). **`join()`** faz quem chamou esperar a outra terminar. **`interrupt()`** pede o cancelamento de forma **cooperativa**: a tarefa deve verificar `isInterrupted()` ou tratar `InterruptedException`; **nunca engula** essa exceção (restaure com `Thread.currentThread().interrupt()`). `setPriority` é só uma dica ao SO.
- **Daemon x usuário**: a JVM encerra quando todas as threads de **usuário** terminam; as *daemon* (`setDaemon(true)`, antes de `start`) não a seguram.

**Problemas clássicos**

- **Condição de corrida (*race condition*)**: duas threads leem/alteram o mesmo dado sem coordenação. `saldo -= v` **não é atômico** (ler, calcular, escrever: três passos). O padrão ***check-then-act*** (`if (saldo >= x) saldo -= x`) também é corrida: duas threads passam no `if` antes de qualquer uma debitar.
- **Visibilidade**: sem sincronização, uma thread pode **nunca enxergar** a escrita de outra (cache de CPU, reordenação de instruções). O **Modelo de Memória do Java** define a relação ***happens-before*** (liberar/adquirir um lock, escrever/ler um `volatile`, `start`, `join`) que garante visibilidade e ordem.
- **Deadlock**: duas threads esperam uma pela outra para sempre, cada uma segurando um lock que a outra quer. Prevenção: **mesma ordem** de aquisição de locks, `tryLock` com tempo, menos locks aninhados. Diagnóstico: *thread dump* (`jstack`). Também existem ***starvation*** (uma thread nunca consegue o recurso) e ***livelock***.

**Ferramentas de sincronização**

- **`synchronized`**: só **uma thread por vez** executa o bloco para o mesmo ***monitor*** (objeto de lock). Em método de instância o lock é `this`; em método `static`, a `Class`. Garante **exclusão mútua** e **visibilidade**. Todos os acessos (inclusive **leituras**) ao dado compartilhado devem usar o **mesmo** lock; mantenha a seção crítica **pequena** e sem I/O.
- **`volatile`**: garante **visibilidade** e ordenação de uma variável, mas **não atomicidade** (`contador++` continua inseguro). Bom para *flags* de parada.
- **`wait()` / `notify()` / `notifyAll()`**: espera e notificação dentro de `synchronized` sobre o **mesmo objeto** (senão `IllegalMonitorStateException`). `wait` **libera o lock** e dorme; deve estar **em um `while`** que recheca a condição (existem *spurious wakeups*). É o padrão **produtor-consumidor**. Esquecer de notificar quando o estado muda (por exemplo, ao "fechar" a fila) deixa consumidores dormindo para sempre.
- **Atômicos** (`AtomicInteger`, `AtomicLong`, `AtomicReference`, `LongAdder`): operações compostas **indivisíveis** (`incrementAndGet`, `compareAndSet`) sem lock, via instruções da CPU (CAS). Ideais para **contadores**.
- **`Lock` / `ReentrantLock`**: lock explícito, **sempre** em `lock(); try { ... } finally { unlock(); }`. Extras sobre `synchronized`: `tryLock()`/`tryLock(tempo)`, `lockInterruptibly()`, modo **justo**, `isHeldByCurrentThread()`. **Reentrante**: a mesma thread pode readquirir (precisa liberar o mesmo número de vezes). **`Condition`** (`newCondition()`, `await`, `signalAll`) é o `wait/notify` por lock, com várias condições.
- **`ReentrantReadWriteLock`**: vários **leitores** simultâneos **ou** um **escritor** exclusivo; vale quando há muito mais leitura que escrita. Não devolva visões vivas (`keySet()`) para fora do lock.
- **Sincronizadores**: `CountDownLatch` (espera N eventos), `CyclicBarrier`, `Semaphore` (limita acessos simultâneos), `Phaser`. **`ThreadLocal`**: um valor por thread.

**Coleções concorrentes**

- **`ConcurrentHashMap`**: mapa *thread-safe* com concorrência fina; operações atômicas (`putIfAbsent`, `computeIfAbsent`, `merge`); **não aceita `null`** (nem chave nem valor). `Collections.synchronizedList/Map` bloqueiam a coleção toda e ainda exigem sincronizar a iteração manualmente.
- **`CopyOnWriteArrayList`**: cada escrita copia o array; os iteradores usam um *snapshot* e **nunca** lançam `ConcurrentModificationException`. Ótima para **muita leitura e pouca escrita** (listas de *listeners*).
- **`BlockingQueue`** (`ArrayBlockingQueue`, `LinkedBlockingQueue`, `SynchronousQueue`): `put` **bloqueia** se cheia; `take` **bloqueia** se vazia; `offer`/`poll` com tempo. É o **produtor-consumidor pronto**. **`TransferQueue`** acrescenta `transfer()`, que só retorna quando um consumidor **recebeu** o item.

**Pontos importantes**: a forma mais simples de ser seguro entre threads é **não compartilhar estado mutável**: objetos **imutáveis** (records, campos `final`, `List.of`) podem ser lidos por todos sem lock. Quando precisar compartilhar, prefira as ferramentas de alto nível (`java.util.concurrent`) a `wait/notify` manual.

**Exemplo**:

```java
class Contador {
    private int n;
    public synchronized void inc() { n++; }          // exclusão mútua
    public synchronized int get() { return n; }
}
Contador c = new Contador();
AtomicInteger at = new AtomicInteger();              // alternativa sem lock
Runnable tarefa = () -> {
    for (int i = 0; i < 10_000; i++) { c.inc(); at.incrementAndGet(); }
};
Thread t1 = new Thread(tarefa), t2 = new Thread(tarefa);
t1.start(); t2.start();
t1.join(); t2.join();                                // espera as duas terminarem
System.out.println(c.get() + " " + at.get());        // 20000 20000

BlockingQueue<String> fila = new ArrayBlockingQueue<>(10);
fila.put("x");                                       // produtor
System.out.println(fila.take());                     // consumidor (bloqueia se vazia): x
```

**Explicação do exemplo**: duas threads executam 10.000 incrementos cada. Com `synchronized` (ou com `AtomicInteger`), nenhum incremento se perde, e o resultado é sempre **20000**. Sem proteção, `n++` (ler, somar, gravar) seria intercalado entre as threads e o resultado ficaria **menor e diferente a cada execução**. O `join()` impede que o `main` leia o valor antes de as threads terminarem. A fila bloqueante mostra a troca de dados entre threads sem `wait/notify` manual.

## 4.6 Executors, `Future` e `CompletableFuture`

**O que é**: criar uma `Thread` por tarefa é **caro** (memória, criação) e não escala. Um **`ExecutorService`** mantém um **pool de threads reutilizáveis**: você envia **tarefas** e o pool as executa, sem você gerenciar threads.

**Para que serve**: controlar o grau de paralelismo, reaproveitar threads, devolver **resultados** das tarefas e, com `CompletableFuture`, **encadear** etapas assíncronas sem bloquear threads esperando.

**Como funciona**:

| Fábrica (`Executors`) | Comportamento |
| --- | --- |
| `newFixedThreadPool(n)` | n threads fixas; tarefas excedentes ficam em **fila** (para CPU: `availableProcessors()`) |
| `newCachedThreadPool()` | cria threads sob demanda e reaproveita as ociosas; pode explodir sob carga |
| `newSingleThreadExecutor()` | uma só thread; tarefas em ordem |
| `newScheduledThreadPool(n)` | tarefas atrasadas ou periódicas |
| `newVirtualThreadPerTaskExecutor()` | uma **thread virtual** por tarefa (tópico 4.7) |

- **`execute(Runnable)`** não devolve nada; **`submit`** devolve um **`Future`**. `invokeAll`/`invokeAny` rodam coleções de tarefas.
- **Encerrar**: **sempre** `shutdown()` (para de aceitar novas tarefas e termina as em andamento) e, se preciso, `awaitTermination` e `shutdownNow()` (interrompe). Sem isso o programa pode **nunca terminar** (as threads do pool são de usuário). Desde o Java 19, `ExecutorService` é `AutoCloseable` (use em *try-with-resources*).
- **`ScheduledExecutorService`**: `schedule` (uma vez, com atraso), `scheduleAtFixedRate` (período contado do **início** de cada execução) e `scheduleWithFixedDelay` (espera contada do **fim** da anterior).
- **`Callable<V>`**: como `Runnable`, mas **devolve valor e pode lançar `Exception`**.
- **`Future<V>`**: promessa de resultado: `get()` **bloqueia** até ficar pronto; `get(tempo, unidade)` lança `TimeoutException`; `cancel`, `isDone`. Falhas da tarefa chegam em `ExecutionException` (use `getCause()`). Limitação: **não compõe nem encadeia**.

**`CompletableFuture<T>`**: um `Future` **composável** para pipelines assíncronos.

| Método | Uso |
| --- | --- |
| `supplyAsync(Supplier[, executor])` / `runAsync(Runnable)` | inicia a tarefa assíncrona |
| `thenApply` / `thenAccept` / `thenRun` | transformar (`map`) / consumir / só executar |
| `thenCompose` | a função **devolve** um `CompletableFuture` (como `flatMap`; evita aninhar) |
| `thenCombine(outro, fn)` | junta o resultado de **dois** futures independentes |
| `allOf(...)` / `anyOf(...)` | espera **todos** / o **primeiro** |
| `exceptionally` / `handle` / `whenComplete` | tratamento de erro |
| `orTimeout` / `completeOnTimeout` | limites de tempo (Java 9) |
| `join()` x `get()` | `join` lança *unchecked* (`CompletionException`); `get` lança *checked* |

- **Sem passar executor**, usa o `ForkJoinPool.commonPool()` (threads *daemon*; tamanho ≈ núcleos − 1). Para **I/O bloqueante**, passe um executor próprio, para não esgotar o pool comum.
- **Armadilha da stream preguiçosa**: em `stream.map(x -> supplyAsync(...)).map(CompletableFuture::join)`, cada elemento passa por todo o *pipeline* antes do próximo, e as tarefas rodam **uma de cada vez**. **Colete** os futures em uma lista primeiro (todos iniciam) e só depois dê `join`.
- Os métodos com sufixo `Async` (`thenApplyAsync`) executam a etapa em outro thread do pool.

**Exemplo**:

```java
try (ExecutorService ex = Executors.newFixedThreadPool(4)) {    // fecha sozinho (Java 19+)
    Future<Integer> f = ex.submit(() -> { Thread.sleep(500); return 42; });
    System.out.println(f.get(2, TimeUnit.SECONDS));            // 42 (espera até 2 s)
}

List<CompletableFuture<Double>> futuros = lojas.stream()
        .map(l -> CompletableFuture.supplyAsync(() -> buscarPreco(l)))
        .toList();                                             // inicia TODAS as buscas
List<Double> precos = futuros.stream().map(CompletableFuture::join).toList();

CompletableFuture.supplyAsync(() -> "dado")
        .thenApply(String::toUpperCase)
        .thenAccept(System.out::println)                       // DADO
        .exceptionally(e -> { System.err.println(e); return null; })
        .join();
```

**Explicação do exemplo**: o `submit` envia um `Callable` ao pool e devolve um `Future`; `get` espera o resultado (42) por até 2 segundos. Na segunda parte, **todas** as buscas de preço começam antes de qualquer `join`, de modo que rodam em paralelo; o tempo total é o da **mais lenta**, não a soma. Na terceira, cada `then...` é uma etapa que executa quando a anterior termina, sem bloquear a thread principal; `exceptionally` captura falhas de qualquer etapa anterior e o `join()` final apenas espera para o programa não terminar antes.

## 4.7 Threads virtuais (Java 21)

**O que é**: threads **leves**, gerenciadas pela **JVM**, e não mapeadas 1:1 para threads do sistema operacional (recurso final no Java 21, JEP 444). Uma thread virtual roda **"montada"** em uma *thread de plataforma* (a *carrier thread*); quando **bloqueia** em I/O (rede, banco, `sleep`), a JVM a **desmonta**, libera a *carrier* para outra thread virtual e a remonta quando o I/O termina.

**Para que serve**: servidores com **muitas conexões simultâneas bloqueantes**. Em vez de programação reativa/assíncrona complexa, você escreve código **síncrono e simples** (uma thread por requisição) e ainda escala para dezenas ou centenas de milhares de tarefas concorrentes.

**Como funciona**:

- **Criar**: `Thread.ofVirtual().start(tarefa)`, `Thread.startVirtualThread(r)` ou, mais comum, `Executors.newVirtualThreadPerTaskExecutor()`.
- **Não use *pool* de threads virtuais**: são baratas, então crie **uma por tarefa**. Para limitar o acesso concorrente a um recurso (banco, API), use `Semaphore`.
- Servem para tarefas **I/O-bound**; para **CPU-bound** não trazem ganho (o gargalo continua sendo os núcleos).
- ***Pinning***: em certos casos a thread virtual fica "presa" à *carrier* durante o bloqueio, perdendo o benefício. Até o Java 23, um bloco `synchronized` com I/O bloqueante causava isso (a alternativa era `ReentrantLock`); a partir do Java 24 (JEP 491) esse caso foi em grande parte resolvido, mas chamadas nativas ainda podem prender a thread (verifique na documentação da sua versão).
- **`ThreadLocal`** funciona, mas, com milhões de threads, evite guardar objetos grandes nele.
- No **Spring Boot**: `spring.threads.virtual.enabled=true` faz o framework usar threads virtuais (requisições web, tarefas `@Async`).
- **Concorrência estruturada** (tratar um grupo de subtarefas como uma unidade) ainda é recurso em **pré-visualização** nas versões recentes (verifique a sua).

**Exemplo**:

```java
try (var ex = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10_000; i++) {
        ex.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));   // bloqueia, mas não prende thread de plataforma
            return null;
        });
    }
}   // o close() espera todas terminarem
```

**Explicação do exemplo**: são criadas **10.000** threads virtuais, cada uma dormindo 1 segundo. Como o `sleep` apenas **desmonta** a thread virtual, bastam poucas *carrier threads* para atender todas, e o conjunto termina em cerca de **1 segundo**. Com 10.000 threads de plataforma isso consumiria muita memória (cada uma reserva uma pilha) e seria bem mais lento. O `try-with-resources` fecha o executor e espera as tarefas.

## 4.8 Gerenciamento de memória e *Garbage Collector*

**O que é**: a JVM aloca objetos no **heap** e os **libera automaticamente** quando não são mais **alcançáveis** a partir das **raízes** (variáveis locais nas pilhas das threads, campos `static`, entre outras). Você não chama `free`. Um objeto "vivo" é aquele que ainda pode ser alcançado seguindo referências; o restante é lixo.

**Para que serve**: entender por que uma aplicação consome muita memória, tem pausas ou **vaza** memória, e dimensionar e depurar a JVM.

**Como funciona**:

- **Raízes e alcançabilidade**: o GC marca tudo que é alcançável e recolhe o resto. **Vazamento (*leak*) em Java** não é esquecer um `free`, e sim **manter referência** a objetos que nunca mais serão usados: coleções estáticas que só crescem, *caches* sem limite, *listeners* não removidos, `ThreadLocal` esquecido.
- **Gerações** (hipótese de que "a maioria dos objetos morre jovem"): objetos novos nascem na **Young** (*Eden* + áreas *Survivor*); os que sobrevivem a várias coletas são promovidos à **Old**. A coleta da Young (*minor*) é rápida e frequente; a da Old (*major/full*) é mais cara.
- **Pausas (*stop-the-world*)**: parte do trabalho do GC pausa a aplicação. Os coletores diferem em **pausa** x **vazão** (*throughput*).
- **Coletores**: **G1** (padrão em máquinas com recursos de servidor desde o Java 9; divide o heap em regiões, pausas previsíveis), **ZGC** e **Shenandoah** (pausas muito curtas mesmo com heaps enormes; o ZGC passou a ser geracional nas versões recentes), **Parallel** (foco em vazão) e **Serial** (programas pequenos). Os detalhes mudam entre versões: confirme na documentação do seu JDK.
- **Tipos de referência**: **forte** (a normal, impede a coleta), **`SoftReference`** (coletada só sob pressão de memória; útil em *caches*), **`WeakReference`** (coletada na próxima coleta se não houver referência forte; base de `WeakHashMap`), **`PhantomReference`** (notificação pós-coleta).
- **Erros**: `OutOfMemoryError` (heap cheio, ou outras áreas), `StackOverflowError` (recursão profunda demais).
- **Parâmetros comuns**: `-Xms`/`-Xmx` (heap inicial e máximo), `-Xss` (tamanho da pilha de cada thread), `-XX:+UseZGC` etc. Em **contêineres**, a JVM respeita os limites de memória do contêiner nas versões modernas.
- **Ferramentas**: `jcmd`, `jmap` (*heap dump*), `jstat`, **VisualVM**, **JFR** (*Java Flight Recorder*), Eclipse MAT para analisar *dumps*.
- **Boas práticas**: não crie objetos desnecessários em laços muito quentes; **feche recursos** (`try-with-resources`); **limite o tamanho de caches**; use `StringBuilder` em vez de `+=` em laços; **não chame `System.gc()`** (é só uma sugestão); `finalize()` foi descontinuado para remoção.

**Exemplo**:

```java
class Cache {
    // VAZAMENTO: o mapa estático só cresce, e os objetos nunca ficam inalcançáveis
    private static final Map<String, byte[]> dados = new HashMap<>();
    static void guardar(String k) { dados.put(k, new byte[1_000_000]); }
}

// Solução: limitar o tamanho (ex.: LinkedHashMap com removeEldestEntry) ou usar referências fracas
Map<String, Integer> lru = new LinkedHashMap<>(16, 0.75f, true) {   // true = ordem de acesso
    @Override protected boolean removeEldestEntry(Map.Entry<String, Integer> eldest) {
        return size() > 100;                                          // descarta o menos usado
    }
};

Runtime rt = Runtime.getRuntime();
long usadoMb = (rt.totalMemory() - rt.freeMemory()) / 1_048_576;
System.out.println(usadoMb + " MB em uso, máximo " + rt.maxMemory() / 1_048_576 + " MB");
```

**Explicação do exemplo**: o mapa `static` mantém **referência forte** a cada `byte[]` inserido; como o próprio mapa é alcançável durante toda a execução, o GC **nunca** pode liberar esses arrays, e o programa acaba com `OutOfMemoryError`. Um `LinkedHashMap` em modo de acesso, com `removeEldestEntry`, vira um *cache LRU* de tamanho limitado: ao passar de 100 entradas, a menos usada é removida e pode ser coletada. As últimas linhas mostram, via `Runtime`, a memória atualmente em uso e o limite do heap (`-Xmx`).

---

# Parte 5: Persistência e qualidade

## 5.1 Bancos de dados: conceitos essenciais

**O que é**: base conceitual para entender JDBC, JPA e Spring Data. Um **banco de dados** guarda dados de forma persistente e organizada, e um **SGBD** (MySQL, PostgreSQL, Oracle, SQL Server, MongoDB, Redis) permite consultá-los e alterá-los com segurança e concorrência.

**Para que serve**: escolher **onde e como** guardar cada tipo de dado, modelar tabelas e relacionamentos corretamente e entender o comportamento de transações, que afetam diretamente o código Java.

**Como funciona**:

**Modelo relacional (SQL)**

- Dados em **tabelas** (linhas = registros, colunas = atributos). **Chave primária** (`PRIMARY KEY`) identifica cada linha; **chave estrangeira** (`FOREIGN KEY`) referencia a chave de outra tabela e garante **integridade referencial**.
- Comandos: **DML** (`SELECT`, `INSERT`, `UPDATE`, `DELETE`), **DDL** (`CREATE`, `ALTER`, `DROP`), **DCL/TCL** (`GRANT`, `COMMIT`, `ROLLBACK`).
- **`JOIN`** combina tabelas: `INNER JOIN` (só linhas com correspondência nos dois lados), `LEFT JOIN` (todas da esquerda, com `NULL` onde não há correspondência), `RIGHT JOIN`, `FULL JOIN`. `GROUP BY` agrupa; `WHERE` filtra **linhas** antes de agrupar e `HAVING` filtra **grupos** depois. `NULL` significa "desconhecido": `x = NULL` nunca é verdadeiro (use `IS NULL`).
- **Índice** (em geral uma árvore B) acelera buscas por uma coluna, ao custo de espaço e de escritas mais lentas. Uma consulta sem índice faz varredura completa da tabela.

**Normalização** (reduz redundância e anomalias de atualização):

- **1FN**: valores **atômicos** (nada de lista de telefones em uma coluna) e sem grupos repetidos.
- **2FN**: 1FN + cada atributo não-chave depende da chave **inteira** (relevante com chaves compostas; sem dependência parcial).
- **3FN**: 2FN + atributos não-chave **não dependem de outros atributos não-chave** (sem dependência transitiva): o nome da cidade não deve ficar na tabela de clientes se depende do CEP.
- Às vezes se **desnormaliza** de propósito por desempenho de leitura.

**Transações e ACID**: **A**tomicidade (tudo ou nada), **C**onsistência (as regras do banco continuam valendo), **I**solamento (transações concorrentes não se atrapalham), **D**urabilidade (o que foi confirmado sobrevive a falhas). Os **níveis de isolamento** controlam as anomalias permitidas:

| Nível | Evita |
| --- | --- |
| `READ UNCOMMITTED` | nada (permite *dirty read*: ler dado ainda não confirmado) |
| `READ COMMITTED` | *dirty read* |
| `REPEATABLE READ` | + *non-repeatable read* (reler a mesma linha e achar outro valor) |
| `SERIALIZABLE` | + *phantom read* (reexecutar a consulta e aparecerem linhas novas) |

O padrão varia: PostgreSQL usa `READ COMMITTED`; o MySQL/InnoDB usa `REPEATABLE READ`. Mais isolamento = menos anomalias e menos concorrência.

**NoSQL**: **documento** (MongoDB: JSON flexível), **chave-valor** (Redis: muito rápido, TTL, cache), **colunar** (Cassandra), **grafo** (Neo4j). Em geral trocam esquema rígido e *joins* por flexibilidade e escala horizontal. **Persistência poliglota** é usar o banco mais adequado a cada necessidade dentro do mesmo sistema (relacional para pedidos, documento para catálogos, Redis para cache).

**Exemplo**:

```sql
CREATE TABLE producer (
  id   INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL
);
CREATE TABLE anime (
  id          INT PRIMARY KEY AUTO_INCREMENT,
  name        VARCHAR(100),
  producer_id INT,
  FOREIGN KEY (producer_id) REFERENCES producer(id)
);

SELECT a.name AS anime, p.name AS estudio
FROM anime a
INNER JOIN producer p ON a.producer_id = p.id
WHERE a.name LIKE '%Nar%';
```

**Explicação do exemplo**: a tabela `anime` referencia `producer` por chave estrangeira, o que impede cadastrar um anime com estúdio inexistente. A consulta liga as duas tabelas com `INNER JOIN` pela correspondência `producer_id = id` e devolve só os animes cujo nome contém "Nar", junto com o nome do estúdio. Sem o `ON`, o resultado seria o produto cartesiano das duas tabelas.

## 5.2 JDBC (*Java Database Connectivity*)

**O que é**: a API padrão do Java (`java.sql`) para conversar com **bancos relacionais**. Um **driver** (`mysql-connector-j`, `postgresql`) implementa as interfaces `Connection`, `Statement`, `PreparedStatement` e `ResultSet` para cada banco; seu código usa só as interfaces, então trocar de banco é, em tese, trocar o driver e a URL. Desde o JDBC 4, o driver presente no *classpath* é registrado automaticamente (não precisa de `Class.forName`).

**Para que serve**: acesso **direto e de baixo nível** ao banco (executar SQL, ler resultados, controlar transações). É a **base** sobre a qual JPA/Hibernate, Spring Data e `JdbcTemplate` são construídos; entendê-lo explica o que esses frameworks fazem por baixo.

**Como funciona**:

- **Conexão**: `DriverManager.getConnection(url, usuario, senha)` com uma **URL** como `jdbc:mysql://localhost:3306/loja`. Em produção, usa-se um **`DataSource` com *pool* de conexões** (HikariCP): abrir conexão é caro, o pool as **reaproveita** (e `close()` apenas devolve a conexão ao pool). O Spring Boot já configura um *pool*.
- **Escolha do *statement***:

|  | `Statement` | `PreparedStatement` | `CallableStatement` |
| --- | --- | --- | --- |
| SQL | texto montado por concatenação | texto com `?` e parâmetros tipados | chamada de *stored procedure* (`CALL sp(?)`) |
| **SQL Injection** | **vulnerável** | **protegido** (o parâmetro nunca vira SQL) | protegido se usar `?` |

- **SQL Injection**: se a entrada do usuário é concatenada ao SQL, ela pode **alterar a consulta**. Com `"... WHERE nome = '" + nome + "'"`, um valor como `x' OR '1'='1` devolve a tabela inteira. **Regra absoluta: nunca concatene dados externos em SQL.** No `PreparedStatement`, os valores viajam **separados** do comando. Limitação: parâmetros `?` só valem para **valores**, não para nomes de tabela/coluna (esses, se variáveis, precisam de lista branca). Em `LIKE`, o `%` faz parte do **valor** (`ps.setString(1, "%" + nome + "%")`).
- **Execução**: `executeQuery()` (`SELECT` → `ResultSet`), `executeUpdate()` (`INSERT/UPDATE/DELETE` → número de linhas afetadas), `execute()` (genérico). **Chaves geradas**: `prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)` + `getGeneratedKeys()`. **Lote**: `addBatch()` repetido + `executeBatch()` envia vários comandos de uma vez (muito mais rápido).
- **`ResultSet`**: um cursor que começa **antes** da primeira linha; `next()` avança e devolve `false` ao fim. **Colunas começam em 1**; prefira o nome (`rs.getString("name")`). Para colunas anuláveis, `getInt` devolve 0 se for `NULL`: use `wasNull()` ou `getObject`. Tipos de cursor: `TYPE_FORWARD_ONLY` (padrão), `TYPE_SCROLL_INSENSITIVE`/`SENSITIVE` (navegável: `last`, `absolute`, `previous`) e `CONCUR_UPDATABLE` (permite `updateRow`, `insertRow`, `deleteRow` pelo próprio cursor; nem todo driver suporta).
- **Metadados**: `ResultSetMetaData` (colunas, tipos) e `DatabaseMetaData` (o que o driver/banco suporta).
- **Recursos**: `Connection`, `Statement` e `ResultSet` devem ser **fechados**; o vazamento de conexões esgota o *pool* e derruba a aplicação. **Sempre `try-with-resources`**.
- **Transações**: por padrão cada comando já é confirmado (**auto-commit**). Para agrupar vários: `conn.setAutoCommit(false)`, executar, `conn.commit()`; em erro, **um** `conn.rollback()` e propagar a falha (não continue o laço depois de uma falha, ou os próximos itens podem ser confirmados no `commit` final). `Savepoint` desfaz só parte. `setTransactionIsolation` define o isolamento. Restaure o auto-commit ao devolver a conexão ao *pool*.
- **Exceções**: `SQLException` traz `getSQLState()` e `getErrorCode()`.
- **`RowSet`**: `JdbcRowSet` (conectado, "JavaBean") e `CachedRowSet` (desconectado: altera offline e `acceptChanges()` sincroniza); uso raro hoje.

**Pontos importantes**: JDBC é verboso (mapear `ResultSet` para objetos, controlar transação, fechar recursos): é por isso que existem JPA, Spring Data e `JdbcTemplate`. Credenciais **nunca** no código (variáveis de ambiente, cofres). Isole o SQL em uma **camada de repositório**.

**Exemplo**:

```java
String sql = "INSERT INTO producer (name) VALUES (?)";
try (Connection conn = dataSource.getConnection()) {
    conn.setAutoCommit(false);                          // início da transação manual
    try (PreparedStatement ps = conn.prepareStatement(sql)) {
        for (String nome : List.of("Madhouse", "Toei", "Ghibli")) {
            ps.setString(1, nome);                       // parâmetro 1 (índice começa em 1)
            ps.addBatch();
        }
        int[] resultados = ps.executeBatch();            // envia tudo de uma vez
        conn.commit();                                   // confirma os 3 inserts juntos
    } catch (SQLException e) {
        conn.rollback();                                 // desfaz TUDO se algo falhou
        throw e;
    }
}

try (Connection conn = dataSource.getConnection();
     PreparedStatement ps = conn.prepareStatement("SELECT id, name FROM producer WHERE name LIKE ?")) {
    ps.setString(1, "%a%");
    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) System.out.println(rs.getInt("id") + " " + rs.getString("name"));
    }
}
```

**Explicação do exemplo**: na primeira parte, três inserções são enviadas em **lote** e confirmadas **juntas** em uma transação: se qualquer uma falhar, o `rollback` desfaz todas, mantendo o banco consistente (atomicidade). O `PreparedStatement` é reutilizado, trocando apenas o parâmetro. Na segunda, a consulta usa `?` para o valor do `LIKE` (seguro contra *SQL Injection*) e percorre o `ResultSet` com `next()`; os três recursos são fechados automaticamente, na ordem inversa da abertura.

## 5.3 JPA e Hibernate (ORM)

**O que é**: **ORM** (*Object-Relational Mapping*) é a técnica de **mapear classes para tabelas** e gerar o SQL automaticamente. **JPA** (*Jakarta Persistence*, antes *Java Persistence API*) é a **especificação**; **Hibernate** é a implementação mais usada. O **`EntityManager`** gerencia o ciclo de vida das **entidades** dentro de um **contexto de persistência** (um cache de primeiro nível ligado à transação).

**Para que serve**: aumentar a **produtividade** (CRUD sem escrever SQL), manter o código orientado a objetos (trabalhar com `Pedido` e `Item`, não com linhas e colunas) e dar alguma **portabilidade** entre bancos.

**Como funciona**:

- **Entidade**: classe anotada `@Entity`, **não `final`**, com **construtor sem argumentos** (pode ser protegido) e um campo `@Id`. Por isso **records normalmente não servem como entidades**.
- **Mapeamento**: `@Table`, `@Id`, `@GeneratedValue(strategy = IDENTITY/SEQUENCE/UUID)` (`IDENTITY` delega ao auto-incremento do banco e impede *batch* de inserts), `@Column(nullable, unique, length)`, `@Enumerated(EnumType.STRING)`, `@Transient` (não persiste), `@Embedded`/`@Embeddable` (objeto-valor), `@Version`.
- **Relacionamentos**: `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`. O lado **dono** da relação (com a chave estrangeira) é o `@ManyToOne`; o lado inverso usa **`mappedBy`**. `cascade` propaga operações (`PERSIST`, `REMOVE`, `ALL`); `orphanRemoval = true` apaga filhos removidos da coleção. Em relações bidirecionais, mantenha os **dois lados sincronizados**.
- **Estados da entidade**: *transient* (nova, desconhecida do contexto), ***managed*** (gerenciada), *detached* (o contexto foi fechado) e *removed*. `persist` torna gerenciada uma entidade nova; `merge` copia o estado de uma *detached* para uma gerenciada; `find` busca por id.
- ***Dirty checking***: toda alteração em uma entidade **gerenciada** é detectada e gravada no **`flush`/commit**, **sem chamar `save`/`update`**.
- **Lazy x Eager**: `@OneToMany`/`@ManyToMany` são **lazy** por padrão (carregam ao acessar); `@ManyToOne`/`@OneToOne` são **eager**. Acessar uma associação lazy **fora da transação** lança `LazyInitializationException`. O **problema N+1** (1 consulta para a lista + N consultas, uma por item, para a associação) é resolvido com `JOIN FETCH`, `@EntityGraph` ou `@BatchSize`.
- **Consultas**: **JPQL** (orientada a objetos: `select p from Pedido p where p.total > :min`), SQL nativo, *Criteria API*. **Projeções/DTOs** evitam carregar a entidade inteira.
- **Herança**: estratégias `SINGLE_TABLE`, `JOINED`, `TABLE_PER_CLASS`.
- **Concorrência**: **otimista** (`@Version`: o commit falha se alguém alterou antes) e **pessimista** (`PESSIMISTIC_WRITE`: trava a linha).
- **Transações**: `@Transactional` abre o contexto; fora dele não há contexto gerenciado.
- **Esquema**: `hbm2ddl.auto=update/create` serve só para estudo; em produção use **migrações versionadas** (Flyway, Liquibase).
- **Cuidados com `equals`/`hashCode`** em entidades (baseie no id ou em uma chave de negócio estável) e com `@Data` do Lombok em entidades (gera `toString`/`equals` que percorrem relacionamentos).

**Quando não usar**: relatórios muito complexos e operações em massa, em que SQL direto (`JdbcTemplate`) é mais claro e rápido.

**Exemplo**:

```java
@Entity
public class Pedido {
    @Id @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    @Column(nullable = false) private String cliente;

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "pedido_id")
    private List<Item> itens = new ArrayList<>();

    protected Pedido() {}                        // exigido pelo JPA
    public Pedido(String cliente) { this.cliente = cliente; }
    public void renomear(String novo) { this.cliente = novo; }
}

@Transactional
void renomear(UUID id, String novo) {
    Pedido p = em.find(Pedido.class, id);        // entidade GERENCIADA
    p.renomear(novo);                            // sem save(): o dirty checking grava no commit
}

TypedQuery<Pedido> q = em.createQuery(
    "select p from Pedido p join fetch p.itens where p.cliente = :c", Pedido.class);
q.setParameter("c", "Ana");                      // evita o N+1 ao carregar os itens
```

**Explicação do exemplo**: a entidade tem construtor protegido sem argumentos (exigência do JPA) e a lista de itens com `cascade = ALL`: salvar o pedido salva também seus itens. No método `renomear`, a entidade lida dentro da transação é **gerenciada**; ao mudar o nome, o Hibernate percebe a diferença e executa o `UPDATE` no *commit*, sem nenhuma chamada de gravação. A consulta usa `join fetch` para trazer **pedido e itens em um único SQL**, evitando o problema N+1; o `:c` é um parâmetro nomeado (seguro contra *SQL Injection*).

## 5.4 Ferramentas de build, dependências e Lombok

**O que é**: ferramentas que **compilam, testam, empacotam** o projeto e **baixam as dependências** (bibliotecas de terceiros) automaticamente, de forma **reprodutível**. As duas principais em Java são **Maven** e **Gradle**.

**Para que serve**: não copiar `.jar` à mão, garantir que todo o time (e o servidor de integração contínua) compile da mesma forma, resolver as dependências **transitivas** (as que suas dependências precisam) e padronizar tarefas (`test`, `package`).

**Como funciona**:

**Estrutura padrão** (as duas ferramentas a seguem): `src/main/java` (código), `src/main/resources` (configurações), `src/test/java` (testes), `src/test/resources`.

**Maven**: configuração declarativa em **XML** (`pom.xml`).

- **Coordenadas**: `groupId` (organização), `artifactId` (projeto), `version`. Uma versão `-SNAPSHOT` é "em desenvolvimento".
- **Escopos** (`<scope>`): `compile` (padrão: todos os momentos), `provided` (necessária para compilar, fornecida pelo ambiente), `runtime` (só na execução, como o driver JDBC), `test` (só nos testes), `import` (para BOM).
- **Ciclo de vida** (fases em ordem): `validate → compile → test → package → verify → install → deploy`. Pedir uma fase executa todas as anteriores (`mvn clean package`). `clean` apaga `target/`. **Plugins** executam as tarefas (`maven-compiler-plugin`, `surefire` para testes unitários, `failsafe` para os de integração `*IT`, `spring-boot-maven-plugin` para o JAR executável).
- **`dependencyManagement`** e **BOM** (*Bill of Materials*): fixam as versões compatíveis de um conjunto de bibliotecas (Spring Boot, Spring Cloud), para você declarar a dependência **sem versão**. O Maven **Wrapper** (`mvnw`) fixa a versão do próprio Maven no projeto.

**Gradle**: script em **Groovy** (`build.gradle`) ou **Kotlin** (`build.gradle.kts`); mais flexível e geralmente mais rápido (build **incremental**, *cache*, *daemon*).

- **Configurações de dependência**: `implementation` (usada na compilação e na execução, **não exposta** a quem consome seu módulo), `api` (exposta; plugin `java-library`), `compileOnly`, `runtimeOnly`, `annotationProcessor`, `testImplementation`, `testRuntimeOnly`.
- **Toolchain**: `java { toolchain { languageVersion = JavaLanguageVersion.of(25) } }` escolhe a versão do JDK independentemente da máquina. Tarefas: `./gradlew build`, `test`, `bootRun`. O **Wrapper** (`gradlew`, `gradle-wrapper.properties`) fixa a versão do Gradle.

**Dependências transitivas e conflitos**: se duas bibliotecas pedem versões diferentes de uma terceira, a ferramenta escolhe uma (Maven: a mais próxima na árvore; Gradle: a mais nova), o que pode quebrar em execução. Diagnóstico: `mvn dependency:tree` / `gradle dependencies`.

**Empacotamento**: **JAR** (`java -jar`); *fat/uber JAR* (Spring Boot inclui todas as dependências); WAR (servidores de aplicação); **imagem de contêiner** (Dockerfile ou o plugin **Jib**, que gera a imagem em camadas sem Dockerfile).

**Lombok e MapStruct (processadores de anotação)**: geram código **durante a compilação** (tópico 3.16).

- **Lombok**: `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, `@Data` (conjunto dos anteriores), `@Value` (imutável), `@Builder` (com `@Builder.Default` para valores padrão), `@NoArgsConstructor`, `@AllArgsConstructor`, `@RequiredArgsConstructor` (campos `final`), `@Slf4j`. Em Gradle: `compileOnly` + `annotationProcessor`. Cuidados: `@Data` em **entidades JPA** (`equals`/`toString` percorrem relacionamentos); o IDE precisa do plugin; `record` já elimina parte da necessidade.
- **MapStruct**: gera, em compilação, os *mappers* DTO ↔ entidade (sem *reflection*; rápido). Com Lombok exige `lombok-mapstruct-binding` e a ordem correta dos processadores.

**Exemplo**:

```xml
<!-- Maven: pom.xml (trecho) -->
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>        <!-- versão vem do BOM do Boot -->
  </dependency>
  <dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>                                     <!-- só na execução -->
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

```groovy
// Gradle: build.gradle (trecho)
java { toolchain { languageVersion = JavaLanguageVersion.of(25) } }
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    runtimeOnly 'com.mysql:mysql-connector-j'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

```bash
./mvnw clean package     # compila, roda os testes e gera target/app.jar
./gradlew bootRun        # executa a aplicação Spring Boot
```

**Explicação do exemplo**: as duas configurações declaram o mesmo: o *starter* web (versão controlada pelo BOM do Spring Boot, por isso sem `<version>`), o driver do MySQL só para a execução (`runtime`/`runtimeOnly`) e o *starter* de testes apenas nos testes. No Gradle, o Lombok é `compileOnly` (não vai no pacote final) **e** `annotationProcessor` (para o `javac` executá-lo). O `clean package` do Maven percorre as fases até `package`, rodando os testes no caminho.

## 5.5 Testes automatizados

**O que é**: código que **verifica automaticamente** se o sistema se comporta como esperado, executado a cada mudança. Dá confiança para **refatorar** e funciona como **documentação executável** do comportamento.

**Para que serve**: detectar **regressões** cedo, permitir mudanças seguras, guiar o design (código fácil de testar costuma ser bem desacoplado) e sustentar a integração contínua.

**Como funciona**:

- **Tipos (pirâmide de testes)**: **unitários** (uma classe isolada, sem rede nem banco; rápidos e numerosos, a **base**), **de integração** (várias partes juntas, com banco real ou em memória; menos), **ponta a ponta/UI** (fluxo completo; poucos e lentos). Outros: de **contrato**, de **carga**, de **aceitação**.
- **JUnit 5**: composto por *Platform*, *Jupiter* (a API atual) e *Vintage* (testes JUnit 4). Cada método de teste roda em uma **nova instância** da classe (por padrão), logo um teste não vê o estado do outro.

| Anotação | Função |
| --- | --- |
| `@Test` | marca o teste |
| `@BeforeEach` / `@AfterEach` | rodam antes/depois de **cada** teste (montar o cenário) |
| `@BeforeAll` / `@AfterAll` | uma vez por classe (métodos `static`, salvo `@TestInstance(PER_CLASS)`) |
| `@DisplayName` | nome legível |
| `@Disabled` | desativa |
| `@Nested` | agrupa cenários em classes internas |
| `@ParameterizedTest` + `@ValueSource` / `@CsvSource` / `@MethodSource` / `@EnumSource` | o mesmo teste com vários dados |
| `@TempDir`, `@Tag`, `@ExtendWith` | pasta temporária, categorias, extensões |

- **Asserções**: `assertEquals(esperado, atual)`, `assertTrue`, `assertNotNull`, `assertAll`, **`assertThrows(Excecao.class, () -> ...)`**: devolve a exceção capturada (para verificar a **mensagem**, compare `getMessage()`); o **terceiro argumento é só a mensagem de falha** do teste, não valida a mensagem da exceção. **AssertJ** (`assertThat(x).isEqualTo(...).hasSize(2)`) é mais fluente e legível.
- **Estrutura de cada teste**: **Arrange / Act / Assert** (ou *Given / When / Then*): prepara, executa, verifica. Teste **um comportamento por vez**, com nomes que descrevem o cenário (`metodo_Resultado_Quando`). Princípios **F.I.R.S.T.**: rápidos (*fast*), independentes, repetíveis, auto-validáveis, oportunos.
- **Dublês de teste (*test doubles*)**: ***mock*** (verifica interações), ***stub*** (devolve respostas prontas), ***spy*** (objeto real parcialmente espionado), ***fake*** (implementação simples e funcional, como um repositório em memória). **Mockito**: `@Mock`, `@InjectMocks`, `when(x.metodo()).thenReturn(y)`, `doThrow`, `verify(m, times(1)).metodo()`, `any()`, `ArgumentCaptor`; com `MockitoExtension`, *stubs* não usados geram erro (`UnnecessaryStubbingException`). **Não mocke a classe sob teste**: isso testa o Mockito, não o seu código; mocke suas **dependências**.
- **Testes de integração**: `H2` em memória é rápido, mas o dialeto difere do banco real; **Testcontainers** sobe o banco de verdade em um contêiner. Convenção: classes `*IT` rodam no Failsafe.
- **Teste de contrato**: uma **classe de teste abstrata** com os comportamentos que **toda** implementação de uma interface deve cumprir; cada implementação a estende (Liskov na prática).
- **TDD** (*Test-Driven Development*): escreva o teste **antes** (vermelho), faça passar (verde), refatore. **Cobertura** (JaCoCo) mede quais linhas foram executadas, **não** se as asserções são boas; testes de **mutação** (PIT) medem isso.

**Exemplo**:

```java
@ExtendWith(MockitoExtension.class)
class PedidoServiceTest {
    @Mock PedidoRepository repo;                  // dublê da dependência
    @InjectMocks PedidoService service;           // classe sob teste, com o mock injetado

    @Test
    @DisplayName("Deve lançar exceção quando o pedido não existe")
    void buscar_Lanca_QuandoNaoExiste() {
        when(repo.findById(1L)).thenReturn(Optional.empty());          // Arrange

        var ex = assertThrows(NaoEncontradoException.class,            // Act
                () -> service.buscar(1L));

        assertEquals("Pedido 1 não encontrado", ex.getMessage());      // Assert
        verify(repo).findById(1L);                                     // a interação esperada ocorreu
    }

    @ParameterizedTest
    @ValueSource(ints = {18, 30, 65})
    void maioridade(int idade) { assertTrue(new Pessoa(idade).isMaior()); }
}
```

**Explicação do exemplo**: o `PedidoRepository` é substituído por um *mock* configurado para devolver `Optional.empty()`, de modo que o teste **não precisa de banco**. O `assertThrows` executa o código e exige que lance `NaoEncontradoException`; a exceção capturada permite conferir a mensagem. O `verify` garante que o serviço realmente consultou o repositório com o id esperado. O teste parametrizado executa o mesmo cenário três vezes, uma para cada idade da lista.

---

# Parte 6: Arquitetura e boas práticas

## 6.1 Princípios SOLID

**O que é**: **SOLID** é um conjunto de cinco princípios de projeto orientado a objetos (popularizados por Robert C. Martin) que ajudam a produzir código **coeso, flexível e fácil de manter e testar**. São **orientações**, não leis: servem para avaliar se o design está ficando rígido (mudar uma coisa quebra outras), frágil ou difícil de testar.

**Para que serve**: reduzir o **acoplamento** (quanto uma classe depende de outras) e aumentar a **coesão** (quanto o que está dentro de uma classe tem a ver entre si), para que mudanças de requisito atinjam poucos pontos do código.

**Como funciona**: cada princípio, com o sintoma que denuncia sua violação:

| Princípio | Ideia | Sintoma de violação | Caminho de correção |
| --- | --- | --- | --- |
| **S**: *Single Responsibility* | uma classe deve ter **um motivo para mudar** (atender a um único "ator"/responsabilidade) | classe "faz tudo": regra de negócio, SQL e e-mail juntos; qualquer mudança mexe nela | separar em classes coesas (`Service`, `Repository`, `Notificador`) |
| **O**: *Open/Closed* | **aberto para extensão, fechado para modificação** | cadeias de `if/else` ou `switch` por tipo, que crescem a cada novo caso | abstração + novas implementações (polimorfismo, *Strategy*) |
| **L**: *Liskov Substitution* | um **subtipo** deve poder ser usado **onde o tipo base é esperado, sem surpresas** | subclasse que lança `UnsupportedOperationException`, ignora ou enfraquece o contrato do pai; `instanceof` espalhado | reavaliar a herança; usar composição ou interfaces menores |
| **I**: *Interface Segregation* | **várias interfaces pequenas** são melhores que uma grande | implementadores com métodos vazios só para cumprir a interface | dividir a interface por papel |
| **D**: *Dependency Inversion* | módulos de **alto nível** dependem de **abstrações**, não de implementações concretas | `new RepositorioMySql()` dentro do serviço; impossível testar sem banco | depender de uma interface e **receber** a implementação (injeção) |

**Detalhando**:

- **SRP**: não significa "uma classe com um método"; significa que as **razões de mudança** vêm de uma só fonte. Uma classe `Relatorio` que calcula, formata e envia por e-mail muda se mudar a regra, o layout **ou** o servidor de e-mail.
- **OCP**: quando surge um novo tipo de desconto, você **adiciona uma classe** em vez de editar um `if` central e arriscar quebrar os demais.
- **LSP**: o exemplo clássico é `Quadrado extends Retangulo`: se `Retangulo` permite alterar largura e altura de forma independente, o `Quadrado` quebra essa expectativa. Regras: um subtipo **não pode exigir mais** (pré-condições) nem **prometer menos** (pós-condições) que o tipo base, e deve preservar suas invariantes.
- **ISP**: interfaces coesas e específicas (`Leitor`, `Gravador`) em vez de uma `Arquivo` gigante.
- **DIP** não é o mesmo que **injeção de dependência** (DI) nem que **IoC**: o DIP é o **princípio** (depender de abstrações); a DI é uma **técnica** para fornecer as dependências (pelo construtor); IoC é o **conceito** de o framework controlar a criação (tópico 7.1).

**Pontos importantes**: aplicar SOLID sem critério gera excesso de interfaces e classes (*over-engineering*). Use quando houver **variação real** ou necessidade de **testes isolados**; para um cálculo trivial que nunca muda, o código direto é melhor. SOLID também orienta a **arquitetura** (tópico 6.6).

**Exemplo (OCP + DIP)**:

```java
// ANTES (viola OCP): cada novo tipo de cliente exige editar este método
double descontoAntes(String tipo, double valor) {
    if (tipo.equals("VIP")) return valor * 0.20;
    if (tipo.equals("REGULAR")) return valor * 0.05;
    return 0;
}

// DEPOIS: abstração + implementações; o cálculo não conhece os tipos
interface RegraDesconto { double calcular(double valor); }
class DescontoVip     implements RegraDesconto { public double calcular(double v) { return v * 0.20; } }
class DescontoRegular implements RegraDesconto { public double calcular(double v) { return v * 0.05; } }

class Pedido {
    private final RegraDesconto regra;                    // DIP: depende da abstração
    Pedido(RegraDesconto regra) { this.regra = regra; }  // recebida por injeção
    double total(double valor) { return valor - regra.calcular(valor); }
}

System.out.println(new Pedido(new DescontoVip()).total(100));      // 80.0
System.out.println(new Pedido(new DescontoRegular()).total(100));  // 95.0
```

**Explicação do exemplo**: na versão "antes", incluir o tipo `PREMIUM` obriga a **modificar** o método, arriscando quebrar os existentes. Na versão "depois", basta **criar uma nova classe** `DescontoPremium` (extensão), sem tocar em `Pedido` (fechado para modificação). `Pedido` depende de `RegraDesconto` (abstração) e **recebe** a regra pelo construtor, o que também permite testá-lo com uma regra falsa. Com a regra VIP, 100 − 20 = 80.0; com a regular, 100 − 5 = 95.0.

## 6.2 Clean Code e boas práticas de código

**O que é**: um conjunto de práticas para escrever código **legível, simples e fácil de mudar** (termo popularizado pelo livro *Clean Code*, de Robert C. Martin). A premissa: o código é **lido muito mais vezes do que escrito**, então o custo real está em entendê-lo e modificá-lo.

**Para que serve**: reduzir bugs e o custo de manutenção, facilitar a entrada de novas pessoas no projeto e permitir evoluir o sistema com segurança.

**Como funciona**:

- **Nomes que revelam intenção**: classes são substantivos, métodos são verbos; sem abreviações obscuras; nomes pronunciáveis e pesquisáveis; a mesma palavra para o mesmo conceito em todo o código. Se precisa de um comentário para explicar o nome, o nome está ruim.
- **Funções pequenas**, que fazem **uma coisa** e em **um único nível de abstração**; **poucos parâmetros** (mais de três sugere agrupar em um objeto/record); sem ***flag arguments*** (`salvar(true)`: crie dois métodos).
- **Comentários**: explicam o **porquê** (decisão, restrição, armadilha), não o quê. Código claro dispensa comentário, e comentário desatualizado **mente**. Evite código comentado (o Git guarda o histórico).
- **DRY** (*Don't Repeat Yourself*): não repita **conhecimento**; cuidado com abstrações cedo demais (duplicação "acidental", em que dois trechos parecem iguais, mas mudam por razões diferentes). **KISS** (*Keep It Simple*). **YAGNI** (*You Aren't Gonna Need It*): não construa o que não é preciso agora.
- **Sem números ou textos "mágicos"**: use constantes nomeadas (`MARGEM_LUCRO = 0.20`, enums).
- **Guard clauses** (retorno antecipado) em vez de `if` aninhado; **imutabilidade** por padrão (campos `final`).
- **Lei de Demeter**: um objeto só fala com seus **vizinhos imediatos** (evite `a.getB().getC().getD()`). **Tell, Don't Ask**: peça ao objeto para **agir**, em vez de buscar seus dados para decidir fora dele.
- **Tratamento de erros**: exceções, não códigos de retorno; sem `catch` vazio; evite receber e devolver `null` (`Optional`, coleções vazias).
- **Convenções**: pacotes em minúsculas, `PascalCase` em classes, `camelCase` em métodos e variáveis, `UPPER_SNAKE` em constantes; formatação consistente (use formatadores automáticos).
- **Refatoração**: melhorar a estrutura **sem mudar o comportamento**, protegida por testes. **Cheiros de código** (*code smells*) que pedem refatoração: método longo, classe gigante, código duplicado, lista longa de parâmetros, *primitive obsession* (usar `String`/`int` no lugar de tipos de domínio), *feature envy* (método que usa mais dados de outra classe que da sua). ***Boy Scout Rule***: deixe o código um pouco mais limpo do que o encontrou.

**Pontos importantes**: clean code é **julgamento**, não regra mecânica; o objetivo é a clareza para quem lê. Revisões de código (*code review*), testes automatizados e análise estática (Checkstyle, SonarQube, SpotBugs) ajudam a manter o padrão.

**Exemplo**:

```java
// antes: aninhado, nomes ruins, número mágico
if (u != null) { if (u.getA() > 17) { if (u.isOk()) { liberar(u, 3); } } }

// depois: guard clauses, nomes que explicam, constante nomeada
private static final int LIMITE_TENTATIVAS = 3;

void liberarAcesso(Usuario usuario) {
    if (usuario == null) return;
    if (!usuario.isMaiorDeIdade()) return;
    if (!usuario.estaAtivo()) return;
    liberar(usuario, LIMITE_TENTATIVAS);
}
```

**Explicação do exemplo**: as três condições aninhadas viraram **cláusulas de guarda**: cada uma elimina um caso inválido e retorna, deixando o "caminho feliz" no fim, sem recuo. Os nomes (`isMaiorDeIdade`, `estaAtivo`) dizem **o que** é testado, e o `3` ganhou nome. O comportamento é o mesmo, mas o código se lê quase como uma frase.

## 6.3 Padrões de projeto criacionais

**O que é**: **padrões de projeto** (*design patterns*, catalogados pelo livro do GoF, "Gang of Four") são **soluções nomeadas e reaproveitáveis** para problemas recorrentes de design, descrevendo **intenção, estrutura e consequências**; não são bibliotecas nem código pronto. Dividem-se em **criacionais** (como **criar** objetos), **estruturais** (como **compor** classes e objetos) e **comportamentais** (como objetos **se comunicam**). Este tópico trata dos criacionais.

**Para que serve**: dar um vocabulário comum à equipe ("use um *Strategy* aqui"), reaproveitar soluções comprovadas e **desacoplar** o código que usa um objeto do código que o cria.

### Singleton

**O que é**: garante **uma única instância** da classe e um ponto global de acesso: construtor `private`, atributo estático e `getInstance()`.

| Variante | Como | Observações |
| --- | --- | --- |
| **Eager** | cria no carregamento da classe | *thread-safe* e simples; cria mesmo sem uso |
| **Lazy simples** | cria no primeiro acesso | **não é *thread-safe***: duas threads podem criar duas instâncias |
| **Double-checked locking** | `if (null) synchronized { if (null) ... }` | exige o campo **`volatile`** e construtor `private` |
| **Holder (Bill Pugh)** | classe interna estática guarda a instância | *lazy* e *thread-safe* sem `synchronized`, graças à inicialização de classe da JVM |
| **Enum** | `enum X { INSTANCE; }` | à prova de **reflexão** e **serialização**; recomendação do *Effective Java* |

**Para que serve**: recurso compartilhado que deve existir uma só vez (configuração, *pool*, *registry*). **Crítica**: é **estado global** disfarçado: dificulta os testes e esconde dependências. No **Spring**, os *beans* têm escopo *singleton* **gerenciado pelo container** (uma instância por contexto), o que dá o benefício **sem** o antipadrão (e permite substituir por *mocks*).

```java
class Config {
    private Config() {}                                        // ninguém cria de fora
    private static class Holder { static final Config INSTANCE = new Config(); }
    static Config getInstance() { return Holder.INSTANCE; }    // lazy e thread-safe
}
System.out.println(Config.getInstance() == Config.getInstance());   // true
```

**Explicação**: a classe `Holder` só é carregada na **primeira chamada** a `getInstance()` (preguiça), e a JVM garante que o carregamento de uma classe é *thread-safe*; por isso não há `synchronized`. As duas chamadas devolvem o **mesmo** objeto.

### Builder

**O que é**: separa a **construção** de um objeto complexo da sua representação, montando-o **passo a passo** por uma interface **fluente** (cada método devolve `this`) e terminando com `build()`.

**Para que serve**: resolver o ***telescoping constructor*** (construtores com muitos parâmetros, em várias combinações) e o risco do estilo *JavaBeans* (criar vazio e chamar setters, deixando o objeto **parcialmente construído** e mutável). O `build()` é o lugar de **validar as invariantes** e entregar um objeto **imutável**. Implementações: **manual**, **Lombok** (`@Builder`) ou **gerada por annotation processor**.

```java
public class Pessoa {
    private final String nome;
    private final String email;
    private final int idade;

    private Pessoa(Builder b) { nome = b.nome; email = b.email; idade = b.idade; }
    public static Builder builder() { return new Builder(); }

    public static class Builder {
        private String nome; private String email; private int idade;
        public Builder nome(String n)  { this.nome = n; return this; }
        public Builder email(String e) { this.email = e; return this; }
        public Builder idade(int i)    { this.idade = i; return this; }
        public Pessoa build() {
            if (nome == null) throw new IllegalStateException("nome obrigatório");
            return new Pessoa(this);                           // objeto final, imutável
        }
    }
}

Pessoa p = Pessoa.builder().nome("Ana").email("ana@x.com").idade(30).build();
```

**Explicação**: cada chamada configura um campo do *builder* e devolve o próprio *builder*, o que permite o encadeamento legível; os argumentos são **nomeados** pelos métodos (sem ordem de parâmetros para decorar). `build()` valida (sem nome, lança exceção) e cria a `Pessoa` com campos `final`.

### Factory (Simple Factory / Factory Method)

**O que é**: encapsula **qual classe concreta instanciar**; o cliente conhece só a **interface**. Variantes: a **fábrica estática/Simple Factory** (um método estático que escolhe a implementação, como no exemplo) e o **Factory Method** do GoF (um método na classe-base que as **subclasses sobrescrevem** para decidir o tipo criado). Há ainda o **Abstract Factory** (cria **famílias** de objetos relacionados) e o **Prototype** (cria por **clonagem** de um modelo).

**Para que serve**: isolar a lógica de criação (que implementação usar, com que configuração), permitindo trocar ou acrescentar tipos sem alterar o código que consome. Com `switch` **exaustivo** sobre um `enum`, o compilador obriga a tratar um novo valor (sem `default`, que esconderia o esquecimento).

```java
interface Moeda { String simbolo(); }
class Real  implements Moeda { public String simbolo() { return "R$"; } }
class Dolar implements Moeda { public String simbolo() { return "$"; } }
enum Pais { BRASIL, EUA }

static Moeda criar(Pais pais) {
    return switch (pais) {
        case BRASIL -> new Real();
        case EUA    -> new Dolar();
    };                                         // sem default: o compilador confere a cobertura
}
System.out.println(criar(Pais.BRASIL).simbolo());   // R$
```

**Explicação**: quem chama `criar` recebe uma `Moeda` e **não conhece** `Real` nem `Dolar`. Se um dia entrar `Pais.JAPAO`, este `switch` **deixa de compilar** até que o novo caso seja tratado, apontando o ponto exato a corrigir.

## 6.4 Padrões de projeto estruturais

**O que é**: padrões que tratam de **como compor classes e objetos** em estruturas maiores e mais flexíveis, sem acoplá-los demais.

**Para que serve**: integrar código existente, simplificar APIs complexas, acrescentar comportamento a objetos sem herança e controlar o acesso a eles.

**Como funciona**:

- **Facade (fachada)**: oferece uma **interface simples** para um subsistema complexo. O cliente chama **uma** operação, e a fachada orquestra as classes internas. Reduz o acoplamento do cliente com os detalhes; o risco é a fachada virar uma "classe deus".
- **Adapter (adaptador)**: **converte** a interface de uma classe na que o cliente espera, permitindo que classes incompatíveis trabalhem juntas. É a base da **arquitetura hexagonal**: os adaptadores (JPA, REST, mensageria) implementam as **portas** definidas pelo domínio (tópico 6.6). Também é a forma de integrar bibliotecas de terceiros sem espalhá-las pelo código.
- **Decorator (decorador)**: **acrescenta comportamento** a um objeto **embrulhando-o** em outro do **mesmo tipo**, sem herança e em tempo de execução. Os fluxos de I/O são o exemplo clássico (`new BufferedReader(new FileReader(f))`: o `BufferedReader` decora o `FileReader`).
- **Proxy**: um **representante** que **controla o acesso** ao objeto real (carregamento tardio, segurança, *cache*, transações). O Spring usa proxies para `@Transactional`, `@Cacheable` e `@Async`; o Feign gera um proxy para cada cliente HTTP. *Proxy* mantém a **mesma interface** do objeto real; o *Adapter* **muda** a interface.
- **Composite**: trata **objetos individuais e grupos** de forma uniforme (pastas e arquivos; um menu e seus itens).
- **DTO (*Data Transfer Object*)**: não é um padrão do GoF (vem do catálogo de padrões corporativos): um objeto **só de dados** que transporta informação entre camadas ou serviços, **evitando expor a entidade de domínio** e reduzindo chamadas. Hoje costuma ser um `record`.

**Pontos importantes**: *Facade* **simplifica** uma API; *Adapter* **traduz** uma interface; *Decorator* **acrescenta** comportamento mantendo a interface; *Proxy* **controla** o acesso mantendo a interface. A semelhança estrutural entre os três últimos (um objeto que envolve outro) é o que mais confunde: o que os diferencia é a **intenção**.

**Exemplo (Facade)**:

```java
class MigracaoFacade {
    private final CepApi cepApi;                 // subsistema 1
    private final CrmService crm;                // subsistema 2
    MigracaoFacade(CepApi cepApi, CrmService crm) { this.cepApi = cepApi; this.crm = crm; }

    void migrarCliente(String nome, String cep) {
        String cidade = cepApi.cidade(cep);
        String estado = cepApi.estado(cep);
        crm.gravar(nome, cep, cidade, estado);   // o cliente não conhece esses passos
    }
}
new MigracaoFacade(cepApi, crm).migrarCliente("Venilton", "14801788");
```

**Explicação do exemplo**: quem usa a fachada chama **um único método** com dois dados e não precisa saber que, por trás, é preciso consultar a cidade e o estado em uma API de CEP e depois gravar tudo no CRM. Se o CRM mudar, só a fachada é alterada. As dependências entram pelo construtor, o que facilita testá-la com *mocks*.

## 6.5 Padrões de projeto comportamentais

**O que é**: padrões que tratam de **como os objetos se comunicam e dividem responsabilidades** e de como **encapsular comportamento**.

**Para que serve**: desacoplar **quem pede** de **quem executa**, trocar algoritmos sem alterar o cliente, reagir a eventos e organizar fluxos de decisão sem montanhas de `if/else`.

**Como funciona**:

- **Strategy**: define uma **família de algoritmos** intercambiáveis, encapsula cada um e os deixa **trocáveis em tempo de execução**. O contexto guarda uma estratégia e **delega** a ela. Elimina `if/else` por tipo e respeita Aberto/Fechado. Em Java moderno, a estratégia pode ser uma **lambda** (`Comparator`, `Predicate`).
- **Template Method**: a classe-base (abstrata) define o **esqueleto** do algoritmo (um método `final`) e deixa **passos** para as subclasses (tópico 2.9). *Strategy* usa **composição** (troca o algoritmo inteiro em execução); *Template Method* usa **herança** (fixa a estrutura, varia os passos em compilação).
- **Observer**: objetos (**observadores**) **assinam** um assunto e são notificados quando ele muda; desacopla quem gera o evento de quem reage. Aparece em *listeners* de GUI, `ApplicationEventPublisher` + `@EventListener` do Spring e `RowSetListener`. (As classes `Observable`/`Observer` do JDK estão **depreciadas** desde o Java 9.)
- **Command**: encapsula uma **requisição como objeto** (permite fila, histórico, desfazer/refazer, execução adiada). `Runnable`/`Callable` são comandos simples.
- **Iterator**: percorre uma coleção **sem expor sua estrutura interna** (`Iterator`, *for-each*).
- **State**: o objeto **muda de comportamento** conforme seu estado interno (máquina de estados: pedido `ABERTO → PAGO → ENVIADO`). Estruturalmente parecido com *Strategy*, mas as "estratégias" (estados) **trocam entre si** sozinhas.
- **Chain of Responsibility**: a requisição percorre uma **cadeia de tratadores**, e cada um decide tratar ou repassar (filtros de servlet, a cadeia de filtros do Spring Security).

**Exemplo (Strategy)**:

```java
interface Comportamento { void mover(); }
class Defensivo implements Comportamento { public void mover() { System.out.println("Movendo defensivamente"); } }
class Agressivo implements Comportamento { public void mover() { System.out.println("Movendo agressivamente"); } }

class Robo {
    private Comportamento estrategia;                       // composição
    void setEstrategia(Comportamento c) { this.estrategia = c; }
    void mover() { estrategia.mover(); }                    // delega à estratégia atual
}

Robo r = new Robo();
r.setEstrategia(new Defensivo());
r.mover();                                                  // Movendo defensivamente
r.setEstrategia(new Agressivo());                           // troca em tempo de execução
r.mover();                                                  // Movendo agressivamente
```

**Explicação do exemplo**: o `Robo` não sabe **como** se mover: delega à `Comportamento` atual. Trocar a estratégia muda o comportamento **sem alterar `Robo`** nem usar `if`. Para acrescentar um modo "furtivo", cria-se uma nova classe que implementa `Comportamento`. Como a interface tem um único método, uma lambda (`() -> System.out.println("...")`) também serviria.

## 6.6 Arquitetura de software

**O que é**: a organização das **camadas, responsabilidades e dependências** do sistema. Uma boa arquitetura mantém as **regras de negócio** protegidas das decisões de tecnologia (framework, banco, web) e facilita testar e evoluir.

**Para que serve**: permitir que o sistema **cresça** sem virar uma massa emaranhada, trocar tecnologias (de banco, de framework web) com baixo custo e testar as regras de negócio isoladamente.

**Como funciona**:

- **Arquitetura em camadas**: *controller* (HTTP) → *service* (regras) → *repository* (dados) → banco. Cada camada fala só com a de baixo. É simples e comum; o risco é o **domínio ficar acoplado à persistência** (entidades JPA invadindo todas as camadas).
- **Hexagonal (*Ports and Adapters*) e Clean Architecture**: o **domínio fica no centro**, sem depender de framework, banco ou web. Ele declara **portas** (interfaces, como `TaskRepository`), e a **infraestrutura** fornece **adaptadores** (JPA, Mongo, REST, mensageria). A **regra da dependência**: o código-fonte só pode apontar **para dentro** (a infraestrutura conhece o domínio; o domínio não conhece a infraestrutura). Camadas usuais: `domain` (entidades, *value objects*, regras), `application` (**casos de uso**, orquestram o domínio) e `infrastructure` (controllers, repositórios, clientes externos). Isso é o **DIP** (6.1) aplicado à arquitetura.
- **Casos de uso**: uma classe por ação do sistema (`CreateTaskUseCase`), com um método `execute`.
- **Dois pares de DTOs**: *Request/Response* (borda HTTP: JSON, validação) e *Input/Output* (borda da aplicação). Mudar o JSON não afeta os casos de uso.
- **DDD (*Domain-Driven Design*)**: modelar o software a partir do **domínio** do negócio, com uma **linguagem ubíqua** compartilhada com os especialistas. Blocos: **entidade** (tem identidade), ***value object*** (imutável, igualdade por valor), **agregado** (conjunto consistente acessado pela **raiz**), **repositório**, **serviço de domínio**, **evento de domínio** e **contextos delimitados** (*bounded contexts*), que se comunicam por eventos ou contratos.
- **Camada anticorrupção**: converte modelos externos para o do domínio, isolando-o de mudanças de terceiros.
- **Monólito x microsserviços**: microsserviços separam contextos em serviços **independentes** (deploy e escala isolados) ao custo de **complexidade distribuída** (rede, consistência eventual, observabilidade, testes). Comece com um **monólito modular** bem dividido; separe quando houver motivo.
- **Resiliência** (tópico 7.12): *timeout*, *retry* com *backoff*, **circuit breaker**, *bulkhead*, *fallback*; escolha entre **fail-open** e **fail-closed** conforme o risco do negócio.

**Pontos importantes**: arquitetura é **trade-off**; para um CRUD simples, camadas simples bastam. Camadas extras só se justificam quando há regra de negócio relevante ou necessidade de trocar tecnologia.

**Exemplo (porta e adaptador)**:

```java
// domain: a PORTA (sem nenhuma tecnologia)
interface TaskRepository {
    Task save(Task task);
    Optional<Task> findById(TaskId id);
}

// application: o caso de uso depende só da porta
class CreateTaskUseCase {
    private final TaskRepository repo;
    CreateTaskUseCase(TaskRepository repo) { this.repo = repo; }
    Task execute(String titulo) { return repo.save(new Task(titulo)); }
}

// infrastructure: um ADAPTADOR (pode ser trocado por JPA, sem tocar nas camadas internas)
class InMemoryTaskRepository implements TaskRepository {
    private final Map<TaskId, Task> dados = new HashMap<>();
    public Task save(Task t) { dados.put(t.getId(), t); return t; }
    public Optional<Task> findById(TaskId id) { return Optional.ofNullable(dados.get(id)); }
}
```

**Explicação do exemplo**: o caso de uso conhece **apenas** a interface `TaskRepository`, que pertence ao domínio. A implementação em memória fica na infraestrutura, e nada no domínio importa classes de Spring ou JPA. Para usar um banco real, cria-se `JpaTaskRepository implements TaskRepository`, **sem alterar** o caso de uso. Nos testes, usa-se a versão em memória, rápida e sem banco.

## 6.7 Princípios de design de APIs REST

**O que é**: **REST** (*Representational State Transfer*) é um **estilo arquitetural** para APIs sobre HTTP, centrado em **recursos** identificados por URLs (substantivos) e manipulados pelos **verbos** HTTP. Uma API "RESTful" segue essas convenções, que a tornam previsível.

**Para que serve**: criar APIs fáceis de entender, consumir e evoluir por qualquer cliente (front-end, mobile, outros serviços).

**Como funciona**:

- **Recursos e verbos**: `GET /tasks` (listar), `GET /tasks/{id}` (consultar), `POST /tasks` (criar), `PUT /tasks/{id}` (**substituir** o recurso inteiro), `PATCH /tasks/{id}` (alterar **parcialmente**), `DELETE /tasks/{id}`. URLs no plural, **sem verbos** (`/tasks`, não `/criarTask`).
- **Segurança e idempotência**: `GET` é **seguro** (não altera estado). **Idempotente** = repetir a chamada tem o mesmo efeito de uma só: `GET`, `PUT` e `DELETE` são idempotentes; `POST` **não** é; `PATCH` depende da operação. Isso orienta o *retry* seguro (e a ideia de uma *chave de idempotência* em `POST` de pagamentos).
- **Códigos de status**: `200 OK`; `201 Created` (com cabeçalho `Location` do novo recurso); `204 No Content` (sucesso sem corpo, típico de `DELETE`/`PUT`); `400 Bad Request` (requisição inválida/validação); `401 Unauthorized` (**não autenticado**); `403 Forbidden` (autenticado **sem permissão**); `404 Not Found`; `409 Conflict` (conflito de estado/duplicidade); `422 Unprocessable Entity` (semântica inválida); `429 Too Many Requests`; `500 Internal Server Error`; `503 Service Unavailable`.
- **Sem estado (*stateless*)**: cada requisição carrega tudo o necessário (por exemplo, um *token*); o servidor não guarda sessão do cliente, o que facilita escalar horizontalmente.
- **Representações**: normalmente JSON; negociação de conteúdo por `Accept`/`Content-Type`; datas em ISO-8601 (`2026-10-24T12:00:00Z`).
- **Paginação, ordenação e filtros** por parâmetros (`?page=0&size=20&sort=name,desc&status=ABERTO`). **Versionamento** (`/v1/...` ou por cabeçalho) para evoluir sem quebrar clientes.
- **Erros padronizados**: o formato ***Problem Details*** (RFC 9457) define campos como `type`, `title`, `status`, `detail`. **HATEOAS** inclui links nas respostas para guiar o cliente (nível mais alto do modelo de maturidade de Richardson).
- **Documentação**: **OpenAPI/Swagger** (gerada por *springdoc*) descreve a API e gera clientes.
- **Segurança**: HTTPS sempre; autenticação (sessão, JWT, OAuth2), **autorização por papel**; **validar toda entrada**; não expor entidades internas nem mensagens de erro com detalhes sensíveis; CORS configurado com critério.

**Exemplo**:

```
POST   /tasks            {"title":"Estudar"}  →  201 Created
                                                 Location: /tasks/42
GET    /tasks/42                               →  200 OK      {"id":42,"title":"Estudar","status":"PENDING"}
PATCH  /tasks/42         {"status":"DONE"}     →  200 OK
DELETE /tasks/42                               →  204 No Content
GET    /tasks/999                              →  404 Not Found
POST   /tasks            {"title":""}          →  400 Bad Request (título obrigatório)
GET    /tasks?page=0&size=10&sort=title,asc    →  200 OK      (primeira página)
```

**Explicação do exemplo**: criar devolve **201** e o endereço do novo recurso em `Location`. `PATCH` altera só o campo enviado. `DELETE` bem-sucedido devolve **204** (nada a mostrar). Buscar um id inexistente devolve **404**, e enviar um corpo inválido devolve **400**. A última linha mostra a paginação por parâmetros de consulta, sem mudar a URL do recurso.

---

# Parte 7: Ecossistema Spring

> **Sobre versões**: o Spring Boot 3 exige **Java 17+**, usa o **Spring Framework 6** e o pacote **`jakarta.*`** (no lugar de `javax.*`). O Spring Boot 4 usa o **Spring Framework 7** e o **Jackson 3** (pacote `tools.jackson`) como padrão. Alguns nomes de propriedades, anotações e pacotes mudaram entre as versões: este guia indica as diferenças principais, mas confirme sempre nas notas de migração da versão que você usa.

## 7.1 Spring Framework: IoC e Injeção de Dependência

**O que é**: o **Spring** é um framework que **gerencia os objetos** da aplicação. Em vez de uma classe criar suas dependências com `new`, o **container IoC** (o `ApplicationContext`) **instancia** os objetos (chamados **beans**), **resolve** as dependências entre eles e as **injeta** onde são necessárias. **Inversão de Controle (IoC)** é o conceito: o controle sobre a criação e o ciclo de vida dos objetos sai do seu código e vai para o framework. **Injeção de Dependência (DI)** é a técnica de receber as dependências de fora.

**Para que serve**: **baixo acoplamento** (a classe depende de uma interface, não de quem a implementa), **troca de implementações por configuração**, **testes** com *mocks* (basta passar um dublê no construtor) e ciclo de vida gerenciado (criação, destruição, escopo).

**Como funciona**:

- **Como o Spring descobre os beans**: classes marcadas com **estereótipos** são registradas durante o *component scan*, que varre o pacote da classe principal **e seus subpacotes**:
  - `@Component` (genérico), `@Service` (regra de negócio/caso de uso), `@Repository` (acesso a dados; traduz exceções de persistência), `@Controller` / `@RestController` (web).
  - `@Configuration` com métodos `@Bean`: **você** cria o objeto e o entrega ao container (necessário para classes de terceiros, como `PasswordEncoder` ou `DataSource`).
- **Formas de injeção**: **por construtor** (**recomendada**: dependências explícitas e obrigatórias, campos `final`, fácil de testar sem Spring; com **um único construtor** nem precisa de `@Autowired`), por *setter* (dependência opcional) e por campo (**evite**: esconde dependências e dificulta testes).
- **Desambiguar** quando há mais de uma implementação do mesmo tipo: `@Primary` (a preferida), `@Qualifier("nome")` (escolher uma), `@Bean(defaultCandidate = false)` (só entra se pedido explicitamente). Injetar **todas** as implementações: `List<Strategy>` ou `Map<String, Strategy>` (útil para *Strategy* + *Factory*).
- **Escopos**: **singleton** (padrão: **uma** instância por contexto, o que exige que beans sejam *stateless* ou *thread-safe*), `prototype` (uma nova a cada pedido), `request` e `session` (web).
- **Ciclo de vida**: instanciar → injetar dependências → `@PostConstruct` → uso → `@PreDestroy`. `@Lazy` adia a criação.
- **Perfis**: `@Profile("dev")` ativa beans conforme o ambiente (`spring.profiles.active=dev`).
- **Dependência circular** (A precisa de B e B precisa de A) é um erro de design: o Boot 2.6+ a proíbe por padrão. Resolva extraindo uma terceira classe.

**Pontos importantes**: o Spring **não é mágica**: usa *reflection* e *proxies* (tópico 7.4). "Singleton" no Spring é **um bean por contexto**, não o padrão GoF (tópico 6.3). Prefira a injeção por construtor mesmo que o Lombok (`@RequiredArgsConstructor`) a gere para você.

**Exemplo**:

```java
interface Notificador { void enviar(String msg); }

@Component("email")
class EmailNotificador implements Notificador {
    public void enviar(String msg) { System.out.println("E-mail: " + msg); }
}
@Component("sms")
class SmsNotificador implements Notificador {
    public void enviar(String msg) { System.out.println("SMS: " + msg); }
}

@Service
public class PedidoService {
    private final PedidoRepository repo;            // dependência obrigatória
    private final Notificador notificador;

    public PedidoService(PedidoRepository repo, @Qualifier("email") Notificador notificador) {
        this.repo = repo;                            // injetados pelo container
        this.notificador = notificador;
    }
    public void concluir(Long id) { /* ... */ notificador.enviar("Pedido " + id + " concluído"); }
}

@Configuration
class AppConfig {
    @Bean PasswordEncoder passwordEncoder() { return new BCryptPasswordEncoder(); }
}
```

**Explicação do exemplo**: ninguém escreve `new PedidoService(...)`: ao iniciar, o Spring detecta as classes anotadas, cria os beans e percebe que `PedidoService` precisa de um `PedidoRepository` e de um `Notificador`. Como há **duas** implementações de `Notificador`, o `@Qualifier("email")` diz qual usar; trocar para `sms` é alterar uma palavra, sem mexer na lógica. Em um teste, basta criar `new PedidoService(repoFalso, notificadorFalso)`. O método `@Bean` ensina o container a fabricar um `PasswordEncoder`, classe que o Spring não conhece.

## 7.2 Configuração externa e propriedades

**O que é**: separar a **configuração** (URLs, credenciais, portas, parâmetros de negócio) do **código**, para que o **mesmo artefato** rode em ambientes diferentes (desenvolvimento, teste, produção) sem recompilar.

**Para que serve**: trocar de banco, porta, taxa ou nível de log apenas mudando a configuração; manter **segredos fora do repositório**; ter um valor por ambiente.

**Como funciona**:

- **Fontes** (das menos para as mais prioritárias, de forma simplificada): `application.properties`/`application.yml` → arquivos por perfil (`application-dev.yml`) → **variáveis de ambiente** → argumentos de linha de comando. A fonte mais prioritária **vence**.
- **Formatos**: `.properties` (`chave.sub=valor`) ou `.yml` (hierárquico). Em variáveis de ambiente, `spring.datasource.url` vira `SPRING_DATASOURCE_URL`.
- **Injetar valores**:
  - `@Value("${chave:padrao}")`: um valor simples, com padrão opcional.
  - **`@ConfigurationProperties("prefixo")`**: liga um **grupo** de propriedades a um objeto **tipado** (com validação e conversão automática). Ideal para vários `DataSource`s (`registration.datasource.*`, `catalog.datasource.*`).
- **Perfis**: `spring.profiles.active=dev` ativa `application-dev.yml` e os `@Profile("dev")`.
- **Segredos** (senhas, chaves): **nunca** no repositório; use variáveis de ambiente, *Secrets* do Kubernetes ou cofres (Vault, AWS Secrets Manager).
- **Propriedades comuns**: `server.port`, `spring.application.name`, `spring.datasource.url|username|password`, `spring.jpa.hibernate.ddl-auto`, `logging.level.pacote=DEBUG`, `spring.threads.virtual.enabled=true`.

**Exemplo**:

```properties
# application.properties
spring.application.name=loja
server.port=8081
app.taxa=0.15
app.cache-ttl=PT10M
```

```java
@ConfigurationProperties("app")
public record AppProps(double taxa, Duration cacheTtl) {}   // app.cache-ttl -> cacheTtl (PT10M = 10 min)

// ou, para um valor isolado:
@Value("${app.taxa:0.10}") double taxa;                      // 0.10 se a propriedade não existir
```

**Explicação do exemplo**: o Spring lê `app.taxa` e `app.cache-ttl` e preenche o record `AppProps`, convertendo o texto `PT10M` em `Duration` e usando a convenção *kebab-case* (`cache-ttl`) → *camelCase* (`cacheTtl`). Em produção, definir a variável de ambiente `APP_TAXA=0.20` **sobrepõe** o valor do arquivo sem recompilar. Para o `@ConfigurationProperties` ser registrado, é preciso habilitá-lo (`@EnableConfigurationProperties` ou `@ConfigurationPropertiesScan`).

## 7.3 Spring Boot

**O que é**: camada sobre o Spring Framework que **elimina a configuração manual** seguindo **"convenção sobre configuração"**: você declara o que quer (dependências) e o Boot configura o resto com padrões sensatos, que você pode sobrescrever.

**Para que serve**: criar rapidamente aplicações **prontas para produção** (servidor embutido, métricas, saúde, configuração externa), sem arquivos XML e sem montar o servidor de aplicação manualmente.

**Como funciona**:

- **Starters**: dependências agrupadas por finalidade; um só *starter* traz as bibliotecas compatíveis entre si: `spring-boot-starter-web`, `-data-jpa`, `-security`, `-validation`, `-actuator`, `-test`, `-data-redis`, `-data-mongodb`. As **versões** vêm do BOM do Boot (por isso não se declaram).
- **Auto-configuração**: o Boot examina o ***classpath*** e as **propriedades** e cria os *beans* necessários. Se há `mysql-connector-j` e `spring.datasource.url`, ele monta o `DataSource`. Se você declarar o seu próprio *bean*, o do Boot **recua**. Para ver o que foi (ou não) configurado e por quê: `--debug` (relatório de condições).
- **`@SpringBootApplication`** = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` (varre a partir do pacote da classe principal).
- **Servidor embutido** (Tomcat por padrão): a aplicação é um **JAR executável** (*fat JAR*): `java -jar app.jar`. Não há WAR/servidor externo, salvo se necessário.
- **Ferramentas**: **DevTools** (reinício automático em desenvolvimento), **Actuator** (operação, tópico 7.14), **Docker Compose support** (`spring-boot-docker-compose` sobe os contêineres do `compose.yml` ao iniciar em desenvolvimento).
- **Versões e migração**: do Boot 2 para o 3: `javax.*` → **`jakarta.*`**, Java 17+, `WebSecurityConfigurerAdapter` → bean `SecurityFilterChain`, `antMatchers` → `requestMatchers`, `@EnableGlobalMethodSecurity` → `@EnableMethodSecurity`, `springdoc-openapi-ui` → `springdoc-openapi-starter-webmvc-ui`, Hibernate 6. O Boot 4 traz Spring Framework 7 e Jackson 3.

**Exemplo**:

```java
@SpringBootApplication
public class LojaApplication {
    public static void main(String[] args) {
        SpringApplication.run(LojaApplication.class, args);
    }
}
```

**Explicação do exemplo**: essa classe mínima basta para subir uma aplicação completa. `SpringApplication.run` cria o contexto, aplica a **auto-configuração** conforme as dependências do projeto (por exemplo, com `starter-web` ele sobe um Tomcat na porta 8080) e executa o *component scan* a partir do pacote desta classe. Por isso as demais classes devem ficar no **mesmo pacote ou abaixo** dele.

## 7.4 AOP (Programação Orientada a Aspectos)

**O que é**: técnica para separar as **preocupações transversais** (*cross-cutting concerns*: transação, segurança, log, cache, métricas), que atravessam várias classes, da **lógica de negócio**. Termos: **aspecto** (o módulo da preocupação), ***advice*** (o código a executar: `@Before`, `@After`, `@AfterReturning`, `@AfterThrowing`, `@Around`), ***pointcut*** (a expressão que diz **onde** aplicar) e ***join point*** (um ponto de execução, no Spring sempre uma **chamada de método**).

**Para que serve**: reaproveitar comportamento transversal **sem repeti-lo** em cada método (nem sujar a regra de negócio). É a base de `@Transactional`, `@Cacheable`, `@Async`, `@PreAuthorize` e `@Retryable`.

**Como funciona**:

- O Spring implementa AOP com **proxies**: em vez de entregar o bean real, o container entrega um **objeto intermediário** (proxy dinâmico do JDK, para interfaces, ou **CGLIB**, por subclasse) que, ao receber a chamada, executa o *advice* e então delega ao método real.
- **`@Around`** é o mais poderoso: recebe um `ProceedingJoinPoint`, pode executar código **antes e depois** e decide se chama `proceed()` (o método real) e com quais argumentos.
- **Pegadinha da autoinvocação**: a anotação só funciona em chamadas **vindas de fora** do bean. Se um método chama outro **da mesma classe** (`this.metodo()`), o proxy é **ignorado** e o `@Transactional`/`@Cacheable` do segundo **não tem efeito**. Solução: mover o método para outro bean.
- Restrições comuns: o método precisa ser `public` (para a maioria dos casos) e a classe não pode ser `final`; o AOP do Spring só intercepta **beans do container** (para algo mais amplo existe o AspectJ completo).

**Pontos importantes**: raramente você escreve aspectos próprios, mas **entender o proxy explica** vários "bugs" (transação que não abre, cache que não funciona). Uma aplicação disso: auditoria, medição de tempo, controle de acesso transversal.

**Exemplo**:

```java
@Aspect
@Component
class TempoAspect {
    @Around("execution(* br.com.loja.service..*(..))")     // todo método dos serviços
    Object medir(ProceedingJoinPoint pjp) throws Throwable {
        long t0 = System.nanoTime();
        try {
            return pjp.proceed();                           // executa o método real
        } finally {
            long ms = (System.nanoTime() - t0) / 1_000_000;
            System.out.println(pjp.getSignature() + " levou " + ms + " ms");
        }
    }
}
```

**Explicação do exemplo**: a expressão do *pointcut* seleciona qualquer método de qualquer classe do pacote `service` e subpacotes. Ao chamar um deles, o proxy executa `medir`: guarda o tempo, chama `pjp.proceed()` (o método de negócio) e, no `finally`, imprime a duração, mesmo que o método lance exceção. Nenhum serviço precisou ser alterado: a medição é aplicada de fora.

## 7.5 Spring MVC e APIs REST

**O que é**: o módulo web do Spring, baseado em **servlets**. Uma requisição HTTP chega ao **`DispatcherServlet`**, que descobre qual **método de controller** (o *handler*) deve atendê-la, **converte** a entrada (corpo JSON → objeto, parâmetros da URL → argumentos), executa o método e **converte** o retorno (objeto → JSON) para a resposta. A conversão de JSON é feita pelo **Jackson**.

**Para que serve**: expor a lógica da aplicação como **APIs HTTP** consumidas por front-end, aplicativos móveis e outros serviços.

**Como funciona**:

- **Controller**: `@RestController` (= `@Controller` + `@ResponseBody`: o retorno vira o **corpo** da resposta). `@RequestMapping("/tasks")` define o prefixo; `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping` definem o verbo e o caminho.
- **Entrada**: `@PathVariable` (`/tasks/{id}`), `@RequestParam` (`?nome=x`), `@RequestBody` (JSON → objeto), `@RequestHeader`, `@AuthenticationPrincipal` (usuário logado), `Pageable` (paginação).
- **Saída**: devolver um objeto (vira JSON); **`@ResponseStatus`** (status fixo, como `CREATED`); ou **`ResponseEntity<T>`** quando status, cabeçalhos e corpo são decididos em execução (`ResponseEntity.created(uri).body(x)`, `ResponseEntity.notFound().build()`).
- **Validação** (Bean Validation, `jakarta.validation`): `@Valid` em um `@RequestBody` ativa as restrições do DTO (`@NotNull`, `@NotBlank`, `@Size`, `@Email`, `@Min`, `@Pattern`...). Falha lança `MethodArgumentNotValidException` (→ **400**). Para validar parâmetros simples (`@RequestParam`, `@PathVariable`), use `@Validated` na classe.
- **Tratamento global de erros**: `@RestControllerAdvice` + `@ExceptionHandler` traduz exceções em respostas HTTP **em um só lugar**: o domínio lança `TaskNotFoundException`, e a borda decide que isso é **404**. O formato moderno é o **`ProblemDetail`** (RFC 9457), que o Spring 6 suporta (`spring.mvc.problemdetails.enabled=true`).
- **DTOs**: separe o contrato HTTP (request/response) das entidades; *records* são ideais. Isso evita expor campos internos (senha) e permite evoluir cada lado.
- **Jackson**: `@JsonProperty`, `@JsonIgnore`, `@JsonInclude(NON_NULL)`, `@JsonFormat`; o módulo `java.time` já vem registrado. Em um `PATCH`, um campo `Optional` **não distingue** "ausente" de "`null` explícito".
- **Filtro x interceptador**: um `Filter` (servlet) atua **antes** do Spring e vê toda requisição (segurança, log); um `HandlerInterceptor` atua dentro do MVC, ao redor do controller.
- **Content negotiation** (`Accept`/`Content-Type`), **CORS** (`@CrossOrigin` ou configuração global), **HATEOAS** (links nas respostas).
- **Clientes HTTP**: `RestClient` (Spring 6.1, síncrono e moderno), `WebClient` (reativo), **interfaces `@HttpExchange`** (declarativas, nativas do Spring) e `RestTemplate` (em modo de manutenção); o **OpenFeign** é outra opção declarativa (tópico 7.12).
- **Documentação**: **springdoc-openapi** gera OpenAPI/Swagger UI a partir das anotações (`@Operation`, `@ApiResponse`, `@Schema`); **Spring REST Docs** gera a documentação a partir dos **testes** (se a API mudar e o teste não, a build quebra).
- **MVC x WebFlux**: o MVC é **bloqueante** (uma thread por requisição); o **WebFlux** é reativo (`Mono`/`Flux`). Com **threads virtuais** (tópico 4.7), o MVC síncrono atende bem a maior parte dos casos com código mais simples.

**Exemplo**:

```java
public record CreateTaskRequest(
        @NotBlank @Size(min = 3, max = 100) String title,
        String description) {}

@RestController
@RequestMapping("/tasks")
class TaskController {
    private final CreateTaskUseCase create;
    private final GetTaskByIdUseCase getById;
    TaskController(CreateTaskUseCase create, GetTaskByIdUseCase getById) {
        this.create = create; this.getById = getById;
    }

    @PostMapping
    ResponseEntity<TaskResponse> criar(@RequestBody @Valid CreateTaskRequest req) {
        TaskResponse nova = TaskResponse.from(create.execute(req.title(), req.description()));
        return ResponseEntity.created(URI.create("/tasks/" + nova.id())).body(nova);   // 201 + Location
    }

    @GetMapping("/{id}")
    TaskResponse buscar(@PathVariable UUID id) {
        return TaskResponse.from(getById.execute(id));   // se não existir, a exceção vira 404 abaixo
    }
}

@RestControllerAdvice
class GlobalExceptionHandler {
    @ExceptionHandler(TaskNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    String naoEncontrada(TaskNotFoundException ex) { return ex.getMessage(); }
}
```

**Explicação do exemplo**: ao receber `POST /tasks` com `{"title":""}`, o `@Valid` rejeita o corpo **antes** de entrar no método (resposta **400**). Com um título válido, o Jackson converte o JSON em `CreateTaskRequest`, o caso de uso cria a tarefa e a resposta é **201 Created** com o cabeçalho `Location: /tasks/{id}`. No `GET /tasks/{id}`, se o caso de uso lançar `TaskNotFoundException`, o `@RestControllerAdvice` a captura e devolve **404** com a mensagem: o controller não tem `try/catch`.

## 7.6 Spring Data JPA

**O que é**: módulo que **gera repositórios a partir de interfaces**, em cima de JPA/Hibernate. Você declara uma interface que estende um repositório base, e o Spring fornece a implementação em tempo de execução (por *proxy*).

**Para que serve**: eliminar o código repetitivo de acesso a dados (CRUD, paginação, consultas simples) mantendo as consultas **expressivas**, e deixar a persistência trocável (tópico 6.6, adaptadores).

**Como funciona**:

- **Hierarquia de repositórios** (Spring Data 3): `Repository` (marcador) → `CrudRepository` (CRUD; `findAll()` devolve `Iterable`) → `ListCrudRepository` (devolve `List`); `PagingAndSortingRepository` (paginação e ordenação; **a partir do Spring Data 3 ela não estende mais `CrudRepository`**, daí combiná-las ou usar `JpaRepository`); **`JpaRepository`** (reúne CRUD em lista, paginação, `flush`, `saveAll`...). Um repositório pode estender só o que precisa (menos métodos expostos).
- **Query methods (consultas derivadas)**: o **nome do método** vira consulta: `findByEmail`, `findByNomeStartingWithIgnoreCase`, `findByAtivoTrueOrderByNomeAsc`, `existsByEmail`, `countByCidade`, `deleteByStatus`, `findTop3ByOrderByPrecoDesc`. Navegam associações com `_` (`findBySetores_Assentos_Codigo`). Nomes muito longos são sinal para usar `@Query`.
- **`@Query`**: JPQL (ou SQL nativo com `nativeQuery = true`) com `@Param`; `@Modifying` para `UPDATE`/`DELETE`.
- **Projeções**: **interface** com só os getters desejados (a consulta traz menos colunas; podem ter SpEL), **records/DTOs** (por construtor) ou dinâmicas.
- **Paginação**: `PageRequest.of(0, 20, Sort.by("nome"))` + um método que receba `Pageable` devolve `Page<T>` (conteúdo + total, com uma consulta `COUNT` extra) ou `Slice<T>` (sem total, mais barata).
- **`save`**: faz `persist` se a entidade é nova (id nulo, ou `@Version` nula) e `merge` caso contrário.
- **Auditoria**: `@CreatedDate`, `@LastModifiedDate`, `@CreatedBy` com `@EnableJpaAuditing`.
- **`@Transactional`**: abre a transação (e o contexto de persistência) ao redor do método, via **proxy**. **Propagação** padrão `REQUIRED` (participa da transação existente ou cria uma); `REQUIRES_NEW` cria uma nova. **Rollback por padrão só em exceções *unchecked***: use `rollbackFor` para as *checked*. `readOnly = true` em consultas. Vale **só em chamadas externas** ao bean (autoinvocação não passa pelo proxy, tópico 7.4). Normalmente fica na camada de **serviço/caso de uso**, não no controller nem no repositório.
- **Vários bancos**: a auto-configuração cobre **um** `DataSource`. Para vários, declare por contexto os beans `DataSourceProperties`, `HikariDataSource`, `LocalContainerEntityManagerFactoryBean` e `JpaTransactionManager`, e ligue com `@EnableJpaRepositories(basePackages, entityManagerFactoryRef, transactionManagerRef)`. Um contexto é `@Primary`; os demais usam `@Qualifier` e, se preciso, `defaultCandidate = false`. Cada `EntityManagerFactory` escaneia **só o seu pacote**. `@Transactional("nomeDoTransactionManager")` escolhe qual.
- **Open Session in View** (`spring.jpa.open-in-view`, ativo por padrão): mantém a sessão aberta até o fim da requisição, o que "esconde" `LazyInitializationException` e o **N+1** (muitas equipes o desativam).
- **Entity listeners**: `@PrePersist`, `@PostPersist`, `@PostUpdate`, `@PostRemove` (e `@EntityListeners`).

**Exemplo**:

```java
public interface ClienteRepository extends JpaRepository<Cliente, UUID> {
    List<Cliente> findByNomeStartingWithIgnoreCase(String prefixo);     // consulta derivada
    boolean existsByEmail(String email);
    Page<Cliente> findByAtivoTrue(Pageable pageable);

    @Query("select c from Cliente c where c.cidade = :cidade")
    List<Cliente> porCidade(@Param("cidade") String cidade);
}

@Service
class ClienteService {
    private final ClienteRepository repo;
    ClienteService(ClienteRepository repo) { this.repo = repo; }

    @Transactional
    public Cliente cadastrar(Cliente c) {
        if (repo.existsByEmail(c.getEmail())) throw new IllegalStateException("e-mail já cadastrado");
        return repo.save(c);
    }
}

Page<Cliente> pagina = repo.findByAtivoTrue(PageRequest.of(0, 10, Sort.by("nome")));
System.out.println(pagina.getTotalElements() + " " + pagina.getContent().size());
```

**Explicação do exemplo**: nenhuma das implementações foi escrita: `findByNomeStartingWithIgnoreCase` gera `WHERE UPPER(nome) LIKE 'ANA%'`, e `existsByEmail` gera uma consulta de existência. O `cadastrar` roda em **uma transação**: se `save` ou qualquer etapa lançar exceção *unchecked*, tudo sofre *rollback*. A consulta paginada pede a **primeira página de 10 clientes ordenados por nome** e devolve, além do conteúdo, o **total** de registros ativos (`getTotalElements`).

## 7.7 Spring Data REST

**O que é**: módulo que **expõe repositórios Spring Data como uma API REST HATEOAS automaticamente**, **sem escrever controller**: CRUD, paginação, ordenação e busca (`/search`) já vêm prontos, em formato **HAL** (com links). Inclui um *HAL Explorer* para navegar pela API.

**Para que serve**: protótipos, painéis internos e CRUD simples, com o mínimo de código.

**Como funciona**:

- **`@RepositoryRestResource(path = "customers", excerptProjection = X.class)`** personaliza o caminho e a **projeção resumida** usada nas listagens. Por padrão, o caminho deriva do nome da entidade no plural.
- **Controle do que é exposto**: `exported = false` no repositório esconde-o da API (ou o **`@RestResource(exported = false)`** em um método, como `deleteById`, para impedir o `DELETE`). Repositórios internos (como o de travas de assentos) **devem** ficar escondidos.
- **Consultas**: os *query methods* aparecem em `/recurso/search/nomeDoMetodo?param=valor` (use `@Param`).
- **Eventos**: `@RepositoryEventHandler` com `@HandleBeforeCreate`, `@HandleAfterCreate`, `@HandleAfterSave`, `@HandleAfterDelete` reagem a operações que chegam **pela API REST**. Diferença para `@PostPersist` (JPA): este dispara em **qualquer** persistência, não só pela API.
- **Limitação**: expõe a **entidade diretamente**, acoplando o contrato HTTP ao modelo de dados e dificultando regras de negócio e validações. Para domínios com regras, use controllers e DTOs.

**Exemplo**:

```java
@RepositoryRestResource(path = "customers", excerptProjection = CustomerExcerpt.class)
public interface CustomerEntityRepository extends ListCrudRepository<Customer, UUID> {
    List<Customer> findByFirstNameStartingWithIgnoreCase(@Param("firstName") String firstName);

    @Override
    @RestResource(exported = false)                 // DELETE não é exposto
    void deleteById(UUID id);
}
// GET  /customers                                              (lista paginada, com links)
// GET  /customers/search/findByFirstNameStartingWithIgnoreCase?firstName=an
```

**Explicação do exemplo**: só de declarar a interface, a aplicação ganha `GET/POST/PUT/PATCH` em `/customers` e uma busca em `/customers/search/...`. O `@RestResource(exported = false)` em `deleteById` faz o endpoint `DELETE /customers/{id}` **deixar de existir**, protegendo os dados de remoção pela API.

## 7.8 Spring Data MongoDB

**O que é**: integração do Spring com o **MongoDB**, banco de **documentos** (JSON/BSON) de **esquema flexível**: cada objeto vira um documento dentro de uma *collection*, e os documentos de uma mesma *collection* podem ter campos diferentes.

**Para que serve**: armazenar dados que **variam de estrutura** ou são lidos como um documento inteiro (catálogos, metadados, perfis, logs), sem criar colunas ou tabelas para cada variação.

**Como funciona**:

- **Mapeamento**: `@Document` (a classe vira *collection*), `@Id` (use `org.springframework.data.annotation.Id`; tipo `String` ou `ObjectId`), `@Field`, `@Indexed`. Tipos aninhados (`List`, `Map<String, Object>`, classes internas) viram **subdocumentos**: o modelo favorece **embutir** dados relacionados no mesmo documento (em vez de *joins*).
- **Repositório**: `MongoRepository<T, String>` com *query methods* (`findByEventId`) e `@Query` em JSON. **`MongoTemplate`** para consultas dinâmicas e **pipelines de agregação**.
- **Auditoria**: `@CreatedDate`, `@LastModifiedDate` com `@EnableMongoAuditing`. **Eventos de ciclo de vida**: `AbstractMongoEventListener` (`onBeforeSave`, `onAfterSave`, `onAfterDelete`).
- **Quando usar**: esquema variável, leitura por documento, escala horizontal. **Evite** para dados fortemente relacionais e para operações que precisam de **transações entre várias entidades** (o suporte a transações exige um *replica set*).
- Configuração por propriedades (URI de conexão, representação de `UUID`); os nomes das propriedades **mudaram entre versões do Boot** (verifique na sua).

**Exemplo**:

```java
@Document
public class EventMetadata {
    @Id private String id;
    private UUID eventId;                                  // elo com o evento no banco relacional
    private Map<String, Object> technicalRequirements;     // estrutura livre: varia por evento
    private List<Sector> sectors;                          // subdocumentos embutidos
    @CreatedDate private Instant createdOn;
}

public interface EventMetadataRepository extends MongoRepository<EventMetadata, String> {
    Optional<EventMetadata> findByEventId(UUID eventId);
}
```

**Explicação do exemplo**: um show de rock e uma peça de teatro podem ter `technicalRequirements` completamente diferentes; como é um `Map`, cada documento guarda os campos que fizerem sentido, sem alterar um esquema. Os setores ficam **embutidos** no documento, e carregá-los é uma única leitura. O `findByEventId` é gerado pelo nome, como nos repositórios JPA.

## 7.9 Spring Data Redis e cache

**O que é**: **Redis** é um banco **chave-valor em memória**, muito rápido, com estruturas (strings, *hashes*, listas, *sets*, *sorted sets*) e **expiração (TTL)** por chave. Processa os comandos em uma única thread, então cada comando é **atômico**. O Spring Data Redis o acessa por um cliente (**Lettuce**, o padrão, ou **Jedis**).

**Para que serve**: **cache** (reduzir latência e carga no banco), **dados temporários** com expiração (travas, sessões, *tokens*), contadores, *rate limiting*, filas leves e *pub/sub*.

**Como funciona**:

- **`RedisTemplate`/`StringRedisTemplate`**: acesso de baixo nível. Atenção ao **serializador**: o padrão (serialização Java) guarda bytes ilegíveis e exige `Serializable`; para JSON ou texto, configure `StringRedisSerializer`/`GenericJackson2JsonRedisSerializer`.
- **`@RedisHash(value = "seat_locks", timeToLive = 30)`**: a entidade é guardada como *hash*, e o Redis a **apaga sozinho** após 30 s (uma trava temporária sem *job* de limpeza). `@Indexed` cria um índice secundário para buscar por outro campo (o `@Id` deve ser o do Spring Data).
- **Trava (lock) atômica**: `existsById` seguido de `save` são **duas operações**: duas requisições simultâneas podem passar pela verificação juntas (**corrida**). O correto é **uma operação atômica**: `SET chave valor NX EX 30` (em Spring: `setIfAbsent(chave, valor, Duration)`). Para travas distribuídas robustas, use bibliotecas dedicadas (Redisson).
- **Cache declarativo** (`@EnableCaching` + um `CacheManager`, como o `RedisCacheManager`):

| Anotação | Efeito |
| --- | --- |
| `@Cacheable("nome")` | se a chave já está no cache, devolve-a e **nem executa o método**; senão executa e guarda o resultado |
| `@CachePut` | **sempre** executa e atualiza o cache |
| `@CacheEvict` | remove a entrada (`allEntries = true` limpa tudo) |

`condition` e `unless` (`unless = "#result.isEmpty()"`) decidem quando cachear; `key` define a chave (SpEL). Funciona por **proxy**: não vale em autoinvocação (tópico 7.4). O valor cacheado precisa ser **serializável** (conforme o serializador) e é preciso definir **TTL** (senão nunca expira) e lidar com dados **desatualizados** (*stale*).

- Outros usos: sessões distribuídas (Spring Session), filas e *streams*.

**Exemplo**:

```java
@Service
class VitrineService {
    @Cacheable(value = "vitrine", unless = "#result.isEmpty()")   // não cacheia lista vazia
    public List<EventoOutput> listar() {
        return repo.findAll().stream().map(EventoOutput::from).toList();   // consulta cara
    }
    @CacheEvict(value = "vitrine", allEntries = true)
    public void invalidar() { }
}

@RedisHash(value = "seat_locks", timeToLive = 30)               // expira em 30 s
class SeatLock {
    @Id String id;
    @Indexed String customerId;
}
// trava atômica: true só para quem ganhou a corrida
Boolean ok = redisTemplate.opsForValue().setIfAbsent("seat:A1", cliente, Duration.ofSeconds(30));
```

**Explicação do exemplo**: na primeira chamada a `listar()` o método executa e o resultado vai para o cache `vitrine`; nas seguintes, devolve do Redis **sem consultar o banco** até o cache ser invalidado ou expirar. O `unless` evita guardar uma lista vazia por engano. O `SeatLock` some sozinho em 30 s, liberando o assento se o cliente desistir. O `setIfAbsent` com expiração é **uma única operação atômica**: se duas pessoas tentam reservar o assento A1 no mesmo instante, **apenas uma** recebe `true`.

## 7.10 Eventos, assincronismo e agendamento

**O que é**: formas de **executar trabalho fora do fluxo principal** e de **desacoplar** partes do sistema: (1) **eventos de aplicação** (um componente avisa, outro reage), (2) **`@Async`** (executar um método em outra thread) e (3) **`@Scheduled`** (executar rotinas em horários ou intervalos).

**Para que serve**: não bloquear a requisição com tarefas lentas (enviar e-mail, gerar relatório), manter **contextos desacoplados** (o cadastro de clientes não precisa conhecer o módulo de vendas) e rodar **rotinas periódicas** (limpeza, conciliação).

**Como funciona**:

- **Eventos de aplicação (padrão *Observer*)**: `ApplicationEventPublisher.publishEvent(objeto)` publica; um método `@EventListener` recebe. O evento pode ser **qualquer objeto** (um `record` serve). Por padrão, o *listener* roda **de forma síncrona**, na **mesma thread e dentro da mesma transação** do publicador, então uma exceção no *listener* **propaga** para quem publicou.
- **`@TransactionalEventListener`**: reage só **depois** (por padrão, `AFTER_COMMIT`) que a transação do publicador **confirma**, evitando agir sobre algo que sofreu *rollback*. Se **não há transação ativa**, o evento é **ignorado** (a menos que `fallbackExecution = true`). Para fazer acesso ao banco nesse *listener*, use `@Transactional(propagation = REQUIRES_NEW)`.
- **Limites dos eventos em memória**: se a aplicação cair entre o *commit* e o *listener*, o evento **se perde**. Para entrega garantida, use o padrão ***outbox*** (gravar o evento no banco na mesma transação e publicá-lo depois) ou **mensageria** (Kafka, RabbitMQ).
- **`@Async`** (exige `@EnableAsync`): o método roda em **outra thread**, em um *executor* (o Boot configura um `applicationTaskExecutor`, que usa threads virtuais se `spring.threads.virtual.enabled=true`). Pode devolver `void`, `Future` ou **`CompletableFuture<T>`**. Funciona por **proxy** (não vale em autoinvocação) e o método deve ser público. Exceções em métodos `void` **não chegam** ao chamador (configure um `AsyncUncaughtExceptionHandler`). Combinar vários `@Async` que devolvem `CompletableFuture` e dar `join` paraleliza consultas independentes.
- **`@Scheduled`** (exige `@EnableScheduling`): `fixedRate` (a cada N ms desde o início), `fixedDelay` (N ms depois do fim da anterior), `cron` (no Spring, 6 campos, **com segundos**: `"0 0 3 * * *"` = todo dia às 3h). Por padrão, roda em **uma única thread** e, em um sistema com **várias instâncias**, executa **em todas** (use *ShedLock* ou similar para garantir uma só execução).

**Exemplo**:

```java
record ClienteCriado(String id, String nome) {}

@Service
class Cadastro {
    private final ClienteRepository repo;
    private final ApplicationEventPublisher publisher;
    Cadastro(ClienteRepository repo, ApplicationEventPublisher publisher) {
        this.repo = repo; this.publisher = publisher;
    }
    @Transactional
    void criar(Cliente c) {
        repo.save(c);
        publisher.publishEvent(new ClienteCriado(c.getId().toString(), c.getNome()));
    }
}

@Component
class VendasListener {
    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    void aoCriarCliente(ClienteCriado e) {
        /* cria a cópia local do cliente no contexto de vendas */
    }
}

@Scheduled(cron = "0 0 3 * * *")          // todos os dias às 03:00:00
void limpezaNoturna() { /* ... */ }
```

**Explicação do exemplo**: `criar` salva o cliente e **publica** o evento dentro da transação. O `VendasListener` só é acionado **depois do *commit***, em outra thread (`@Async`), então o cadastro não espera o módulo de vendas e, se o `save` falhar, o evento nunca é processado. O módulo de cadastro **não conhece** o de vendas: só publica um fato. A rotina `limpezaNoturna` é disparada pelo agendador às 3h.

## 7.11 Spring Security

**O que é**: framework de **autenticação** (*quem é você?*) e **autorização** (*o que você pode fazer?*). Funciona como uma **cadeia de filtros servlet** (`SecurityFilterChain`) executada **antes** de a requisição chegar ao controller. Ao incluir o *starter* de segurança, **todos os endpoints passam a exigir autenticação** (com uma senha gerada no log) até você configurar.

**Para que serve**: proteger endpoints e dados, controlar acesso por **papel/permissão**, guardar credenciais com segurança e se proteger de ataques comuns (CSRF, fixação de sessão, *clickjacking* via cabeçalhos).

**Como funciona**:

**Fluxo de autenticação**:

```
Filtro (monta um Authentication NÃO autenticado: usuário + senha)
  → AuthenticationManager (ProviderManager)
    → DaoAuthenticationProvider
        ├─ UserDetailsService.loadUserByUsername   (busca o usuário)
        └─ PasswordEncoder.matches                 (confere a senha com o HASH)
  → Authentication AUTENTICADO → guardado no SecurityContext (por padrão, na sessão)
```

- **`UserDetails`**: contrato do usuário (`getUsername`, `getPassword`, `getAuthorities`, flags da conta). **Papéis/permissões** são `GrantedAuthority` (`ROLE_ADMIN`). **`UserDetailsService`** carrega o usuário (de banco, LDAP...).
- **Senhas**: **nunca em texto puro**. `BCryptPasswordEncoder` aplica *salt* e é **deliberadamente lento** (dificulta força bruta). `DelegatingPasswordEncoder` (`{bcrypt}...`) permite migrar de algoritmo. **Não registre** senhas nem *hashes* em logs.
- **Configuração moderna (Security 6+)**: um bean `SecurityFilterChain` com a DSL lambda (`authorizeHttpRequests`, `requestMatchers`). `WebSecurityConfigurerAdapter` e `antMatchers` foram **removidos**.
- **Autorização em dois níveis**:
  - **Por URL**: `.requestMatchers("/admin/**").hasRole("ADMIN")`, `.anyRequest().authenticated()` (**negar por padrão**: libere só o necessário).
  - **Por método**: `@EnableMethodSecurity` + `@PreAuthorize("hasRole('ADMIN')")` (aceita SpEL: `#id == authentication.principal.id`) e `@PostAuthorize`. `hasRole('X')` procura a *authority* **`ROLE_X`** (o prefixo é automático); `hasAuthority('ROLE_X')` não acrescenta prefixo.
- **401 x 403**: **401** = não autenticado; **403** = autenticado, mas **sem permissão**. Sem formulário/Basic habilitados, o padrão para quem não está autenticado também é 403; configure um `AuthenticationEntryPoint` para devolver 401.
- **`@AuthenticationPrincipal`** injeta o usuário logado no controller; `SecurityContextHolder.getContext().getAuthentication()` obtém programaticamente.
- **Sessão x *stateless***: login por **sessão** (cookie `JSESSIONID`; o servidor guarda o estado) ou **token** (**JWT**/**OAuth2** no cabeçalho `Authorization: Bearer ...`, sem sessão no servidor: `SessionCreationPolicy.STATELESS`, escala melhor). O Spring tem suporte a *Resource Server* OAuth2/OIDC.
- **CSRF**: protege contra requisições forjadas quando o **navegador envia o cookie automaticamente**. Só desabilite em API ***stateless* com token no cabeçalho**; **com sessão por cookie, mantenha ativo** (ou use cookie `SameSite`). **CORS** é outra coisa: controla quais origens de navegador podem chamar a API.
- **Filtro de autenticação customizado**: estenda `UsernamePasswordAuthenticationFilter` e sobrescreva `attemptAuthentication` (por exemplo, para ler o login em **JSON**), e registre com `addFilterAt`. O filtro **não valida a senha**: apenas monta o `Authentication` e delega ao `AuthenticationManager`. Atenção: um `Filter` declarado como `@Component` também é registrado pelo Boot na cadeia do servlet e pode rodar **duas vezes**; desative com um `FilterRegistrationBean` (`setEnabled(false)`).
- **Autorização de dados** (quais *registros* o usuário vê) é **regra de negócio**, não de URL: resolva no caso de uso, por exemplo com **Strategy** por escopo (`OWN` x `ALL`).

**Exemplo**:

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
class SecurityConfig {
    @Bean
    SecurityFilterChain chain(HttpSecurity http) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)            // só em API stateless com token
            .authorizeHttpRequests(a -> a
                .requestMatchers("/api/auth/login").permitAll()
                .anyRequest().authenticated())                // negar por padrão
            .build();
    }
    @Bean PasswordEncoder encoder() { return new BCryptPasswordEncoder(); }
}

@RestController
class PropostaController {
    @PostMapping("/proposals")
    @PreAuthorize("hasRole('INFLUENCER')")                    // só quem tem ROLE_INFLUENCER
    ProposalResponse criar(@RequestBody CreateProposalRequest r, @AuthenticationPrincipal User user) {
        /* usa user.getId() para identificar o dono da proposta */
        return null;
    }
}
```

**Explicação do exemplo**: a cadeia libera só o endpoint de login e exige autenticação para todo o resto. A anotação `@PreAuthorize` restringe **este método** a quem tem o papel `INFLUENCER`: um usuário autenticado com outro papel recebe **403**. O `@AuthenticationPrincipal` entrega o usuário já autenticado, sem consultar o banco de novo. O `BCryptPasswordEncoder` é o *bean* usado pelo provedor para conferir senhas armazenadas como *hash*. (Neste exemplo, o CSRF está desligado porque se supõe uma API *stateless*; com sessão por cookie, não desligue.)

## 7.12 Spring Cloud OpenFeign e resiliência

**O que é**: o **OpenFeign** é um **cliente HTTP declarativo**: você descreve o contrato de uma API externa em uma **interface**, com as mesmas anotações do Spring MVC, e o framework gera a **implementação** (por proxy). Evita montar URLs, serializar/desserializar JSON e tratar erros à mão com `RestTemplate`/`WebClient`. (O Spring Cloud OpenFeign está em modo de manutenção; a alternativa nativa são as interfaces `@HttpExchange` com `RestClient`. Verifique o status atual.)

**Para que serve**: integrar serviços de forma **limpa** e, combinado a um **circuit breaker**, **tolerar falhas** de dependências externas sem derrubar a aplicação.

**Como funciona**:

- **Declaração**: `@FeignClient(name = "...", url = "...")` na interface + `@EnableFeignClients` na aplicação. Os métodos usam `@GetMapping`, `@PostMapping`, `@PathVariable`, `@RequestBody`, `@RequestHeader`. O cliente é injetado como qualquer bean.
- **Configuração por propriedades**, por nome do cliente: `spring.cloud.openfeign.client.config.<nome>.url`, `.logger-level=full`, `.connect-timeout`, `.read-timeout`, `.default-request-headers.*`. Em produção, URLs e chaves vêm de variáveis de ambiente ou de um cofre. **Sempre defina *timeouts***.
- **Por que resiliência**: chamadas remotas **falham ou ficam lentas**, e uma dependência lenta pode **prender todas as threads** e derrubar sua aplicação em cascata.
- **Circuit Breaker (Resilience4j)**: observa as chamadas e funciona como uma máquina de estados:

```
  CLOSED ──(falhas acima do limite)──▶ OPEN ──(tempo de espera)──▶ HALF-OPEN
     ▲                                                                  │
     └────────────(chamadas de teste com sucesso)───────────────────────┘
```

`CLOSED`: tudo passa normalmente. `OPEN`: **falha imediatamente**, sem tocar no serviço (dá fôlego a ele). `HALF-OPEN`: deixa passar algumas chamadas de teste. Ative com `spring.cloud.openfeign.circuitbreaker.enabled=true` e o *starter* do Resilience4j. Outras proteções: **timeout**, ***retry*** com *backoff* (só em operações idempotentes), ***bulkhead*** (isola recursos) e ***rate limiter***.

- **Fallback**: resposta alternativa quando a chamada falha ou o circuito está aberto (`fallback = X.class`; `fallbackFactory` dá acesso à causa). **Decisão de negócio crítica**: um fallback que devolve "lista vazia" pode significar **fail-open** ("seguir como se estivesse tudo bem"), **perigoso em compliance e segurança**, pois pode aprovar sem verificar; o **fail-closed** (barrar ou encaminhar para revisão manual) é o mais seguro. Defina fallback para **todos** os clientes dependentes.
- **Camada anticorrupção**: os DTOs externos **convertem a si mesmos** para o domínio (`toDomain()`), **normalizando `null`** e isolando o domínio de mudanças do fornecedor. Dados externos são **não confiáveis**.
- **Mocks de APIs externas** (Mockoon, WireMock) permitem desenvolver e testar cenários (sucesso, erro 500, lentidão) sem o fornecedor real.

**Exemplo**:

```java
@FeignClient(name = "sanction-client", fallback = SanctionClient.Fallback.class)
public interface SanctionClient {
    @GetMapping("/sanctions/companies/{registro}")
    SanctionResult consultar(@PathVariable String registro);

    @Component
    class Fallback implements SanctionClient {
        public SanctionResult consultar(String registro) {
            throw new ServicoIndisponivelException("Serviço de sanções fora do ar");   // fail-closed
        }
    }
}
```

```properties
spring.cloud.openfeign.client.config.sanction-client.url=http://localhost:3001
spring.cloud.openfeign.client.config.sanction-client.read-timeout=3000
spring.cloud.openfeign.circuitbreaker.enabled=true
```

**Explicação do exemplo**: chamar `consultar("123")` produz um `GET http://localhost:3001/sanctions/companies/123` sem nenhum código HTTP escrito. Se o serviço estiver fora do ar, estourar o *timeout* de 3 s ou o circuito estiver aberto, o Feign executa o `Fallback`. Aqui o fallback **lança uma exceção** (*fail-closed*): a análise de risco é interrompida em vez de concluir, erradamente, que "não há sanções". Se a regra do negócio permitir, o fallback poderia marcar a empresa como "revisão manual".

## 7.13 Testes no Spring

**O que é**: o `spring-boot-starter-test` traz **JUnit 5, Mockito, AssertJ, Hamcrest, JSONassert** e utilitários do Spring para testar desde a regra de negócio até a API completa.

**Para que serve**: validar o sistema **na velocidade certa para cada nível**: rápido e isolado para a regra de negócio, e mais pesado só onde a integração com o Spring/banco realmente importa.

**Como funciona**:

- **Sem Spring (mais rápido)**: `@ExtendWith(MockitoExtension.class)` com `@Mock`/`@InjectMocks` (tópico 5.5). **Prefira para a lógica de serviços e casos de uso.** Evite `@SpringBootTest` quando só precisa de Mockito (ele carrega o contexto inteiro à toa).
- **`@SpringBootTest`**: sobe o **contexto completo** (lento). `webEnvironment = RANDOM_PORT` + **`TestRestTemplate`** (ou `WebTestClient`) testa a API de ponta a ponta com servidor real. O Spring **reaproveita o contexto** entre classes de teste com a mesma configuração (*cache de contexto*); **`@DirtiesContext`** o descarta (use com parcimônia: é lento).
- **Testes de fatia (*slices*)**: carregam **só uma parte** do contexto:
  - **`@WebMvcTest(Controller.class)`**: apenas a camada web, com **`MockMvc`** (chama o controller **sem servidor**); as dependências (casos de uso) são fornecidas como *mocks*.
  - **`@DataJpaTest`**: apenas JPA e repositórios, com **banco em memória** substituindo o real (para usar o banco real: `@AutoConfigureTestDatabase(replace = NONE)`); cada teste roda em transação com *rollback*.
  - `@DataMongoTest`, `@JsonTest`, `@RestClientTest`.
- **Mocks no contexto**: **`@MockitoBean`** e `@MockitoSpyBean` (Spring Framework 6.2 / Boot 3.4+) **substituem** o antigo `@MockBean`/`@SpyBean`, que foi depreciado.
- **Segurança**: `@WithMockUser(roles = "ADMIN")`, `SecurityMockMvcRequestPostProcessors` (`csrf()`, `user(...)`).
- **Banco real nos testes**: **Testcontainers** sobe o banco em um contêiner; `@ServiceConnection` (Boot 3.1+) liga o contêiner à aplicação sem configurar URLs. H2 é rápido, mas o dialeto difere do banco de produção.
- **Dados e propriedades**: `@Sql` carrega scripts; `@ActiveProfiles("test")`, `@TestPropertySource`, `@DynamicPropertySource`.
- **APIs externas**: **WireMock**/`MockWebServer` simulam o fornecedor (cenários de erro e lentidão), importante para testar *fallbacks*.
- **Spring REST Docs**: os testes `MockMvc` geram *snippets* que o Asciidoctor junta na documentação da API.

**Exemplo**:

```java
@WebMvcTest(TaskController.class)
class TaskControllerTest {
    @Autowired MockMvc mvc;
    @MockitoBean CreateTaskUseCase useCase;                // substitui o bean real por um mock

    @Test
    void deveRetornar400QuandoTituloVazio() throws Exception {
        mvc.perform(post("/tasks")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"title\":\"\"}"))
           .andExpect(status().isBadRequest());
    }
}

@DataJpaTest
class ClienteRepositoryTest {
    @Autowired ClienteRepository repo;

    @Test
    void deveSalvarEBuscarPorEmail() {
        repo.save(new Cliente("Ana", "ana@x.com"));
        assertTrue(repo.existsByEmail("ana@x.com"));
    }
}
```

**Explicação do exemplo**: o primeiro teste sobe **só a camada web** e usa `MockMvc` para simular um `POST` com título vazio, verificando o **400** da validação, sem servidor, banco ou caso de uso real (substituído pelo `@MockitoBean`). O segundo sobe **só o JPA** com banco em memória, salva um cliente e confirma a consulta derivada `existsByEmail`; ao final, a transação é desfeita, deixando o banco limpo para o próximo teste.

## 7.14 Observabilidade, empacotamento e execução

**O que é**: **observabilidade** é a capacidade de entender **o que a aplicação está fazendo em produção** a partir do que ela emite. As três bases: **logs** (eventos), **métricas** (números ao longo do tempo) e **traces** (o caminho de uma requisição por vários serviços). **Empacotamento e execução** tratam de entregar a aplicação de forma reprodutível (JAR, contêiner).

**Para que serve**: detectar problemas rapidamente (alertas), entender desempenho e capacidade, diagnosticar incidentes e implantar de forma previsível em qualquer ambiente.

**Como funciona**:

- **Actuator**: endpoints de **operação** em `/actuator`: `health` (estado da aplicação e das dependências, como banco e Redis), `info`, `metrics`, `prometheus`, `env`, `loggers` (alterar nível de log **em execução**). Exponha **só o necessário** (`management.endpoints.web.exposure.include=health,info,metrics,prometheus`) e **proteja** os demais, pois vazam informações sensíveis. `management.endpoint.health.show-details=always` mostra o detalhe de cada componente. Há grupos de **liveness** (a aplicação está viva?) e **readiness** (está pronta para receber tráfego?), usados por orquestradores como o Kubernetes.
- **Micrometer**: **fachada de métricas** (como o SLF4J para logs): contadores, medidores, temporizadores; `MeterRegistry` e *tags* comuns (`application`). Os dados são exportados para sistemas como o **Prometheus** (que **coleta**, por *scrape*, o endpoint `/actuator/prometheus` periodicamente) e visualizados no **Grafana** (dashboards e alertas). **Traces distribuídos**: Micrometer Tracing/**OpenTelemetry** (Zipkin, Jaeger).
- **Logs**: SLF4J + Logback (tópico 3.17); em produção, para o *stdout* e coletados por uma plataforma (formato JSON/estruturado).
- **Docker**: o **`Dockerfile`** ou o plugin **Jib** (gera a imagem em camadas, sem Dockerfile) geram a imagem. **Docker Compose** sobe vários serviços (aplicação, banco, Prometheus, Grafana), com `healthcheck` e `depends_on: condition: service_healthy` para respeitar a ordem, e `mem_limit` para limitar a memória.
- **Boas práticas (*12-factor app*)**: configuração por **variáveis de ambiente**, **um processo por contêiner**, logs no *stdout*, aplicação **sem estado** (estado em banco/cache), ***graceful shutdown*** (`server.shutdown=graceful`), *health checks*, imagens com versão fixa (não `latest`).

**Exemplo**:

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  endpoint:
    health:
      show-details: when-authorized
```

```yaml
# compose.yml (trecho)
services:
  db:
    image: mysql:8.0
    environment: { MYSQL_ROOT_PASSWORD: root, MYSQL_DATABASE: app }
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      retries: 10
  app:
    image: minha-app:1.0
    depends_on:
      db: { condition: service_healthy }     # só sobe quando o banco responde
    ports: ["8080:8080"]
```

**Explicação do exemplo**: o primeiro trecho expõe só quatro endpoints do Actuator e mostra os detalhes da saúde apenas para usuários autorizados. Com isso, o Prometheus pode coletar `/actuator/prometheus` e o orquestrador pode consultar `/actuator/health`. No `compose.yml`, a aplicação só inicia depois que o *healthcheck* do banco passa, evitando erros de conexão na partida. (As credenciais `root/root` são só para desenvolvimento local.)

---

# Checklist de revisão

Marque o que você consegue **explicar em voz alta, com um exemplo**:

- [ ] Diferença entre JDK, JRE e JVM; stack x heap; como o GC decide o que liberar.
- [ ] Tipos primitivos, casting, overflow, `==` x `equals`, imutabilidade de `String`.
- [ ] Passagem por valor em objetos; sobrecarga x sobrescrita; polimorfismo em tempo de execução.
- [ ] Classe abstrata x interface (`default`, `static`); quando usar `enum`, `record`, classe interna.
- [ ] Contrato `equals`/`hashCode` e por que quebram `HashSet`/`HashMap`.
- [ ] Checked x unchecked; `finally`; try-with-resources; exceções customizadas.
- [ ] `List`, `Set`, `Map`, `Queue`: qual usar e a complexidade de cada uma; `Comparable` x `Comparator`.
- [ ] Generics: invariância, wildcards (PECS) e *type erasure*.
- [ ] Lambdas, `Predicate`/`Function`/`Consumer`/`Supplier`, method reference (4 tipos), `Optional` (`orElse` x `orElseGet`).
- [ ] Streams: lazy, terminais, `flatMap`, `groupingBy`, `toMap` com duplicata, paralelas (quando não usar).
- [ ] Threads: `start` x `run`, race condition, `synchronized`, `volatile`, deadlock, `wait/notify`.
- [ ] `ExecutorService`, `Future` x `CompletableFuture`, threads virtuais, coleções concorrentes.
- [ ] JDBC: `PreparedStatement` x SQL Injection, `ResultSet` (índice 1), transação com `commit`/`rollback`.
- [ ] JPA: entidade, relacionamentos, lazy, N+1, `@Transactional` e a armadilha da autoinvocação.
- [ ] SOLID (um exemplo de violação e correção de cada), Clean Code, ao menos 5 padrões de projeto.
- [ ] Arquitetura em camadas x hexagonal; DTO; regra da dependência.
- [ ] Spring: IoC/DI, escopos, auto-configuração, controllers REST, validação, `@RestControllerAdvice`.
- [ ] Spring Data (query methods, projeções, paginação), Redis (cache e TTL), Mongo (documentos).
- [ ] Spring Security: fluxo de autenticação, BCrypt, `hasRole`, 401 x 403, CSRF, sessão x JWT.
- [ ] OpenFeign + Circuit Breaker + fallback (fail-open x fail-closed).
- [ ] Testes: unitário x integração x slice, Mockito, `MockMvc`, Testcontainers.
