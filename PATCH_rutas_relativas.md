# Patch de rutas relativas — ML_23F_unav

Objetivo: que cualquier notebook se ejecute desde la carpeta del repo sin tocar rutas,
en local o en Colab, y que la cadena de datos (CSV → embeddings → grafo) encaje.

---

## 0) Bloque de cabecera ESTÁNDAR (pegar como 1ª celda de código de CADA notebook)

Reemplaza todos los inventos de `os.path.dirname(os.path.abspath('__file__'))`
(que está MAL: `'__file__'` entre comillas es un string literal, no la variable).

```python
# === Rutas del proyecto (robusto en .py, notebook y Colab) ===
import os
from pathlib import Path

def _project_root() -> Path:
    # 1) Si se ejecuta como script .py, __file__ existe
    try:
        return Path(__file__).resolve().parent
    except NameError:
        pass
    # 2) Notebook/Colab: usar el cwd y subir hasta encontrar config.json o el CSV
    p = Path.cwd().resolve()
    for cand in [p, *p.parents]:
        if (cand / "config.json").exists() or (cand / "23f_scrappedDF.csv").exists():
            return cand
    return p  # fallback: cwd

BASE_DIR    = _project_root()
DATA_DIR    = BASE_DIR                      # los CSV viven en la raíz del repo
OUTPUT_DIR  = BASE_DIR / "outputs"
FIGURES_DIR = OUTPUT_DIR / "figures"
TABLES_DIR  = OUTPUT_DIR / "tables"
for d in (OUTPUT_DIR, FIGURES_DIR, TABLES_DIR):
    d.mkdir(parents=True, exist_ok=True)

CSV_PRINCIPAL = DATA_DIR / "23f_scrappedDF.csv"
print("BASE_DIR:", BASE_DIR)
```

A partir de aquí, en TODOS los notebooks:
- `pd.read_csv('algo.csv')`  →  `pd.read_csv(DATA_DIR / 'algo.csv')`
- `df.to_csv('salida.csv')`  →  `df.to_csv(OUTPUT_DIR / 'salida.csv', index=False)`

---

## 1) 23f_clasificacion_participantes.ipynb

RUTA ABSOLUTA → relativa.

Celda 3:
```python
# ANTES
DATA_PATH = Path("/mnt/data/23f_scrappedDF.csv")
# DESPUÉS
DATA_PATH = CSV_PRINCIPAL
```

Celda 22:
```python
# ANTES
OUTPUT_ALL   = Path("/mnt/data/23f_frases_candidatas.csv")
OUTPUT_LABEL = Path("/mnt/data/23f_frases_para_etiquetar.csv")
# DESPUÉS
OUTPUT_ALL   = OUTPUT_DIR / "23f_frases_candidatas.csv"
OUTPUT_LABEL = OUTPUT_DIR / "23f_frases_para_etiquetar.csv"
```

---

## 2) Buscar_contradicciones_v4.5.ipynb

Bug del `'__file__'`. El bloque de cabecera ya define BASE_DIR; usa config.json relativo.

Celda 2:
```python
# ANTES
CONFIG_PATH = os.path.join(os.path.dirname(os.path.abspath('__file__')), 'config.json')
with open(CONFIG_PATH, encoding='utf-8') as f:
    cfg = json.load(f)
RUTA_CSV        = cfg['rutas']['csv_principal']
RUTA_STATEMENTS = cfg['rutas']['output_statements']
RUTA_EMBEDDINGS = cfg['rutas']['output_embeddings']
RUTA_CONTRA     = cfg['rutas']['output_contradicciones']
...
os.makedirs('outputs', exist_ok=True)

# DESPUÉS
CONFIG_PATH = BASE_DIR / 'config.json'
with open(CONFIG_PATH, encoding='utf-8') as f:
    cfg = json.load(f)
# Las rutas del config son relativas al repo → anclarlas a BASE_DIR
RUTA_CSV        = BASE_DIR / cfg['rutas']['csv_principal']
RUTA_STATEMENTS = BASE_DIR / cfg['rutas']['output_statements']
RUTA_EMBEDDINGS = BASE_DIR / cfg['rutas']['output_embeddings']
RUTA_CONTRA     = BASE_DIR / cfg['rutas']['output_contradicciones']
# (OUTPUT_DIR ya se crea en el bloque de cabecera; la línea os.makedirs sobra)
```

---

## 3) embedding.ipynb

Dos variables SIN DEFINIR (`ruta`, `guardar`) → NameError. Genera el embeddings.csv
que luego consume el grafo, así que debe escribir en outputs/.

Celda 1 (quitar el login con clave en claro):
```python
# ANTES
from huggingface_hub import login
login("Tu clave de huggings face")
# DESPUÉS  (el modelo es público; no hace falta login. Si algún día lo necesitas:)
# import os; from huggingface_hub import login
# if os.getenv("HF_TOKEN"): login(os.environ["HF_TOKEN"])
```

