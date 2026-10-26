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

**O que é**: Java é uma linguagem compilada para um formato intermediário, o **bytecode** (`.class`), executado pela **JVM** (Java Virtual Machine). Daí a ideia de "escreva uma vez, rode em qualquer lugar": o mesmo bytecode roda em qualquer sistema que tenha uma JVM.

- **JVM**: carrega as classes (*class loader*), verifica o bytecode, executa. Começa interpretando e, com o **JIT** (*Just-In-Time compiler*), compila para código nativo os trechos executados com frequência.
- **JRE**: JVM + bibliotecas padrão. **JDK**: JRE + ferramentas de desenvolvimento (`javac`, `jar`, `javadoc`, `jshell`, `jlink`).
- **Memória da JVM**: **stack** (uma pilha por thread; guarda variáveis locais e referências de cada chamada de método), **heap** (onde vivem os objetos; compartilhado por todas as threads) e **metaspace** (metadados das classes).
- **Garbage Collector (GC)**: libera automaticamente objetos que não são mais alcançáveis por nenhuma referência. Você não faz `free`. Coletores: G1 (padrão), ZGC e Shenandoah (baixa latência), Parallel, Serial.
- **Versões**: nova versão a cada 6 meses; as **LTS** (suporte longo) são 8, 11, 17, 21 e 25.

**Para que serve**: portabilidade, gerenciamento automático de memória, segurança (o verificador de bytecode impede operações inválidas) e um ecossistema enorme (back-end, Android, big data).

**Exemplo**:

```bash
javac Ola.java     # gera Ola.class (bytecode)
java Ola           # JVM executa
java Ola.java      # Java 11+: compila e executa direto, em um passo
```

## 1.2 Estrutura de um programa Java

**O que é**: todo código vive dentro de **classes**. A execução começa no método `main`. Classes são organizadas em **pacotes** (`package`) e usam outras por `import`.

- Uma classe `public` deve estar em arquivo de **mesmo nome** (`Ola.java` → `public class Ola`).
- **Comentários**: `//` linha, `/* ... */` bloco, `/** ... */` **Javadoc** (gera documentação com a ferramenta `javadoc`).
- **Convenções de nomes**: classes em `PascalCase`, métodos e variáveis em `camelCase`, constantes em `MAIUSCULAS_COM_UNDERSCORE`, pacotes em minúsculas (`br.com.empresa.projeto`).
- **Ponto de entrada**: `public static void main(String[] args)` (clássico). Do **Java 25** (JEP 512) também vale `static void main(String[] args)` sem `public`, `main()` sem argumentos e até `main` de instância, e a classe `java.io.IO` oferece `IO.println` e `IO.readln`.
- `import java.util.List;` traz uma classe; `import java.util.*;` todas do pacote; `import static java.lang.Math.*;` membros estáticos. `java.lang` é importado automaticamente.

**Para que serve**: organiza o código, evita conflito de nomes (pacotes funcionam como "pastas" lógicas) e define o ponto onde o programa inicia.

**Exemplo**:

```java
package br.com.exemplo;

import java.util.List;

/** Programa de exemplo. */
public class Ola {
    public static void main(String[] args) {
        List<String> nomes = List.of("Ana", "Bia");
        System.out.println("Olá, " + nomes.get(0)); // Olá, Ana
    }
}
```

## 1.3 Variáveis e tipos de dados

**O que é**: uma variável é um espaço nomeado na memória com um **tipo** fixo (Java é **estaticamente tipado**). Há dois grupos de tipos:

- **Primitivos** (8): guardam o valor diretamente.
- **Referência** (classes, interfaces, arrays, enums, records): guardam o **endereço** de um objeto no heap.

| Tipo | Tamanho | Faixa / uso | Padrão (atributo) |
| --- | --- | --- | --- |
| `byte` | 8 bits | -128 a 127 | 0 |
| `short` | 16 bits | -32.768 a 32.767 | 0 |
| `int` | 32 bits | ≈ ±2,1 bilhões (padrão para inteiros) | 0 |
| `long` | 64 bits | ≈ ±9,2 × 10^18 | 0L |
| `float` | 32 bits | decimal, \~7 dígitos | 0.0f |
| `double` | 64 bits | decimal, \~15 dígitos (padrão para decimais) | 0.0 |
| `char` | 16 bits | um caractere UTF-16 | '\\u0000' |
| `boolean` | 1 valor | `true` ou `false` | false |

**Regras importantes**:

- **Variáveis locais não têm valor padrão**: usar sem inicializar é erro de compilação. Atributos de classe e elementos de array recebem o padrão da tabela (`null` para referências).
- **Escopo**: a variável existe só no bloco `{ }` onde foi declarada.
- **`final`** torna a variável atribuível uma única vez (constante).
- **`var`** (Java 10+) deixa o compilador inferir o tipo de variável **local**: `var total = 10;` continua sendo `int` (não é tipagem dinâmica).
- Literais: inteiro é `int`, decimal é `double`. Use sufixos `L` (`long`) e `F` (`float`), e `_` para legibilidade (`1_000_000`). Prefixos `0x` (hexa), `0b` (binário).
- `float`/`double` **não representam decimais exatamente** (`0.1 + 0.2 != 0.3`). Para dinheiro use `BigDecimal` (tópico 3.7).

**Para que serve**: o tipo define quanta memória é usada, que operações são permitidas e o intervalo de valores, e o compilador detecta erros de tipo antes de executar.

**Exemplo**:

```java
int idade = 25;
long populacao = 8_000_000_000L;
double preco = 19.90;
char letra = 'A';
boolean ativo = true;
final double PI = 3.14159;
var nome = "Ka";            // String inferida
```

## 1.4 Conversão de tipos (casting)

**O que é**: transformar um valor de um tipo em outro.

- **Widening (ampliação)**: de tipo menor para maior, **automática** e sem perda: `byte → short → int → long → float → double` (e `char → int`).
- **Narrowing (estreitamento)**: de maior para menor, exige **cast explícito** e pode **perder dados**: `int i = (int) 3.99;` vale 3 (trunca, não arredonda).
- Em expressões, `byte`/`short`/`char` são promovidos a `int`; se há `long`, `float` ou `double`, o resultado sobe para o maior tipo presente.
- **Overflow** (estouro) é silencioso: `Integer.MAX_VALUE + 1` vira o menor `int`. `Math.addExact` lança `ArithmeticException` em vez de estourar.
- Para objetos, **upcast** (para o tipo pai) é automático e **downcast** (para o filho) exige cast e pode lançar `ClassCastException`.
- Texto ↔ número: `Integer.parseInt("42")`, `String.valueOf(42)`.

**Para que serve**: misturar tipos em cálculos e adaptar valores a APIs que exigem um tipo específico, consciente da perda de precisão.

**Exemplo**:

```java
int a = 5, b = 2;
System.out.println(a / b);          // 2 (divisão inteira)
System.out.println((double) a / b); // 2.5
long grande = a;                    // widening automático
byte pequeno = (byte) 200;          // narrowing: vira -56
```

## 1.5 Operadores

**O que é**: símbolos que realizam operações sobre valores.

- **Aritméticos**: `+ - * / %`. Divisão entre inteiros trunca. `%` é o resto. `+` com `String` concatena, avaliando da esquerda para a direita (`1 + 2 + "a"` é `"3a"`, `"a" + 1 + 2` é `"a12"`).
- **Relacionais**: `== != > < >= <=` → `boolean`. Em objetos, `==` compara **referência**; para conteúdo use `equals`.
- **Lógicos**: `&&` (E), `||` (OU), `!` (NÃO). `&&` e `||` fazem **curto-circuito** (não avaliam o lado direito se o resultado já está definido); `&` e `|` avaliam sempre os dois lados.
- **Atribuição**: `=` e compostos `+= -= *= /= %=` (fazem cast implícito: `byte b += 1` compila).
- **Incremento/decremento**: `i++` (usa e depois soma) e `++i` (soma e depois usa).
- **Ternário**: `condição ? a : b`.
- **Bit a bit**: `& | ^ ~ << >> >>>`.
- **`instanceof`**: testa o tipo do objeto; com *pattern matching* (Java 16+) já faz o cast: `if (o instanceof String s) { s.length(); }`.
- **Precedência**: `* / %` antes de `+ -`; relacionais antes de `&&`; `&&` antes de `||`. Na dúvida, use parênteses.

**Para que serve**: construir expressões, cálculos, comparações e decisões.

**Exemplo**:

```java
int x = 10;
boolean maiorQue5EPar = x > 5 && x % 2 == 0;   // true
x += 5;                                         // 15
String tipo = x > 10 ? "grande" : "pequeno";    // "grande"
if (lista != null && !lista.isEmpty()) { }     // curto-circuito evita NullPointerException
```

## 1.6 Estruturas condicionais

**O que é**: comandos que decidem qual trecho de código executar.

- **`if / else if / else`**: avalia condições `boolean` em ordem e executa o primeiro bloco verdadeiro.
- **`switch` tradicional**: compara uma variável com constantes (`byte, short, int, char`, wrappers, `String`, `enum`; não aceita `long`, `boolean`, `float`, `double`). Sem `break`, ocorre **fall-through** (continua nos próximos `case`).
- **`switch` moderno (Java 14+)**: usa `->`, sem fall-through, pode **devolver valor** (expressão) e, com `enum`/tipos `sealed`, o compilador exige cobertura de todos os casos. `yield` devolve valor em bloco.
- **Pattern matching em `switch`** (Java 21): `case Circulo c -> ...`, com guarda `when`.
- **Operador ternário**: `if/else` compacto que produz valor.

**Para que serve**: ramificar a lógica conforme dados e estados. Use `if` para faixas e combinações; `switch` para uma variável contra valores discretos.

**Exemplo**:

```java
int nota = 85;
if (nota >= 90) System.out.println("A");
else if (nota >= 70) System.out.println("B");
else System.out.println("C");

int dia = 3;
String nome = switch (dia) {
    case 1, 7 -> "Fim de semana";
    case 2, 3, 4, 5, 6 -> "Dia útil";
    default -> "Inválido";
};
```

## 1.7 Estruturas de repetição

**O que é**: executam um bloco várias vezes.

| Laço | Quando usar |
| --- | --- |
| `for (init; cond; passo)` | número de iterações conhecido/controlado por contador |
| `while (cond)` | repete enquanto a condição for verdadeira (pode nunca executar) |
| `do { } while (cond);` | executa **pelo menos uma vez** |
| `for (T x : colecao)` (*for-each*) | percorrer arrays e coleções sem usar índice |

- `break` encerra o laço mais interno; `continue` pula para a próxima iteração. Com rótulo (`externo:`) controla laços aninhados.
- Cuidado com **laço infinito** (condição que nunca muda) e **erro de uma unidade** (`<` vs `<=`).
- Não remova elementos de uma coleção dentro de um *for-each* (`ConcurrentModificationException`); use `Iterator.remove()` ou `removeIf`.

**Para que serve**: processar coleções, repetir tentativas, ler dados até acabar, cálculos acumulados.

**Exemplo**:

```java
for (int i = 1; i <= 5; i++) {
    if (i == 3) continue;       // pula o 3
    System.out.println(i);      // 1 2 4 5
}
int n = 0;
while (n < 3) { n++; }
int[] nums = {4, 5, 6};
for (int v : nums) System.out.println(v);
```

## 1.8 Arrays

**O que é**: estrutura de **tamanho fixo** que guarda elementos do mesmo tipo em posições numeradas a partir de 0. É um **objeto** (fica no heap) mesmo com primitivos. Acessar índice inválido lança `ArrayIndexOutOfBoundsException`.

- Elementos não inicializados recebem o valor padrão do tipo.
- `length` é **atributo** (sem parênteses).
- **Multidimensional** é array de arrays; linhas podem ter tamanhos diferentes (*jagged*).
- Arrays são cópias por **referência**: `b = a` não copia. Use `Arrays.copyOf` ou `clone`.
- Utilitário `java.util.Arrays`: `toString`, `deepToString`, `sort`, `binarySearch` (array ordenado), `fill`, `copyOf`, `copyOfRange`, `equals`, `asList`, `stream`.

**Para que serve**: armazenar sequências de tamanho conhecido com acesso rápido por índice. Quando o tamanho varia, prefira `List`.

**Exemplo**:

```java
int[] idades = new int[3];          // {0, 0, 0}
idades[0] = 21;
int[] primos = {2, 3, 5, 7};
int[][] matriz = new int[2][3];     // 2 linhas, 3 colunas
System.out.println(Arrays.toString(primos));   // [2, 3, 5, 7]
Arrays.sort(primos);
```

## 1.9 Métodos

**O que é**: bloco de código nomeado e reutilizável que pode receber **parâmetros** e **devolver** um valor. Assinatura: `modificador tipoRetorno nome(parâmetros)`.

- **`void`**: não devolve nada. `return` encerra o método (e devolve valor, se houver). Todo caminho de um método não-`void` deve retornar.
- **Sobrecarga (*overload*)**: mesmo nome com parâmetros diferentes (tipo, quantidade ou ordem); o compilador escolhe pela chamada.
- **Passagem por valor**: Java **sempre** copia o argumento. Em primitivos, a cópia do dado (o original não muda). Em objetos, a cópia da **referência**: dá para alterar o estado do objeto, mas não fazer o original apontar para outro objeto.
- **Varargs** (`int... nums`): quantidade variável de argumentos; é um array dentro do método; deve ser o último parâmetro.
- **Recursão**: método que chama a si mesmo; precisa de **caso base**, senão `StackOverflowError`.
- **Métodos estáticos** pertencem à classe (`Math.max`); **de instância** pertencem ao objeto.

**Para que serve**: evitar repetição (DRY), dividir problemas em partes pequenas e testáveis, dar nome à intenção do código.

**Exemplo**:

```java
static int soma(int... nums) {
    int total = 0;
    for (int n : nums) total += n;
    return total;
}
static double soma(double a, double b) { return a + b; }   // sobrecarga
static int fatorial(int n) { return n <= 1 ? 1 : n * fatorial(n - 1); }

soma(1, 2, 3);   // 6
```

## 1.10 Strings

**O que é**: `String` é uma classe que representa texto. É **imutável**: nenhum método altera o objeto, todos devolvem **uma nova** String. Literais ficam no **String pool** (literais iguais compartilham o mesmo objeto).

- **`==` x `equals`**: `==` compara referência; `equals` compara conteúdo. Para conteúdo, **sempre** `equals` (ou `equalsIgnoreCase`).
- Métodos comuns: `length`, `charAt`, `substring(ini, fim)` (fim exclusivo), `indexOf`, `contains`, `startsWith/endsWith`, `replace`, `split` (regex), `trim`/`strip`, `isEmpty`/`isBlank`, `toUpperCase/toLowerCase`, `compareTo`, `repeat`, `join`, `format`/`formatted`.
- **Text blocks** (`""" ... """`, Java 15): textos multilinha (SQL, JSON).
- **`StringBuilder`**: buffer **mutável** para montar texto em laços sem criar milhares de Strings (`+=` em laço é O(n²)). **`StringBuffer`** é o equivalente sincronizado (thread-safe, mais lento).
- **Comparação de ordem**: `compareTo` (lexicográfica, devolve negativo/0/positivo).

**Para que serve**: representar e manipular qualquer informação textual.

**Exemplo**:

```java
String nome = "  Maria  ";
System.out.println(nome.trim().toUpperCase());      // MARIA
System.out.println("abc".equals("ABC"));            // false
String csv = String.join(",", "a", "b", "c");       // a,b,c
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 3; i++) sb.append(i).append(' ');
System.out.println(sb.toString().trim());           // 0 1 2
String json = """
        {"nome": "Ana"}
        """;
```

## 1.11 Classes wrapper, autoboxing e unboxing

**O que é**: cada primitivo tem uma classe correspondente (`int`→`Integer`, `double`→`Double`, `char`→`Character`, `boolean`→`Boolean`, `long`→`Long`, `byte`→`Byte`, `short`→`Short`, `float`→`Float`). **Autoboxing** converte primitivo em wrapper automaticamente; **unboxing** faz o inverso.

- Existem porque coleções e *generics* só aceitam objetos (`List<Integer>`), e porque oferecem métodos úteis e o valor `null`.
- **Unboxing de `null`** lança `NullPointerException`.
- **Cache de `Integer`** de -128 a 127: `==` entre wrappers pode dar `true` com 127 e `false` com 128. **Compare wrappers com `equals`.**
- Utilitários: `Integer.parseInt`, `Integer.valueOf`, `Integer.MAX_VALUE`, `Integer.compare`, `Double.parseDouble`, `Boolean.parseBoolean`, `Character.isDigit/isLetter/toUpperCase`.
- Boxing em laços pesados custa alocação: use streams primitivas (`IntStream`).

**Para que serve**: usar valores primitivos onde se exigem objetos, converter texto em números e acessar constantes/métodos dos tipos.

**Exemplo**:

```java
Integer a = 10;                 // autoboxing
int b = a;                      // unboxing
int n = Integer.parseInt("42");
List<Integer> lista = new ArrayList<>();
lista.add(5);                   // int → Integer
System.out.println(Character.isDigit('7'));   // true
```

## 1.12 Entrada e saída no console

**O que é**: `System.out` (saída padrão), `System.err` (erros) e `System.in` (entrada). Para ler, usa-se `Scanner` ou `BufferedReader`.

- Saída: `println` (com quebra), `print`, `printf`/`format` (formatado: `%d` inteiro, `%s` texto, `%.2f` decimal com 2 casas, `%n` quebra de linha, `%,d` com separador de milhar).
- **`Scanner`**: `nextLine()` (linha), `nextInt()`, `nextDouble()`, `next()` (palavra), `hasNext...()`. **Pegadinha**: `nextInt()` não consome a quebra de linha, então um `nextLine()` seguinte devolve vazio; resolva com um `nextLine()` extra ou lendo tudo como linha e convertendo com `parseInt`.
- **Argumentos de linha de comando**: o array `args` do `main`.
- Feche `Scanner` de arquivos com *try-with-resources* (não feche o de `System.in` se ainda for usá-lo).

**Para que serve**: interagir com o usuário em programas de console e exibir resultados e logs simples.

**Exemplo**:

```java
Scanner sc = new Scanner(System.in);
System.out.print("Nome: ");
String nome = sc.nextLine();
System.out.print("Idade: ");
int idade = Integer.parseInt(sc.nextLine());
System.out.printf("%s tem %d anos e R$ %.2f%n", nome, idade, 1234.5);
```

---

# Parte 2: Orientação a objetos

## 2.1 Classes e objetos

**O que é**: **classe** é o molde que define os **atributos** (estado) e **métodos** (comportamento) de um tipo de coisa. **Objeto** é uma **instância** da classe, criada com `new`, com seu próprio espaço no heap e seus próprios valores. Uma variável de tipo objeto guarda uma **referência** (endereço), não o objeto: `b = a` faz as duas apontarem para o **mesmo** objeto. Só `new` cria outro. Atributos não inicializados recebem o valor padrão do tipo (`null`, `0`, `false`).

**Para que serve**: modelar o domínio do problema agrupando dados e regras que operam sobre eles, em vez de variáveis e funções soltas. É a base dos quatro pilares: **abstração, encapsulamento, herança e polimorfismo**.

**Exemplo**:

