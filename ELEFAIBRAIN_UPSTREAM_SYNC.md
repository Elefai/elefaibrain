# Upstream sync (Mem0 → ElefAI Brain fork)

Este repositório (`Elefai/elefaibrain`) é um fork do upstream `mem0ai/mem0`.

Objetivo: manter o fork “andando junto” com o upstream, sem perder os hotfixes necessários para o deploy self-hosted.

## Onde estão os hotfixes

Os hotfixes atuais estão documentados em `ELEFAIBRAIN_PATCHES.md` e aplicados no código do fork (principalmente `pgvector` e `neo4j`).

## Estratégia recomendada (patchset rebaseável)

Mesmo com o `main` contendo os hotfixes, a forma mais segura de acompanhar o upstream é manter uma branch “patchset” rebaseável:

- `elefaibrain-patches`: contém apenas commits de hotfix (sem mudanças grandes misturadas).
- `main`: pode receber merge/PR dessa branch quando você quiser promover para produção.

Se você preferir simplificar, pode manter só o `main` com hotfixes — mas a chance de conflito aumenta quando o upstream muda bastante.

## Setup local (uma vez)

```bash
git clone https://github.com/Elefai/elefaibrain.git
cd elefaibrain
git remote add upstream https://github.com/mem0ai/mem0.git
git fetch upstream
```

Confira:

```bash
git remote -v
```

## Atualizar trazendo mudanças do upstream (rotina)

### Opção A (recomendado): rebase do patchset

1) Atualize referências:

```bash
git fetch upstream
git fetch origin
```

2) Rebase da branch de patches em cima do upstream:

```bash
git checkout elefaibrain-patches
git rebase upstream/main
```

3) Se tiver conflitos: resolva, rode smoke tests e finalize o rebase.

4) Publique a branch rebaseada:

```bash
git push --force-with-lease origin elefaibrain-patches
```

5) Abra PR `elefaibrain-patches` → `main` e faça merge quando estiver ok.

### Opção B: merge upstream direto no main (mais simples, menos controlado)

```bash
git checkout main
git merge upstream/main
git push origin main
```

Funciona, mas tende a “misturar” histórico e dificultar manter hotfixes isolados.

## Como decidir se um hotfix ainda é necessário

Sempre que atualizar o upstream:

1) Leia `ELEFAIBRAIN_PATCHES.md`
2) Verifique no diff do upstream se o bug foi corrigido.
3) Se upstream já corrigiu: remova o hotfix (um commit separado) e documente.

## Smoke test mínimo (sempre)

Depois de atualizar e antes de promover para produção:

- `GET /docs` (FastAPI)
- `POST /memories` (criar)
- `POST /search` (buscar)
- Neo4j (se habilitado): validar retorno de `relations` sem erro Cypher

## Tagging (recomendado)

Quando você promover hotfixes para `main`:

- use tags que indiquem upstream + patch:
  - exemplo: `v1.0.1-elefai.1`

Assim você sabe exatamente o que está rodando no Swarm.

