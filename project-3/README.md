# DSL `WebNavi`

## Descrição Resumida da DSL

> A WebNavi é uma Linguagem de Domínio Específico (DSL) voltada para a modelagem, validação e análise de fluxos de navegação em aplicações web. A linguagem permite descrever páginas, elementos de interface, eventos, transições, condições, invariantes e fluxos de navegação de forma declarativa e legível.
>
> Nesta terceira etapa do projeto, a WebNavi foi implementada em Scheme/Racket utilizando macros, permitindo que programas escritos na DSL sejam convertidos para uma representação interna estruturada (AST) sobre a qual podem ser realizadas análises e validações.
>
> Fluxos de navegação web normalmente são documentados em textos, diagramas ou protótipos, enquanto sua implementação e seus testes automatizados são desenvolvidos separadamente em ferramentas específicas. Essa separação pode gerar inconsistências entre a documentação, as regras de negócio e os testes executados.
>
> A WebNavi foi criada para fornecer uma representação única e formal desses fluxos, permitindo que a especificação da navegação seja utilizada tanto para documentação quanto para validação automática e geração de artefatos.
>
> Além de definir a sintaxe e a gramática da linguagem, a implementação atual permite construir uma AST por meio de macros, realizar validações semânticas, verificar propriedades do fluxo de navegação e gerar automaticamente código de teste para o framework Cypress. Dessa forma, a linguagem demonstra como conceitos de DSLs e metaprogramação podem ser aplicados ao domínio de aplicações web.

## Slides

> Coloque aqui o link para o PDF da apresentação final.

# Sintaxe da Linguagem WebNavi

## Estrutura Geral

Todo programa WebNavi é definido dentro de um bloco `webnavi`.

```scheme
(webnavi nome-do-site

  declaração*

)
```

Exemplo:

```scheme
(webnavi loja

  (pagina login
    ...)

  (pagina catalogo
    ...)

)
```

---

## Páginas

Uma página representa um estado do fluxo de navegação.

## Sintaxe

```scheme
(pagina nome

  rota?

  elemento*

)
```

## Exemplo

```scheme
(pagina login

  (rota "/login")

  (elemento campo_email entrada)

  (elemento campo_senha entrada)

  (elemento botao_entrar botao)

)
```

---

## Rotas

Rotas identificam o endereço associado a uma página.

## Sintaxe

```scheme
(rota "/caminho")
```

## Exemplo

```scheme
(rota "/checkout")
```

---

## Elementos

Elementos representam componentes da interface.

## Sintaxe

```scheme
(elemento nome tipo modificadores*)
```

## Tipos disponíveis

```scheme
botao
entrada
link
formulario
modal
aviso
selecao
caixa
```

## Exemplo

```scheme
(elemento campo_email entrada)
```

---

## Modificadores

Modificadores adicionam propriedades a elementos.

## Obrigatório

```scheme
obrigatorio
```

Exemplo:

```scheme
(elemento email entrada obrigatorio)
```

---

## Formato

```scheme
formato email
formato senha
formato cpf
formato cnpj
formato telefone
formato cep
formato data
```

Exemplo:

```scheme
(elemento email entrada formato email)
```

---

## Tamanho mínimo

```scheme
minimo NUMERO
```

Exemplo:

```scheme
(elemento senha entrada minimo 8)
```

---

## Tamanho máximo

```scheme
maximo NUMERO
```

Exemplo:

```scheme
(elemento senha entrada maximo 20)
```

---

## Texto

```scheme
texto "conteudo"
```

Exemplo:

```scheme
(elemento entrar botao texto "Entrar")
```

---

## Seletor

```scheme
seletor "#id"
```

Exemplo:

```scheme
(elemento email entrada seletor "#email")
```

---

## Visibilidade Condicional

```scheme
visivel quando IDENT
```

Exemplo:

```scheme
(elemento aviso_erro aviso
          visivel quando login_falhou)
```

---

## Estado Inicial

Define a página inicial do sistema.

## Sintaxe

```scheme
(inicio em pagina)
```

## Exemplo

```scheme
(inicio em login)
```

---

## Estados Finais

Define páginas finais válidas.

## Sintaxe

```scheme
(final em pagina*)
```

## Exemplo

```scheme
(final em confirmacao)
```

ou

```scheme
(final em sucesso erro cancelado)
```

---

## Transições

Representam mudanças de estado.

## Sintaxe

```scheme
(transicao

  de origem

  para destino

  (via elemento evento)

  (somente se expressao)?

  (entao efeito*)?

)
```

