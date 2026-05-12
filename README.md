# Projeto `WebNavi`
# Project `WebNavi`

> |Nome  | RA |
> |--|--|
> | Thiago Dorta   | 231413  |
> | Gabriel Silva  | 200342  |

# WebNavi — DSL para Validação de Fluxos Web

Repositório do projeto da disciplina **Paradigmas de Programação**, organizado em entregas parciais.

A proposta geral do projeto é desenvolver a **WebNavi**, uma Linguagem de Domínio Específico (DSL) em português para especificação e validação de fluxos de navegação web.

A linguagem permite descrever páginas, rotas, elementos de interface, eventos, transições, condições, invariantes, dados de teste e fluxos de navegação de forma declarativa e legível.

---

## Visão Geral

Em sistemas web, fluxos de navegação costumam ser descritos em documentos, protótipos, apresentações ou conversas, enquanto os testes automatizados são implementados separadamente em ferramentas como Cypress ou Playwright.

Essa separação pode gerar inconsistência entre:

- o fluxo especificado;
- o comportamento esperado;
- a documentação;
- os testes automatizados.

A WebNavi busca reduzir essa distância ao transformar a especificação do fluxo em um artefato estruturado, legível e passível de validação.

A linguagem usa autômatos finitos como base formal:

- páginas funcionam como estados;
- ações do usuário funcionam como eventos;
- transições representam mudanças entre páginas.

Porém, diferente de uma linguagem genérica de autômatos, a WebNavi incorpora conceitos específicos do domínio web, como botões, campos, links, rotas, condições, efeitos e invariantes.

---

## Estrutura do Repositório

```text
.
├── project-1/
│   ├── README.md
│   └── WebNavi.pdf
│
├── project-2/
│   ├── README.md
│   ├── WebNavi_notebook.ipynb
│   └── WebNavi_Apresentacao (1)
│
└── README.md
```

---

## Project 1 — Primeira Entrega

A primeira entrega apresentou a ideia inicial da WebNavi como uma linguagem para modelar navegação usando uma estrutura próxima de autômatos.

Nessa versão inicial, o foco estava em representar páginas como estados e mudanças de tela como transições.

### Principais características

- modelagem inicial de navegação;
- uso de estados e transições;
- primeiras primitivas da linguagem;
- primeiros exemplos de fluxo;
- estrutura inicial da DSL.

### Limitações identificadas

Após a primeira entrega, foram levantados alguns pontos importantes para evolução:

- a linguagem estava muito próxima de um autômato puro;
- ainda não estava claro o que diferenciava a DSL de uma linguagem genérica de autômatos;
- faltava representar melhor elementos reais de páginas web;
- era necessário explicitar o que dispara uma transição;
- era necessário separar melhor o evento da ação e o local onde ela ocorre;
- faltava diferenciar navegação possível de navegação executada;
- a validação ainda era limitada.

Esses pontos motivaram a reformulação da linguagem na segunda entrega.

---

## Project 2 — Segunda Entrega

A segunda entrega apresenta uma versão revisada e mais completa da WebNavi.

Nesta etapa, a linguagem deixa de ser apenas uma representação textual de autômatos e passa a incorporar conceitos próprios do domínio de navegação web.

### Principais evoluções

- inclusão de páginas com rotas;
- declaração de elementos de interface;
- separação entre elemento e evento;
- transições com condições;
- efeitos de estado;
- invariantes;
- dados de teste;
- fluxos de navegação;
- gramática em EBNF;
- exemplos mais completos;
- discussão sobre análise estática e futuras saídas para testes automatizados.

### Exemplo de sintaxe

```webnavi
site loja

pagina login
  rota /login
  elemento campo_email entrada obrigatorio formato email seletor "#email"
  elemento campo_senha entrada obrigatorio formato senha minimo 8 seletor "#senha"
  elemento botao_entrar botao texto "Entrar" seletor "#entrar"
fim

inicio em inicio
final em confirmacao

transicao de login para catalogo
  via botao_entrar clicado
  somente se campo_email preenchido e campo_senha preenchido
  entao sessao.autenticado recebe verdadeiro
fim
```

Nesse exemplo, a transição não é apenas uma seta entre duas páginas. Ela define:

- a página de origem;
- a página de destino;
- o elemento que dispara a transição;
- o evento realizado;
- a condição necessária;
- o efeito produzido.

---

## Conceitos Centrais da WebNavi

| Conceito | Representação na WebNavi |
|---|---|
| Sistema modelado | `site` |
| Estado do fluxo | `pagina` |
| Endereço da página | `rota` |
| Elemento da interface | `elemento` |
| Mudança possível de tela | `transicao` |
| Disparo da transição | `via elemento evento` |
| Condição da transição | `somente se` |
| Efeito da transição | `entao ... recebe ...` |
| Regra global | `invariante` |
| Valores reutilizáveis | `dados` |
| Cenário concreto de navegação | `fluxo` |

---

## Navegação Possível vs. Navegação Executada

Um ponto importante da WebNavi é a separação entre transições e fluxos.

### Transições

As transições definem a navegação possível dentro do sistema.

```webnavi
transicao de login para catalogo
  via botao_entrar clicado
fim
```

Isso significa que existe um caminho possível de `login` para `catalogo`, disparado pelo clique em `botao_entrar`.

### Fluxos

Os fluxos definem uma sequência concreta de passos.

```webnavi
fluxo compra completa
  comecar em inicio
  passo clicar botao_login aguardar pagina login
  passo preencher campo_email com usuario_valido.email
  passo clicar botao_entrar aguardar pagina catalogo
  terminar em catalogo
fim
```

Assim:

- transições descrevem o espaço de navegação permitido;
- fluxos descrevem cenários específicos que podem ser simulados, analisados ou usados futuramente para geração de testes.

---

## Estado Atual do Projeto

A WebNavi deve ser entendida, nesta etapa, como:

- uma especificação de linguagem;
- uma gramática proposta;
- um protótipo parcial de implementação;
- uma base para análise estática de fluxos web;
- uma possível entrada futura para geração de testes automatizados.

A linguagem em si não executa testes sozinha. Ela define uma representação estruturada do fluxo. Ferramentas construídas sobre essa representação podem realizar análise, simulação ou geração de código para frameworks como Cypress ou Playwright.

---

## Trabalhos Futuros

Possíveis evoluções do projeto incluem:

- implementar um parser formal completo;
- refinar a gramática ANTLR;
- fortalecer a análise semântica;
- detectar páginas inalcançáveis;
- detectar transições inconsistentes;
- validar invariantes de forma mais robusta;
- diferenciar melhor avisos e erros;
- simular fluxos completos;
- gerar testes Cypress ou Playwright;
- produzir relatórios de cobertura;
- criar uma interface ou editor para arquivos `.webnavi`.

---

## Referências

- Material da disciplina e laboratório ANTLR4 utilizado em aula.
- Cypress. *Cypress Documentation*. Disponível em: https://docs.cypress.io/
- Playwright. *Playwright Documentation*. Disponível em: https://playwright.dev/
- SANTANCHÈ, André. ANTLR4 Lab. Program2Learn. Disponível em: https://santanche.github.io/program2learn/formal-languages/antlr4-lab/