```java
class Carro {
    String nome;
    int ano;
    void ligar() { System.out.println(nome + " ligado"); }
}

Carro a = new Carro();
a.nome = "Fusca";
Carro b = a;          // mesma referência
b.nome = "Gol";
System.out.println(a.nome);   // Gol
Carro c = new Carro();        // outro objeto, independente
```

## 2.2 Construtores

**O que é**: método especial chamado em `new`, com o **nome da classe** e **sem tipo de retorno**. Inicializa o objeto em um estado válido.

- Se a classe **não declara nenhum**, o compilador cria o **construtor padrão** (sem parâmetros). Ao declarar **qualquer** construtor, o padrão deixa de existir.
- **Sobrecarga**: vários construtores com parâmetros diferentes.
- **`this(...)`** chama outro construtor da mesma classe (deve ser a primeira instrução) e evita duplicar código.
- Quando há muitos parâmetros opcionais, use o padrão **Builder** (tópico 6.3).

**Para que serve**: garantir que todo objeto nasça com os dados obrigatórios e consistentes.

**Exemplo**:

```java
class Pessoa {
    private final String nome;
    private final int idade;
    Pessoa(String nome, int idade) { this.nome = nome; this.idade = idade; }
    Pessoa(String nome) { this(nome, 0); }    // delega ao outro construtor
}
Pessoa p = new Pessoa("Ana", 30);
```

## 2.3 As palavras `this` e `super`

**O que é**: `this` é a referência ao **objeto atual**; `super` é a referência à **parte da superclasse** do objeto atual.

- `this.campo` desambigua atributo de parâmetro de mesmo nome. `this` também pode ser passado ou devolvido (`return this`, base dos builders).
- `super(...)` chama o construtor da superclasse (primeira instrução do construtor). Se omitido, o compilador insere `super()` e falha se o pai não tiver construtor sem argumentos.
- `super.metodo()` chama a versão do pai de um método sobrescrito.

**Para que serve**: acessar membros próprios sem ambiguidade e reaproveitar a inicialização/comportamento do pai.

**Exemplo**:

```java
class Animal { String nome; Animal(String nome) { this.nome = nome; }
               String som() { return "..."; } }
class Cao extends Animal {
    Cao(String nome) { super(nome); }
    @Override String som() { return super.som() + "Au!"; }
}
```

## 2.4 Encapsulamento e modificadores de acesso

**O que é**: esconder o estado interno do objeto e só permitir acesso por métodos controlados. O atributo fica `private`; leitura e escrita passam por **getters** e **setters**, que podem **validar**.

| Modificador | Mesma classe | Mesmo pacote | Subclasse (outro pacote) | Qualquer lugar |
| --- | --- | --- | --- | --- |
| `private` | sim | não | não | não |
| (padrão) | sim | sim | não | não |
| `protected` | sim | sim | sim | não |
| `public` | sim | sim | sim | sim |

- Convenção: `getNome()`/`setNome()`; para `boolean`, `isAtivo()`.
- Regra: comece com tudo `private` e abra o mínimo. Nem todo atributo precisa de setter; objetos **imutáveis** (campos `final`, sem setters) são mais seguros.
- Cuidado com getters que devolvem arrays/coleções mutáveis: expõem o estado interno (devolva cópia ou visão imutável).

**Para que serve**: proteger **invariantes** (regras que sempre devem valer), reduzir acoplamento e permitir mudar a implementação interna sem quebrar quem usa a classe.

**Exemplo**:

```java
class Conta {
    private double saldo;
    public void depositar(double v) {
        if (v <= 0) throw new IllegalArgumentException("Valor inválido");
        saldo += v;
    }
    public double getSaldo() { return saldo; }
}
```

## 2.5 Membros `static` e blocos de inicialização

**O que é**: membro `static` pertence à **classe**, não a cada objeto: existe **uma cópia** compartilhada. Acessa-se por `Classe.membro`. Método estático não tem `this` e não acessa membros de instância diretamente.

- **Bloco de instância** `{ ... }`: roda a cada `new`, antes do corpo do construtor.
- **Bloco estático** `static { ... }`: roda **uma vez**, quando a classe é carregada.
- **Ordem ao criar um objeto de subclasse**: blocos estáticos da superclasse → blocos estáticos da subclasse → (campos, blocos de instância, construtor) da superclasse → (campos, blocos de instância, construtor) da subclasse.
- Constantes: `static final` em MAIÚSCULAS.
- Cuidado: estado estático mutável é estado **global** (difícil de testar, perigoso com threads).

**Para que serve**: constantes, utilitários sem estado (`Math.sqrt`), contadores compartilhados, fábricas, inicialização única de recursos.

**Exemplo**:

```java
class Contador {
    static int total;                 // compartilhado
    static final double TAXA = 0.1;
    static { System.out.println("classe carregada"); }
    Contador() { total++; }
    static int getTotal() { return total; }
}
new Contador(); new Contador();
System.out.println(Contador.getTotal());   // 2
```

## 2.6 Herança

**O que é**: uma classe (**subclasse**) herda atributos e métodos de outra (**superclasse**) com `extends`, podendo acrescentar e especializar. Java tem herança **simples** de classes (uma superclasse), e toda classe herda de `Object`.

- Membros `private` não são acessíveis diretamente na subclasse; `protected` são.
- Relação **"é um"** (*is-a*). Use herança só quando vale o **Princípio de Substituição de Liskov**: qualquer subclasse pode ser usada onde se espera a superclasse sem quebrar nada.
- Herança cria acoplamento forte (a subclasse depende de detalhes do pai). Para reaproveitar código sem "é um", prefira **composição** (tópico 2.15).

**Para que serve**: reaproveitar código e modelar hierarquias de especialização (`Funcionario` → `Gerente`).

**Exemplo**:

```java
class Funcionario {
    protected double salario;
    Funcionario(double s) { salario = s; }
    double bonus() { return salario * 0.05; }
}
class Gerente extends Funcionario {
    Gerente(double s) { super(s); }
    @Override double bonus() { return salario * 0.2; }
}
```

## 2.7 Sobrescrita (*override*) e sobrecarga (*overload*)

**O que é**:

|  | Sobrecarga | Sobrescrita |
| --- | --- | --- |
| Onde | mesma classe | subclasse redefine método do pai |
| Assinatura | **diferente** (parâmetros) | **igual** |
| Resolvida | em **compilação** | em **execução** (polimorfismo) |

Regras da sobrescrita: mesma assinatura; retorno igual ou **covariante** (subtipo); acesso igual ou **mais amplo**; exceções *checked* iguais ou mais restritas; não vale para `private`, `static` e `final`. Use sempre **`@Override`**: o compilador acusa se você errar a assinatura.

**Para que serve**: sobrecarga dá várias formas de chamar a mesma operação; sobrescrita permite que cada subclasse tenha seu comportamento.

**Exemplo**:

```java
class Calc {
    int soma(int a, int b) { return a + b; }
    double soma(double a, double b) { return a + b; }   // sobrecarga
}
class Animal { String som() { return "..."; } }
class Gato extends Animal {
    @Override String som() { return "Miau"; }              // sobrescrita
}
```

## 2.8 Polimorfismo

**O que é**: capacidade de tratar objetos de tipos diferentes pela **mesma interface**. Uma variável do tipo pai (classe ou interface) aponta para um objeto de qualquer subtipo, e a versão do método executada é a do **objeto real**, escolhida em **tempo de execução** (*dynamic dispatch*).

- **Upcast** é automático; **downcast** exige cast e pode lançar `ClassCastException` (proteja com `instanceof`).
- Não são polimórficos: atributos, métodos `static`, `private` e a escolha entre **sobrecargas**.
- Princípio de projeto: **programe para a interface, não para a implementação**.

**Para que serve**: escrever código genérico que funciona para qualquer subtipo, eliminando `if/else` por tipo e permitindo estender o sistema sem alterar o código existente (Aberto/Fechado).

**Exemplo**:

```java
List<Animal> bichos = List.of(new Cao("Rex"), new Gato());
for (Animal a : bichos) {
    System.out.println(a.som());   // cada objeto responde do seu jeito
}
```

## 2.9 Classes abstratas

**O que é**: classe marcada `abstract` que **não pode ser instanciada**. Pode ter atributos, construtores, métodos concretos e **métodos abstratos** (sem corpo), que as subclasses concretas são obrigadas a implementar.

- Representa um conceito incompleto ("Forma" não existe sozinha, só Círculo, Quadrado).
- Útil no padrão **Template Method**: a classe abstrata define o esqueleto do algoritmo e deixa passos para as subclasses.
- Cuidado: chamar método abstrato/sobrescrevível dentro do construtor da superclasse executa a versão da subclasse **antes** de os campos dela estarem inicializados.

**Para que serve**: compartilhar estado e código comum entre classes relacionadas, impondo um contrato parcial.

**Exemplo**:

```java
abstract class Forma {
    abstract double area();
    void imprimir() { System.out.println("Área: " + area()); }   // template
}
class Quadrado extends Forma {
    double lado = 2;
    double area() { return lado * lado; }
}
```

## 2.10 Interfaces

**O que é**: **contrato** que define *o que* uma classe faz, sem dizer *como*. Uma classe pode implementar **várias** interfaces (`implements A, B`), contornando a ausência de herança múltipla de classes.

- Métodos são `public abstract` por padrão; campos são `public static final`.
- **`default`** (Java 8): método com implementação, permite evoluir a interface sem quebrar implementadores.
- **`static`**: método chamado pela própria interface (`Interface.metodo()`), não herdado.
- **`private`** (Java 9): reaproveita código entre `default`s.
- **Conflito de `default`**: a classe deve sobrescrever e pode escolher com `Interface.super.metodo()`. Regra: **classe vence** interface; interface mais específica vence.
- **Interface funcional**: exatamente um método abstrato (`@FunctionalInterface`), usada com lambdas.
- **`sealed interface`** limita quem pode implementar.
- **Interface x classe abstrata**: interface é capacidade/contrato (sem estado de instância); abstrata é base com estado.

**Para que serve**: desacoplar (depender de abstrações), permitir implementações intercambiáveis (injeção de dependência, testes com mocks) e definir papéis.

**Exemplo**:

```java
interface Pagamento {
    void pagar(double valor);
    default void recibo() { System.out.println("Pago"); }
}
class Pix implements Pagamento {
    public void pagar(double v) { System.out.println("Pix " + v); }
}
Pagamento p = new Pix();
p.pagar(50);
```

## 2.11 O modificador `final`

**O que é**: impede mudança.

- **Variável/atributo**: atribuída uma única vez. Em referências, a **referência** não muda, mas o objeto pode mudar.
- **Método**: não pode ser sobrescrito.
- **Classe**: não pode ser estendida (`String`, `Integer`).
- **Parâmetro/variável local**: não pode ser reatribuída; lambdas e classes anônimas só capturam variáveis *final* ou *efetivamente final*.

**Para que serve**: constantes, **imutabilidade** (segurança em concorrência), travar comportamento que não deve ser alterado.

**Exemplo**:

```java
class Config {
    static final int MAX = 10;
    final List<String> itens = new ArrayList<>();
    void teste() { itens.add("x"); /* ok */  /* itens = null; // erro */ }
}
```

## 2.12 Enumerações (`enum`)

**O que é**: tipo com um **conjunto fixo e conhecido** de constantes. Cada constante é uma instância única da classe `enum`. Pode ter atributos, construtor (sempre privado), métodos, implementar interfaces e ter **método abstrato com corpo por constante**.

- Métodos prontos: `values()`, `valueOf(String)`, `name()`, `ordinal()` (evite persistir o ordinal), `compareTo`.
- Funciona em `switch`; `EnumMap`/`EnumSet` são coleções otimizadas para enums.
- Em JPA, use `@Enumerated(EnumType.STRING)`.

**Para que serve**: representar categorias/estados (status de pedido, dias da semana, tipos de pagamento) com segurança de tipos, em vez de `int`/`String` soltos.

**Exemplo**:

```java
enum Status {
    ABERTO("Aberto"), FECHADO("Fechado");
    private final String rotulo;
    Status(String rotulo) { this.rotulo = rotulo; }
    String rotulo() { return rotulo; }
}
Status s = Status.valueOf("ABERTO");
for (Status x : Status.values()) System.out.println(x + " " + x.rotulo());
```

## 2.13 Records

**O que é** (Java 16+): classe **de dados imutável** com sintaxe enxuta. Declarando os componentes, o compilador gera construtor canônico, *accessors* (`nome()`, sem `get`), `equals`, `hashCode` e `toString`.

- Campos `private final`; o record é `final` e não estende classes (pode implementar interfaces).
- **Construtor compacto** valida/normaliza; pode ter métodos e membros estáticos.

**Para que serve**: DTOs, *value objects* (identificadores, coordenadas), chaves de mapa, retorno de vários valores. Não serve para entidades mutáveis nem herança.

**Exemplo**:

```java
record Ponto(int x, int y) {
    Ponto {                                  // construtor compacto
        if (x < 0) throw new IllegalArgumentException();
    }
    double distancia() { return Math.sqrt(x * x + y * y); }
}
Ponto p = new Ponto(3, 4);
System.out.println(p.x() + " " + p);    // 3 Ponto[x=3, y=4]
```

## 2.14 Classes internas, locais e anônimas

**O que é**: classes declaradas dentro de outras.

| Tipo | Declaração | Precisa de instância externa? |
| --- | --- | --- |
| **Interna (inner)** | dentro da classe, sem `static` | sim; acessa até membros `private` da externa |
| **Aninhada estática** | dentro da classe, com `static` | não; só acessa membros estáticos da externa |
| **Local** | dentro de um método | só existe naquele método |
| **Anônima** | `new Tipo() { ... }` | classe sem nome criada na hora |

- Criar inner: `outer.new Inner()`. Dentro dela, `Externa.this` é o objeto externo.
- A inner guarda **referência implícita** à externa (risco de *memory leak*); se não precisar dela, torne-a `static`.
- Anônima: implementação de uso único; para interfaces funcionais, a **lambda** é a forma moderna (e `this` na lambda é a classe que a contém).

**Para que serve**: agrupar classes auxiliares fortemente ligadas à externa (o `Builder` de uma classe), callbacks, esconder detalhes de implementação.

**Exemplo**:

```java
class Externa {
    private String msg = "oi";
    class Interna { void mostrar() { System.out.println(msg); } }
    static class Aninhada { }
}
Externa e = new Externa();
Externa.Interna i = e.new Interna();
Runnable r = new Runnable() {                     // anônima
    public void run() { System.out.println("rodando"); }
};
```

## 2.15 Associação, agregação e composição

**O que é**: formas de relacionar objetos (relação **"tem um"**, *has-a*).

- **Associação**: um objeto referencia outro. Pode ser **unidirecional** (só A conhece B) ou **bidirecional** (os dois se conhecem; cuidado com dessincronização e recursão em `toString`/`equals`). Cardinalidade: 1-1, 1-N, N-N.
- **Agregação**: o todo "tem" partes que **existem sem ele** (Escola e Professor).
- **Composição**: as partes **dependem do todo** e morrem com ele (Pedido e ItemDoPedido).
- **Composição x herança**: "Favoreça composição sobre herança". Compor objetos dá flexibilidade (trocar partes em execução) sem o acoplamento da herança.

**Para que serve**: modelar relações reais entre entidades do domínio e montar comportamento complexo combinando objetos simples.

**Exemplo**:

```java
class Motor { void ligar() { } }
class Carro {
    private final Motor motor = new Motor();     // composição
    void ligar() { motor.ligar(); }              // delega
}
class Time { List<Jogador> jogadores = new ArrayList<>(); }   // agregação 1-N
```

## 2.16 A classe `Object`: `equals`, `hashCode`, `toString`

**O que é**: raiz de toda hierarquia; todo objeto herda seus métodos.

- **`toString()`**: texto do objeto. O padrão é `Classe@hash`; sobrescreva para depuração e logs (não inclua senhas).
- **`equals(Object)`**: igualdade lógica. O padrão compara **referência** (`==`). Contrato: **reflexivo**, **simétrico**, **transitivo**, **consistente** e `x.equals(null)` é `false`.
- **`hashCode()`**: inteiro usado por `HashMap`/`HashSet`. **Regra de ouro**: objetos **iguais** (`equals`) devem ter o **mesmo** `hashCode`; sobrescreva **sempre os dois juntos** e use os mesmos campos (`Objects.equals`, `Objects.hash`). Não use campos mutáveis em objetos que servem de chave.
- `getClass()`, `clone()` (cópia; frágil, prefira construtor de cópia), `finalize()` (obsoleto, não use).

**Para que serve**: comparar objetos por valor, usá-los em coleções baseadas em hash e exibi-los de forma legível.

**Exemplo**:

```java
class Produto {
    private final String codigo;
    Produto(String c) { codigo = c; }
    @Override public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        return Objects.equals(codigo, ((Produto) o).codigo);
    }
    @Override public int hashCode() { return Objects.hash(codigo); }
    @Override public String toString() { return "Produto{" + codigo + "}"; }
}
```

## 2.17 Tratamento de exceções

**O que é**: mecanismo para sinalizar e tratar **situações anormais** sem misturar com o fluxo normal. Hierarquia:

```
Throwable
├── Error                (problemas da JVM: OutOfMemoryError, StackOverflowError; não trate)
└── Exception
    ├── checked          (IOException, SQLException: o compilador EXIGE tratar ou declarar)
    └── RuntimeException (unchecked: NullPointerException, IllegalArgumentException, ...)
```

- **Checked**: condição externa previsível (arquivo, rede, banco). **Unchecked**: normalmente **bug** de programação; frameworks modernos (Spring) preferem unchecked.
- **`try / catch / finally`**: `catch` são avaliados **em ordem** (específicas antes das genéricas); **multi-catch** (`A | B`) para tratamento igual; `finally` roda sempre (use para liberar recursos; nunca coloque `return` nele).
- **`throw`** lança uma exceção; **`throws`** declara que o método pode lançá-la.
- **try-with-resources**: `try (var r = ...) { }` fecha recursos `AutoCloseable` automaticamente, na ordem inversa. Exceções do `close` ficam em `getSuppressed()`.
- **Exceções customizadas**: estenda `Exception` (checked) ou `RuntimeException` (unchecked); ofereça construtor com `cause`.
- **Sobrescrita**: o método filho não pode declarar exceção *checked* nova ou mais ampla que a do pai.
- **Boas práticas**: nunca engula (`catch` vazio); não capture `Exception`/`Throwable` genéricas fora da borda da aplicação; preserve a causa (`new X(msg, e)`); *fail fast* (`Objects.requireNonNull`); não use exceção para controle de fluxo; **log OU relance**, não os dois; não devolva `null` para indicar erro.

**Para que serve**: separar o caminho feliz do tratamento de erro, propagar falhas até quem sabe resolvê-las e garantir a liberação de recursos.

**Exemplo**:

```java
class SaldoInsuficienteException extends Exception {
    SaldoInsuficienteException(String m) { super(m); }
}
void sacar(double v) throws SaldoInsuficienteException {
    if (v > saldo) throw new SaldoInsuficienteException("Saldo baixo");
}
try (BufferedReader br = new BufferedReader(new FileReader("a.txt"))) {
    System.out.println(br.readLine());
} catch (FileNotFoundException e) {
    System.out.println("Arquivo não existe");
} catch (IOException e) {
    throw new UncheckedIOException(e);
}
```