## Exemplo

```scheme
(transicao

  de login

  para catalogo

  (via botao_entrar clicado)

)
```

---

## Eventos

Eventos disparam transições.

## Sintaxe

```scheme
(via elemento evento)
```

## Eventos disponíveis

```scheme
clicado
enviado
selecionado
```

## Exemplo

```scheme
(via botao_pagar clicado)
```

---

## Condições

Condições controlam quando uma transição pode ocorrer.

## Sintaxe

```scheme
(somente se expressao)
```

## Exemplo

```scheme
(somente se
  (e
    (preenchido campo_email)
    (preenchido campo_senha)))
```

---

## Efeitos

Efeitos alteram valores do sistema após uma transição.

## Sintaxe

```scheme
(entao

  (recebe alvo valor)

)
```

## Exemplo

```scheme
(entao
  (recebe sessao.autenticado verdadeiro))
```

---

## Expressões

## E lógico

```scheme
(e expr1 expr2 ...)
```

## Ou lógico

```scheme
(ou expr1 expr2 ...)
```

## Negação

```scheme
(nao expr)
```

## Campo preenchido

```scheme
(preenchido campo)
```

## Campo vazio

```scheme
(vazio campo)
```

## Elemento visível

```scheme
(visivel elemento)
```

## Elemento oculto

```scheme
(oculto elemento)
```

## Página atual

```scheme
(estiver em pagina)
```

## Comparação maior

```scheme
(maior acesso que numero)
```

## Comparação menor

```scheme
(menor acesso que numero)
```

## Comparação igual

```scheme
(igual acesso a valor)
```

## Comparação diferente

```scheme
(diferente acesso de valor)
```

---

## Invariantes

Representam regras globais do sistema.

## Sintaxe

```scheme
(invariante nome

  (sempre que
      expressao
      entao
      expressao)

)
```

## Exemplo

```scheme
(invariante checkout_exige_autenticacao

  (sempre que
      (estiver em checkout)

      entao

      (igual sessao.autenticado a verdadeiro))
)
```

---

## Dados

Representam conjuntos de valores reutilizáveis.

## Sintaxe

```scheme
(dados nome

  (campo valor)*

)
```

## Exemplo

```scheme
(dados usuario_valido

  (email "user@loja.com")

  (senha "Senha@123")

)
```

---

## Fluxos

Representam cenários concretos de navegação.

## Sintaxe

```scheme
(fluxo nome

  (comecar em pagina)

  passo+

  (terminar em pagina)

)
```

## Exemplo

```scheme
(fluxo login_sucesso

  (comecar em login)

  (passo clicar botao_entrar
         aguardar pagina catalogo)

  (terminar em catalogo)

)
```

---

## Passos

## Clique simples

```scheme
(passo clicar elemento)
```

## Clique aguardando página

```scheme
(passo clicar elemento
       aguardar pagina destino)
```

## Clique aguardando elemento visível

```scheme
(passo clicar elemento
       aguardar elemento visivel)
```

## Preenchimento

```scheme
(passo preencher campo
       com valor)
```

## Navegação

```scheme
(passo navegar pagina)
```

## Navegação aguardando página

```scheme
(passo navegar pagina
       aguardar pagina destino)
```

---

## Exemplo Completo

```scheme
(webnavi loja

  (pagina login
    (rota "/login")
    (elemento campo_email entrada obrigatorio formato email)
    (elemento campo_senha entrada obrigatorio formato senha)
    (elemento botao_entrar botao))

  (pagina catalogo
    (rota "/catalogo"))

  (inicio em login)

  (final em catalogo)

  (transicao
    de login
    para catalogo
    (via botao_entrar clicado)
    (somente se
      (e
        (preenchido campo_email)
        (preenchido campo_senha)))
    (entao
      (recebe sessao.autenticado verdadeiro)))

  (fluxo login_sucesso
    (comecar em login)
    (passo clicar botao_entrar
           aguardar pagina catalogo)
    (terminar em catalogo))
)
```

## Gramática da Linguagem

