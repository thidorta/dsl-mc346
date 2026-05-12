# DSL WebNavi

## Descrição Resumida da DSL

A **WebNavi** é uma Linguagem de Domínio Específico (DSL) em português para especificação e validação de fluxos de navegação web.

A linguagem foi criada para representar, de forma declarativa e legível, como um usuário navega por um sistema web: quais páginas existem, quais elementos aparecem em cada página, quais ações disparam mudanças de tela, quais condições precisam ser satisfeitas e quais regras globais devem ser preservadas.

A motivação do projeto está no fato de que fluxos web normalmente são descritos em documentos, protótipos ou conversas, enquanto testes automatizados são implementados separadamente em ferramentas como Cypress ou Playwright. Essa separação pode gerar inconsistência entre documentação, regra de negócio e teste.

A WebNavi busca reduzir essa distância ao transformar o fluxo web em um artefato estruturado, legível e validável. A linguagem usa autômatos finitos como base formal: páginas são estados, ações do usuário são eventos e transições representam mudanças entre páginas. Porém, diferente de uma linguagem genérica de autômatos, a WebNavi inclui conceitos próprios do domínio web, como rotas, botões, campos, links, eventos, condições, efeitos, dados de teste, invariantes e fluxos executáveis/simuláveis.

Nesta entrega, a WebNavi é apresentada como uma especificação de linguagem e um protótipo parcial de implementação. A linguagem foi projetada para permitir análise estática, simulação de fluxos e futura geração de testes automatizados, como Cypress ou Playwright.

---

## Slides

PDF da apresentação:

- [Apresentação WebNavi](WebNavi_Apresentacao%20(1).pdf)

---

## Sintaxe da Linguagem na Forma de Tutorial

A sintaxe da WebNavi foi pensada para ser próxima do português e fácil de ler por pessoas que conhecem o fluxo do sistema, mesmo que não conheçam frameworks de teste.

Algumas decisões de design da linguagem:

| Decisão | Justificativa |
|---|---|
| Sem ponto-e-vírgula | O fim da linha encerra a declaração |
| Sem chaves `{}` | Blocos são encerrados com a palavra `fim` |
| Sem operadores simbólicos | `->` vira `para`, `=` vira `recebe`, `==` vira `igual a` |
| Palavras-chave em português | `site`, `pagina`, `elemento`, `transicao`, `fluxo`, `dados` |
| Transições com eventos explícitos | O elemento e o evento são separados, como em `via botao_entrar clicado` |

### 1. Declaração do site

Todo arquivo WebNavi começa com o nome do site ou sistema modelado.

```webnavi
site loja
```

### 2. Declaração de páginas

Uma página representa um estado do fluxo de navegação. Cada página pode ter uma rota e elementos de interface.

```webnavi
pagina login
  rota /login
fim
```

### 3. Declaração de elementos

Elementos representam objetos da interface com os quais o usuário pode interagir, como botões, campos, links, avisos e seletores.

```webnavi
pagina login
  rota /login
  elemento campo_email entrada obrigatorio formato email seletor "#email"
  elemento campo_senha entrada obrigatorio formato senha minimo 8 seletor "#senha"
  elemento botao_entrar botao texto "Entrar" seletor "#entrar"
fim
```

Nesse exemplo:

- `campo_email` é uma entrada obrigatória com formato de e-mail;
- `campo_senha` é uma entrada obrigatória com formato de senha e tamanho mínimo;
- `botao_entrar` é um botão que pode disparar uma transição.

### 4. Estado inicial e final

A linguagem permite declarar onde o fluxo começa e onde ele pode terminar.

```webnavi
inicio em inicio
final em confirmacao
```

### 5. Transições

Transições representam mudanças entre páginas. Elas indicam:

- página de origem;
- página de destino;
- elemento que dispara a transição;
- evento realizado sobre esse elemento;
- condição opcional;
- efeito opcional.

```webnavi
transicao de login para catalogo
  via botao_entrar clicado
  somente se campo_email preenchido e campo_senha preenchido
  entao sessao.autenticado recebe verdadeiro
fim
```

Esse exemplo indica que o usuário sai da página `login` e vai para `catalogo` quando o botão `botao_entrar` é clicado, desde que os campos de e-mail e senha estejam preenchidos. Como efeito, a sessão passa a estar autenticada.