## 2.18 Pacotes e módulos

**O que é**: **pacotes** agrupam classes relacionadas e definem *namespaces* (`br.com.app.servico`); a estrutura de pastas espelha o nome. O acesso *package-private* e `protected` dependem do pacote. **Módulos** (JPMS, Java 9+) agrupam pacotes em unidades com `module-info.java`, declarando o que o módulo `requires` e o que `exports`, para encapsulamento forte e imagens de runtime menores (`jlink`).

**Para que serve**: organizar projetos grandes, evitar colisão de nomes e controlar o que é API pública. A maioria dos projetos de aplicação usa só pacotes (Maven/Gradle dependem do *classpath*).

**Exemplo**:

```java
// module-info.java
module meu.app {
    requires java.sql;
    exports br.com.app.api;
}
```

## 2.19 Classes seladas e pattern matching

**O que é**: **`sealed`** (Java 17) restringe quais classes/interfaces podem estender/implementar um tipo (`permits`); os herdeiros devem ser `final`, `sealed` ou `non-sealed`. Com `switch` e *pattern matching* (Java 21), o compilador verifica se todos os casos foram cobertos (**exaustividade**) e dispensa `default`. **Record patterns** desestruturam records.

**Para que serve**: modelar **tipos algébricos** (um valor é uma de poucas alternativas conhecidas) com segurança: resultado `Sucesso | Erro`, formas geométricas, eventos.

**Exemplo**:

```java
sealed interface Forma permits Circulo, Retangulo {}
record Circulo(double r) implements Forma {}
record Retangulo(double l, double a) implements Forma {}

double area(Forma f) {
    return switch (f) {
        case Circulo c -> Math.PI * c.r() * c.r();
        case Retangulo(double l, double a) -> l * a;
    };
}
```

---

# Parte 3: API padrão do Java

## 3.1 Visão geral do Collections Framework

**O que é**: conjunto de interfaces e classes (`java.util`) para guardar e manipular **grupos de objetos** de tamanho variável. Hierarquia:

```
Iterable
└── Collection
    ├── List   (ordenada, aceita duplicados, acesso por índice)
    ├── Set    (sem duplicados)
    └── Queue  (fila / prioridade)  →  Deque (fila de duas pontas)
Map  (chave → valor, chaves únicas; NÃO é uma Collection)
```

| Implementação | Estrutura interna | Ordem | Custo típico |
| --- | --- | --- | --- |
| `ArrayList` | array redimensionável | inserção | `get(i)` O(1); inserir/remover no meio O(n) |
| `LinkedList` | lista duplamente ligada | inserção | `get(i)` O(n); pontas O(1) |
| `HashSet` / `HashMap` | tabela hash | **sem garantia** | O(1) médio |
| `LinkedHashSet` / `LinkedHashMap` | hash + lista ligada | inserção | O(1) |
| `TreeSet` / `TreeMap` | árvore rubro-negra | **ordenada** | O(log n) |
| `PriorityQueue` | heap binário | por prioridade | `offer`/`poll` O(log n) |
| `ArrayDeque` | array circular | pontas | O(1) nas pontas |

- **Programe para a interface**: `List<String> l = new ArrayList<>();`. O diamante `<>` infere o tipo.
- Coleções guardam **objetos**: `List<int>` não compila, use `List<Integer>`.

**Para que serve**: substituir arrays quando o tamanho varia e oferecer estruturas prontas e otimizadas (busca, ordenação, unicidade, filas).

**Como escolher**: lista com índice → `ArrayList`; sem duplicados → `HashSet` (mantendo inserção `LinkedHashSet`, ordenado `TreeSet`); chave→valor → `HashMap`; fila de prioridade → `PriorityQueue`; pilha/fila → `ArrayDeque`; enum como chave → `EnumMap`.

**Exemplo**:

```java
List<String> nomes = new ArrayList<>();
nomes.add("Ana");
Set<Integer> unicos = new HashSet<>(List.of(1, 2, 3));
Map<String, Integer> idades = new HashMap<>();
idades.put("Ana", 30);
```

## 3.2 List

**O que é**: coleção **ordenada** (mantém a ordem de inserção), que **permite duplicados** e acesso por **índice**. `ArrayList` (array interno) é a escolha padrão; `LinkedList` só em uso intenso de inserção/remoção nas pontas.

- Operações: `add(e)`, `add(i, e)`, `get(i)`, `set(i, e)`, `remove(i)` (por **índice**) e `remove(obj)` (por **objeto**; em `List<Integer>`, `remove(1)` remove o índice 1), `size`, `contains`/`indexOf` (usam `equals`), `subList`, `sort`, `addAll`, `removeIf`.
- **Fábricas imutáveis**: `List.of(...)` e `List.copyOf` (sem `null`; alterar lança `UnsupportedOperationException`). `Arrays.asList(array)` é uma **visão de tamanho fixo** sobre o array (`set` funciona, `add`/`remove` não); para lista mutável: `new ArrayList<>(Arrays.asList(...))`.
- Remover durante um *for-each* lança `ConcurrentModificationException`: use `Iterator.remove()` ou `removeIf`.
- A capacidade inicial (`new ArrayList<>(100)`) é só otimização, não limita o tamanho.
- **Java 21**: `getFirst()`, `getLast()`, `reversed()` (interface `SequencedCollection`).

**Para que serve**: sequências onde a ordem importa e repetições são válidas (itens de um pedido, resultados de consulta).

**Exemplo**:

```java
List<String> l = new ArrayList<>(List.of("c", "a", "b"));
l.add("d");
Collections.sort(l);               // [a, b, c, d]
l.removeIf(s -> s.equals("b"));    // [a, c, d]
System.out.println(l.get(0) + l.size());
```

## 3.3 Set

**O que é**: coleção **sem elementos duplicados** e sem acesso por índice. O que é "duplicado" depende de `equals`/`hashCode` (`HashSet`) ou do `compareTo`/`Comparator` (`TreeSet`).

- **`HashSet`**: mais rápido, ordem imprevisível. **`LinkedHashSet`**: mantém a ordem de inserção. **`TreeSet`** (`NavigableSet`): sempre ordenado; exige `Comparable` ou `Comparator`.
- `add` devolve `boolean` (`false` se já existia).
- **Navegação** (`NavigableSet`): `lower` (<), `floor` (<=), `higher` (>), `ceiling` (>=), `first`, `last`, `pollFirst`, `headSet`, `tailSet`, `descendingSet`.
- **Cuidado no `TreeSet`**: quem decide a unicidade é o **comparator**, não o `equals`. Dois objetos que "empatam" na comparação (ex.: mesmo preço) são tratados como o mesmo elemento e o segundo **não é inserido**; desempate com `thenComparing`.
- Operações de conjunto: `addAll` (união), `retainAll` (interseção), `removeAll` (diferença), `containsAll`.

**Para que serve**: eliminar duplicatas, testar pertencimento rapidamente, operações de conjuntos.

**Exemplo**:

```java
Set<String> tags = new TreeSet<>(List.of("java", "spring", "java"));
System.out.println(tags);                  // [java, spring]
System.out.println(tags.contains("java")); // true
NavigableSet<Integer> n = new TreeSet<>(List.of(10, 20, 30));
System.out.println(n.floor(25));           // 20
```

## 3.4 Map

**O que é**: guarda pares **chave → valor**; a **chave é única** (inserir chave existente substitui o valor). Não implementa `Collection`.

- **`HashMap`** (sem ordem, aceita uma chave `null`), **`LinkedHashMap`** (ordem de inserção), **`TreeMap`** (`NavigableMap`, chaves ordenadas; `firstKey`, `floorKey`, `ceilingKey`, `headMap`, `tailMap`).
- Operações: `put`, `get` (devolve `null` se ausente), `getOrDefault`, `putIfAbsent`, `computeIfAbsent`, `merge` (útil para contadores), `remove`, `containsKey`, `forEach`.
- Três visões: `keySet()`, `values()`, `entrySet()`. Use **`entrySet()`** quando precisar de chave e valor.
- **Chaves objeto exigem `equals`/`hashCode` corretos e, de preferência, imutáveis.**
- `Map.of(...)`/`Map.copyOf` criam mapas imutáveis. `HashMap` não é thread-safe: use `ConcurrentHashMap` (tópico 4.5).

**Para que serve**: buscas por identificador, contagem de ocorrências, agrupamento, caches, tabelas de configuração.

**Exemplo**:

```java
Map<String, Integer> cont = new HashMap<>();
for (String p : "a b a c a b".split(" ")) {
    cont.merge(p, 1, Integer::sum);
}
System.out.println(cont);                  // {a=3, b=2, c=1}
for (Map.Entry<String, Integer> e : cont.entrySet()) {
    System.out.println(e.getKey() + " -> " + e.getValue());
}
Map<String, List<String>> grupos = new HashMap<>();
grupos.computeIfAbsent("x", k -> new ArrayList<>()).add("1");
```

## 3.5 Queue e Deque

**O que é**: **`Queue`** é uma fila (normalmente FIFO: primeiro a entrar, primeiro a sair). **`Deque`** (fila de duas pontas) permite inserir/remover nos dois lados e serve como **pilha** (LIFO).

| Operação | Lança exceção | Devolve valor especial |
| --- | --- | --- |
| Inserir | `add(e)` | `offer(e)` |
| Remover cabeça | `remove()` | `poll()` (`null` se vazia) |
| Ver cabeça | `element()` | `peek()` |

- **`PriorityQueue`** **não é FIFO**: `poll` devolve o elemento de **maior prioridade** (menor pela ordem natural ou pelo `Comparator`). A iteração não sai ordenada; só `poll` garante a ordem. Não aceita `null`.
- **`ArrayDeque`**: implementação recomendada para pilha (`push/pop`) e fila (`offer/poll`); substitui `Stack` e, em geral, `LinkedList`.
- Filas **bloqueantes** para threads (`BlockingQueue`): tópico 4.5.

**Para que serve**: processar elementos por ordem de chegada ou prioridade (tarefas, eventos), pilhas de desfazer, buscas em largura/profundidade.

**Exemplo**:

```java
Queue<String> fila = new ArrayDeque<>();
fila.offer("a"); fila.offer("b");
System.out.println(fila.poll());   // a

Queue<Integer> pq = new PriorityQueue<>(Comparator.reverseOrder());
pq.addAll(List.of(3, 9, 1));
System.out.println(pq.poll());     // 9

Deque<Integer> pilha = new ArrayDeque<>();
pilha.push(1); pilha.push(2);
System.out.println(pilha.pop());   // 2
```

## 3.6 Comparable, Comparator, Iterator e utilitários

**O que é**:

- **`Comparable<T>`**: define a ordem **natural** dentro da própria classe: `int compareTo(T outro)` devolve negativo (this menor), 0 (igual) ou positivo. Usado por `Collections.sort`, `TreeSet`, `TreeMap`, `PriorityQueue`. Ideal manter consistente com `equals`.
- **`Comparator<T>`**: critério de ordenação **externo** (`int compare(a, b)`). Construção moderna: `Comparator.comparing(Pessoa::nome).thenComparing(Pessoa::idade).reversed()`. Para primitivos use `Integer.compare`/`Double.compare` (subtração pode estourar).
- **`Iterator<T>`**: percorre uma coleção (`hasNext`, `next`, `remove`). O *for-each* usa um por baixo dos panos.
- **`Collections`**: `sort`, `reverse`, `shuffle`, `max`, `min`, `frequency`, `binarySearch` (**lista ordenada** com o mesmo critério; se não achar devolve `-(ponto de inserção) - 1`), `unmodifiableList`, `emptyList`, `swap`.
- **Imutável x visão**: `Collections.unmodifiableList(l)` é só uma **visão** (acompanha mudanças em `l`); `List.copyOf(l)` é uma cópia imutável.

**Para que serve**: ordenar, buscar e percorrer coleções de objetos com critérios variados, sem alterar a classe dos elementos.

**Exemplo**:

```java
record Pessoa(String nome, int idade) {}
List<Pessoa> ps = new ArrayList<>(List.of(new Pessoa("Bia", 30), new Pessoa("Ana", 30)));
ps.sort(Comparator.comparing(Pessoa::idade).thenComparing(Pessoa::nome));
Iterator<Pessoa> it = ps.iterator();
while (it.hasNext()) if (it.next().idade() > 40) it.remove();
```

## 3.7 Números precisos: BigDecimal, BigInteger e Math

**O que é**: `float`/`double` usam ponto flutuante binário e **não representam decimais exatamente** (`0.1 + 0.2 = 0.30000000000000004`). **`BigDecimal`** representa decimais com **precisão arbitrária e escala controlada**; **`BigInteger`** representa inteiros **sem limite**. Ambos são **imutáveis** (cada operação devolve novo objeto).

- Crie `BigDecimal` a partir de **String** (`new BigDecimal("0.1")` ou `BigDecimal.valueOf(0.1)`), nunca de `double` direto.
- Operações: `add`, `subtract`, `multiply`, `divide`, `compareTo` (use em vez de `equals`, que considera a escala: `2.0` ≠ `2.00`), `setScale(2, RoundingMode.HALF_UP)`.
- `divide` sem escala/arredondamento lança `ArithmeticException` em dízimas; informe `scale` e `RoundingMode` (ou `MathContext`).
- **`Math`**: `abs`, `max`, `min`, `pow`, `sqrt`, `round`, `floor`, `ceil`, `random`, `addExact` (estouro lança exceção).

**Para que serve**: dinheiro, impostos e qualquer cálculo decimal que exige exatidão; fatoriais e criptografia com inteiros gigantes.

**Exemplo**:

```java
BigDecimal preco = new BigDecimal("19.90");
BigDecimal total = preco.multiply(BigDecimal.valueOf(3)).setScale(2, RoundingMode.HALF_UP);
System.out.println(total);                    // 59.70
BigInteger f = BigInteger.ONE;
for (int i = 2; i <= 25; i++) f = f.multiply(BigInteger.valueOf(i));
```

## 3.8 Generics

**O que é**: parametrizar classes, interfaces e métodos com **tipos**, movendo a checagem de tipos para a **compilação** e eliminando *casts*. Antes dos generics, coleções guardavam `Object` e o erro só aparecia em execução (`ClassCastException`).

- **Classe genérica**: `class Caixa<T> { ... }`. Convenção: `T` (type), `E` (element), `K`/`V` (key/value), `R` (return).
- **Método genérico**: `<T> List<T> listaDe(T x)`; o `<T>` antes do retorno declara o parâmetro do método; o compilador infere `T`.
- **Tipos limitados**: `<T extends Comparable<T>>` (também vale para interfaces; vários limites com `&`).
- **Invariância**: `List<Cachorro>` **não** é `List<Animal>`, mesmo que `Cachorro extends Animal`. (Arrays são covariantes e por isso inseguros: `ArrayStoreException`.)
- **Wildcards**: `?` qualquer tipo; **`? extends T`** (produtor: você só **lê**); **`? super T`** (consumidor: você só **escreve**). Regra **PECS** (*Producer Extends, Consumer Super*).
- **Type erasure**: os parâmetros de tipo são apagados na compilação. Consequências: não existe `new T()`, `new T[10]`, `instanceof List<String>`, nem sobrecarga que só difere no tipo genérico; tipos primitivos não podem ser argumento (`List<int>`); campos `static` não usam o `T` da classe. *Raw types* (`List` sem `<>`) desligam a checagem: evite.

**Para que serve**: reutilizar código para qualquer tipo com segurança (coleções, repositórios genéricos, utilitários como `filter`/`map`).

**Exemplo**:

```java
class Caixa<T> {
    private T item;
    void guardar(T item) { this.item = item; }
    T pegar() { return item; }
}
static <T extends Comparable<T>> T maior(List<T> l) {
    T m = l.get(0);
    for (T t : l) if (t.compareTo(m) > 0) m = t;
    return m;
}
static void imprimir(List<? extends Number> nums) {      // só leitura
    for (Number n : nums) System.out.println(n);
}
static void preencher(List<? super Integer> destino) {   // só escrita
    destino.add(1);
}
```

## 3.9 Datas e horas (`java.time`)

**O que é**: API moderna (Java 8) para datas e horas, com classes **imutáveis** e thread-safe (toda operação devolve **novo objeto**). Substitui `Date` e `Calendar` (legado: mutáveis, meses de `Calendar` começam em 0, nomes enganosos).

| Classe | Representa |
| --- | --- |
| `LocalDate` | data sem hora e sem fuso (`2026-10-24`) |
| `LocalTime` | hora sem data (`14:30`) |
| `LocalDateTime` | data + hora, sem fuso |
| `Instant` | ponto exato na linha do tempo (UTC), contado desde a *epoch* 1970-01-01 |
| `ZonedDateTime` | data-hora + `ZoneId` (regras de horário de verão) |
| `OffsetDateTime` | data-hora + deslocamento fixo (`-03:00`) |
| `Duration` | quantidade de tempo em horas/min/s/ns |
| `Period` | quantidade em anos/meses/dias |
| `ZoneId` / `ZoneOffset` | fuso por região (`America/Sao_Paulo`) / desvio fixo |
| `DayOfWeek`, `Month`, `Year`, `YearMonth` | enums/tipos auxiliares |

- **`Duration` x `Period`**: `Duration` é tempo de relógio; `Period` é tempo de calendário (um mês tem tamanho variável). `Duration.between` com `LocalDate` lança `UnsupportedTemporalTypeException`.
- **`ChronoUnit.DAYS.between(a, b)`** devolve a diferença inteira (truncada) numa unidade. `ChronoField` acessa campos.
- **`TemporalAdjusters`** (`next(DayOfWeek.MONDAY)`, `firstDayOfMonth()`, `lastDayOfMonth()`) e `TemporalAdjuster` customizado (`data.with(adjuster)`).
- **Boas práticas**: armazene instantes em **UTC** (`Instant`) e converta para o fuso do usuário só na borda; para testes, injete um `Clock` em vez de chamar `now()` direto; nunca monte datas por concatenação de String.

**Para que serve**: aniversários, vencimentos, agendamentos, medição de intervalos, timestamps de auditoria e conversões entre fusos.

**Exemplo**:

```java
LocalDate hoje = LocalDate.now();
LocalDate venc = hoje.plusDays(30).withDayOfMonth(10);
long dias = ChronoUnit.DAYS.between(hoje, venc);
Period idade = Period.between(LocalDate.of(1998, 8, 6), hoje);
ZonedDateTime tokyo = Instant.now().atZone(ZoneId.of("Asia/Tokyo"));
Duration d = Duration.ofMinutes(90);        // PT1H30M
System.out.println(hoje.getDayOfWeek() + " " + dias + " " + idade.getYears());
```

## 3.10 Formatação e internacionalização

**O que é**: adaptar a **exibição** de números, moedas, datas e textos às convenções de uma região.