```ebnf
programa      ::= (webnavi IDENT declaracao*)
declaracao    ::= pagina | inicio | final | transicao | invariante | dados | fluxo

pagina        ::= (pagina IDENT rota? elemento*)
rota          ::= (rota STRING)
elemento      ::= (elemento IDENT tipo_elem modificador*)
tipo_elem     ::= botao | entrada | link | formulario | modal | aviso | selecao | caixa
modificador   ::= obrigatorio
                | formato tipo_fmt
                | minimo NUMERO
                | maximo NUMERO
                | texto STRING
                | seletor STRING
                | visivel quando IDENT
tipo_fmt      ::= email | senha | cpf | cnpj | telefone | cep | data

inicio        ::= (inicio em IDENT)
final         ::= (final em IDENT*)

transicao     ::= (transicao de IDENT para IDENT via somente? entao?)
via           ::= (via IDENT evento)
evento        ::= clicado | enviado | selecionado
somente       ::= (somente se expressao)
entao         ::= (entao efeito*)
efeito        ::= (recebe ACESSO valor)
valor         ::= STRING | NUMERO | verdadeiro | falso | IDENT | ACESSO

invariante    ::= (invariante IDENT+ (sempre que expressao entao expressao))

dados         ::= (dados IDENT campo*)
campo         ::= (IDENT valor)

fluxo         ::= (fluxo IDENT+ (comecar em IDENT) passo+ (terminar em IDENT))
passo         ::= (passo clicar IDENT)
                | (passo clicar IDENT aguardar pagina IDENT)
                | (passo clicar IDENT aguardar IDENT visivel)
                | (passo preencher IDENT com REF_DADOS)
                | (passo navegar IDENT)
                | (passo navegar IDENT aguardar pagina IDENT)
REF_DADOS     ::= IDENT.IDENT | STRING

expressao     ::= (e expressao+)
                | (ou expressao+)
                | (nao expressao)
                | (preenchido IDENT)
                | (vazio IDENT)
                | (visivel IDENT)
                | (oculto IDENT)
                | (estiver em IDENT)
                | (maior ACESSO que NUMERO)
                | (menor ACESSO que NUMERO)
                | (igual ACESSO a valor)
                | (diferente ACESSO de valor)

IDENT         ::= símbolo Scheme, por exemplo login, campo_email, sessao.autenticado
STRING        ::= texto entre aspas
NUMERO        ::= inteiro
ACESSO        ::= símbolo com ponto, por exemplo sessao.autenticado
```

## Notebook

>
> [WebNavi Notebook](webnavi_notebook.ipynb)
>

## Exemplos Selecionados

> # Exemplo Completo — Loja Virtual