Essa estrutura responde a uma questão central do projeto: a transição não é apenas uma seta entre estados. Ela tem um evento, acontece sobre um elemento específico e pode depender de condições.

### 6. Invariantes

Invariantes representam regras globais que devem ser respeitadas em qualquer ponto do fluxo.

```webnavi
invariante checkout exige autenticacao
  sempre que estiver em checkout entao sessao.autenticado igual a verdadeiro
fim
```

Esse exemplo expressa que a página `checkout` só deve ser acessada se o usuário estiver autenticado.

### 7. Dados de teste

Dados permitem declarar valores reutilizáveis nos fluxos.

```webnavi
dados usuario_valido
  email "user@loja.com"
  senha "Senha@123"
fim
```

### 8. Fluxos

Fluxos representam sequências concretas de navegação. Enquanto as transições descrevem a navegação possível, os fluxos descrevem uma navegação executada ou simulável.

```webnavi
fluxo compra completa
  comecar em inicio
  passo clicar botao_login aguardar pagina login
  passo preencher campo_email com usuario_valido.email
  passo preencher campo_senha com usuario_valido.senha
  passo clicar botao_entrar aguardar pagina catalogo
  terminar em catalogo
fim
```

A separação entre transições e fluxos é importante:

- **transições** definem o espaço de navegação possível;
- **fluxos** definem cenários concretos dentro desse espaço.

---

## Gramática da Linguagem

A seguir está uma versão resumida da gramática da WebNavi em EBNF.

```ebnf
programa      ::= declaracao_site
                  declaracao_pagina*
                  declaracao_inicio
                  declaracao_final?
                  transicao*
                  invariante*
                  dados*
                  fluxo*

site          ::= 'site' IDENT

pagina        ::= 'pagina' IDENT NEWLINE
                  INDENT rota? elemento* DEDENT
                  'fim'

rota          ::= 'rota' CAMINHO

elemento      ::= 'elemento' IDENT tipo_elem modificador*

tipo_elem     ::= 'botao'
                | 'entrada'
                | 'link'
                | 'formulario'
                | 'modal'
                | 'aviso'
                | 'selecao'
                | 'caixa'

modificador   ::= 'obrigatorio'
                | 'formato' tipo_fmt
                | 'minimo' NUMERO
                | 'maximo' NUMERO
                | 'texto' STRING
                | 'seletor' STRING
                | 'visivel' 'quando' IDENT

tipo_fmt      ::= 'email'
                | 'senha'
                | 'cpf'
                | 'cnpj'
                | 'telefone'
                | 'cep'
                | 'data'

inicio        ::= 'inicio' 'em' IDENT

final         ::= 'final' 'em' IDENT (',' IDENT)*

transicao     ::= 'transicao' 'de' IDENT 'para' IDENT NEWLINE
                  INDENT via somente_se? entao? DEDENT
                  'fim'

via           ::= 'via' IDENT acao_evento

acao_evento   ::= 'clicado'
                | 'enviado'
                | 'selecionado'

somente_se    ::= 'somente' 'se' expressao

entao         ::= 'entao' atribuicao ('e' atribuicao)*

atribuicao    ::= ACESSO 'recebe' valor

valor         ::= IDENT
                | STRING
                | NUMERO
                | 'verdadeiro'
                | 'falso'

invariante    ::= 'invariante' IDENT+ NEWLINE
                  INDENT 'sempre' 'que' expressao 'entao' expressao DEDENT
                  'fim'

dados         ::= 'dados' IDENT NEWLINE
                  INDENT campo+ DEDENT
                  'fim'

campo         ::= IDENT valor

fluxo         ::= 'fluxo' IDENT+ NEWLINE
                  INDENT comecar passo+ terminar DEDENT
                  'fim'

comecar       ::= 'comecar' 'em' IDENT

terminar      ::= 'terminar' 'em' IDENT

passo         ::= 'passo' acao_passo ('aguardar' espera)?

acao_passo    ::= 'clicar' IDENT
                | 'preencher' IDENT 'com' ref_dados
                | 'navegar' IDENT

ref_dados     ::= ACESSO
                | STRING

espera        ::= 'pagina' IDENT
                | IDENT 'visivel'
                | NUMERO 'segundos'

expressao     ::= expressao ('e' | 'ou') expressao
                | 'nao' expressao
                | comparacao

comparacao    ::= IDENT ('preenchido' | 'vazio' | 'visivel' | 'oculto')
                | ACESSO 'maior' 'que' NUMERO
                | ACESSO 'menor' 'que' NUMERO
                | ACESSO 'igual' 'a' valor
                | ACESSO 'diferente' 'de' valor
                | 'estiver' 'em' IDENT

IDENT         ::= [a-zA-Z_][a-zA-Z0-9_]*
ACESSO        ::= IDENT '.' IDENT
CAMINHO       ::= '/' [a-zA-Z0-9/_:-]*
STRING        ::= '"' (~["])* '"'
NUMERO        ::= [0-9]+
```