- **`Locale`**: idioma + país (`pt-BR`, `en-US`). O construtor `new Locale("pt", "BR")` está depreciado (Java 19+): use `Locale.of("pt", "BR")` ou `Locale.forLanguageTag("pt-BR")`. Informe sempre o `Locale` explicitamente (o padrão da JVM varia por máquina).
- **`NumberFormat`**: `getInstance`, `getCurrencyInstance`, `getPercentInstance`; `setMaximumFractionDigits`. `1.234,56` (pt-BR) x `1,234.56` (en-US). `DecimalFormat` aceita padrão (`"#,##0.00"`).
- **`DateTimeFormatter`** (moderno, thread-safe): constantes ISO (`ISO_LOCAL_DATE`) e `ofPattern("dd/MM/yyyy", locale)`; formata (`data.format(f)`) e **lê** (`LocalDate.parse(txt, f)`). Estilos localizados: `ofLocalizedDate(FormatStyle.LONG)`.
- Letras do padrão: `yyyy` ano (**`YYYY` é ano da semana, bug clássico**), `MM` mês (**`mm` é minuto**), `dd` dia, `HH` hora 0-23 (`hh` é 1-12), `mm` minuto, `ss` segundo, `MMMM` nome do mês (depende do `Locale`).
- **`SimpleDateFormat`/`DateFormat`** (legado): **não são thread-safe**.
- **`String.format`/`printf`**, `MessageFormat` (textos com `{0}`) e **`ResourceBundle`** (arquivos `.properties` por idioma: `mensagens_pt_BR.properties`) para internacionalizar mensagens.

**Para que serve**: exibir valores no formato esperado pelo usuário e traduzir interfaces/mensagens.

**Exemplo**:

```java
Locale br = Locale.of("pt", "BR");
System.out.println(NumberFormat.getCurrencyInstance(br).format(1500.5));   // R$ 1.500,50
DateTimeFormatter f = DateTimeFormatter.ofPattern("dd/MM/yyyy");
System.out.println(LocalDate.of(2026, 10, 24).format(f));                 // 24/10/2026
LocalDate d = LocalDate.parse("18/08/2026", f);
```

## 3.11 Expressões regulares (regex)

**O que é**: linguagem para descrever **padrões de texto**. `java.util.regex`: **`Pattern`** compila a expressão (operação cara: guarde em `static final`); **`Matcher`** aplica sobre um texto.

- **`matches()`**: o texto **inteiro** casa. **`find()`**: procura a **próxima ocorrência** (pegadinha: um texto pode falhar em `matches` e ser encontrado por `find`). **`lookingAt()`**: casa no início. Depois: `group()`, `group(n)`, `start()`, `end()`, `replaceAll`.
- **Escape duplo**: a regex `\d` em Java é `"\\d"`.

| Elemento | Significado |
| --- | --- |
| `.` | qualquer caractere (menos quebra de linha) |
| `\d` `\D` | dígito / não dígito |
| `\w` `\W` | `[a-zA-Z0-9_]` / o resto |
| `\s` `\S` | espaço em branco / não branco |
| `[abc]` `[a-z]` `[^a-z]` | conjunto, intervalo, negação |
| `?` `*` `+` | 0 ou 1 / 0 ou mais / 1 ou mais |
| `{n}` `{n,m}` | exatamente n / de n a m |
| `( )` `\|` | grupo (captura) / alternância |
| `^` `$` | início / fim |

- Quantificadores são **gananciosos** (*greedy*); `?` depois (`.*?`) torna-os mínimos.
- `String.matches`, `replaceAll` e `split` aceitam regex (e recompilam o padrão a cada chamada).
- Cuidado com **ReDoS** (repetições aninhadas travam a CPU). Para validar e-mail, prefira Bean Validation (`@Email`) a regex caseira.
- `Scanner.useDelimiter(",")` também aceita regex.

**Para que serve**: validar formatos (CPF, CEP, telefone), extrair partes de texto, limpar/substituir conteúdo, dividir strings complexas.

**Exemplo**:

```java
Pattern p = Pattern.compile("(\\d{3})-(\\d{4})");
Matcher m = p.matcher("Ligue 123-4567 agora");
if (m.find()) {
    System.out.println(m.group());    // 123-4567
    System.out.println(m.group(2));   // 4567
}
System.out.println("a1b22c".replaceAll("\\d+", "#"));   // a#b#c
System.out.println("12345".matches("\\d+"));            // true
```

## 3.12 Arquivos e I/O clássico (`java.io`)

**O que é**: leitura e escrita de dados em **fluxos** (*streams*). Dois grupos: **bytes** (`InputStream`/`OutputStream`, para binários) e **caracteres** (`Reader`/`Writer`, para texto).

- **`File`**: representa um **caminho** e seus metadados, não o conteúdo: `exists`, `isFile`, `isDirectory`, `length`, `lastModified`, `delete`, `createNewFile` (devolve `boolean`), `mkdir`/`mkdirs`, `renameTo`, `list`.
- **`FileReader`/`FileWriter`**: texto, caractere a caractere. `new FileWriter(arquivo, true)`: o segundo argumento é **append** (senão o arquivo é sobrescrito). `read()` devolve `int` (`-1` = fim).
- **`BufferedReader`/`BufferedWriter`**: guardam dados em memória e reduzem acessos ao disco; `readLine()` (devolve `null` no fim), `newLine()`, `flush()` (o `close()` já faz).
- **Decorator**: os streams se encadeiam (`new BufferedReader(new FileReader(f))`). Outros: `PrintWriter`, `DataInputStream`, `ObjectOutputStream`, `ByteArrayInputStream`.
- Sempre use **try-with-resources**. Explicite o *charset* (`StandardCharsets.UTF_8`).
- Portabilidade: evite `\\` em caminhos (só Windows); use `/`, `File.separator` ou `Path.of(...)`.

**Para que serve**: ler/gravar arquivos de texto e binários, logs, importação/exportação de dados.

**Exemplo**:

```java
try (BufferedWriter bw = new BufferedWriter(new FileWriter("saida.txt", true))) {
    bw.write("linha 1");
    bw.newLine();
}
try (BufferedReader br = new BufferedReader(new FileReader("saida.txt"))) {
    String linha;
    while ((linha = br.readLine()) != null) System.out.println(linha);
} catch (IOException e) {
    e.printStackTrace();
}
```

## 3.13 NIO.2 (`java.nio.file`)

**O que é**: API moderna de arquivos: **`Path`** (caminho; `Path.of(...)`) e **`Files`** (operações), separando as responsabilidades que `File` misturava.

- **`Path`**: `getFileName`, `getParent`, `toAbsolutePath`, `normalize()` (remove `.` e `..`), `resolve(outro)` (concatena; se `outro` é absoluto devolve ele), `relativize(outro)` (caminho de um até o outro).
- **`Files`**: `createFile`, `createDirectory/ies`, `copy`/`move` (com `StandardCopyOption.REPLACE_EXISTING`), `delete`/`deleteIfExists`, `exists`, `readAllLines`, `readString`, `writeString`, `lines` (Stream; feche!), `list`, `walk`, `size`.
- **Atributos**: `BasicFileAttributes` (tamanho, datas; todo SO), `DosFileAttributes` (Windows: oculto, somente leitura), `PosixFileAttributes` (Unix: dono, grupo, permissões `rwxr-xr-x`); `Files.getFileAttributeView` também **altera**.
- **Percorrer diretórios**: `DirectoryStream` (**um nível**) e `Files.walkFileTree` com `SimpleFileVisitor` (**recursivo**, com callbacks `preVisitDirectory`, `visitFile`, `postVisitDirectory`).
- **`PathMatcher`** com **glob**: `*.java`, `**/*.txt`, `*.{bkp,txt}`.
- **`WatchService`** observa mudanças em pastas. **`ZipOutputStream`**/`ZipFile` criam e leem `.zip`.

**Para que serve**: manipular arquivos e pastas de forma concisa, portátil e com erros claros (exceções específicas).

**Exemplo**:

```java
Path dir = Path.of("dados", "sub");
Files.createDirectories(dir);
Path arq = dir.resolve("a.txt");
Files.writeString(arq, "olá\nmundo");
List<String> linhas = Files.readAllLines(arq);
try (Stream<Path> s = Files.walk(Path.of("dados"))) {
    s.filter(p -> p.toString().endsWith(".txt")).forEach(System.out::println);
}
Files.copy(arq, dir.resolve("b.txt"), StandardCopyOption.REPLACE_EXISTING);
```

## 3.14 Serialização

**O que é**: transformar o estado de um objeto em **bytes** (`ObjectOutputStream.writeObject`) e reconstruí-lo depois (`ObjectInputStream.readObject`). A classe implementa a interface marcadora **`Serializable`**; todos os campos (exceto `transient` e `static`) também devem ser serializáveis, senão `NotSerializableException`.

- **`serialVersionUID`**: versão da classe; se não declarado, é calculado pela estrutura e qualquer mudança causa `InvalidClassException`. Declare explicitamente.
- **`transient`**: campo ignorado (volta com o valor padrão): senhas, caches, conexões. Campos `static` não são gravados.
- **Serialização customizada**: métodos privados `writeObject`/`readObject` (com `defaultWriteObject()`/`defaultReadObject()`); `Externalizable` dá controle total.
- Se a superclasse **não** é `Serializable`, precisa de construtor sem argumentos acessível.
- **Segurança**: desserializar dados não confiáveis é vetor clássico de ataque; use `ObjectInputFilter`. Para APIs e persistência, prefira **JSON** (Jackson) ou formatos explícitos (Protobuf).

**Para que serve**: salvar/transmitir objetos (cache em disco, sessões distribuídas, RMI). Não use como formato de armazenamento de longo prazo.

**Exemplo**:

```java
class Aluno implements Serializable {
    @Serial private static final long serialVersionUID = 1L;
    String nome;
    transient String senha;      // não é gravada
}
try (ObjectOutputStream out = new ObjectOutputStream(Files.newOutputStream(Path.of("a.ser")))) {
    out.writeObject(aluno);
}
try (ObjectInputStream in = new ObjectInputStream(Files.newInputStream(Path.of("a.ser")))) {
    Aluno lido = (Aluno) in.readObject();
}
```

## 3.15 Reflection

**O que é**: capacidade de **inspecionar e manipular classes, métodos, campos e construtores em tempo de execução**, mesmo sem conhecê-los em compilação. Ponto de entrada: `Class<?>` (`obj.getClass()`, `Pessoa.class`, `Class.forName("...")`).

- Inspeção: `getDeclaredFields()`, `getDeclaredMethods()`, `getDeclaredConstructors()`, `getModifiers()`, `getAnnotation(...)`, `getInterfaces()`, `getSuperclass()`.
- Uso: `constructor.newInstance(...)`, `method.invoke(obj, args)`, `field.get/set` (com `setAccessible(true)` para quebrar `private`; restrito por módulos em Java 17+).
- **`java.lang.reflect.Proxy`**: cria objetos que implementam interfaces em execução (base de AOP, `@Transactional`, Feign, repositórios Spring Data).
- **Custos**: mais lento, sem checagem em compilação, quebra encapsulamento, exige configuração em GraalVM *native image*.

**Para que serve**: é o alicerce de **frameworks**: Spring (injeção e *scanning*), JPA/Hibernate (mapear campos), Jackson (JSON), JUnit (descobrir `@Test`). No dia a dia, você raramente precisa escrever reflection.

**Exemplo**:

```java
Class<?> c = Pessoa.class;
for (Field f : c.getDeclaredFields()) System.out.println(f.getName() + " " + f.getType());
Object p = c.getDeclaredConstructor(String.class).newInstance("Ana");
Method m = c.getMethod("getNome");
System.out.println(m.invoke(p));
```

## 3.16 Annotations e Annotation Processing

**O que é**: **anotações** são metadados no código (`@Nome`) que não mudam a lógica diretamente, mas são lidas por compilador, ferramentas e frameworks. Declaradas com `@interface`.

- **Embutidas**: `@Override`, `@Deprecated`, `@SuppressWarnings`, `@FunctionalInterface`, `@SafeVarargs`.
- **Meta-anotações**: `@Retention` (`SOURCE` só no fonte, `CLASS` no `.class` (padrão), `RUNTIME` acessível por reflection), `@Target` (`TYPE`, `FIELD`, `METHOD`, `PARAMETER`...), `@Documented`, `@Inherited`, `@Repeatable`.
- Podem ter **elementos** (`String value() default "";`).
- **Annotation Processing (JSR 269)**: o `javac` chama **processadores** (`AbstractProcessor`) **durante a compilação** para **gerar novos arquivos** `.java` (nunca alterar os existentes, exceto hacks como o Lombok). Atua em *build-time*, com custo zero em execução. Registro via `@AutoService` ou `META-INF/services/javax.annotation.processing.Processor`. Geração de código com **JavaPoet**. Exemplos: Lombok, MapStruct, Dagger.
- Reflection (runtime) x Annotation Processing (compilação): o segundo valida e gera código antes de executar e funciona bem com GraalVM.

**Para que serve**: declarar intenção (`@Entity`, `@Test`, `@GetMapping`), validar regras (`@NotNull`), gerar código repetitivo (builders, getters, mappers).

**Exemplo**:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Auditar { String motivo() default ""; }

class Servico {
    @Auditar(motivo = "saque")
    void sacar() { }
}
Method m = Servico.class.getDeclaredMethod("sacar");
if (m.isAnnotationPresent(Auditar.class))
    System.out.println(m.getAnnotation(Auditar.class).motivo());   // saque
```

## 3.17 Logging

**O que é**: registrar eventos da aplicação (diagnóstico, auditoria, erros) com **níveis** e destinos configuráveis, em vez de `System.out`/`printStackTrace`.

- **Níveis**: `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR` (e `FATAL` em alguns). Em produção costuma-se `INFO`; `DEBUG` para investigar.
- **Implementações**: `java.util.logging` (JDK), **Logback**, **Log4j2**. **SLF4J** é a **fachada** padrão: o código usa a API e a biblioteca de log é trocável. Lombok oferece `@Slf4j`/`@Log4j2`.
- **Mensagem parametrizada** (`log.info("Usuário {} logou", id)`) evita concatenação desnecessária. Para registrar o *stack trace*, passe a exceção como **último** argumento (`log.error("Falha ao salvar {}", id, e)`).
- **Appenders** definem o destino (console, arquivo com rotação, rede); o **layout/pattern** o formato. **MDC** adiciona contexto (id da requisição).
- Boas práticas: não registre dados sensíveis (senhas, tokens, cartão); não use log **e** `throw` na mesma camada; use nível adequado; evite logar em laços quentes.

**Para que serve**: depurar produção, rastrear incidentes, auditoria e métricas.

**Exemplo**:

```java
private static final Logger log = LoggerFactory.getLogger(PedidoService.class);