```scheme
(webnavi loja

  (pagina inicio
    (rota "/")
    (elemento botao_login botao texto "Entrar")
    (elemento link_produtos link texto "Ver produtos"))

  (pagina login
    (rota "/login")
    (elemento campo_email entrada obrigatorio formato email seletor "#email")
    (elemento campo_senha entrada obrigatorio formato senha minimo 8 seletor "#senha")
    (elemento botao_entrar botao texto "Entrar" seletor "#entrar")
    (elemento aviso_erro aviso visivel quando login_falhou))

  (pagina catalogo
    (rota "/produtos")
    (elemento campo_busca entrada seletor "#busca")
    (elemento botao_buscar botao texto "Buscar"))

  (pagina produto
    (rota "/produto")
    (elemento botao_adicionar botao texto "Adicionar ao carrinho" seletor "#adicionar")
    (elemento seletor_qtd selecao seletor "#quantidade"))

  (pagina carrinho
    (rota "/carrinho")
    (elemento botao_finalizar botao texto "Finalizar compra" seletor "#finalizar")
    (elemento botao_remover botao texto "Remover"))

  (pagina checkout
    (rota "/checkout")
    (elemento campo_cartao entrada obrigatorio minimo 16 maximo 16 seletor "#cartao")
    (elemento campo_cvv entrada obrigatorio minimo 3 maximo 4 seletor "#cvv")
    (elemento campo_validade entrada obrigatorio formato data)
    (elemento botao_pagar botao texto "Pagar" seletor "#pagar")
    (elemento aviso_recusa aviso visivel quando pagamento_recusado))

  (pagina confirmacao
    (rota "/confirmacao")
    (elemento numero_pedido entrada seletor "#pedido")
    (elemento link_continuar link texto "Continuar comprando"))

  (pagina erro_pagamento
    (rota "/checkout/erro")
    (elemento botao_tentar botao texto "Tentar novamente"))

  (inicio em inicio)

  (final em confirmacao)

  (transicao de inicio para login
    (via botao_login clicado))

  (transicao de inicio para catalogo
    (via link_produtos clicado))

  (transicao de login para catalogo
    (via botao_entrar clicado)
    (somente se
      (e
        (preenchido campo_email)
        (preenchido campo_senha)))
    (entao
      (recebe sessao.autenticado verdadeiro)
      (recebe pagamento_recusado falso)))

  (transicao de login para login
    (via botao_entrar clicado)
    (somente se
      (ou
        (vazio campo_email)
        (vazio campo_senha)))
    (entao
      (recebe login_falhou verdadeiro)))

  (transicao de catalogo para produto
    (via botao_buscar clicado))

  (transicao de produto para carrinho
    (via botao_adicionar clicado)
    (somente se
      (maior produto.estoque que 0))
    (entao
      (recebe carrinho.quantidade 1)))

  (transicao de carrinho para checkout
    (via botao_finalizar clicado)
    (somente se
      (e
        (maior carrinho.quantidade que 0)
        (igual sessao.autenticado a verdadeiro))))

  (transicao de checkout para confirmacao
    (via botao_pagar clicado)
    (somente se
      (e
        (preenchido campo_cartao)
        (preenchido campo_cvv)
        (preenchido campo_validade)
        (igual pagamento_recusado a falso)))
    (entao
      (recebe pedido.numero gerado)
      (recebe carrinho.quantidade 0)))

  (transicao de checkout para erro_pagamento
    (via botao_pagar clicado)
    (somente se
      (igual pagamento_recusado a verdadeiro)))

  (transicao de erro_pagamento para checkout
    (via botao_tentar clicado)
    (entao
      (recebe pagamento_recusado falso)))

  (transicao de confirmacao para inicio
    (via link_continuar clicado))

  (invariante checkout exige autenticacao
    (sempre que
      (estiver em checkout)
      entao
      (igual sessao.autenticado a verdadeiro)))

  (invariante checkout exige carrinho nao vazio
    (sempre que
      (estiver em checkout)
      entao
      (maior carrinho.quantidade que 0)))

  (invariante confirmacao exige pedido gerado
    (sempre que
      (estiver em confirmacao)
      entao
      (visivel numero_pedido)))

  (dados usuario_valido
    (email "user@loja.com")
    (senha "Senha@123"))

  (dados cartao_valido
    (numero "4111111111111111")
    (cvv "123")
    (validade "12/28"))

  (dados cartao_invalido
    (numero "0000000000000000")
    (cvv "000")
    (validade "01/20"))

  (fluxo compra completa
    (comecar em inicio)
    (passo clicar botao_login aguardar pagina login)
    (passo preencher campo_email com usuario_valido.email)
    (passo preencher campo_senha com usuario_valido.senha)
    (passo clicar botao_entrar aguardar pagina catalogo)
    (passo clicar botao_buscar aguardar pagina produto)
    (passo clicar botao_adicionar aguardar pagina carrinho)
    (passo clicar botao_finalizar aguardar pagina checkout)
    (passo preencher campo_cartao com cartao_valido.numero)
    (passo preencher campo_cvv com cartao_valido.cvv)
    (passo preencher campo_validade com cartao_valido.validade)
    (passo clicar botao_pagar aguardar pagina confirmacao)
    (terminar em confirmacao))

  (fluxo pagamento recusado
    (comecar em checkout)
    (passo preencher campo_cartao com cartao_invalido.numero)
    (passo preencher campo_cvv com cartao_invalido.cvv)
    (passo preencher campo_validade com cartao_invalido.validade)
    (passo clicar botao_pagar aguardar pagina erro_pagamento)
    (terminar em erro_pagamento)))
```
>Resultado:

```
WebNavi — loja
========================================
Páginas: 8 | Transições: 11 | Invariantes: 3 | Fluxos: 2

Alcançabilidade:
  [ok] inicio
  [ok] login
  [ok] catalogo
  [ok] produto
  [ok] carrinho
  [ok] checkout
  [ok] confirmacao
  [ok] erro_pagamento

Condições concorrentes:
  [ok] login via botao_entrar -> catalogo e login via botao_entrar -> login são mutuamente exclusivas
  [ok] checkout via botao_pagar -> confirmacao e checkout via botao_pagar -> erro_pagamento são mutuamente exclusivas

Invariantes:
  [ok] checkout exige autenticacao
  [ok] checkout exige carrinho nao vazio
  [ok] confirmacao exige pedido gerado

Fluxos:
  [ok] compra completa — 11 passos
  [ok] pagamento recusado — 4 passos

Resultado: 0 erros | 2 avisos
  [aviso] produto -> carrinho via botao_adicionar: sem transição alternativa para produto.estoque igual a 0
  [aviso] carrinho -> checkout via botao_finalizar: sem transição alternativa para carrinho.quantidade igual a 0
```

