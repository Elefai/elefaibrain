FROM python:3.12-slim

# Build context expected: elefaibrain-project/mem0-main/ (repo root)
WORKDIR /app

# Install server deps (installs mem0ai from PyPI)
COPY server/requirements.txt /app/requirements.txt
RUN pip install --no-cache-dir -r /app/requirements.txt

# Copy server code
COPY server/ /app/server/

# Hotfix: override mem0ai's pgvector implementation in site-packages
# with the patched file from this repo to avoid mem0migrations duplicate inserts
# causing psycopg connection loss on /search.
COPY mem0/vector_stores/pgvector.py /tmp/pgvector.py
RUN python -c "import os, shutil, mem0.vector_stores.pgvector as pg; shutil.copy('/tmp/pgvector.py', os.path.join(os.path.dirname(pg.__file__), 'pgvector.py'))"

# Hotfix: sanitize graph relationship types and avoid Cypher injection in deletes
COPY mem0/memory/utils.py /tmp/mem0_memory_utils.py
COPY mem0/memory/graph_memory.py /tmp/mem0_graph_memory.py
RUN python -c "import os, shutil, mem0.memory.utils as u; shutil.copy('/tmp/mem0_memory_utils.py', os.path.join(os.path.dirname(u.__file__), 'utils.py'))"
RUN python -c "import os, shutil, mem0.memory.graph_memory as gm; shutil.copy('/tmp/mem0_graph_memory.py', os.path.join(os.path.dirname(gm.__file__), 'graph_memory.py'))"

WORKDIR /app/server

EXPOSE 8000
ENV PYTHONUNBUFFERED=1

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