Celda 2:
```python
# ANTES
df = pd.read_csv(ruta)
# DESPUÉS
df = pd.read_csv(CSV_PRINCIPAL)
```

Celda 14:
```python
# ANTES
guardar = r'Tu ruta de descarga/'
df_chunks.to_csv(guardar+'embeddings.csv', index=False)
# DESPUÉS
df_chunks.to_csv(OUTPUT_DIR / 'embeddings.csv', index=False)
```

---

## 4) Grafo-Flujo-InformacionesFINAL-LISTO.ipynb  (versión a conservar)

Mismo bug de `'__file__'` + busca embeddings.csv en sitios que no existen.
Ahora el embeddings.csv lo escribe embedding.ipynb en outputs/.

Celda 9:
```python
# ANTES
BASE_DIR = os.path.dirname(os.path.abspath('__file__'))
DATA_CANDIDATES = [
    os.path.join(BASE_DIR, 'embeddings.csv'),
    os.path.join(BASE_DIR, 'data', 'embeddings.csv'),
    os.path.join(BASE_DIR, '..', 'data', 'embeddings.csv'),
    os.path.join(BASE_DIR, 'embedding-2.csv'),
    os.path.join(BASE_DIR, 'data', 'embedding-2.csv'),
]
CSV_PATH = next((p for p in DATA_CANDIDATES if os.path.exists(p)), None)
...
OUTPUT_DIR  = os.path.join(BASE_DIR, 'outputs')
FIGURES_DIR = os.path.join(OUTPUT_DIR, 'figures')
TABLES_DIR  = os.path.join(OUTPUT_DIR, 'tables')

# DESPUÉS  (BASE_DIR / OUTPUT_DIR / FIGURES_DIR / TABLES_DIR ya vienen de la cabecera)
DATA_CANDIDATES = [
    OUTPUT_DIR / 'embeddings.csv',     # lo genera embedding.ipynb
    DATA_DIR   / 'embeddings.csv',
]
CSV_PATH = next((p for p in DATA_CANDIDATES if p.exists()), None)
if CSV_PATH is None:
    raise FileNotFoundError(
        "Falta embeddings.csv. Ejecuta antes embedding.ipynb.\n"
        "Rutas buscadas:\n" + "\n".join(f"  - {p}" for p in DATA_CANDIDATES)
    )
```

Celda 11 (`pd.read_csv(`): asegúrate de que lee de `CSV_PATH`.

> Borra el duplicado `Grafo_flujo_informaciones.ipynb` para no confundir al profesor.

---

## 5) Relaciones_23F_v1_3.ipynb

Quitar Colab y pip inline; ruta relativa de salida.

Celda 2 (`!pip install ...`): elimínala y pon las libs en requirements.txt.

Celda 5:
```python
# ANTES
from google.colab import files
files.upload()
# DESPUÉS  (no subir a mano: leer del repo)
# (eliminar la celda; el df se carga abajo)
```

Celda 6:
```python
# ANTES
df = pd.read_csv('23f_scrappedDF.csv')
# DESPUÉS
df = pd.read_csv(CSV_PRINCIPAL)
```

Celda 45:
```python
# ANTES
df_entities.to_csv("fase1_entities_by_doc.csv", index=False)
df_mentions.to_csv("fase1_mentions.csv", index=False)
# DESPUÉS
df_entities.to_csv(OUTPUT_DIR / "fase1_entities_by_doc.csv", index=False)
df_mentions.to_csv(OUTPUT_DIR / "fase1_mentions.csv", index=False)
```

---

## 6) 23-F_Download.ipynb  (scraper; conservar uno de los dos de descarga)

Celda 1:
```python
# ANTES
filepath = os.path.join("transcripts", filename)
...
df.to_csv("23f_scrappedDF.csv", index=False, encoding="utf-8")
# DESPUÉS
(BASE_DIR / "transcripts").mkdir(exist_ok=True)
filepath = BASE_DIR / "transcripts" / filename
...
df.to_csv(CSV_PRINCIPAL, index=False, encoding="utf-8")
```

Celda 6 (`from google.colab import files; files.upload()`): eliminar.

Celda 7:
```python
df = pd.read_csv(CSV_PRINCIPAL)
```

---

## Nota sobre spaCy (no es ruta pero rompe la ejecución)
Varios notebooks hacen `spacy.load("es_core_news_lg")`. Ese modelo NO se instala con
pip normal; hay que descargarlo. Añade una celda (o al requirements):
```python
import spacy
try:
    nlp = spacy.load("es_core_news_lg")
except OSError:
    from spacy.cli import download
    download("es_core_news_lg")
    nlp = spacy.load("es_core_news_lg")
```