A gramática acima representa a especificação da linguagem. A implementação atual cobre um subconjunto funcional dessa gramática. A implementação completa de um parser formal, por exemplo com ANTLR ou outra ferramenta, é considerada um trabalho futuro.

---

## Notebook

A implementação atual está organizada em um notebook, que funciona como protótipo executável da linguagem.

O fluxo da implementação é: primeiro recebemos uma especificação WebNavi; depois o parser interpreta as declarações; em seguida montamos uma representação interna com páginas, elementos, transições, dados e fluxos; por fim, realizamos uma análise estática e produzimos saídas como relatório e exemplos de tradução para testes.

A WebNavi, enquanto linguagem, não gera Cypress sozinha. O que ela faz é fornecer uma especificação estruturada que permite implementar um gerador de testes.

No protótipo, exploramos como alguns fluxos podem ser traduzidos para Cypress em casos simples. A geração completa e robusta de Cypress é parte dos trabalhos futuros.

Notebook da implementação:

- [Notebook WebNavi](WebNavi_notebook.ipynb)

## Exemplos Selecionados

### Exemplo 1 — Login simples

Este exemplo apresenta um fluxo básico de autenticação.

```webnavi
site portal

pagina inicio
  rota /
  elemento botao_login botao texto "Acessar" seletor "#acessar"
fim

pagina login
  rota /login
  elemento campo_email entrada obrigatorio formato email seletor "#email"
  elemento campo_senha entrada obrigatorio formato senha minimo 6 seletor "#senha"
  elemento botao_entrar botao texto "Entrar" seletor "#entrar"
  elemento aviso_erro aviso visivel quando login_falhou
fim

pagina painel
  rota /painel
  elemento botao_sair botao texto "Sair" seletor "#sair"
fim

inicio em inicio
final em painel

transicao de inicio para login
  via botao_login clicado
fim

transicao de login para painel
  via botao_entrar clicado
  somente se campo_email preenchido e campo_senha preenchido
  entao sessao.autenticado recebe verdadeiro
fim

transicao de login para login
  via botao_entrar clicado
  somente se campo_email vazio ou campo_senha vazio
  entao login_falhou recebe verdadeiro
fim

invariante painel exige autenticacao
  sempre que estiver em painel entao sessao.autenticado igual a verdadeiro
fim

dados usuario_valido
  email "ana@portal.com"
  senha "Senha@456"
fim

fluxo login com sucesso
  comecar em inicio
  passo clicar botao_login aguardar pagina login
  passo preencher campo_email com usuario_valido.email
  passo preencher campo_senha com usuario_valido.senha
  passo clicar botao_entrar aguardar pagina painel
  terminar em painel
fim
```

#### Resultado esperado

Uma ferramenta implementada sobre a WebNavi poderia produzir uma análise como:

```text
WebNavi — portal.webnavi
========================

Páginas: 3
Transições: 3
Invariantes: 1
Fluxos: 1

Alcançabilidade:
  [ok] inicio — ponto de entrada
  [ok] login — alcançável via inicio
  [ok] painel — alcançável via login

Condições:
  [ok] login > painel e login > login usam condições alternativas

Invariantes:
  [ok] painel exige autenticacao

Resultado: 0 erros
```

---

### Exemplo 2 — Loja virtual com checkout

Este exemplo apresenta um fluxo mais completo, com autenticação, catálogo, carrinho, checkout, confirmação e erro de pagamento.