void salvar(Pedido p) {
    log.info("Salvando pedido {}", p.getId());
    try { repo.save(p); }
    catch (Exception e) { log.error("Erro ao salvar pedido {}", p.getId(), e); throw e; }
}
```

## 3.18 Classes utilitárias úteis

**O que é**: classes pequenas e muito usadas da biblioteca padrão.

- **`Objects`**: `requireNonNull(x, msg)`, `equals`, `hash`, `hashCode`, `toString(x, padrão)`, `isNull`, `requireNonNullElse`.
- **`UUID`**: identificador universal (`UUID.randomUUID()`), usado como chave primária e id de correlação.
- **Aleatórios**: `java.util.Random`, `ThreadLocalRandom.current().nextInt(1, 100)` (melhor em multithread), `SecureRandom` (criptográfico: senhas/tokens).
- **`System`**: `currentTimeMillis`, `nanoTime` (medir intervalos), `getProperty`, `getenv` (variáveis de ambiente), `exit`, `lineSeparator`. **`Runtime`**: memória e processadores (`availableProcessors`).
- **`Properties`**: carrega `chave=valor` de arquivos `.properties`.
- **`StringJoiner`**, **`Optional`** (tópico 4.3), **`TimeUnit`** (`TimeUnit.SECONDS.sleep(2)`), **`Timer`** (legado; use `ScheduledExecutorService`).

**Para que serve**: validações rápidas, geração de ids e números aleatórios, leitura de configuração, medição de desempenho.

**Exemplo**:

```java
Objects.requireNonNull(nome, "nome é obrigatório");
String id = UUID.randomUUID().toString();
int dado = ThreadLocalRandom.current().nextInt(1, 7);
String env = System.getenv().getOrDefault("AMBIENTE", "dev");
Properties props = new Properties();
try (InputStream in = Files.newInputStream(Path.of("app.properties"))) { props.load(in); }
long t0 = System.nanoTime();
long ms = (System.nanoTime() - t0) / 1_000_000;
```

---

# Parte 4: Java avançado

## 4.1 Lambdas e interfaces funcionais

**O que é**: uma **interface funcional** tem **exatamente um método abstrato** (pode ter `default`/`static`); `@FunctionalInterface` faz o compilador garantir isso. Uma **expressão lambda** é uma forma compacta de implementar essa interface sem criar classe: `(parâmetros) -> corpo`. O compilador infere a interface e os tipos pelo contexto.

```
(a, b) -> a + b                 // expressão: retorno implícito
s -> s.length()                 // um parâmetro: parênteses opcionais
() -> new ArrayList<String>()   // sem parâmetros
(x, y) -> { int s = x + y; return s; }   // bloco: return explícito
```

Interfaces prontas em `java.util.function`:

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

- Versões primitivas evitam *boxing*: `IntPredicate`, `ToIntFunction`, `IntBinaryOperator`...
- **Composição**: `p1.and(p2)`, `p1.or(p2)`, `p.negate()`, `f.andThen(g)`, `f.compose(g)`, `Function.identity()`.
- **Captura**: a lambda só usa variáveis locais **finais ou efetivamente finais**. Dentro dela, `this` é a classe que a contém (diferente da classe anônima).
- **Parametrizar comportamento**: em vez de um método por critério (`filtrarVerdes`, `filtrarVermelhos`), receba um `Predicate<T>` e passe a regra como argumento. É o princípio por trás das Streams e do padrão Strategy.

**Para que serve**: escrever código mais curto e declarativo, passar comportamento como dado (callbacks, filtros, ordenações), base de Streams, `CompletableFuture` e frameworks.

**Exemplo**:

```java
static <T> List<T> filtrar(List<T> lista, Predicate<T> regra) {
    List<T> r = new ArrayList<>();
    for (T e : lista) if (regra.test(e)) r.add(e);
    return r;
}
List<Integer> pares = filtrar(List.of(1, 2, 3, 4), n -> n % 2 == 0);   // [2, 4]
Function<String, Integer> tam = String::length;
Predicate<String> longo = s -> s.length() > 3;
System.out.println(longo.and(s -> s.startsWith("j")).test("java"));    // true
```

## 4.2 Method references

**O que é**: atalho `::` para uma lambda que **apenas chama um método já existente**. Não muda o comportamento, só remove ruído. Quatro categorias:

| Categoria | Sintaxe | Equivale a |
| --- | --- | --- |
| Método **estático** | `Integer::parseInt` | `s -> Integer.parseInt(s)` |
| Método de **objeto específico** | `System.out::println`, `comparador::comparar` | `x -> System.out.println(x)` |
| Método de **objeto arbitrário de um tipo** | `String::compareTo`, `String::toUpperCase`, `List::contains` | `(a, b) -> a.compareTo(b)` (o **1º parâmetro vira o receptor**) |
| **Construtor** | `ArrayList::new`, `Anime::new`, `int[]::new` | `() -> new ArrayList<>()` |

- A interface funcional de destino decide qual construtor/sobrecarga é usado (`Supplier<X> s = X::new`; `BiFunction<String,Integer,X> f = X::new`).
- Se há lógica extra (condição, conversão), mantenha a lambda.

**Para que serve**: deixar pipelines e ordenações mais legíveis (`.map(Pessoa::getNome)`, `.forEach(System.out::println)`).

**Exemplo**:

```java
List<String> nomes = new ArrayList<>(List.of("rui", "ana"));
nomes.sort(String::compareTo);                       // 3: objeto arbitrário
nomes.forEach(System.out::println);                  // 2: objeto específico
Function<String, Integer> conv = Integer::parseInt;  // 1: estático
Supplier<List<String>> novaLista = ArrayList::new;   // 4: construtor
```

## 4.3 Optional

**O que é**: contêiner que **pode ou não** ter um valor. Torna a ausência **explícita no tipo de retorno**, em vez de devolver `null` (invisível na assinatura, fonte de `NullPointerException`).

- **Criar**: `Optional.of(x)` (NPE se `x` for `null`), `Optional.ofNullable(x)`, `Optional.empty()`.
- **Consumir**: `ifPresent(consumer)`, `ifPresentOrElse`, `orElse(valor)` (**sempre avalia** o argumento), `orElseGet(supplier)` (só executa se vazio; use quando criar o padrão custa), `orElseThrow(...)` (ausência é erro), `isPresent`/`isEmpty`.
- **Transformar**: `map`, `flatMap` (quando a função já devolve `Optional`; evita `Optional<Optional<T>>`), `filter`, `or`, `stream()`.
- **`get()` sem checar** lança `NoSuchElementException`: prefira `orElseThrow()`.
- **Quando NÃO usar**: campos de classe, parâmetros de método, elementos de coleção, e no lugar de coleções (devolva lista vazia). Foi projetado para **tipo de retorno**.
- Variantes primitivas: `OptionalInt`, `OptionalDouble`, `OptionalLong`.

**Para que serve**: métodos de busca que podem não achar resultado (`findById`), evitando verificações de `null` espalhadas.

**Exemplo**:

```java
Optional<String> achado = buscarNome(7);
String nome = achado.map(String::toUpperCase).orElse("DESCONHECIDO");
achado.ifPresent(n -> System.out.println(n));
String obrigatorio = buscarNome(7).orElseThrow(() -> new IllegalStateException("Não achei"));
```

## 4.4 Streams

**O que é**: **pipeline** declarativo para processar sequências de elementos. A Stream **não guarda dados nem altera a fonte**, é **preguiçosa** (*lazy*) e de **uso único**. Estrutura: **fonte** → **operações intermediárias** (devolvem outra Stream) → **operação terminal** (dispara o processamento e produz o resultado). Sem terminal, nada executa; `limit`/`findFirst` fazem **curto-circuito**.

- **Fontes**: `coleção.stream()`, `Arrays.stream(arr)`, `Stream.of(...)`, `IntStream.range/rangeClosed`, `Files.lines(path)` (fechar!), `Stream.iterate`, `Stream.generate`.
- **Intermediárias**: `filter`, `map`, `flatMap` (1:N e **achata** `Stream<List<X>>` em `Stream<X>`), `distinct` (usa `equals`/`hashCode`), `sorted`, `limit`, `skip`, `peek`, `takeWhile`/`dropWhile`, `mapToInt/Long/Double`, `boxed`.
- **Terminais**: `forEach`, `collect`, `toList()` (imutável), `toArray`, `count`, `min`/`max`, `reduce`, `anyMatch`/`allMatch`/`noneMatch`, `findFirst`/`findAny` (devolvem `Optional`).
- **`reduce`**: combina tudo num valor; o operador deve ser **associativo**; com identidade (`reduce(0, Integer::sum)`) devolve o tipo direto, sem identidade devolve `Optional`.
- **Streams primitivas** (`IntStream`, `LongStream`, `DoubleStream`): sem *boxing*; `sum`, `average` (`OptionalDouble`), `summaryStatistics`. `range` exclui o fim, `rangeClosed` inclui.
- **Infinitas**: `Stream.iterate(1, n -> n * 2)`, `Stream.generate(supplier)`; exigem `limit`.
- **`Collectors`**: `toList`, `toSet`, `toMap(chave, valor[, merge])` (**chave duplicada sem `merge` lança `IllegalStateException`**), `joining`, `counting`, `summingX`, `averagingX`, `summarizingX`, `maxBy`/`minBy`, `mapping`, `collectingAndThen`, `teeing`, `toCollection`.
- **Agrupamento**: `groupingBy(classificador)` → `Map<Chave, List<T>>`; aceita **downstream collector** (`groupingBy(f, counting())`) e é aninhável (multinível); `partitioningBy(predicado)` separa em `true`/`false`.
- **Paralelas**: `.parallel()` divide o trabalho no *ForkJoin common pool*; só compensa com muitos dados, custo alto por elemento, fonte fácil de dividir (`ArrayList`, `range`) e operações **sem estado e sem efeitos colaterais**. Meça antes de usar.
- **Boas práticas**: não altere a fonte nem variáveis externas dentro do pipeline; `filter` antes de `map`/`sorted` caros; não use stream onde um laço simples é mais claro.

**Para que serve**: filtrar, transformar, agrupar e agregar dados de coleções de forma legível e concisa.

**Exemplo**:

```java
record Livro(String titulo, double preco, String categoria) {}
List<Livro> livros = List.of(new Livro("A", 10, "X"), new Livro("B", 30, "X"), new Livro("C", 20, "Y"));

List<String> baratos = livros.stream()
        .filter(l -> l.preco() < 25)
        .sorted(Comparator.comparing(Livro::titulo))
        .map(Livro::titulo)
        .toList();                                                    // [A, C]
Map<String, Long> porCategoria = livros.stream()
        .collect(Collectors.groupingBy(Livro::categoria, Collectors.counting()));   // {X=2, Y=1}
double soma = livros.stream().mapToDouble(Livro::preco).sum();       // 60.0
int somaPares = IntStream.rangeClosed(1, 10).filter(n -> n % 2 == 0).sum();   // 30
```

## 4.5 Concorrência: threads, sincronização e coleções concorrentes

**O que é**: executar várias tarefas **ao mesmo tempo** (ou intercaladas). Uma **thread** é uma linha de execução independente dentro do processo; todas compartilham o **heap** (por isso há risco) e cada uma tem sua **pilha**. Todo programa tem a thread `main`. *Concorrência* é lidar com várias coisas ao mesmo tempo; *paralelismo* é executá-las simultaneamente em vários núcleos.

**Criar threads**: implemente `Runnable` (preferido: separa a tarefa de quem a executa) ou estenda `Thread`.

- **`start()` x `run()`**: `start()` cria uma nova thread e executa `run()` nela. Chamar `run()` direto roda na thread atual, **sem concorrência nenhuma**.
- **Estados** (`Thread.State`): `NEW` → `RUNNABLE` → `BLOCKED` (esperando um monitor), `WAITING`/`TIMED_WAITING` (`wait`, `sleep`, `join`) → `TERMINATED`.
- `Thread.sleep(ms)` pausa a atual (lança `InterruptedException`). **`join()`** espera outra thread terminar. **`interrupt()`** pede cancelamento **cooperativo**; nunca engula `InterruptedException` (restaure com `Thread.currentThread().interrupt()`). `setPriority` é só uma dica ao SO.
- **Daemon x usuário**: a JVM encerra quando todas as threads de **usuário** terminam; threads *daemon* (`setDaemon(true)`) não a seguram.

**Problemas clássicos**:

- **Condição de corrida (*race condition*)**: duas threads leem/alteram o mesmo dado sem coordenação. `saldo -= v` não é atômico (ler, calcular, escrever); o padrão *check-then-act* (`if (saldo >= x) saldo -= x`) também é corrida.
- **Visibilidade**: sem sincronização, uma thread pode **não enxergar** a escrita de outra (cache de CPU, reordenação). O **Modelo de Memória do Java** define a relação *happens-before* que garante visibilidade (locks, `volatile`, `join`, `start`).
- **Deadlock**: duas threads esperam uma pela outra para sempre, cada uma segurando um lock que a outra quer. Prevenção: **mesma ordem** de aquisição de locks, `tryLock` com tempo, menos locks aninhados. Diagnóstico: *thread dump* (`jstack`).
- **Starvation/livelock**: thread nunca consegue recurso / threads reagem uma à outra sem progresso.

**Ferramentas de sincronização**:

- **`synchronized`**: só **uma thread por vez** executa o bloco para o mesmo **monitor**. Em método de instância o lock é `this`; em `static`, a `Class`. Garante **exclusão mútua** e **visibilidade**. Todos os acessos ao dado compartilhado (inclusive leitura) devem usar o **mesmo** lock; mantenha a seção crítica pequena.
- **`volatile`**: garante **visibilidade** e ordenação de uma variável, mas **não atomicidade** (`count++` continua inseguro). Bom para *flags* de parada.
- **`wait()` / `notify()` / `notifyAll()`**: espera/notificação dentro de `synchronized` sobre o mesmo objeto; `wait` libera o lock. **Sempre em `while`** (rechecar a condição por causa de *spurious wakeups*). É o padrão **produtor-consumidor**. Esquecer de notificar na mudança de estado (ex.: ao "fechar") deixa consumidores dormindo para sempre.
- **Atômicos** (`AtomicInteger`, `AtomicLong`, `AtomicReference`, `LongAdder`): operações compostas indivisíveis (`incrementAndGet`, `compareAndSet`) sem lock. Ideais para contadores.
- **`Lock` / `ReentrantLock`**: lock explícito, **sempre** em `lock(); try { ... } finally { unlock(); }`. Extras: `tryLock()`/`tryLock(tempo)`, `lockInterruptibly()`, modo justo, `isHeldByCurrentThread()`. **Reentrante**: a mesma thread pode readquirir. **`Condition`** (`newCondition()`, `await`, `signalAll`) é o `wait/notify` por lock, com várias condições.
- **`ReentrantReadWriteLock`**: vários **leitores** simultâneos ou **um escritor** exclusivo; vale com muito mais leitura que escrita. Não devolva visões vivas (`keySet()`) para fora do lock.
- **Sincronizadores**: `CountDownLatch` (espera N eventos), `CyclicBarrier`, `Semaphore` (limita acessos simultâneos), `Phaser`.

**Coleções concorrentes**:

- **`ConcurrentHashMap`**: mapa thread-safe com concorrência fina; operações atômicas (`putIfAbsent`, `computeIfAbsent`, `merge`). `Collections.synchronizedList/Map` bloqueiam a coleção toda e ainda exigem sincronizar a iteração.
- **`CopyOnWriteArrayList`**: cada escrita copia o array; iteradores usam um *snapshot* e **nunca** lançam `ConcurrentModificationException`. Ótima para muita leitura e pouca escrita (listeners).
- **`BlockingQueue`** (`ArrayBlockingQueue`, `LinkedBlockingQueue`, `SynchronousQueue`): `put` **bloqueia** se cheia; `take` **bloqueia** se vazia; `offer/poll` com tempo. É o produtor-consumidor pronto. **`TransferQueue`** acrescenta `transfer()`, que só retorna quando um consumidor **recebeu** o item.

**Imutabilidade** é a forma mais simples de ser thread-safe: objetos que nunca mudam podem ser compartilhados sem lock (records, `final`, `List.of`).

**Para que serve**: aproveitar vários núcleos, manter a aplicação responsiva e não bloquear enquanto se espera I/O.

**Exemplo**:

```java
class Contador {
    private int n;
    public synchronized void inc() { n++; }
    public synchronized int get() { return n; }
}
Contador c = new Contador();
Runnable tarefa = () -> { for (int i = 0; i < 10_000; i++) c.inc(); };
Thread t1 = new Thread(tarefa), t2 = new Thread(tarefa);
t1.start(); t2.start();
t1.join(); t2.join();                    // espera as duas terminarem
System.out.println(c.get());             // 20000

AtomicInteger a = new AtomicInteger();   // alternativa sem lock
a.incrementAndGet();
BlockingQueue<String> fila = new ArrayBlockingQueue<>(10);
fila.put("x");                            // produtor
String item = fila.take();                // consumidor (bloqueia se vazia)
```

## 4.6 Executors, Future e CompletableFuture

**O que é**: criar uma `Thread` por tarefa é caro e não escala. Um **`ExecutorService`** mantém um **pool de threads reutilizáveis** e recebe tarefas.

| Fábrica (`Executors`) | Comportamento |
| --- | --- |
| `newFixedThreadPool(n)` | n threads fixas; excedentes ficam em fila (CPU: `availableProcessors()`) |
| `newCachedThreadPool()` | cria sob demanda, reaproveita ociosas; pode explodir sob carga |
| `newSingleThreadExecutor()` | uma thread, tarefas em ordem |
| `newScheduledThreadPool(n)` | tarefas atrasadas/periódicas |
| `newVirtualThreadPerTaskExecutor()` | uma thread virtual por tarefa (tópico 4.7) |

- `execute(Runnable)` não devolve nada; **`submit`** devolve **`Future`**. `invokeAll`/`invokeAny` rodam coleções de tarefas.
- **Sempre** `shutdown()` (para de aceitar novas tarefas e termina as em andamento) e, se necessário, `awaitTermination` / `shutdownNow()` (interrompe). Sem isso o programa pode nunca terminar. Em Java 19+, `ExecutorService` é `AutoCloseable` (use em *try-with-resources*).
- **`ScheduledExecutorService`**: `schedule` (uma vez, com atraso), `scheduleAtFixedRate` (período contado do **início** de cada execução), `scheduleWithFixedDelay` (espera contada do **fim** da anterior).
- **`Callable<V>`**: como `Runnable`, mas **devolve valor e pode lançar `Exception`**.
- **`Future<V>`**: promessa de resultado. `get()` **bloqueia** até ficar pronto; `get(tempo, unidade)` lança `TimeoutException`; `cancel`, `isDone`. Falhas vêm em `ExecutionException`. Limitação: sem encadeamento nem composição.

**`CompletableFuture<T>`**: `Future` **composável**, para pipelines assíncronos sem bloquear cedo demais.

| Método | Uso |
| --- | --- |
| `supplyAsync(Supplier[, executor])` / `runAsync(Runnable)` | inicia tarefa assíncrona |
| `thenApply` / `thenAccept` / `thenRun` | transformar (`map`) / consumir / só executar |
| `thenCompose` | função que **devolve** `CompletableFuture` (`flatMap`; evita aninhar) |
| `thenCombine(outro, fn)` | une dois futures independentes |
| `allOf(...)` / `anyOf(...)` | espera **todos** / o **primeiro** |
| `exceptionally` / `handle` / `whenComplete` | tratamento de erro |
| `join()` x `get()` | `join` lança unchecked; `get` lança checked |

- Sem executor explícito usa o `ForkJoinPool.commonPool()` (threads **daemon**); para I/O bloqueante passe um executor próprio.
- **Armadilha**: em `stream.map(x -> supplyAsync(...)).map(CompletableFuture::join)`, a Stream preguiçosa faz cada elemento passar por tudo antes do próximo e **serializa**. **Colete** os futures numa lista primeiro (todos iniciam) e só depois dê `join`.

**Para que serve**: rodar tarefas em paralelo com custo controlado, buscar dados de vários serviços ao mesmo tempo, encadear etapas assíncronas.

**Exemplo**:

```java
try (ExecutorService ex = Executors.newFixedThreadPool(4)) {
    Future<Integer> f = ex.submit(() -> { Thread.sleep(500); return 42; });
    System.out.println(f.get(2, TimeUnit.SECONDS));   // 42
}

List<CompletableFuture<Double>> futuros = lojas.stream()
        .map(l -> CompletableFuture.supplyAsync(() -> buscarPreco(l)))
        .toList();                                    // inicia todos
List<Double> precos = futuros.stream().map(CompletableFuture::join).toList();

CompletableFuture.supplyAsync(() -> "dado")
        .thenApply(String::toUpperCase)
        .thenAccept(System.out::println)
        .exceptionally(e -> { System.err.println(e); return null; });
```

## 4.7 Threads virtuais (Java 21)

**O que é**: threads **leves**, gerenciadas pela JVM (não 1:1 com threads do SO). Milhões delas cabem em memória; quando uma bloqueia em I/O (rede, banco), a JVM a **desmonta** do thread de plataforma, que passa a executar outra. Programa-se no estilo síncrono e simples (uma thread por requisição), com escala de código assíncrono.

- Criar: `Thread.ofVirtual().start(tarefa)`, `Thread.startVirtualThread(r)` ou `Executors.newVirtualThreadPerTaskExecutor()`.
- **Não use pool** de threads virtuais (são baratas; crie uma por tarefa). Limite concorrência com `Semaphore`.
- Evite *pinning* (bloco `synchronized` longo com I/O pode prender o *carrier thread*; prefira `ReentrantLock`).
- Úteis para tarefas **I/O-bound**; para **CPU-bound** não trazem ganho.
- No Spring Boot: `spring.threads.virtual.enabled=true`.

**Para que serve**: servidores com muitas conexões simultâneas bloqueantes sem a complexidade de programação reativa.

**Exemplo**:

```java
try (var ex = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10_000; i++) {
        ex.submit(() -> { Thread.sleep(Duration.ofSeconds(1)); return null; });
    }
}   // fecha esperando todas; 10 mil tarefas em ~1 segundo
```

## 4.8 Gerenciamento de memória e Garbage Collector

**O que é**: a JVM aloca objetos no **heap** e os libera automaticamente quando **não são mais alcançáveis** a partir das **raízes** (variáveis locais nas pilhas das threads, campos estáticos, etc.). Não há `free`; existe o **vazamento lógico** (*memory leak*): objetos ainda referenciados que nunca serão usados (caches e `static` coleções que só crescem, listeners não removidos, `ThreadLocal` esquecido).

- **Gerações**: objetos novos nascem na **Young** (*Eden* + *Survivors*); sobreviventes são promovidos à **Old**. Coleta *minor* (Young, rápida e frequente) e *major/full* (Old).
- **Coletores**: **G1** (padrão; regiões, pausas previsíveis), **ZGC** e **Shenandoah** (pausas ínfimas, heaps enormes), **Parallel** (throughput), **Serial** (pequenos).
- **Referências**: forte (normal), `SoftReference` (liberada sob pressão de memória, para caches), `WeakReference` (liberada no próximo GC; `WeakHashMap`), `PhantomReference`.
- **Erros**: `OutOfMemoryError` (heap cheio), `StackOverflowError` (recursão profunda demais).
- **Parâmetros comuns**: `-Xms`/`-Xmx` (heap inicial/máximo), `-XX:+UseZGC`, `-Xss` (pilha). Ferramentas: `jcmd`, `jmap`, `jstat`, VisualVM, JFR (*Flight Recorder*), *heap dump*.
- **Boas práticas**: não crie objetos desnecessários em laços quentes, feche recursos, limite o tamanho de caches, prefira `StringBuilder` a `+=` em laços, não chame `System.gc()`.

**Para que serve**: entender por que uma aplicação consome muita memória, tem pausas ou vaza, e dimensionar a JVM.

**Exemplo**:

```java
class Cache {
    private static final Map<String, byte[]> dados = new HashMap<>();   // cresce para sempre: vazamento
}
Map<Object, String> fraco = new WeakHashMap<>();   // chave sem outras referências é removida pelo GC
Runtime rt = Runtime.getRuntime();
System.out.println((rt.totalMemory() - rt.freeMemory()) / 1_048_576 + " MB em uso");
```

---

# Parte 5: Persistência e qualidade

## 5.1 Bancos de dados: conceitos essenciais

**O que é**: base para entender JDBC, JPA e Spring Data.

- **Relacional (SQL)**: dados em **tabelas** (linhas e colunas) ligadas por **chaves** (`PRIMARY KEY`, `FOREIGN KEY`). Comandos: `SELECT`, `INSERT`, `UPDATE`, `DELETE` (DML); `CREATE`, `ALTER`, `DROP` (DDL). `JOIN` (`INNER`, `LEFT`, `RIGHT`) combina tabelas; `GROUP BY` agrupa; `INDEX` acelera buscas (custa escrita e espaço).
- **Normalização**: 1FN (valores atômicos, sem grupos repetidos), 2FN (sem dependência parcial da chave), 3FN (sem dependência transitiva). Reduz redundância e anomalias; às vezes se **desnormaliza** por desempenho.
- **Transações e ACID**: **A**tomicidade (tudo ou nada), **C**onsistência, **I**solamento, **D**urabilidade. **Níveis de isolamento**: `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`; evitam anomalias (*dirty read*, *non-repeatable read*, *phantom read*) ao custo de concorrência.
- **NoSQL**: **documento** (MongoDB: JSON flexível), **chave-valor** (Redis: rápido, cache, TTL), **colunar** (Cassandra), **grafo** (Neo4j). Troca de rigidez e transações por flexibilidade e escala; **persistência poliglota** é usar o banco certo para cada necessidade.

**Para que serve**: escolher onde e como guardar os dados; modelar tabelas e relacionamentos corretamente.

**Exemplo**:

```sql
CREATE TABLE producer (id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(100) NOT NULL);
CREATE TABLE anime (
  id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(100),
  producer_id INT, FOREIGN KEY (producer_id) REFERENCES producer(id));
