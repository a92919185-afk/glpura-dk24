# PRD_INDEX — Clone Page Superpowers

> **Para qualquer LLM/IA lendo este projeto:** este arquivo é o ponto de entrada obrigatório.
> Não execute nenhuma tarefa sem antes verificar o guia de carga abaixo.

---

## O que este projeto faz

**Clone Page Superpowers** é uma fábrica de landing pages de alta conversão para Google Ads.

Dado um produto e uma URL de referência, o sistema extrai o design real (cores, tipografia, imagens), gera copy culturalmente adaptado ao país-alvo, e produz um único `index.html` com estrutura imutável de 11 blocos (SEMA7), 100% compatível com Google Ads.

**Saída sempre:** um único `index.html` + diretório `images/` em `output/[produto]/`.
**Nunca:** páginas secundárias, grids de preço separados, ou seções inventadas.

---

## Dois modos de operação

| Modo | Como usar | Quando usar |
|---|---|---|
| **Agent Mode** (primário) | Prompt manual seguindo `prd_squad.md` | Geração guiada por LLM, refinamentos estéticos |
| **CLI Mode** | `npx tsx src/cli.ts generate --product "X" --url "URL"` | Geração rápida automatizada |

---

## Guia de Carga para LLMs

Carregue **apenas** os arquivos necessários para a tarefa. Não carregue tudo.

| Tarefa | Arquivos obrigatórios |
|---|---|
| **Iniciar projeto / entender o sistema** | `PRD_INDEX.md` + `prd_squad.md` |
| **Extrair design (Scout)** | `PRD_INDEX.md` + `agents/scout.agent.md` |
| **Gerar copy persuasivo (Copywriter)** | `PRD_INDEX.md` + `prd_copy.md` + `prd_compliance.md` + `agents/copywriter.agent.md` |
| **Extrair e classificar imagens (Scout)** | `PRD_INDEX.md` + `prd_images.md` + `agents/scout.agent.md` |
| **Gerar SEO Block (Copy SEO)** | `PRD_INDEX.md` + `agents/copy-seo.agent.md` |
| **Criar design system (Designer)** | `PRD_INDEX.md` + `prd_compliance.md` + `agents/designer.agent.md` |
| **Escolher modelo de página** | `PRD_INDEX.md` + `prd_page_models.md` |
| **Construir página (Builder)** | `PRD_INDEX.md` + `prd_page_models.md` + `prd_architecture.md` + `prd_compliance.md` + `prd_images.md` + `agents/builder.agent.md` |
| **Revisar HTML (Critic)** | `PRD_INDEX.md` + `prd_compliance.md` + `agents/critic.agent.md` |
| **QA visual (Validator)** | `PRD_INDEX.md` + `prd_compliance.md` + `prd_images.md` + `agents/validator.agent.md` |
| **Deploy para Vercel** | `PRD_INDEX.md` (seção "Deploy para Vercel") |

---

## Mapa de Arquivos

```
AGENT.md                          ← Master prompt (leia primeiro, brevemente)
docs/prd/
  PRD_INDEX.md                    ← Este arquivo (roteador central)
  prd_compliance.md               ← FONTE ÚNICA: Google Ads + legal + anti-alucinação
  prd_architecture.md             ← Os 11 blocos SEMA7 (imutável)
  prd_squad.md                    ← Workflow dos 7 agentes + como iniciar
  prd_copy.md                     ← Perfis culturais (10 países) + framework de copy por bloco
  prd_images.md                   ← Classificação, placement, CSS e UX/UI de imagens
  prd_page_models.md              ← Dois modelos: SEMA7 Standard (A) e Flag Page Presell (B)
agents/
  scout.agent.md                  ← Extração de design (Agente 1)
  copywriter.agent.md             ← Copy persuasivo B1–B9 + B11 (Agente 2)
  copy-seo.agent.md               ← SEO Block B10 — Google Search Network (Agente 3)
  designer.agent.md               ← Design system premium (Agente 4)
  builder.agent.md                ← Geração de HTML (Agente 5)
  critic.agent.md                 ← Revisão e compliance (Agente 6)
  validator.agent.md              ← QA visual (Agente 7)
squads/page-builder/_memory/
  memories.md                     ← Aprendizados acumulados de sessões passadas
```

---

## Prompt Mestre

**Model A — SEMA7 Standard (um país):**
```
Produto: [NOME] · URL: [URL] · País: [CÓDIGO] · Modelo: A
Leia: prd_architecture.md + prd_compliance.md + prd_copy.md + prd_images.md
```

**Model B — Flag Page Presell (multi-país):**
```
Produto: [NOME] · URL: [URL] · Modelo: B
Países: [código]: [URL checkout] | [código]: [URL checkout] | ...
Leia: prd_page_models.md + prd_architecture.md + prd_compliance.md + prd_images.md
```

→ Especificação dos dois modelos: `prd_page_models.md`

---

## Deploy para Vercel (Git + Static)

### Regra absoluta — causa de 404 se violada

> O Vercel serve **apenas o que está na raiz do repositório**.
> Um `index.html` em `output/stopwatt-presell/index.html` resulta em **404: NOT_FOUND**.
> O arquivo **deve estar na raiz** do repo que será conectado ao Vercel.

### Estrutura correta do repo de deploy

Cada página vai para um **repositório Git próprio e dedicado**. A raiz do repo = a raiz da página.

```
[repo-name]/          ← raiz do repositório
  index.html          ← a página (copiada de output/[slug]/index.html)
  vercel.json         ← configuração obrigatória
  images/             ← se houver imagens locais (opcional)
```

### vercel.json obrigatório

Criar sempre na raiz do repo de deploy:

```json
{
  "cleanUrls": true,
  "trailingSlash": false
}
```

### Passo a passo de deploy (sequência exata)

```
1. Gerar a página → output/[slug]/index.html  (pipeline normal)

2. Criar repo Git dedicado para essa página:
   git init
   git remote add origin https://github.com/[user]/[repo].git

3. Copiar index.html para a raiz do repo:
   cp output/[slug]/index.html ./index.html

4. Criar vercel.json na raiz (conteúdo acima)

5. Commitar apenas esses dois arquivos:
   git add index.html vercel.json
   git commit -m "feat: deploy [produto] landing page"
   git push -u origin main

6. No Vercel Dashboard → Import Git Repository → selecionar o repo
   - Framework Preset: Other
   - Root Directory: ./  (padrão — não alterar)
   - Build Command: (vazio)
   - Output Directory: (vazio)
   → Deploy

7. Vercel serve index.html diretamente na URL gerada ✓
```

### Erros comuns e correções

| Erro | Causa | Correção |
|---|---|---|
| `404: NOT_FOUND` | `index.html` não está na raiz do repo | Copiar para a raiz, commitar, push |
| `404: NOT_FOUND` | Root Directory configurado errado no Vercel | Dashboard → Settings → Root Directory → `./` |
| Página em branco | Vercel apontando para subpasta errada | Deletar projeto no Vercel, reimportar com settings corretos |
| CDNs não carregam | Não é problema do Vercel — verificar URLs dos CDNs (FA, flag-icons) | Testar abrindo o HTML local primeiro |

### Referência real — StopWatt Presell

```
Repo:  https://github.com/a92919185-afk/stopwatt-economy
Fonte: output/stopwatt-presell/index.html  →  copiado para raiz como index.html
```