## Discussão
>
> O objetivo inicial do projeto era criar uma Linguagem de Domínio Específico capaz de representar fluxos de navegação web de forma mais estruturada do que descrições textuais tradicionais. Durante o desenvolvimento, a proposta evoluiu de uma modelagem baseada apenas em autômatos para uma DSL especializada no domínio web, incorporando conceitos como páginas, elementos de interface, eventos, transições, invariantes e fluxos de navegação.
>
>Os resultados obtidos demonstram que a linguagem consegue representar cenários reais de navegação de forma legível e próxima do domínio do problema. A utilização de macros em Scheme permitiu construir uma sintaxe própria para a linguagem, eliminando a necessidade de um parser externo e integrando a especificação diretamente ao ambiente de execução.
>
> Além da definição sintática, a implementação foi capaz de construir uma representação interna estruturada (AST) e realizar análises semânticas sobre os modelos definidos. Entre essas análises estão a validação de referências a páginas, a identificação de páginas inalcançáveis, a verificação de transições e a validação de fluxos declarados pelo usuário.
>
> Outro resultado relevante foi a geração automática de testes Cypress a partir dos fluxos especificados na DSL. Essa funcionalidade reforça a proposta original do projeto de aproximar documentação, modelagem e automação de testes em um único artefato.
>
> Os resultados indicam que a abordagem é viável para especificação de aplicações web orientadas a navegação. Entretanto, a implementação atual ainda possui limitações, especialmente na expressividade de algumas regras de negócio e na cobertura de componentes modernos de aplicações web. Essas limitações sugerem oportunidades para evolução futura da linguagem e do mecanismo de análise.

## Conclusão

>O desenvolvimento da WebNavi demonstrou que é possível utilizar conceitos de Linguagens de Domínio Específico para representar fluxos de navegação web de forma declarativa, legível e passível de validação automática.
>
>A implementação em Scheme utilizando macros permitiu explorar conceitos de metaprogramação estudados na disciplina, transformando construções da DSL em estruturas internas que podem ser analisadas e processadas automaticamente. Dessa forma, a linguagem deixou de ser apenas uma especificação conceitual e passou a possuir uma implementação funcional.
>
>Entre os principais resultados alcançados estão a definição formal da gramática da linguagem, a implementação das macros da DSL, a construção de uma AST, a realização de validações semânticas sobre os modelos e a geração automática de testes Cypress a partir dos fluxos descritos.
>
>O principal desafio enfrentado durante o projeto foi encontrar um equilíbrio entre simplicidade sintática e capacidade de representar comportamentos reais de aplicações web. Também foi necessário adaptar a proposta inicial para incorporar uma implementação efetiva em Scheme, utilizando macros para construir a linguagem diretamente no ambiente hospedeiro.
>
>Como principal aprendizado, o projeto evidenciou a importância de separar sintaxe, semântica e domínio de aplicação durante o desenvolvimento de uma DSL. Além disso, mostrou na prática como mecanismos de metaprogramação podem ser utilizado

# Trabalhos Futuros

> Diversas extensões podem ser incorporadas à WebNavi em trabalhos futuros.
>
>Uma possibilidade é ampliar o conjunto de componentes suportados pela linguagem, permitindo modelar comportamentos mais complexos encontrados em aplicações web modernas, como navegação assíncrona, chamadas a APIs, autenticação multifator e interfaces dinâmicas.
>
>Outra evolução interessante seria expandir o sistema de análise estática, incorporando verificações mais sofisticadas sobre propriedades do fluxo de navegação, detecção de inconsistências e validação automática de invariantes mais complexos.
>
>Também seria possível melhorar o gerador de testes, oferecendo suporte a outros frameworks além do Cypress, como Playwright, Selenium ou Robot Framework.
>
> Do ponto de vista acadêmico, a linguagem poderia ser utilizada como base para estudos sobre verificação formal de fluxos web, análise de alcançabilidade, geração automática de casos de teste e modelagem baseada em estados.
>
> Por fim, uma possível evolução prática seria o desenvolvimento de ferramentas visuais capazes de gerar especificações WebNavi a partir de diagramas ou protótipos de interface, tornando a linguagem mais acessível para profissionais que não possuem experiência com programação.

# Referências Bibliográficas

> - Documentação oficial do Cypress. Disponível em: https://docs.cypress.io/
> - SANTANCHÈ, A. Material de apoio da disciplina Paradigmas de Programação, 2026.
> - FLATT, M.; FINDLER, R. B. The Racket Reference. PLT Inc., 2024. Disponível em: https://docs.racket-lang.org/reference/