SELECT a.name, p.name FROM anime a INNER JOIN producer p ON a.producer_id = p.id WHERE a.name LIKE '%Nar%';
```

## 5.2 JDBC (Java Database Connectivity)

**O que é**: API padrão (`java.sql`) para falar com **bancos relacionais**. Um **driver** (`mysql-connector-j`, `postgresql`) implementa as interfaces `Connection`, `Statement`, `ResultSet`. A conexão usa uma **URL** (`jdbc:mysql://localhost:3306/db`), usuário e senha, via `DriverManager` ou `DataSource`.

- **Connection pool** (HikariCP): abrir conexão é caro; o pool reaproveita conexões. Em produção sempre use pool (o Spring Boot já configura).
- **`Statement`**: executa SQL como texto. **Nunca concatene entrada do usuário**: abre **SQL Injection** (`' OR '1'='1`).
- **`PreparedStatement`**: SQL com `?` e parâmetros tipados (`setString(1, nome)`; índices começam em **1**). Previne SQL Injection, pode reutilizar o plano. O `%` do `LIKE` faz parte do **valor**, não do SQL.
- **`CallableStatement`**: chama *stored procedures* (`CALL sp(?)`, `registerOutParameter`).
- Execução: `executeQuery` (SELECT → `ResultSet`), `executeUpdate` (linhas afetadas), `execute` (genérico). Chaves geradas: `RETURN_GENERATED_KEYS` + `getGeneratedKeys()`. **Batch**: `addBatch()` + `executeBatch()`.
- **`ResultSet`**: cursor que começa **antes** da primeira linha; `next()` avança. Colunas começam em **1**; prefira por nome (`rs.getString("name")`). `wasNull()` para colunas anuláveis. Tipos de cursor: `TYPE_FORWARD_ONLY` (padrão), `TYPE_SCROLL_INSENSITIVE`/`SENSITIVE` (navega com `last`, `absolute`, `previous`); `CONCUR_UPDATABLE` permite alterar/inserir/apagar por ele (`updateRow`, `moveToInsertRow`/`insertRow`, `deleteRow`).
- **Metadados**: `ResultSetMetaData` (colunas) e `DatabaseMetaData` (o que o driver suporta).
- **Recursos**: `Connection`, `Statement` e `ResultSet` devem ser fechados (vazamento de conexões esgota o pool). **Sempre try-with-resources**.
- **Transações**: por padrão há **auto-commit**. Para agrupar: `setAutoCommit(false)`, executar, `commit()`; em erro, **um** `rollback()` e propagar a falha (não continue o laço após falhar). `Savepoint` desfaz parte. `setTransactionIsolation` define o isolamento.
- **`RowSet`**: `JdbcRowSet` (conectado, JavaBean) e `CachedRowSet` (desconectado: altera offline e `acceptChanges()` sincroniza). Uso raro hoje.
- **Boas práticas**: credenciais fora do código; camada de repositório isolando SQL; mapear `ResultSet` → objeto em um só lugar.

**Para que serve**: acesso direto e de baixo nível ao banco; é a base sobre a qual JPA, Spring Data e `JdbcTemplate` são construídos.

**Exemplo**:

```java
String sql = "SELECT id, name FROM producer WHERE name LIKE ?";
try (Connection conn = DriverManager.getConnection(url, user, pass);
     PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.setString(1, "%" + nome + "%");
    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) System.out.println(rs.getInt("id") + " " + rs.getString("name"));
    }
}
// transação
try (Connection conn = ds.getConnection()) {
    conn.setAutoCommit(false);
    try { /* vários executeUpdate */ conn.commit(); }
    catch (SQLException e) { conn.rollback(); throw e; }
}
```

## 5.3 JPA e Hibernate (ORM)

**O que é**: **ORM** (*Object-Relational Mapping*) mapeia classes para tabelas e gera o SQL. **JPA** (Jakarta Persistence) é a **especificação**; **Hibernate** é a implementação mais usada. O **`EntityManager`** gerencia o ciclo de vida das entidades dentro de um **contexto de persistência** (cache de 1º nível).

- **Mapeamento**: `@Entity`, `@Table`, `@Id`, `@GeneratedValue(strategy = IDENTITY/SEQUENCE/UUID)`, `@Column(nullable, unique, length)`, `@Enumerated(EnumType.STRING)`, `@Transient`, `@Embedded`.
- **Relacionamentos**: `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`; `mappedBy` define o lado inverso; `@JoinColumn`; `cascade` propaga operações; `orphanRemoval` apaga filhos removidos da coleção.
- **Lazy x Eager**: `*ToMany` é **lazy** por padrão (carrega ao acessar). Fora da transação causa `LazyInitializationException`. O **problema N+1** (1 consulta + N consultas dos filhos) resolve-se com `JOIN FETCH`, `@EntityGraph` ou `@BatchSize`.
- **Estados da entidade**: *transient* (nova), *managed* (gerenciada: mudanças são salvas automaticamente no `flush`/`commit` — *dirty checking*), *detached*, *removed*.
- **JPQL**: consulta orientada a objetos (`SELECT p FROM Pedido p WHERE p.total > :min`); também SQL nativo e *Criteria API*.
- **Transações**: `@Transactional`; **locking**: otimista (`@Version`, detecta conflito na gravação) e pessimista (`PESSIMISTIC_WRITE`).
- **Esquema**: `hbm2ddl.auto = update/create` só para estudo; em produção use **migrações versionadas** (Flyway, Liquibase).
- **Quando não usar**: relatórios muito complexos e operações em massa (prefira SQL/`JdbcTemplate`).

**Para que serve**: produtividade (CRUD sem SQL manual), portabilidade entre bancos e modelo de domínio rico.

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
}
TypedQuery<Pedido> q = em.createQuery(
    "SELECT p FROM Pedido p JOIN FETCH p.itens WHERE p.cliente = :c", Pedido.class);
q.setParameter("c", "Ana");
```

## 5.4 Build, dependências e Lombok

**O que é**: ferramentas que **compilam, testam, empacotam** o projeto e **baixam dependências** (bibliotecas).

- **Maven**: configuração em XML (`pom.xml`): `groupId`/`artifactId`/`version` (coordenadas), `<dependencies>` com `scope` (`compile`, `test`, `runtime`, `provided`), `<properties>`, plugins. Ciclo de vida: `validate → compile → test → package → verify → install → deploy`. Comandos: `mvn clean package`, `mvn test`. **Wrapper** (`mvnw`) fixa a versão do Maven.
- **Gradle**: script em Groovy/Kotlin (`build.gradle`): `plugins`, `repositories`, `dependencies` (`implementation`, `testImplementation`, `runtimeOnly`, `compileOnly`, `annotationProcessor`), tarefas (`./gradlew build`, `test`, `bootRun`). Build incremental e cache; **toolchain** escolhe a versão do JDK.
- **BOM** / `dependencyManagement`: fixa versões compatíveis de um conjunto de bibliotecas (Spring Boot, Spring Cloud).
- **Dependências transitivas**: trazidas por outras; conflitos de versão exigem atenção (`mvn dependency:tree`, `gradle dependencies`).
- **Estrutura padrão**: `src/main/java`, `src/main/resources`, `src/test/java`.
- **Empacotamento**: JAR (`java -jar`), *fat/uber JAR* (Spring Boot), WAR, imagem de contêiner (Dockerfile ou **Jib**).
- **Lombok**: *annotation processor* que gera código repetitivo em compilação: `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, `@Data`, `@Value` (imutável), `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@RequiredArgsConstructor`, `@Slf4j`. Cuidado: `@Data` em entidades JPA (`equals`/`hashCode`/`toString` sobre relacionamentos); `records` cobrem boa parte dos casos.
- **MapStruct**: processor que gera mappers DTO ↔ entidade em compilação.

**Para que serve**: automatizar o ciclo de vida do projeto e garantir builds reprodutíveis.

**Exemplo**:

```xml
<!-- Maven -->
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter</artifactId>
  <version>5.11.0</version>
  <scope>test</scope>
</dependency>
```

```groovy
// Gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

## 5.5 Testes automatizados

**O que é**: código que **verifica automaticamente** se o sistema se comporta como esperado. Dá confiança para refatorar e documenta o comportamento.

- **Tipos (pirâmide)**: **unitários** (uma classe, rápidos, sem rede/banco; base da pirâmide), **de integração** (várias partes/banco real), **ponta a ponta** (fluxo completo; poucos). Também **de contrato**, **de carga**, **de aceitação**.
- **JUnit 5**: `@Test`, `@BeforeEach`/`@AfterEach` (a cada teste), `@BeforeAll`/`@AfterAll` (uma vez; métodos `static`), `@DisplayName`, `@Disabled`, `@Nested`, `@ParameterizedTest` (+ `@ValueSource`, `@CsvSource`), `@ExtendWith`.
- **Asserções**: `assertEquals(esperado, atual)`, `assertTrue`, `assertNotNull`, `assertThrows(Excecao.class, () -> ...)` (o 3º argumento é só a **mensagem de falha**; para validar a mensagem da exceção, compare `getMessage()` do retorno), `assertAll`. **AssertJ**: `assertThat(x).isEqualTo(...).hasSize(2)` (fluente).
- **Estrutura**: **Arrange / Act / Assert** (ou Given / When / Then); um comportamento por teste; nomes descritivos (`metodo_Resultado_Quando`).
- **Mockito**: cria **dublês** para isolar a unidade. `@Mock`, `@InjectMocks`, `when(x).thenReturn(y)`, `verify(m, times(1)).metodo()`, `any()`, `ArgumentCaptor`. **Mock** (verifica interações), **stub** (respostas prontas), **spy** (objeto real parcial), **fake** (implementação simples em memória). Não mocke a classe sob teste.
- **TDD**: escrever o teste antes (vermelho → verde → refatorar). **Cobertura** (JaCoCo) mede linhas executadas, não a qualidade das asserções.
- **Testes de banco**: H2 em memória ou **Testcontainers** (banco real em contêiner).
- **Teste de contrato de interface**: classe de teste abstrata reutilizada por cada implementação (Liskov na prática).
- Princípios **F.I.R.S.T.**: Rápidos, Independentes, Repetíveis, Auto-validáveis, Oportunos.

**Para que serve**: detectar regressões, permitir mudanças seguras e servir de documentação executável.

**Exemplo**:

```java
@ExtendWith(MockitoExtension.class)
class PedidoServiceTest {
    @Mock PedidoRepository repo;
    @InjectMocks PedidoService service;

    @Test
    @DisplayName("Deve lançar exceção quando pedido não existe")
    void buscar_Lanca_QuandoNaoExiste() {
        when(repo.findById(1L)).thenReturn(Optional.empty());
        var ex = assertThrows(NaoEncontradoException.class, () -> service.buscar(1L));
        assertEquals("Pedido 1 não encontrado", ex.getMessage());
        verify(repo).findById(1L);
    }

    @ParameterizedTest
    @ValueSource(ints = {18, 30, 65})
    void maioridade(int idade) { assertTrue(new Pessoa(idade).isMaior()); }
}
```

---

# Parte 6: Arquitetura e boas práticas

## 6.1 Princípios SOLID

**O que é**: cinco princípios de projeto orientado a objetos que tornam o código **coeso, flexível e fácil de manter**.

- **S: Responsabilidade Única (SRP)**: uma classe deve ter **um único motivo para mudar**. Separe regra de negócio, acesso a dados e apresentação. Sintoma de violação: classe "faz tudo", qualquer mudança mexe nela. Resultado: alta **coesão**.
- **O: Aberto/Fechado (OCP)**: aberto para **extensão**, fechado para **modificação**. Novos comportamentos entram por **novas implementações** de uma abstração (polimorfismo, Strategy), sem editar código já testado. Sintoma: cadeias de `if/else` ou `instanceof` por tipo.
- **L: Substituição de Liskov (LSP)**: um subtipo deve poder ser usado **onde o tipo base é esperado, sem surpresas** (sem lançar `UnsupportedOperationException`, sem endurecer pré-condições). Se `Quadrado extends Retangulo` quebra expectativas, a herança está errada: prefira composição.
- **I: Segregação de Interfaces (ISP)**: **vários contratos pequenos** em vez de um grande. Ninguém deve implementar métodos vazios só para cumprir uma interface.
- **D: Inversão de Dependência (DIP)**: módulos de alto nível **dependem de abstrações**, não de implementações concretas; ambos dependem de interfaces. A dependência é **injetada** (construtor), o que permite trocar por mock ou outra implementação. É a base da injeção de dependência do Spring.

**Para que serve**: reduzir acoplamento, facilitar testes e evolução, evitar código frágil e rígido.

**Exemplo (DIP + OCP)**:

```java
interface Notificador { void enviar(String msg); }
class EmailNotificador implements Notificador { public void enviar(String m) { /* e-mail */ } }
class SmsNotificador implements Notificador { public void enviar(String m) { /* SMS */ } }

class PedidoService {
    private final Notificador notificador;                 // depende da abstração
    PedidoService(Notificador n) { this.notificador = n; }  // injetada
    void concluir() { notificador.enviar("Pedido concluído"); }
}
new PedidoService(new SmsNotificador()).concluir();   // trocar canal não altera PedidoService
```

## 6.2 Clean Code e boas práticas de código

**O que é**: conjunto de práticas para escrever código **legível, simples e fácil de mudar**. Código é lido muito mais vezes do que escrito.

- **Nomes que revelam intenção**: classes são substantivos, métodos são verbos; sem abreviações obscuras; nomes pronunciáveis e pesquisáveis.
- **Funções pequenas**, que fazem **uma coisa**, em **um nível de abstração**; poucos parâmetros (mais de 3: use um objeto/record); sem *flag arguments* (`salvar(true)`).
- **Comentários**: explique o **porquê**, não o quê. Código claro dispensa comentário; comentário desatualizado mente.
- **DRY** (não repita lógica, mas não abstraia cedo demais), **KISS** (mantenha simples), **YAGNI** (não construa o que não precisa agora).
- **Sem números/strings mágicos**: use constantes nomeadas (`MARGEM_LUCRO = 0.20`).
- **Guard clauses** (retorno antecipado) em vez de `if` aninhado; **imutabilidade** por padrão.
- **Lei de Demeter**: converse só com vizinhos imediatos (evite `a.getB().getC().getD()`). **Tell, Don't Ask**: mande o objeto agir em vez de puxar seus dados para decidir fora.
- **Tratamento de erros**: exceções em vez de códigos de retorno; sem `catch` vazio; não devolva nem receba `null` à toa.
- **Convenções**: pacotes em minúsculas, `PascalCase` em classes, `camelCase` em métodos, `UPPER_SNAKE` em constantes.
- **Refatoração**: melhorar a estrutura sem mudar o comportamento, protegido por testes. **Boy Scout Rule**: deixe o código mais limpo do que encontrou. Cheiros comuns: método longo, classe gigante, código duplicado, *primitive obsession*, *feature envy*.

**Para que serve**: reduzir bugs e o custo de manutenção, facilitar a leitura por outras pessoas (e por você no futuro).

**Exemplo**:

```java
// antes
if (u != null) { if (u.getA() > 17) { if (u.isOk()) { liberar(u); } } }