```webnavi
site loja

pagina inicio
  rota /
  elemento botao_login botao texto "Entrar" seletor "#login"
  elemento link_produtos link texto "Ver produtos"
fim

pagina login
  rota /login
  elemento campo_email entrada obrigatorio formato email seletor "#email"
  elemento campo_senha entrada obrigatorio formato senha minimo 8 seletor "#senha"
  elemento botao_entrar botao texto "Entrar" seletor "#entrar"
  elemento aviso_erro aviso visivel quando login_falhou
fim

pagina catalogo
  rota /produtos
  elemento campo_busca entrada seletor "#busca"
  elemento botao_buscar botao texto "Buscar" seletor "#buscar"
fim

pagina produto
  rota /produto
  elemento botao_adicionar botao texto "Adicionar ao carrinho" seletor "#adicionar"
  elemento seletor_qtd selecao seletor "#quantidade"
fim

pagina carrinho
  rota /carrinho
  elemento botao_finalizar botao texto "Finalizar compra" seletor "#finalizar"
fim

pagina checkout
  rota /checkout
  elemento campo_cartao entrada obrigatorio minimo 16 maximo 16 seletor "#cartao"
  elemento campo_cvv entrada obrigatorio minimo 3 maximo 4 seletor "#cvv"
  elemento campo_validade entrada obrigatorio formato data seletor "#validade"
  elemento botao_pagar botao texto "Pagar" seletor "#pagar"
  elemento aviso_recusa aviso visivel quando pagamento_recusado
fim

pagina confirmacao
  rota /confirmacao
  elemento numero_pedido entrada seletor "#pedido"
fim

pagina erro_pagamento
  rota /checkout/erro
  elemento botao_tentar botao texto "Tentar novamente"
fim

inicio em inicio
final em confirmacao

transicao de inicio para login
  via botao_login clicado
fim

transicao de inicio para catalogo
  via link_produtos clicado
fim

transicao de login para catalogo
  via botao_entrar clicado
  somente se campo_email preenchido e campo_senha preenchido
  entao sessao.autenticado recebe verdadeiro
fim

transicao de login para login
  via botao_entrar clicado
  somente se campo_email vazio ou campo_senha vazio
  entao login_falhou recebe verdadeiro
fim

transicao de catalogo para produto
  via botao_buscar clicado
fim

transicao de produto para carrinho
  via botao_adicionar clicado
  somente se produto.estoque maior que 0
  entao carrinho.quantidade recebe carrinho.quantidade
fim

transicao de carrinho para checkout
  via botao_finalizar clicado
  somente se carrinho.quantidade maior que 0 e sessao.autenticado igual a verdadeiro
fim

transicao de checkout para confirmacao
  via botao_pagar clicado
  somente se campo_cartao preenchido e campo_cvv preenchido e campo_validade preenchido
  entao pedido.numero recebe gerado e carrinho.quantidade recebe 0
fim

transicao de checkout para erro_pagamento
  via botao_pagar clicado
  somente se pagamento_recusado igual a verdadeiro
fim

transicao de erro_pagamento para checkout
  via botao_tentar clicado
  entao pagamento_recusado recebe falso
fim

invariante checkout exige autenticacao
  sempre que estiver em checkout entao sessao.autenticado igual a verdadeiro
fim

invariante checkout exige carrinho nao vazio
  sempre que estiver em checkout entao carrinho.quantidade maior que 0
fim

invariante confirmacao exige pedido gerado
  sempre que estiver em confirmacao entao numero_pedido visivel
fim

dados usuario_valido
  email "user@loja.com"
  senha "Senha@123"
fim

dados cartao_valido
  numero "4111111111111111"
  cvv "123"
  validade "12/28"
fim

fluxo compra completa
  comecar em inicio
  passo clicar botao_login aguardar pagina login
  passo preencher campo_email com usuario_valido.email
  passo preencher campo_senha com usuario_valido.senha
  passo clicar botao_entrar aguardar pagina catalogo
  passo clicar botao_buscar aguardar pagina produto
  passo clicar botao_adicionar aguardar pagina carrinho
  passo clicar botao_finalizar aguardar pagina checkout
  passo preencher campo_cartao com cartao_valido.numero
  passo preencher campo_cvv com cartao_valido.cvv
  passo preencher campo_validade com cartao_valido.validade
  passo clicar botao_pagar aguardar pagina confirmacao
  terminar em confirmacao
fim
```

#### Resultado esperado

Uma ferramenta construída sobre a WebNavi poderia produzir um relatório como:

