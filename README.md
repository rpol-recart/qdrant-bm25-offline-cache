# qdrant-bm25-offline-cache

Pre-populated [FastEmbed](https://github.com/qdrant/fastembed) / Hugging Face cache
for the **`Qdrant/bm25`** model, packaged for **fully offline** use.

The tree mirrors the standard `huggingface_hub` cache layout (blobs + snapshot
symlinks), so pointing `FASTEMBED_CACHE_PATH` at it lets FastEmbed load the model
without any network access.

## Contents

```
fastembed_cache/
├── CACHEDIR.TAG
└── models--Qdrant--bm25/
    ├── blobs/                    # content-addressed files (stopword lists + config)
    ├── files_metadata.json
    ├── refs/main                 # -> 22b8d2af71a76161e18dd432d2cee0eefa66e412
    └── snapshots/
        └── 22b8d2af71a76161e18dd432d2cee0eefa66e412/
            ├── config.json -> ../../blobs/9e26dfeeb6e641a33dae4961196235bdb965b21b
            └── <lang>.txt -> ../../blobs/<blob_id>   (17 languages)
```

- Snapshot revision: `22b8d2af71a76161e18dd432d2cee0eefa66e412`
- 17 stopword lists (ar, da, de, en, es, fi, fr, el, hu, it, nl, no, pt, ro, ru, sv, tr) + `config.json`
- Total on-disk size: ~120 KB

## Usage

### Python (FastEmbed >= 0.8.0)

```python
import os
os.environ["FASTEMBED_CACHE_PATH"] = "/path/to/qdrant-bm25-offline-cache/fastembed_cache"

from fastembed import SparseTextEmbedding

model = SparseTextEmbedding(model_name="Qdrant/bm25")  # resolved from local cache, no download
embeddings = list(model.embed(["offline embedding works"]))
```

### Docker (read-only mount)

```yaml
services:
  app:
    image: your-app
    environment:
      FASTEMBED_CACHE_PATH: /opt/fastembed_cache
    volumes:
      - ./fastembed_cache:/opt/fastembed_cache:ro
```

The symlinks inside `snapshots/` are **relative** and always resolve inside the
exported tree, so the directory can be relocated or bind-mounted anywhere
(e.g. read-only) without modification.

## Provenance

- Model: [`Qdrant/bm25`](https://huggingface.co/Qdrant/bm25) (BM25 sparse
  vocabulary / stopword files used by FastEmbed's `SparseTextEmbedding`).
- Snapshot revision: `22b8d2af71a76161e18dd432d2cee0eefa66e412`.
- This repository is a **verbatim copy** of that snapshot's cache files (blobs,
  snapshot symlinks, refs, `config.json`, `files_metadata.json`). No files were
  modified, added to, or removed from the model data.
- FastEmbed version this cache was produced/validated with: **0.8.0**.

## License

The upstream model `Qdrant/bm25` is licensed under the **Apache License 2.0**
(see the model card tag `license:apache-2.0`). These redistributed model files
are provided under the same terms — see [`LICENSE`](./LICENSE) and
[`NOTICE`](./NOTICE).
