# Stackmaker patches (Mem0 fork)

Este repositório (`Elefai/elefaibrain`) é um fork do upstream `mem0ai/mem0`, com hotfixes aplicados para melhorar estabilidade do servidor self-hosted (principalmente `pgvector/Postgres` e `Neo4j`).

## O que está “pinned”

- O build instala `mem0ai` via `server/requirements.txt` (atualmente `mem0ai==1.0.1`).
- Em seguida o `server/Dockerfile` sobrescreve alguns arquivos dentro do `site-packages/mem0/` com as versões deste repo.

Isso existe para manter o servidor REST estável em produção no Swarm (quando usado com o Stackmaker).

## Patches aplicados (hotfix)

### `mem0/vector_stores/pgvector.py`
- Evita inserts duplicados na tabela de migrations (usando `ON CONFLICT DO NOTHING`) que geravam erros no Postgres e podiam derrubar a conexão (`the connection is lost`).
- Melhora resiliência do pool do `psycopg`:
  - valida conexão (`SELECT 1`) antes de emprestar do pool
  - retry 1x em falhas transientes para `search()` e `list()`
- Mantém compatibilidade do retorno de `list()` com o código do Mem0 (`vector_store.list(...)[0]`).

### `mem0/memory/utils.py`
- Sanitiza tipos de relação do grafo (Neo4j) para evitar tipos inválidos e reduzir chance de Cypher inválido.

### `mem0/memory/graph_memory.py`
- Evita Cypher inválido em deleções de relação: usa `type(r) = $relationship` em vez de interpolar `:[REL]`.

## Onde isso é aplicado

Veja `server/Dockerfile` (linhas de `COPY ...` + `python -c "shutil.copy(...)"`).

## Como atualizar o Mem0 com segurança

1) Atualize o `mem0ai==...` em `server/requirements.txt`
2) Rebuild da imagem:
   - `cd <repo>`
   - `docker build -f server/Dockerfile -t mem0-api-server:local .`
3) Rode um smoke-test:
   - `GET /docs`
   - `POST /memories` (cria)
   - `POST /search` (busca)
4) Re-deploy da stack `mem0-api`:
   - `docker stack deploy -c "stacks modelos/mem0-api.yaml" mem0-api`

Se o upstream alterar as APIs internas do Mem0, revise os patches acima (principalmente o contrato de `list()`).