```text
WebNavi — loja.webnavi
======================

Páginas: 8
Transições: 10
Invariantes: 3
Conjuntos de dados: 2
Fluxos: 1

Alcançabilidade:
  [ok] inicio — ponto de entrada
  [ok] login — alcançável via inicio
  [ok] catalogo — alcançável via inicio ou login
  [ok] produto — alcançável via catalogo
  [ok] carrinho — alcançável via produto
  [ok] checkout — alcançável via carrinho
  [ok] confirmacao — alcançável via checkout
  [ok] erro_pagamento — alcançável via checkout

Condições:
  [ok] login possui transições alternativas para sucesso e falha
  [ok] checkout possui transições alternativas para confirmação e erro de pagamento

Invariantes:
  [ok] checkout exige autenticacao
  [ok] checkout exige carrinho nao vazio
  [ok] confirmacao exige pedido gerado

Avisos:
  [aviso] produto > carrinho depende de estoque maior que 0, mas não há transição alternativa para produto.estoque igual a 0

Resultado: 0 erros, 1 aviso
```

---

## Discussão dos Resultados

A principal evolução da WebNavi nesta entrega foi deixar de ser apenas uma representação textual de autômatos e passar a incorporar conceitos específicos de navegação web.

A linguagem agora separa claramente:

| Conceito | Representação na WebNavi |
|---|---|
| Estado | `pagina` |
| Rota | `rota` |
| Elemento de interface | `elemento` |
| Evento | `clicado`, `enviado`, `selecionado` |
| Transição possível | `transicao de A para B` |
| Condição | `somente se` |
| Efeito | `entao ... recebe ...` |
| Regra global | `invariante` |
| Dados de teste | `dados` |
| Navegação concreta | `fluxo` |

Essa separação responde a questões importantes levantadas na primeira etapa do projeto:

- o que diferencia a linguagem de uma linguagem genérica de autômatos;
- o que dispara uma transição;
- onde estão os eventos;
- se a navegação é possível ou executada;
- como separar a ação do local onde ela acontece.

Na WebNavi, as transições descrevem navegações possíveis, enquanto os fluxos descrevem cenários concretos de navegação. Além disso, a cláusula `via` separa o elemento da interface do evento realizado:

```webnavi
via botao_entrar clicado
```

Nesse exemplo:

- `botao_entrar` é onde a ação acontece;
- `clicado` é o evento que dispara a transição.

A linguagem também foi projetada para permitir futuras ferramentas de análise e geração de testes. No entanto, é importante destacar que a WebNavi, enquanto linguagem, não executa testes sozinha. Ela fornece uma especificação estruturada que pode ser usada por analisadores, simuladores ou geradores de código.

---

## Conclusão

A WebNavi é uma DSL em português para especificação e validação de fluxos de navegação web.

A linguagem usa autômatos como base formal, mas se diferencia de uma linguagem genérica de autômatos ao incorporar elementos próprios do domínio web: páginas, rotas, botões, campos, eventos, condições, efeitos, invariantes, dados e fluxos.

A principal contribuição desta entrega é a definição de uma sintaxe mais expressiva e de uma gramática capaz de representar fluxos web de forma estruturada. A linguagem também estabelece uma separação importante entre navegação possível, descrita pelas transições, e navegação concreta, descrita pelos fluxos.

A implementação atual deve ser entendida como um protótipo parcial. Ela demonstra a viabilidade da proposta, mas ainda não cobre toda a gramática nem todas as validações semânticas desejadas.

Como trabalhos futuros, pretendemos:

- implementar um parser formal completo;
- ampliar a análise semântica;
- detectar estados inalcançáveis e transições inconsistentes;
- verificar invariantes de forma mais robusta;
- implementar geração completa de testes Cypress ou Playwright;
- produzir relatórios de cobertura de fluxos;
- criar uma interface ou editor para arquivos `.webnavi`.

---

# Referências Bibliográficas

- Material da disciplina e laboratório ANTLR4 utilizado em aula.
- Cypress. *Cypress Documentation*. Disponível em: https://docs.cypress.io/
- Playwright. *Playwright Documentation*. Disponível em: https://playwright.dev/
- SANTANCHÈ, André. ANTLR4 Lab. Program2Learn. Disponível em: https://santanche.github.io/program2learn/formal-languages/antlr4-lab/
