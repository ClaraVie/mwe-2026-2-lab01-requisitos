# Lab 01 — Engenharia de Requisitos: PRD e SDD

**Microservice and Web Engineering & IT Services**
Prof. José Romualdo da Costa Filho | FIAP Sistemas de Informação | 1º semestre de 2026-2

> Aula 01 — SDLC, Git, Requisitos (PRD/SDD), DDD e Modelo OSI · 04/08/2026
> **60 minutos, em dupla**

| | |
|---|---|
| Slides desta aula | <https://josercf.github.io/FIAP-2026-2-3SI/aulas-1sem/aulas/aula01.html> |
| Portal da disciplina | <https://josercf.github.io/FIAP-2026-2-3SI/> |
| Repositório do acervo | <https://github.com/josercf/FIAP-2026-2-3SI> |
| Biblioteca de skills | <https://github.com/josercf/skill-library> |

---

## O Mini Mundo

A **LogiTech Enterprise** é uma transportadora com **400 caminhões** operando no Sudeste. Hoje, entre a coleta e a entrega, ninguém sabe onde a carga está: o cliente liga perguntando, o atendente liga para o motorista, o motorista atende quando pode.

A diretoria aprovou a construção de um **serviço de telemetria de frota**. Você foi chamado para especificá-lo.

**Hoje ninguém escreve código.** Equipes que usam técnicas de desenvolvimento modernas não escrevem mais código à mão: escrevem a especificação, e o código é derivado dela. A parte que continua sendo humana é **especificar bem e revisar criticamente** — é isso que vocês vão exercitar.

Na Aula 02 vocês implementam exatamente o que especificaram hoje.

---

## Passo 1 — Fork deste repositório (5 min)

1. Clique em **Fork**, no canto superior direito desta página.
2. Renomeie o fork para `mwe-2026-2-lab01-duplaXX` — `XX` é o número da dupla com **dois dígitos** (`dupla07`, não `dupla7`).
3. Em **Settings → Collaborators**, adicione o colega de dupla e o professor.

> **Por que fork e não repositório novo?** O fork mantém o vínculo com o original. Se algo for corrigido no Lab Kit durante a aula, vocês trazem a correção com `git pull upstream main`.

---

## Passo 2 — Abrir o ambiente (5 min)

Este repositório tem um **Dev Container** que já vem com Python, GitHub CLI, o cliente de IA e o **Ollama com o modelo `qwen2.5:1.5b` baixado**. Nada para instalar.

### Opção A — GitHub Codespaces (recomendado)

No **seu fork**: **Code → Codespaces → Create codespace on main**.

> A primeira criação leva alguns minutos porque baixa o modelo (~1 GB). Depois disso, abre em segundos.

### Opção B — Local, com Docker

```bash
git clone https://github.com/SEU-USUARIO/mwe-2026-2-lab01-duplaXX.git
cd mwe-2026-2-lab01-duplaXX
code .                              # aceite "Reopen in Container"
export GITHUB_TOKEN=$(gh auth token)
```

### Verifique antes de seguir

```bash
python ai/ask.py "diga apenas: ambiente ok"
ollama list                          # qwen2.5:1.5b deve aparecer
```

O `ask.py` tenta o **GitHub Models** primeiro (token que o Codespaces injeta) e cai para o **Ollama local** se a cota acabar ou faltar rede. Para forçar o local:

```bash
OLLAMA_MODEL=qwen2.5:1.5b python ai/ask.py "..."
```

---

## Passo 3 — Instalar a skill de PRD (5 min)

Uma **skill** é um arquivo `SKILL.md` que ensina ao assistente um procedimento — como escrever um PRD, como padronizar commits. Em vez de repetir um prompt gigante toda vez, você instala a skill uma vez e a invoca.

```bash
# 1. Baixar a biblioteca compartilhada da disciplina
git clone https://github.com/josercf/skill-library.git /tmp/skill-library

# 2. Instalar as skills que vamos usar hoje
mkdir -p .claude/skills
cp -r /tmp/skill-library/skills/prd .claude/skills/
cp -r /tmp/skill-library/skills/sdd .claude/skills/

# 3. Conferir
ls .claude/skills/
```

Assistentes que leem `.claude/skills/` (como o Claude Code) passam a enxergar a skill sozinhos. Com o `ai/ask.py`, você anexa o conteúdo dela ao prompt:

```bash
python ai/ask.py "$(cat .claude/skills/prd/SKILL.md)

Agora aplique esta metodologia e escreva o PRD do servico de telemetria
de frota da LogiTech, transportadora com 400 caminhoes que nao sabe onde
a carga esta entre a coleta e a entrega." > docs/PRD.md
```

---

## Passo 4 — Gerar o PRD e o SDD (20 min)

O que o modelo produz é **rascunho**, não entrega.

### 4.1 `docs/PRD.md`

Prompt (já embutido no comando do passo anterior, ou use direto):

```
Escreva um PRD para o servico de telemetria de frota da LogiTech,
transportadora com 400 caminhoes que nao sabe onde a carga esta entre
a coleta e a entrega.

Estruture em: Visao do Produto, Problema, Personas, Casos de Uso,
Requisitos Funcionais, Requisitos Nao Funcionais e Metricas de Sucesso.

Cada requisito deve ser verificavel e ter identificador (RF-01, RNF-01).
Nao proponha solucao tecnica: o PRD descreve o problema, nao a arquitetura.
```

