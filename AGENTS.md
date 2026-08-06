# Documentação da Vext

Site de documentação da API de pagamentos PIX com split da Vext, construído em
[Mintlify](https://mintlify.com).

## Sobre este projeto

- Páginas são arquivos MDX com frontmatter YAML. A configuração vive em `docs.json`.
- **A fonte de verdade do comportamento da API é `api-reference/openapi.yaml`.** Nada é documentado
  aqui que não esteja lá; se o fato é novo, ele entra na spec primeiro, no mesmo PR.
- `mint dev` na raiz para pré-visualizar. `mint broken-links` antes de abrir PR.
- Para conhecimento do produto Mintlify (componentes, configuração), o MCP é
  `https://www.mintlify.com/docs/mcp`.

## Estrutura

| Pasta | O que vive nela |
| --- | --- |
| `essenciais/` | Conceitos transversais: centavos, autenticação, idempotência, erros, limites, ciclo de vida, estornos, saldo |
| `webhooks/` | Guia de webhooks: visão geral, assinatura, entregas |
| `receitas/` | Fluxos completos ponta a ponta |
| `api-reference/` | A spec e as páginas de endpoint/evento, cada uma com `openapi:` no frontmatter |
| `snippets/` | Trechos reutilizáveis. Nunca viram página |

## Política de fonte única

Um fato mora em **um** lugar. O critério não é "conceito vs. endpoint", é o escopo da afirmação.

| Tipo de conteúdo | Dono |
| --- | --- |
| Tipo, enum, obrigatoriedade e exemplo de um campo | `openapi.yaml`. **Nunca redigite schema em MDX** |
| Porquê específico de um campo (piso do `expires_in`, teto de `per_page`, `platform_fee` congelada) | `openapi.yaml` |
| Centavos, autenticação, idempotência, erros, limites de taxa | A página em `essenciais/`. Na spec, uma linha e um link |
| Assinatura de webhook | `webhooks/assinatura.mdx`. Na spec, o formato e o `pattern` |
| Exemplos de request/response | `openapi.yaml`, em `examples:` — alimentam o playground |

Duas regras que decorrem disso:

- **A spec nunca ganha prosa nova.** Ela perde prosa para os MDX e ganha links. Se você quer explicar
  algo em mais de três linhas dentro do YAML, o texto pertence a uma página.
- **Números vêm de `snippets/limites.mdx`.** Se você digitou "24 horas", "90 dias", "5 minutos" ou
  "100 itens" à mão em um MDX, está errado.

## Terminologia

Use **cobrança** (não "transação"), **vendedor** (não "usuário" nem "cliente"), **comprador**,
**estorno** (não "reembolso"), **saldo**, **provedor** ou **adquirente**, **saque**.

A seção do painel onde se criam chaves é **Desenvolvedores** — sempre em negrito.

Identificadores da API (`charge`, `refund`, `amount`, `charges:write`) ficam em inglês, em `code`, e
nunca são traduzidos na prosa.

## Voz

Português do Brasil. "Você" é quem integra. "Nós" é a Vext, e só aparece quando a frase explica uma
decisão nossa — "guardamos apenas um hash SHA-256", "preferimos recusar a adivinhar". Nunca "o
usuário deve" nem "recomenda-se". Instrução no imperativo.

**A regra que define esta documentação: comece pelo ganho, termine na consequência.**

Padrão de três partes: [o que você ganha ou o que fazer] → [por que a regra existe] → [o que
acontece sem ela]. A ordem importa. A mesma informação abre acolhendo ou abre acusando:

> ❌ Confira a assinatura antes de agir. Quem tem o segredo consegue forjar um `charge.paid`, e um
> sistema que libera produto ao recebê-lo entregaria de graça.
>
> ✅ Conferir a assinatura leva poucas linhas de código e é o que garante que o evento veio mesmo de
> nós. Sem ela, qualquer pessoa que descubra a URL do seu endpoint consegue simular um pagamento.

A consequência continua lá, concreta e específica — ela só não é a primeira coisa que a pessoa lê.
Nada de "por questões de segurança" ou "por boas práticas": diga qual ataque, qual bug, qual
prejuízo. Se você não consegue nomear a consequência, provavelmente a regra não precisa ser
documentada.

Evite abrir parágrafo com proibição. "Nunca faça X" vira "Faça Y — é o que evita X". "Não há
exceção" vira "Uma regra só, e ela vale para todos os campos".

Diga explicitamente as distinções que geram chamado no suporte, em vez de deixá-las subentendidas:
`null` não é zero, `403` não é `401`, `partially_refunded` não é `refunded`.

## Frase e formatação

- Uma ideia por frase. Parágrafo de no máximo quatro linhas.
- **Proibidos:** "simplesmente", "basta", "apenas", "é só", "facilmente". Minimizar a dificuldade de
  quem integra pagamento é como um erro passa despercebido.
- Negrito só na palavra que carrega o aviso, no máximo um por parágrafo.
- `code` para campo, código de erro, header, status e caminho.
- Títulos em sentence case, sem gerúndio de manual ("Verificar a assinatura", não "Verificando").
- Dinheiro: sempre o inteiro em centavos e, na primeira menção do trecho, o equivalente —
  "`10000` (R\$ 100,00)".
- **Cifrão: dois `R$` no mesmo trecho viram fórmula LaTeX.** O texto entre eles é renderizado como
  math, e acentos ali dentro quebram a build com `unicodeTextInMathMode`. Onde vale a regra:

  | Onde | O que fazer |
  | --- | --- |
  | Prosa, tabelas, componentes | Escape: `R\$` |
  | Título de bloco de código | Escape: ` ```json Cobrança (R\$ 100,00) ` |
  | `description` do frontmatter | **Não dá para escapar** — reescreva a frase com um `R$` só, ou nenhum |
  | Dentro de crase e de bloco de código | `R$` normal; ali o `\` apareceria literal |

  O `description` é a armadilha silenciosa: ele é renderizado como subtítulo da página, mas o
  detector abaixo não distingue frontmatter de corpo. Se a build reclamar e a prosa estiver limpa,
  olhe o frontmatter.

## Componentes

- `<Warning>` só quando ignorar custa dinheiro ou abre brecha — no máximo um por página. Se tudo é
  aviso, nada é aviso. `<Note>` para contexto, `<Tip>` para atalho.
- `<CodeGroup>` sempre na mesma ordem: cURL, PHP, JavaScript, Python.
- `<ParamField>` e `<ResponseField>` **só** em páginas de guia. Campos de request e response vêm do
  `openapi.yaml`; redigitá-los em MDX cria a segunda cópia que vai divergir.
- Toda página de Essenciais e de Webhooks termina com `<Suporte />`.

## Fronteiras de conteúdo

- Todo exemplo usa os valores canônicos da spec: `ch_01k1y6r6m6q2x0p3d9v4t7c8n2`, `pedido-1042`,
  `amount: 10000`, `platform_fee: 798`, `net_amount: 9202`. Números inventados não fecham conta
  entre páginas.
- Nunca use chave real, nem em exemplo. **Sempre `sk_live_...` ou `sk_test_...`, com reticências.**
  Não preencha o placeholder com caracteres soltos: `sk_live_` seguido de 24+ alfanuméricos casa com
  o detector de chave da Stripe no secret scanning do GitHub, e o push da branch inteira é recusado —
  mesmo sendo só letra `x`.
- Não documente comportamento do provedor adquirente. A mensagem crua dele não sai da nossa API e
  não deve entrar na doc.
- Antes de afirmar que uma conversão numérica quebra, **rode o código**. O exemplo de ponto flutuante
  em `essenciais/valores-em-centavos.mdx` usa `19.99`, `8.20` e `0.29` porque esses truncam de
  verdade nas três linguagens — `10.50` é exato em binário e não serve de exemplo.

## Estrutura de uma página de Essenciais

1. Uma frase dizendo o que a página resolve.
2. A regra, com exemplo mínimo.
3. O porquê.
4. "O que dá errado" — os códigos de erro relacionados e o sintoma que a pessoa vai ver.
5. "Veja também", com `<CardGroup>` para as páginas vizinhas.