// depois: guard clauses e nomes que explicam
if (usuario == null) return;
if (!usuario.isMaiorDeIdade()) return;
if (!usuario.estaAtivo()) return;
liberarAcesso(usuario);
```

## 6.3 Padrões de projeto criacionais

**O que é**: **padrões de projeto** (GoF) são soluções nomeadas para problemas recorrentes de design. Os **criacionais** tratam de **como criar objetos**.

### Singleton

Garante **uma única instância** e um ponto global de acesso: construtor `private`, atributo estático e `getInstance()`.

| Variante | Como | Observações |
| --- | --- | --- |
| **Eager** | cria no carregamento da classe | thread-safe e simples; cria mesmo sem uso |
| **Lazy simples** | cria no primeiro acesso | **não é thread-safe** |
| **Double-checked locking** | `if (null) synchronized { if (null) ... }` | exige o campo **`volatile`** e construtor `private` |
| **Holder (Bill Pugh)** | classe interna estática guarda a instância | lazy e thread-safe sem `synchronized` |
| **Enum** | `enum X { INSTANCE; }` | imune a reflexão e serialização (recomendado) |

**Crítica**: é estado global (dificulta testes e esconde dependências). No Spring, **beans são singletons gerenciados pelo container**, sem o antipadrão.

```java
class Config {
    private Config() {}
    private static class Holder { static final Config INSTANCE = new Config(); }
    static Config getInstance() { return Holder.INSTANCE; }
}
enum Registro { INSTANCE; void log(String m) { System.out.println(m); } }
```

### Builder

Separa a **construção** de um objeto complexo da sua representação; resolve o *telescoping constructor* (muitos construtores com parâmetros opcionais). Interface fluente (cada método devolve `this`) e `build()` no fim, onde se validam invariantes. Pode ser escrito à mão, gerado por **Lombok** (`@Builder`) ou por annotation processor.

```java
Pessoa p = Pessoa.builder().nome("Ana").email("a@x.com").idade(30).build();
```

### Factory Method / Simple Factory

Encapsula **qual classe concreta instanciar**; o cliente depende só da interface/abstração. Com `switch` **exaustivo** sobre `enum`, o compilador obriga a tratar novos casos (sem `default`).

```java
static Moeda criar(Pais pais) {
    return switch (pais) {
        case BRASIL -> new Real();
        case EUA -> new Dolar();
    };
}
```

Outros criacionais: **Abstract Factory** (famílias de objetos) e **Prototype** (clonar).

**Para que serve (criacionais)**: controlar e desacoplar a criação de objetos, garantir instância única, montar objetos complexos de forma legível.

## 6.4 Padrões de projeto estruturais

**O que é**: tratam de **como compor classes e objetos** em estruturas maiores.

- **Facade**: interface **simples** para um subsistema complexo. O cliente chama uma operação e a fachada orquestra várias classes. Reduz acoplamento; cuidado para não virar "classe deus".
- **Adapter**: converte a interface de uma classe na que o cliente espera. É a base da **arquitetura hexagonal** (adaptadores de persistência/HTTP implementando as portas do domínio).
- **Decorator**: **acrescenta comportamento** a um objeto embrulhando-o, sem herança (`new BufferedReader(new FileReader(f))`).
- **Proxy**: representante que **controla o acesso** ao objeto real (carregamento tardio, segurança, transações, cache). É o que o Spring usa em `@Transactional`, `@Cacheable` e clientes Feign.
- **Composite**: trata objetos individuais e grupos de forma uniforme (árvore de pastas/arquivos).
- **DTO (Data Transfer Object)**: objeto só de dados que transporta informação entre camadas/serviços, evitando expor a entidade de domínio e reduzindo chamadas. Hoje costuma ser um `record`.

**Para que serve**: reaproveitar e integrar código existente, simplificar APIs, estender comportamento sem herança.

**Exemplo (Facade)**:

```java
class MigracaoFacade {
    private final CepApi cepApi; private final CrmService crm;
    MigracaoFacade(CepApi c, CrmService m) { cepApi = c; crm = m; }
    void migrarCliente(String nome, String cep) {
        String cidade = cepApi.cidade(cep);
        String estado = cepApi.estado(cep);
        crm.gravar(nome, cep, cidade, estado);     // o cliente chama um único método
    }
}
```

## 6.5 Padrões de projeto comportamentais

**O que é**: tratam de **como objetos se comunicam e dividem responsabilidades**.

- **Strategy**: define uma **família de algoritmos** intercambiáveis em tempo de execução; o contexto guarda uma estratégia e delega. Elimina `if/else` por tipo e respeita Aberto/Fechado. Em Java moderno a estratégia pode ser uma **lambda** (`Comparator`, `Predicate`).
- **Template Method**: a classe abstrata define o **esqueleto** do algoritmo e deixa passos para as subclasses.
- **Observer**: objetos **assinam** eventos de outro e são notificados quando algo muda (`PropertyChangeListener`, `ApplicationEventPublisher` do Spring, listeners).
- **Command**: encapsula uma requisição como objeto (fila, desfazer/refazer, `Runnable`).
- **Iterator**: percorre uma coleção sem expor sua estrutura (`Iterator`, for-each).
- **State**: o comportamento muda conforme o estado interno (máquina de estados).
- **Chain of Responsibility**: a requisição passa por uma cadeia de tratadores (filtros servlet, Spring Security).

**Para que serve**: desacoplar quem pede de quem executa, trocar algoritmos sem alterar o cliente e organizar fluxos de decisão.

**Exemplo (Strategy)**:

```java
interface Comportamento { void mover(); }
class Defensivo implements Comportamento { public void mover() { System.out.println("Defensivo"); } }
class Agressivo implements Comportamento { public void mover() { System.out.println("Agressivo"); } }

class Robo {
    private Comportamento estrategia;
    void setEstrategia(Comportamento c) { estrategia = c; }
    void mover() { estrategia.mover(); }
}
Robo r = new Robo();
r.setEstrategia(new Defensivo()); r.mover();
r.setEstrategia(new Agressivo()); r.mover();     // troca em tempo de execução
```

## 6.6 Arquitetura de software

**O que é**: a organização das **camadas e responsabilidades** do sistema.

- **Arquitetura em camadas**: *controller* (HTTP) → *service* (regras) → *repository* (dados) → banco. Cada camada conversa só com a de baixo. Simples; o risco é o domínio ficar acoplado à persistência.
- **Hexagonal (Ports and Adapters) / Clean Architecture**: o **domínio** fica no centro, sem depender de framework, banco ou web. Ele declara **portas** (interfaces, como `TaskRepository`); a infraestrutura fornece **adaptadores** (JPA, Mongo, REST). **Regra da dependência**: o código-fonte só aponta **para dentro**. Camadas: `domain` (entidades, value objects, regras), `application` (casos de uso), `infrastructure` (controllers, banco, clientes externos).
- **Casos de uso**: uma classe por ação do sistema (`CreateTaskUseCase`), orquestrando o domínio.
- **DDD (Domain-Driven Design)**: modelar o software a partir do domínio. *Entidade* (identidade), *Value Object* (imutável, por valor), *Agregado* (cluster consistente com raiz), *Repositório*, *Serviço de domínio*, **linguagem ubíqua**, **contextos delimitados** (*bounded contexts*) que se comunicam por eventos/contratos.
- **Camada anticorrupção**: converte modelos externos para o do domínio, isolando-o de mudanças de terceiros.
- **Dois pares de DTOs**: *Request/Response* (borda HTTP) e *Input/Output* (borda da aplicação), para mudar o JSON sem afetar os casos de uso.
- **Microsserviços x monólito**: microsserviços separam contextos em serviços independentes (escala e deploy isolados) ao custo de complexidade distribuída (rede, consistência eventual, observabilidade). Comece por um **monólito modular** bem dividido.
- **Resiliência**: *timeout*, *retry*, **circuit breaker**, *bulkhead*, *fallback*; escolha entre **fail-open** e **fail-closed** conforme o risco do negócio.

**Para que serve**: manter o sistema evolutivo, testável e independente de tecnologia.

**Exemplo (porta e adaptador)**:

```java
// domain: porta
interface TaskRepository { Task save(Task t); Optional<Task> findById(TaskId id); }

// application: caso de uso depende da porta
class CreateTaskUseCase {
    private final TaskRepository repo;
    CreateTaskUseCase(TaskRepository r) { this.repo = r; }
    Task execute(String titulo) { return repo.save(new Task(titulo)); }
}

// infrastructure: adaptador (pode ser trocado por JPA, sem tocar nas camadas internas)
class InMemoryTaskRepository implements TaskRepository { /* Map em memória */ }
```

## 6.7 Princípios de design de APIs REST

**O que é**: **REST** é um estilo arquitetural para APIs sobre HTTP, centrado em **recursos** (substantivos) identificados por URL.

- **Recursos e verbos**: `GET /tasks` (listar), `GET /tasks/{id}`, `POST /tasks` (criar), `PUT` (substituir inteiro), `PATCH` (alterar parcialmente), `DELETE`. URLs no plural, sem verbos (`/tasks`, não `/criarTask`).
- **Idempotência**: `GET`, `PUT`, `DELETE` são idempotentes (repetir dá o mesmo resultado); `POST` não. `GET` é **seguro** (não altera estado).
- **Códigos de status**: `200 OK`, `201 Created`, `204 No Content`, `400 Bad Request` (validação), `401 Unauthorized` (não autenticado), `403 Forbidden` (sem permissão), `404 Not Found`, `409 Conflict`, `422`, `500 Internal Server Error`.
- **Sem estado** (*stateless*): cada requisição carrega o necessário (token); facilita escalar.
- **Representações**: JSON; `Content-Type`/`Accept` (negociação de conteúdo); datas em ISO-8601.
- **Paginação, ordenação e filtros** por parâmetros (`?page=0&size=20&sort=name,desc`); **versionamento** (`/v1`); **HATEOAS** (links na resposta); erros padronizados (*Problem Details*, RFC 9457); documentação **OpenAPI/Swagger**.
- **Segurança**: HTTPS, autenticação (sessão/JWT/OAuth2), autorização por papel, validação de entrada, não expor entidades internas.

**Para que serve**: criar APIs previsíveis, fáceis de consumir e de evoluir.

**Exemplo**:

```
POST   /tasks            {"title":"Estudar"}   →  201 Created  + Location: /tasks/42
GET    /tasks/42                                →  200 OK       {"id":42,"title":"Estudar"}
PATCH  /tasks/42         {"status":"DONE"}     →  200 OK
DELETE /tasks/42                                →  204 No Content
GET    /tasks/999                               →  404 Not Found
```

---

# Parte 7: Ecossistema Spring

## 7.1 Spring Framework: IoC e Injeção de Dependência

**O que é**: o **Spring** é um framework que gerencia os objetos da aplicação. Em vez de a classe criar suas dependências com `new`, o **container IoC** (`ApplicationContext`) instancia os objetos (**beans**), resolve as dependências entre eles e os **injeta** (*Inversão de Controle* + *Injeção de Dependência*). O controle de criação e ciclo de vida sai do seu código.

- **Declarar beans**: estereótipos `@Component` (genérico), `@Service` (caso de uso/serviço), `@Repository` (acesso a dados; traduz exceções de persistência), `@Controller`/`@RestController` (web), `@Configuration` + métodos `@Bean` (para criar você mesmo, especialmente classes de terceiros). O `@ComponentScan` descobre os estereótipos a partir do pacote da classe principal **para baixo**.
- **Formas de injeção**: **por construtor** (recomendada: dependências explícitas, campos `final`, fácil de testar; com um só construtor nem precisa de `@Autowired`), por *setter* e por campo (evite).
- **Desambiguar** quando há mais de uma implementação: `@Primary` (preferido), `@Qualifier("nome")`, `@Bean(defaultCandidate = false)` (só entra se pedido explicitamente). Injetar **todas** as implementações: `List<Strategy>` ou `Map<String, Strategy>`.
- **Escopos**: **singleton** (padrão: uma instância por contexto, por isso beans devem ser *stateless* ou thread-safe), `prototype` (nova a cada pedido), `request`, `session`.
- **Ciclo de vida**: instanciar → injetar → `@PostConstruct` → uso → `@PreDestroy`. `InitializingBean`/`DisposableBean` são alternativas.
- **Perfis**: `@Profile("dev")` ativa beans por ambiente (`spring.profiles.active=dev`).

**Para que serve**: baixo acoplamento, troca de implementações por configuração, testes com mocks e ciclo de vida gerenciado.

**Exemplo**:

```java
@Service
public class PedidoService {
    private final PedidoRepository repo;        // dependência por construtor
    private final Notificador notificador;
    public PedidoService(PedidoRepository repo, @Qualifier("email") Notificador n) {
        this.repo = repo;
        this.notificador = n;
    }
}

@Configuration
class AppConfig {
    @Bean PasswordEncoder passwordEncoder() { return new BCryptPasswordEncoder(); }
}
```

## 7.2 Configuração externa e propriedades

**O que é**: separar **configuração** do código, para mudar de ambiente sem recompilar.

- Fontes (da menor para a maior precedência, simplificando): `application.properties`/`application.yml`, arquivos por perfil (`application-dev.yml`), **variáveis de ambiente**, argumentos de linha de comando.
- **`@Value("${chave:padrao}")`** injeta um valor; **`@ConfigurationProperties("prefixo")`** liga um grupo de propriedades a um objeto tipado (com validação). Ótimo para vários `DataSource`s (`registration.datasource.*`, `catalog.datasource.*`).
- **Segredos** (senhas, chaves) nunca no repositório: variáveis de ambiente, cofres (Vault, AWS Secrets Manager).
- Propriedades comuns: `server.port`, `spring.application.name`, `spring.datasource.url|username|password`, `spring.jpa.hibernate.ddl-auto`, `logging.level.pacote=DEBUG`, `spring.threads.virtual.enabled=true`.

**Para que serve**: um mesmo artefato roda em desenvolvimento, teste e produção com configurações diferentes.

**Exemplo**:

```properties
spring.application.name=loja
server.port=8081
app.taxa=0.15
```

```java
@ConfigurationProperties("app")
public record AppProps(double taxa) {}
// ou: @Value("${app.taxa}") double taxa;
```

## 7.3 Spring Boot

**O que é**: camada sobre o Spring que reduz a configuração: **"convenção sobre configuração"**.

- **Starters**: dependências agrupadas por finalidade (`spring-boot-starter-web`, `-data-jpa`, `-security`, `-validation`, `-actuator`, `-test`, `-data-redis`, `-data-mongodb`).
- **Auto-configuração**: o Boot configura beans automaticamente conforme o **classpath** e as propriedades (se há driver MySQL e URL, cria o `DataSource`). Você sobrescreve declarando seu próprio bean. `--debug` mostra o relatório de condições.
- **`@SpringBootApplication`** = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`.
- **Servidor embutido** (Tomcat): `java -jar app.jar` sobe a aplicação. **Fat JAR** inclui tudo.
- **DevTools** (reinício automático), **Docker Compose support** (`spring-boot-docker-compose` sobe os contêineres do `compose.yml` ao iniciar a aplicação).
- **Versões**: Spring Boot 3 exige **Java 17+** e usa `jakarta.*` (em vez de `javax.*`); Boot 4 usa **Jackson 3** (`tools.jackson`). Migração do Boot 2: `javax.*` → `jakarta.*`, `WebSecurityConfigurerAdapter` → bean `SecurityFilterChain`, `antMatchers` → `requestMatchers`, `@EnableGlobalMethodSecurity` → `@EnableMethodSecurity`, `springdoc-openapi-ui` → `springdoc-openapi-starter-webmvc-ui`.

**Para que serve**: criar aplicações prontas para produção rapidamente, sem XML e com padrões sensatos.

**Exemplo**:

```java
@SpringBootApplication
public class LojaApplication {
    public static void main(String[] args) {
        SpringApplication.run(LojaApplication.class, args);
    }
}
```

## 7.4 AOP (Programação Orientada a Aspectos)

**O que é**: separar **preocupações transversais** (transação, segurança, log, cache) da lógica de negócio. Termos: **aspecto** (módulo da preocupação), **advice** (`@Before`, `@After`, `@Around`, `@AfterReturning`, `@AfterThrowing`), **pointcut** (onde aplicar), **join point** (ponto de execução). O Spring implementa com **proxies** (dinâmicos JDK ou CGLIB) que envolvem o bean.

- É a base de `@Transactional`, `@Cacheable`, `@Async`, `@PreAuthorize`, `@Retryable`.
- **Pegadinha do proxy**: a anotação só funciona em chamadas **vindas de fora** do bean. Chamar `this.metodo()` (autoinvocação) **ignora** o proxy e a anotação não tem efeito. Métodos devem ser `public` e a classe não pode ser `final` em certos casos.

**Para que serve**: reaproveitar comportamento transversal sem repetir código em cada método.

**Exemplo**:

```java
@Aspect @Component
class TempoAspect {
    @Around("execution(* br.com.loja.service..*(..))")
    Object medir(ProceedingJoinPoint pjp) throws Throwable {
        long t0 = System.nanoTime();
        try { return pjp.proceed(); }
        finally { System.out.println(pjp.getSignature() + " " + (System.nanoTime() - t0) / 1_000_000 + "ms"); }
    }
}
```

## 7.5 Spring MVC e APIs REST

**O que é**: módulo web baseado em **servlets**; o `DispatcherServlet` recebe a requisição, encontra o *handler* (método do controller), converte entrada/saída (JSON via **Jackson**) e devolve a resposta.

- **Controller**: `@RestController` (= `@Controller` + `@ResponseBody`), `@RequestMapping("/tasks")`; `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping`.
- **Parâmetros**: `@PathVariable` (`/tasks/{id}`), `@RequestParam` (`?nome=x`), `@RequestBody` (JSON → objeto), `@RequestHeader`, `@AuthenticationPrincipal`, `Pageable`.
- **Resposta**: devolver objeto (vira JSON), `@ResponseStatus(HttpStatus.CREATED)` (status fixo) ou **`ResponseEntity<T>`** (status, cabeçalhos e corpo decididos em execução).
- **Validação** (Bean Validation/Jakarta): `@Valid` no `@RequestBody` dispara `@NotNull`, `@NotBlank`, `@Size`, `@Email`, `@Min`, `@Pattern`...; falha lança `MethodArgumentNotValidException` (→ 400). Validação de parâmetros simples com `@Validated`.
- **Tratamento global de erros**: `@RestControllerAdvice` + `@ExceptionHandler` traduz exceções de domínio em respostas HTTP **em um só lugar** (o domínio lança, a borda converte). Padrão moderno: `ProblemDetail` (RFC 9457).
- **DTOs**: separe o contrato HTTP (request/response) das entidades; *records* são ideais.
- **Jackson**: `@JsonProperty`, `@JsonIgnore`, `@JsonInclude(NON_NULL)`, `@JsonFormat`; registra `java.time` automaticamente.
- **Content negotiation** (`Accept`/`Content-Type`), **CORS** (`@CrossOrigin` ou configuração global), **HATEOAS** (links na resposta).
- **Clientes HTTP**: `RestClient` (síncrono, moderno), `WebClient` (reativo), `RestTemplate` (legado; use `exchange` com `ParameterizedTypeReference` para listas genéricas), ou **OpenFeign** (declarativo, tópico 7.12).
- **Documentação**: **springdoc-openapi** gera OpenAPI/Swagger UI (`@Operation`, `@ApiResponse`, `@Schema`); **Spring REST Docs** gera snippets a partir dos **testes** (a doc quebra a build se a API mudar).
- **Reativo (WebFlux)**: alternativa não bloqueante com `Mono`/`Flux`; com threads virtuais, o MVC síncrono atende bem a maior parte dos casos.

**Para que serve**: expor a lógica da aplicação como API HTTP consumível por front-end, mobile e outros serviços.

**Exemplo**:

```java
public record CreateTaskRequest(@NotBlank @Size(min = 3, max = 100) String title, String description) {}

@RestController
@RequestMapping("/tasks")
class TaskController {
    private final CreateTaskUseCase create;
    TaskController(CreateTaskUseCase create) { this.create = create; }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    TaskResponse criar(@RequestBody @Valid CreateTaskRequest req) {
        return TaskResponse.from(create.execute(req.toInput()));
    }

    @GetMapping("/{id}")
    ResponseEntity<TaskResponse> buscar(@PathVariable UUID id) {
        return ResponseEntity.ok(/* ... */ null);
    }
}

@RestControllerAdvice
class GlobalExceptionHandler {
    @ExceptionHandler(TaskNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    String naoEncontrada(TaskNotFoundException ex) { return ex.getMessage(); }
}
```

## 7.6 Spring Data JPA

**O que é**: gera **repositórios** a partir de **interfaces**, em cima de JPA/Hibernate. Hierarquia: `Repository` → `CrudRepository` (CRUD) → `PagingAndSortingRepository` (paginação e ordenação) → `JpaRepository` (`flush`, `saveAll`, `findAll(Pageable)`).