### 4.2 `docs/SDD.md`

```
Com base no PRD acima, escreva um System Design Document.

Inclua: Bounded Contexts identificados, a Linguagem Ubiqua (glossario dos
termos do dominio) e a escolha da camada de transporte para cada fluxo
de dados.

Para cada decisao entre TCP e UDP, justifique a partir do requisito nao
funcional que a motiva.
```

---

## Passo 5 — Revisar criticamente (20 min) — **é o que vale nota**

O modelo **vai errar**. Encontrar onde ele errou é a competência que a disciplina desenvolve.

| Procure por | Por que é problema |
|---|---|
| Requisito que não dá para testar | "O sistema deve ser rápido" não é requisito. Qual latência, em qual percentil, sob qual carga? |
| O mesmo conceito com dois nomes | PRD diz "entrega" e SDD diz "remessa"? A Linguagem Ubíqua quebrou na primeira página |
| Justificativa decorada | O SDD explicou *o que é* UDP, ou *por que este requisito* pede UDP? Só o segundo conta |
| Requisito inventado | Modelos criam funcionalidade para parecer completos. Apague o que não veio do problema |
| Bounded Context que virou camada técnica | "Frontend" e "Backend" não são contextos de negócio. "Faturamento" e "Operação" são |

---

## Entregáveis: exatamente o que precisa estar no repositório

### 1. `docs/PRD.md`

Contendo, obrigatoriamente:

- [ ] **Visão do Produto** e **Problema** — 1 parágrafo cada
- [ ] **No mínimo 2 personas**, com nome do papel e a dor específica de cada uma
- [ ] **No mínimo 3 casos de uso**, no formato `UC-01 — <nome>`, com ator, pré-condição e fluxo principal
- [ ] **No mínimo 5 requisitos funcionais**, numerados `RF-01`, `RF-02`, ...
- [ ] **No mínimo 3 requisitos não funcionais**, numerados `RNF-01`, ..., **cada um com número** (latência em ms, disponibilidade em %, volume em eventos/s). Requisito não funcional sem número não conta
- [ ] **No mínimo 3 métricas de sucesso**, com a linha de base atual e a meta
- [ ] Seção final **"Revisão da Dupla"** (ver item 3)

### 2. `docs/SDD.md`

Contendo, obrigatoriamente:

- [ ] **No mínimo 3 Bounded Contexts**, cada um com uma frase dizendo qual responsabilidade de negócio ele isola
- [ ] **Glossário da Linguagem Ubíqua** com **no mínimo 8 termos** do domínio, cada um com a definição que vale dentro do contexto. Os termos precisam ser os mesmos usados no PRD
- [ ] **Tabela de fluxos de dados** com uma linha por fluxo, nas colunas: `Fluxo | Protocolo (TCP/UDP) | RNF que motiva | Justificativa`
- [ ] **No mínimo 2 fluxos usando TCP e 2 usando UDP**, cada escolha amarrada a um `RNF-XX` do PRD
- [ ] Um diagrama dos componentes, em Mermaid ou ASCII

### 3. Seção "Revisão da Dupla", ao final do `PRD.md`

```markdown
## Revisão da Dupla

| # | O que o modelo entregou | O que corrigimos | Por quê |
|---|---|---|---|
| 1 | "O sistema deve ser rápido" | "RNF-02: p95 de ingestão < 200 ms" | Não era verificável |
| 2 | ... | ... | ... |
```

**No mínimo 4 linhas.** Sem essa tabela, a entrega fica incompleta.

### 4. Histórico Git

- [ ] **No mínimo 2 commits** seguindo [Conventional Commits](https://www.conventionalcommits.org/pt-br/v1.0.0/)
- [ ] Os dois integrantes da dupla aparecendo como autores (usem `git commit --author` ou commitem cada um do seu ambiente)

---

## Como entregar

```bash
git add docs/
git commit -m "docs(telemetria): adiciona PRD e SDD do rastreamento de frota"
git push origin main
```

Submeta a URL do seu fork no formulário da disciplina, **até o fim da aula**. Um envio por dupla.

---

## Como será avaliado

| Peso | Critério |
|---|---|
| Estrutura | Todos os itens obrigatórios do PRD e do SDD presentes |
| Verificabilidade | RNFs com número, não com adjetivo |
| Consistência | Os termos do glossário são os mesmos usados no PRD e no SDD |
| Justificativa | Escolha de TCP/UDP amarrada a um RNF, não à definição do protocolo |
| Revisão crítica | Qualidade da tabela "Revisão da Dupla": achados reais, não cosméticos |
| Git | Commits semânticos e participação dos dois integrantes |

---

## O que tem neste repositório

```
.devcontainer/
  devcontainer.json   ambiente reproduzível (Codespaces e local)
  post-create.sh      dependências + instalação do Ollama + download do modelo
  post-start.sh       garante o Ollama no ar a cada boot
ai/
  ask.py              cliente de IA: GitHub Models, com fallback para Ollama
docs/                 onde entram o seu PRD.md e SDD.md
README.md
```

---

## Na próxima aula

A Aula 02 implementa o servidor de telemetria em Python (Sockets TCP e UDP) **a partir do SDD que vocês escreveram hoje**, e sobe para HTTP e SSE. Se a especificação estiver vaga, a implementação vai doer — e esse é justamente o ponto.
