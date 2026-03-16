# DSL `WebNavi`

## Descrição Resumida da DSL

A **WebNavi** é uma linguagem específica de domínio (DSL) voltada para a modelagem de navegação em websites por meio de autômatos finitos.

A proposta da linguagem é representar páginas ou telas como estados, ações do usuário como eventos e mudanças de navegação como transições entre estados.

A motivação do projeto é aproximar conceitos de linguagens formais e autômatos de um domínio intuitivo e cotidiano: o fluxo de navegação de sistemas web, incluindo ações como login, acesso a perfil, navegação em páginas internas e logout.

A relevância da DSL está em permitir a descrição formal de fluxos de interface de maneira clara, compacta e compreensível, servindo tanto como exercício de especificação de linguagem quanto como base para futuras validações semânticas e simulações.

## Slides

[Apresentação em PDF](WebNavi.pdf)

## Sintaxe da Linguagem na Forma de Tutorial

A WebNavi possui uma sintaxe declarativa e simples. Um programa descreve um website, seus estados e suas transições.

### 1. Declarando o website

Todo programa começa com a declaração do sistema modelado:

```txt
website portal;
```

Nesse exemplo, `portal` é o nome do website.

### 2. Declarando os estados

Os estados representam páginas, telas ou seções do website:

```txt
states home, login, dashboard, perfil, erro;
```

Cada identificador representa um estado possível da navegação.

### 3. Definindo o estado inicial

O estado inicial representa a página em que o fluxo começa:

```txt
initial home;
```

### 4. Definindo estados finais

Os estados finais representam estados de aceitação ou de término relevante do fluxo:

```txt
final dashboard, perfil;
```

Essa declaração é opcional.

### 5. Declarando transições

As transições representam eventos que levam de um estado a outro.

A forma geral é:

```txt
estado_origem -> estado_destino on evento;
```

Exemplo:

```txt
home -> login on clicar_login;
login -> dashboard on autenticar_ok;
login -> erro on autenticar_falha;
```

### 6. Exemplo completo

```txt
website portal;
states home, login, dashboard, perfil, erro;
initial home;
final dashboard, perfil;

home -> login on clicar_login;
login -> dashboard on autenticar_ok;
login -> erro on autenticar_falha;
erro -> login on tentar_novamente;
dashboard -> perfil on abrir_perfil;
perfil -> dashboard on voltar;
dashboard -> home on logout;
```

Nesse fluxo:
- o usuário começa em `home`;
- acessa `login`;
- pode autenticar com sucesso e seguir para `dashboard`;
- pode falhar e ir para `erro`;
- pode acessar `perfil` e retornar;
- pode encerrar a navegação com `logout`.

## Gramática da Linguagem

### Gramática Léxica (ANTLR4)

```antlr
lexer grammar WebNavLexer;

WEBSITE : [Ww][Ee][Bb][Ss][Ii][Tt][Ee] ;
STATES  : [Ss][Tt][Aa][Tt][Ee][Ss] ;
INITIAL : [Ii][Nn][Ii][Tt][Ii][Aa][Ll] ;
FINAL   : [Ff][Ii][Nn][Aa][Ll] ;
ON      : [Oo][Nn] ;

ARROW     : '->' ;
SEMICOLON : ';' ;
COMMA     : ',' ;

IDENT     : [a-zA-Z][a-zA-Z0-9_]* ;

WS        : [ \t\r\n]+ -> skip ;
```

### Gramática Sintática (ANTLR4)

```antlr
parser grammar WebNavParser;

options { tokenVocab = WebNavLexer; }

program
    : websiteDecl statesDecl initialDecl finalDecl? transitionDecl+ EOF
    ;

websiteDecl
    : WEBSITE IDENT SEMICOLON
    ;

statesDecl
    : STATES identList SEMICOLON
    ;

initialDecl
    : INITIAL IDENT SEMICOLON
    ;

finalDecl
    : FINAL identList SEMICOLON
    ;

transitionDecl
    : IDENT ARROW IDENT ON IDENT SEMICOLON
    ;

identList
    : IDENT ( COMMA IDENT )*
    ;
```

### Forma Abstrata da Gramática

```ebnf
program        = websiteDecl, statesDecl, initialDecl, [finalDecl], transitionDecl, {transitionDecl} ;
websiteDecl    = "website", IDENT, ";" ;
statesDecl     = "states", identList, ";" ;
initialDecl    = "initial", IDENT, ";" ;
finalDecl      = "final", identList, ";" ;
transitionDecl = IDENT, "->", IDENT, "on", IDENT, ";" ;
identList      = IDENT, {",", IDENT} ;
```

## Exemplos Selecionados

### Exemplo 1 — fluxo de login

```txt
website portal;
states home, login, dashboard, erro;
initial home;
final dashboard;

home -> login on clicar_login;
login -> dashboard on autenticar_ok;
login -> erro on autenticar_falha;
erro -> login on tentar_novamente;
```

**Resultado alcançado:**
A gramática reconhece um fluxo básico de autenticação, separando claramente o caminho de sucesso (`dashboard`) e o caminho de falha (`erro`).

---

### Exemplo 2 — navegação interna

```txt
website sistema;
states home, dashboard, perfil, configuracoes;
initial home;
final perfil, configuracoes;

home -> dashboard on entrar;
dashboard -> perfil on abrir_perfil;
dashboard -> configuracoes on abrir_configuracoes;
perfil -> dashboard on voltar;
configuracoes -> dashboard on salvar;
```

**Resultado alcançado:**
A linguagem representa adequadamente múltiplas possibilidades de navegação a partir de um estado central (`dashboard`).

---

### Exemplo 3 — fluxo de e-commerce

```txt
website loja;
states home, catalogo, produto, carrinho, checkout, pedido;
initial home;
final pedido;

home -> catalogo on ver_produtos;
catalogo -> produto on selecionar_produto;
produto -> carrinho on adicionar_ao_carrinho;
carrinho -> checkout on finalizar_compra;
checkout -> pedido on pagamento_aprovado;
```

**Resultado alcançado:**
A DSL descreve um fluxo sequencial de compra até o estado final `pedido`, mostrando sua aplicação também em domínios além do login.

---

### Exemplo 4 — fluxo com logout

```txt
website portal;
states home, login, dashboard, perfil;
initial home;
final home;

home -> login on clicar_login;
login -> dashboard on autenticar_ok;
dashboard -> perfil on abrir_perfil;
perfil -> dashboard on voltar;
dashboard -> home on logout;
```

**Resultado alcançado:**
A linguagem permite representar o ciclo completo de entrada, navegação e encerramento de sessão.

# Referências Bibliográficas
- SANTANCHÈ, André. ANTLR4 Lab. Program2Learn. Disponível em: https://santanche.github.io/program2learn/formal-languages/antlr4-lab/
- INKLE STUDIOS. Ink. Disponível em: https://www.inklestudios.com/ink/
- Material da disciplina e laboratório ANTLR4 utilizado em aula.