- **Query methods**: o **nome** do método vira consulta: `findByNome`, `findByEmailAndAtivoTrue`, `findByNomeStartingWithIgnoreCase`, `existsBy...`, `countBy...`, `deleteBy...`, `findTop3ByOrderByPrecoDesc`; navega associações com `_` (`findBySetores_Assentos_Codigo`).
- **`@Query`**: JPQL (ou SQL nativo com `nativeQuery = true`) com `@Param`; `@Modifying` para updates/deletes.
- **Projeções**: interfaces com só os getters desejados (a consulta traz menos colunas), `record`/DTO, ou dinâmicas.
- **Paginação**: `Pageable` (`PageRequest.of(0, 20, Sort.by("nome"))`) → `Page<T>` (conteúdo + total) ou `Slice<T>`.
- **Auditoria**: `@CreatedDate`, `@LastModifiedDate`, `@CreatedBy` com `@EnableJpaAuditing`.
- **Transações**: `@Transactional` (no serviço/caso de uso): abre transação, **rollback por padrão só em exceções unchecked** (`rollbackFor` para as checked), `readOnly = true` em consultas. Só vale em chamadas externas ao bean (proxy).
- **Vários bancos**: a auto-configuração cobre **um** `DataSource`; para vários, declare por contexto `DataSourceProperties`, `HikariDataSource`, `LocalContainerEntityManagerFactoryBean` e `JpaTransactionManager`, e ligue com `@EnableJpaRepositories(basePackages, entityManagerFactoryRef, transactionManagerRef)`. Um contexto é `@Primary`; os demais usam `@Qualifier` + `defaultCandidate = false`.
- **Open Session in View** (`spring.jpa.open-in-view`) mantém a sessão aberta até a resposta; cômodo, mas esconde N+1: muitos times desligam.
- **Entity listeners**: `@PostPersist`, `@PostUpdate`, `@PostRemove`, `@PrePersist`.

**Para que serve**: eliminar o boilerplate de acesso a dados mantendo consultas expressivas.

**Exemplo**:

```java
public interface ClienteRepository extends JpaRepository<Cliente, UUID> {
    List<Cliente> findByNomeStartingWithIgnoreCase(String prefixo);
    boolean existsByEmail(String email);
    Page<Cliente> findByAtivoTrue(Pageable pageable);

    @Query("select c from Cliente c where c.cidade = :cidade")
    List<Cliente> porCidade(@Param("cidade") String cidade);
}

@Service
class ClienteService {
    @Transactional
    public Cliente cadastrar(Cliente c) { return repo.save(c); }
}
```

## 7.7 Spring Data REST

**O que é**: expõe repositórios Spring Data como **API REST HATEOAS** automaticamente, **sem escrever controller**: paginação, links, ordenação e busca (`/search`) já vêm prontos. Inclui o **HAL Explorer** para navegar pela API.

- **`@RepositoryRestResource(path = "...", excerptProjection = X.class)`** personaliza caminho e a projeção resumida usada nas listagens. **`exported = false`** (no repositório) e **`@RestResource(exported = false)`** (no método, ex.: `deleteById`) **escondem** o que não deve ser público.
- **Eventos**: `@RepositoryEventHandler` com `@HandleBeforeCreate`, `@HandleAfterCreate`, `@HandleAfterSave`, `@HandleAfterDelete` reagem a operações que chegam **pela API REST** (diferente de `@PostPersist`, do JPA, que dispara em **qualquer** persistência).
- Limitação: expõe a **entidade** diretamente, acoplando o contrato HTTP ao modelo de dados. Ótimo para protótipos e CRUD simples; para regras de negócio, use controllers e DTOs.

**Para que serve**: CRUD rápido, painéis internos e protótipos.

**Exemplo**:

```java
@RepositoryRestResource(path = "customers", excerptProjection = CustomerExcerpt.class)
public interface CustomerEntityRepository extends PagingAndSortingRepository<Customer, UUID>, CrudRepository<Customer, UUID> {
    List<Customer> findByFirstNameStartingWithIgnoreCase(@Param("firstName") String firstName);

    @Override @RestResource(exported = false)
    void deleteById(UUID id);
}
// GET /customers  ·  GET /customers/search/findByFirstNameStartingWithIgnoreCase?firstName=an
```

## 7.8 Spring Data MongoDB

**O que é**: integração com o **MongoDB**, banco de **documentos** (JSON/BSON) de esquema flexível. Cada objeto vira um documento numa *collection*.

- **Mapeamento**: `@Document`, `@Id` (o correto é `org.springframework.data.annotation.Id`), `@Field`, `@Indexed`; tipos aninhados (`List`, `Map<String, Object>`) viram subdocumentos.
- **Repositório**: `MongoRepository<T, String>` com *query methods* (`findByEventId`). `MongoTemplate` para consultas dinâmicas e agregações.
- **Auditoria**: `@CreatedDate`, `@LastModifiedDate` com `@EnableMongoAuditing`.
- **Eventos de ciclo de vida**: `AbstractMongoEventListener` (`onBeforeSave`, `onAfterSave`, `onAfterDelete`).
- **Quando usar**: esquema variável, dados lidos como um documento inteiro (catálogos, metadados, logs). **Evite** para dados fortemente relacionais e com transações entre várias entidades.
- UUIDs: configure a representação (`spring.mongodb.representation.uuid=standard`).

**Para que serve**: armazenar estruturas heterogêneas sem criar colunas/tabelas para cada variação.

**Exemplo**:

```java
@Document
public class EventMetadata {
    @Id private String id;
    private UUID eventId;
    private Map<String, Object> technicalRequirements;   // estrutura livre
    @CreatedDate private Instant createdOn;
}
public interface EventMetadataRepository extends MongoRepository<EventMetadata, String> {
    Optional<EventMetadata> findByEventId(UUID eventId);
}
```

## 7.9 Spring Data Redis e cache

**O que é**: **Redis** é um banco **chave-valor em memória**, muito rápido, com estruturas (strings, hashes, listas, sets, sorted sets) e **expiração (TTL)**. Clientes: Lettuce ou Jedis.

- **`@RedisHash(value = "seat_locks", timeToLive = 30)`**: entidade guardada como *hash*; o Redis a **apaga sozinho** após 30 s (trava temporária sem job de limpeza). `@Indexed` permite buscar por outro campo.
- **`RedisTemplate`/`StringRedisTemplate`**: acesso de baixo nível. Para **travas**, use operação **atômica** (`setIfAbsent` com expiração = `SET key value NX EX 30`); fazer `existsById` e depois `save` tem **corrida** (duas requisições passam juntas).
- **Cache declarativo** (`@EnableCaching` + um `CacheManager` como `RedisCacheManager`):
  - `@Cacheable("nome")`: guarda o retorno; na próxima chamada com a mesma chave, o método **nem executa**. `unless = "#result.isEmpty()"` evita cachear vazio.
  - `@CachePut`: executa e atualiza o cache. `@CacheEvict`: invalida.
  - O valor cacheado precisa ser **serializável**; defina **TTL**, senão nunca expira. Cuidado com dados desatualizados (*stale*).
- Outros usos: sessões, filas, *rate limiting*, ranking, pub/sub.

**Para que serve**: reduzir latência e carga no banco (cache), travas distribuídas e dados temporários.

**Exemplo**:

```java
@Cacheable(value = "vitrine", unless = "#result.isEmpty()")
public List<EventoOutput> listar() { return repo.findAll().stream().map(EventoOutput::from).toList(); }

@RedisHash(value = "seat_locks", timeToLive = 30)
class SeatLock {
    @Id String id;
    @Indexed String customerId;
}
```

## 7.10 Eventos, assincronismo e agendamento

**O que é**: formas de executar trabalho **fora do fluxo principal** e de **desacoplar** partes do sistema.

- **Eventos de aplicação**: `ApplicationEventPublisher.publishEvent(new ClienteCriado(...))` + `@EventListener` em outro componente (padrão **Observer**). Mantém contextos desacoplados (cadastro não conhece vendas). Use **`@TransactionalEventListener(phase = AFTER_COMMIT)`** para reagir **só depois do commit** (evitar evento de algo que sofreu rollback). Eventos em memória se perdem se a aplicação cair; para garantia use *outbox* ou mensageria (Kafka, RabbitMQ).
- **`@Async`** (com `@EnableAsync`): executa o método em outra thread; pode devolver `void` ou `CompletableFuture<T>`. Funciona via proxy (autoinvocação não vale). Combine com `CompletableFuture.join` para paralelizar buscas.
- **`@Scheduled`** (com `@EnableScheduling`): `fixedRate`, `fixedDelay`, `cron = "0 0 * * * *"`.
- **Threads virtuais**: `spring.threads.virtual.enabled=true` faz o Boot usar threads virtuais para requisições e tarefas.
- **Mensageria** (`spring-kafka`, `spring-amqp`): comunicação assíncrona **entre** serviços, com garantia de entrega e desacoplamento temporal.

**Para que serve**: não bloquear a requisição, reagir a acontecimentos e rodar rotinas periódicas.

**Exemplo**:

```java
record ClienteCriado(String id, String nome) {}

@Service class Cadastro {
    private final ApplicationEventPublisher publisher;
    Cadastro(ApplicationEventPublisher p) { this.publisher = p; }
    @Transactional void criar(Cliente c) { repo.save(c); publisher.publishEvent(new ClienteCriado(c.getId(), c.getNome())); }
}

@Component class VendasListener {
    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    void on(ClienteCriado e) { /* cria cópia local do cliente */ }
}

@Scheduled(cron = "0 0 3 * * *") void limpezaNoturna() { }
```

## 7.11 Spring Security

**O que é**: framework de **autenticação** (*quem é você*) e **autorização** (*o que pode fazer*). Funciona como uma **cadeia de filtros servlet** executada **antes** do controller (`SecurityFilterChain`).

- **Fluxo de autenticação**: o filtro monta um `Authentication` não autenticado → `AuthenticationManager` → `DaoAuthenticationProvider` → **`UserDetailsService.loadUserByUsername`** (busca o usuário) e **`PasswordEncoder.matches`** (confere o hash) → `Authentication` autenticado → guardado no **`SecurityContext`** (sessão).
- **`UserDetails`**: contrato do usuário (`getUsername`, `getPassword`, `getAuthorities`, flags da conta). Papéis são `GrantedAuthority` (`ROLE_ADMIN`).
- **Senhas**: **nunca em texto puro**; `BCryptPasswordEncoder` (salt + custo lento) ou `DelegatingPasswordEncoder` (`{bcrypt}...`).
- **Autorização em dois níveis**: **por URL** (`authorizeHttpRequests(a -> a.requestMatchers("/admin/**").hasRole("ADMIN").anyRequest().authenticated())`: **negar por padrão**) e **por método** (`@EnableMethodSecurity` + `@PreAuthorize("hasRole('ADMIN')")`). `hasRole('X')` procura a authority `ROLE_X`.
- **401 x 403**: **401** = não autenticado; **403** = autenticado sem permissão. Configure o `AuthenticationEntryPoint` para devolver 401 (sem ele o padrão pode devolver 403).
- **`@AuthenticationPrincipal`** injeta o usuário logado no controller.
- **Sessão x stateless**: login por **sessão** (cookie `JSESSIONID`) ou **JWT**/**OAuth2** (token no cabeçalho `Authorization: Bearer`; sem estado no servidor). **CSRF**: protege requisições forjadas quando o navegador envia cookie automaticamente; só desabilite em API *stateless* com token, e **mantenha ativo com sessão por cookie**.
- **Filtro de autenticação customizado**: estenda `UsernamePasswordAuthenticationFilter` e sobrescreva `attemptAuthentication` (por exemplo, para ler login em **JSON**); registre com `addFilterAt`. Atenção: um `Filter` declarado como `@Component` também é registrado pelo Boot na cadeia do servlet (pode rodar duas vezes; desative com `FilterRegistrationBean`).
- **Autorização de dados** (quais registros o usuário vê) é regra de negócio: resolva no caso de uso, por exemplo com **Strategy** por escopo (`OWN`/`ALL`).
- Também: CORS, headers de segurança, *method security* com SpEL, OAuth2 Resource Server, `@WithMockUser` em testes.

**Para que serve**: proteger endpoints e dados, controlar acesso por papel e guardar credenciais com segurança.

**Exemplo**:

```java
@Configuration @EnableWebSecurity @EnableMethodSecurity
class SecurityConfig {
    @Bean SecurityFilterChain chain(HttpSecurity http) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)      // só em API stateless
            .authorizeHttpRequests(a -> a
                .requestMatchers("/api/auth/login").permitAll()
                .anyRequest().authenticated())
            .build();
    }
    @Bean PasswordEncoder encoder() { return new BCryptPasswordEncoder(); }
}

@RestController class PropostaController {
    @PostMapping("/proposals") @PreAuthorize("hasRole('INFLUENCER')")
    ProposalResponse criar(@RequestBody CreateProposalRequest r, @AuthenticationPrincipal User user) { /* ... */ return null; }
}
```

## 7.12 Spring Cloud OpenFeign e resiliência

**O que é**: **OpenFeign** é um cliente HTTP **declarativo**: você descreve o contrato em uma **interface** com as anotações do Spring MVC e o framework gera a implementação (no lugar de `RestTemplate`/`WebClient` verbosos).

- **`@FeignClient(name = "...", url = "...")`** + **`@EnableFeignClients`** na aplicação. Métodos usam `@GetMapping`, `@PostMapping`, `@PathVariable`, `@RequestBody`, `@RequestHeader`.
- **Configuração por propriedades** por nome do cliente: `spring.cloud.openfeign.client.config.<nome>.url`, `.logger-level=full`, `.default-request-headers.*`, `connect-timeout`, `read-timeout`.
- **Resiliência**: chamadas remotas **falham ou ficam lentas**. Com `spring.cloud.openfeign.circuitbreaker.enabled=true` e **Resilience4j**, o **Circuit Breaker** monitora as chamadas: `CLOSED` (normal) → muitas falhas → `OPEN` (falha imediata, sem tocar o serviço) → após um tempo `HALF-OPEN` (testa algumas) → volta a `CLOSED`. Outras proteções: **timeout** (sempre), **retry** com *backoff*, **bulkhead** (isola recursos), **rate limiter**.
- **Fallback**: resposta alternativa quando a chamada falha ou o circuito está aberto (`fallback = X.class`, ou `fallbackFactory` para ver a causa). **Decisão de negócio**: um fallback que devolve "lista vazia" pode significar **fail-open** (seguir como se estivesse tudo bem), perigoso em compliance/segurança; o **fail-closed** (barrar ou mandar para revisão manual) é o mais seguro.
- **Camada anticorrupção**: DTOs externos convertem a si mesmos para o domínio (`toDomain()`), normalizando `null` e isolando o domínio de mudanças do fornecedor.
- **Mocks de APIs externas** (Mockoon, WireMock) permitem desenvolver e testar cenários (erro, lentidão) sem o fornecedor real.
- Alternativas: `RestClient`, `WebClient`, Spring Cloud LoadBalancer/Gateway, **service discovery** (Eureka/Consul), configuração centralizada.

**Para que serve**: integrar serviços de forma limpa e tolerar falhas de dependências externas sem derrubar a aplicação.

**Exemplo**:

```java
@FeignClient(name = "sanction-client", fallback = SanctionClient.Fallback.class)
public interface SanctionClient {
    @GetMapping("/sanctions/companies/{registro}")
    SanctionResult consultar(@PathVariable String registro);

    @Component
    class Fallback implements SanctionClient {
        public SanctionResult consultar(String registro) {
            throw new ServicoIndisponivelException("Sanções fora do ar");   // fail-closed
        }
    }
}
```

```properties
spring.cloud.openfeign.client.config.sanction-client.url=http://localhost:3001
spring.cloud.openfeign.circuitbreaker.enabled=true
```

## 7.13 Testes no Spring

**O que é**: o `spring-boot-starter-test` traz JUnit 5, Mockito, AssertJ, Hamcrest, JSONassert e utilitários de teste do Spring.

- **`@SpringBootTest`**: sobe o **contexto completo** (lento; use para integração). `webEnvironment = RANDOM_PORT` + **`TestRestTemplate`** testa a API de ponta a ponta.
- **Testes de fatia (*slices*)**: carregam só parte do contexto: **`@WebMvcTest`** (controllers + `MockMvc`), **`@DataJpaTest`** (JPA + banco em memória), `@DataMongoTest`, `@JsonTest`, `@RestClientTest`.
- **`MockMvc`**: chama controllers sem subir servidor (`mockMvc.perform(post("/tasks")...).andExpect(status().isCreated())`).
- **Mocks no contexto**: `@MockitoBean` (Spring 6.2+/Boot 3.4+; substitui o antigo `@MockBean`) e `@MockitoSpyBean`.
- **Segurança**: `@WithMockUser(roles = "ADMIN")`, `SecurityMockMvcRequestPostProcessors`.
- **Unitário puro** (sem Spring): `@ExtendWith(MockitoExtension.class)` com `@Mock`/`@InjectMocks`; é o mais rápido. Evite `@SpringBootTest` quando só precisa de Mockito.
- **Banco real**: **Testcontainers** (`@ServiceConnection` liga o contêiner à aplicação). `@DirtiesContext` descarta o contexto (lento, use com parcimônia). `@Sql` carrega dados.
- **Spring REST Docs**: os testes geram os *snippets* que o Asciidoctor junta na documentação.

**Para que serve**: validar da regra de negócio ao fluxo HTTP completo, com a velocidade certa em cada nível da pirâmide.

**Exemplo**:

```java
@WebMvcTest(TaskController.class)
class TaskControllerTest {
    @Autowired MockMvc mvc;
    @MockitoBean CreateTaskUseCase useCase;

    @Test
    void deveRetornar400QuandoTituloVazio() throws Exception {
        mvc.perform(post("/tasks").contentType(MediaType.APPLICATION_JSON)
                .content("{\"title\":\"\"}"))
           .andExpect(status().isBadRequest());
    }
}

@DataJpaTest
class ClienteRepositoryTest {
    @Autowired ClienteRepository repo;
    @Test void deveSalvar() { assertNotNull(repo.save(new Cliente("Ana")).getId()); }
}
```

## 7.14 Observabilidade, empacotamento e execução

**O que é**: saber **o que a aplicação está fazendo em produção** e entregá-la de forma reprodutível.

- **Actuator**: endpoints de operação (`/actuator/health`, `info`, `metrics`, `prometheus`, `env`, `loggers`). Exponha só o necessário (`management.endpoints.web.exposure.include`) e **proteja**; `management.endpoint.health.show-details=always` mostra o estado de cada dependência (banco, Redis).
- **Micrometer**: fachada de métricas (como o SLF4J para logs); `MeterRegistry` e *tags* comuns (`application`). **Prometheus** coleta (*scrape*) as métricas; **Grafana** monta dashboards. As **três bases** de observabilidade: **logs**, **métricas** e **traces** (OpenTelemetry/Zipkin para rastrear uma requisição entre serviços).
- **Docker**: o `Dockerfile` ou o plugin **Jib** (sem Dockerfile, imagem em camadas) geram a imagem; **Docker Compose** sobe aplicação + banco + Prometheus + Grafana, com `healthcheck` e `depends_on: condition: service_healthy`; `mem_limit` limita memória.
- **Boas práticas**: configuração por variável de ambiente, **12-factor app**, logs no *stdout*, *graceful shutdown*, health checks de liveness/readiness, versionamento da imagem.

**Para que serve**: detectar problemas rápido, entender desempenho e implantar de forma previsível.

**Exemplo**:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
```

```yaml
# compose.yml (trecho)
services:
  db:
    image: mysql:8.0
    environment: { MYSQL_ROOT_PASSWORD: root, MYSQL_DATABASE: app }
    healthcheck: { test: ["CMD", "mysqladmin", "ping", "-h", "localhost"], interval: 5s, retries: 10 }
  app:
    image: minha-app:1.0
    depends_on: { db: { condition: service_healthy } }
    ports: ["8080:8080"]
```

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
