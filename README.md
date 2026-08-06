# Documentação da Vext

Site de documentação da API de pagamentos PIX com split da Vext, construído em
[Mintlify](https://mintlify.com).

## Estrutura

```
docs.json              Configuração do site: navegação, marca, playground
index.mdx              Introdução
quickstart.mdx         Primeira cobrança
essenciais/            Conceitos: centavos, autenticação, idempotência, erros,
                       limites, ciclo de vida, estornos, saldo
webhooks/              Guia: visão geral, assinatura, entregas e retentativas
receitas/              Fluxos ponta a ponta
api-reference/
  openapi.yaml         Fonte de verdade do comportamento da API
  ...                  Páginas de endpoint e de evento, ligadas à spec pelo frontmatter
snippets/              Trechos reutilizáveis
logo/, favicon.svg     Marca
```

`AGENTS.md` traz o guia de voz, a política de fonte única e as convenções de escrita. Leia antes de
alterar conteúdo.

## Desenvolvimento

Instale o CLI da Mintlify:

```bash
npm i -g mint
```

Rode na raiz, onde está o `docs.json`:

```bash
mint dev
```

A pré-visualização fica em `http://localhost:3000`.

Antes de abrir um PR:

```bash
mint broken-links
```

E confira que nenhum cifrão escapou sem escape na prosa — dois `R$` na mesma linha viram fórmula
LaTeX e quebram a build (ver `AGENTS.md`):

```bash
grep -rn 'R\$' --include='*.mdx' . | grep -v 'R\\\$'
```

Ocorrências dentro de crase e de bloco de código são esperadas; o que não pode aparecer é `R$` solto
no texto corrido.

## Publicação

O app do GitHub da Mintlify propaga as mudanças do repositório para o deploy. Alterações na branch
padrão vão para produção automaticamente.

## Problemas comuns

- **Editou o `openapi.yaml` e a página não mudou:** o `mint dev` recarrega MDX a quente, mas cacheia
  a spec. Reinicie o servidor. Se você confiar no hot reload aqui, vai revisar a versão antiga
  achando que revisou a nova.
- Ambiente de desenvolvimento não sobe: rode `mint update` para atualizar o CLI.
- Página carrega como 404: confirme que você está rodando em uma pasta com um `docs.json` válido, e
  que o caminho da página está na navegação do `docs.json`.
