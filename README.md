Fundamentos-Ingenieria-Datos
==============================

Proyecto integrador de la asignatura Fundamentos de Ingeniería de Datos

Project Organization
------------

```
fundamentos-ingenieria-datos/
├── LICENSE     
├── README.md                  
├── Makefile                     # Makefile with commands like `make data` or `make train`                   
├── configs                      # Config files (models and training hyperparameters)
│   └── model1.yaml              
│
├── data                         
│   ├── external                 # Data from third party sources.
│   ├── interim                  # Intermediate data that has been transformed.
│   ├── processed                # The final, canonical data sets for modeling.
│   └── raw                      # The original, immutable data dump.
│
├── docs                         # Project documentation.
│
├── models                       # Trained and serialized models.
│
├── notebooks                    # Jupyter notebooks.
│
├── references                   # Data dictionaries, manuals, and all other explanatory materials.
│
├── reports                      # Generated analysis as HTML, PDF, LaTeX, etc.
│   └── figures                  # Generated graphics and figures to be used in reporting.
│
├── requirements.txt             # The requirements file for reproducing the analysis environment.
└── src                          # Source code for use in this project.
    ├── __init__.py              # Makes src a Python module.
    │
    ├── data                     # Data engineering scripts.
    │   ├── build_features.py    
    │   ├── cleaning.py          
    │   ├── ingestion.py         
    │   ├── labeling.py          
    │   ├── splitting.py         
    │   └── validation.py        
    │
    ├── models                   # ML model engineering (a folder for each model).
    │   └── model1      
    │       ├── dataloader.py    
    │       ├── hyperparameters_tuning.py 
    │       ├── model.py         
    │       ├── predict.py       
    │       ├── preprocessing.py 
    │       └── train.py         
    │
    └── visualization        # Scripts to create exploratory and results oriented visualizations.
        ├── evaluation.py        
        └── exploration.py       
```


--------
<p><small>Project based on the <a target="_blank" href="https://github.com/Chim-SO/cookiecutter-mlops/">cookiecutter MLOps project template</a>
that is originally based on <a target="_blank" href="https://drivendata.github.io/cookiecutter-data-science/">cookiecutter data science project template</a>. 
#cookiecuttermlops #cookiecutterdatascience</small></p>

# Scraper de Noticias – El Universal

## 1. Sitio elegido
Se seleccionó la sección **“Minuto x Minuto”** de *El Universal* (`https://www.eluniversal.com.mx/minuto-x-minuto`) como fuente de noticias.

**Motivos:**
- Noticias actualizadas constantemente.
- Acceso público sin autenticación.
- Estructura HTML relativamente consistente para extraer título, enlace, fecha y autor.

---

## 2. Flujo de Adquisición de Datos

### Fase 1: Scraping
- **Script:** `src/scraper.py`
- **Función principal:** `scrape_eluniversal(max_pages=5)`
- **Acciones realizadas:**
  1. Navega por las primeras `max_pages` de la sección Minuto x Minuto.
  2. Extrae títulos y enlaces de noticias.
  3. Accede a cada enlace para obtener fecha de publicación y autor.
  4. Genera un ID único usando `hashlib.md5`.
  5. Almacena cada noticia como diccionario.

### Fase 2: Guardado de datos
- **Función:** `save_jsonl(data, filename="data/raw/noticias.jsonl")`
- **Acción:** Guarda los datos en `data/raw/noticias.jsonl` en formato JSONL, una noticia por línea.

### Fase 3: Perfilado de Calidad
- **Función:** `profile_data(articles, report_path="reports/perfilado.md")`
- **Acción:**  
  - Calcula número total de registros.
  - Detecta valores nulos por campo.
  - Detecta duplicados por URL o ID.
  - Verifica consistencia en formatos de fechas y URLs.
  - Genera reporte Markdown en `reports/perfilado.md`.

---

## 3. Construcción del Scraper
- **Lenguaje:** Python
- **Librerías:** `requests`, `BeautifulSoup`, `json`, `hashlib`, `datetime`, `os`, `time`, `random`
- **Proceso resumido:**
  1. Configura cabeceras HTTP para simular navegador y evitar bloqueos.
  2. Recorre las páginas de noticias (`/todos/n` para paginación).
  3. Extrae título y URL de cada noticia.
  4. Accede a cada noticia para extraer fecha y autor (clase `.sc__author-nota`).
  5. Genera diccionario con campos: `id`, `titulo`, `fecha`, `url`, `fuente`, `autor`, `capturado_ts`.
  6. Guarda datos en JSONL.
  7. Genera perfil de calidad en Markdown.

---

## 4. Problemas encontrados
- **Bloqueos HTTP:** Algunos sitios requieren cabeceras o sesiones especiales para evitar errores 401/403.
- **Estructura HTML cambiante:** Se identificó correctamente la clase para extraer autores.
- **Paginación:** Las páginas siguientes usan `/todos/n` y no `?page=n`.
- **Duplicados:** Algunos enlaces se repetían; se implementó un `set` para URLs.
- **Carga del servidor:** Se agregaron pausas aleatorias (`time.sleep(random.uniform(1,2))`) para no saturar el sitio.

---

## 5. Archivos Generados
- `data/raw/noticias.jsonl` → Contiene todas las noticias obtenidas en formato JSONL.
- `reports/perfilado.md` → Contiene perfil de calidad de los datos (total de registros, nulos, duplicados y consistencia de formatos).
- `contracts/schema.yaml` → Define el contrato de datos (tipo de dato, obligatoriedad, unicidad, formato).

---

## 6. Uso
Ejecuta el scraper con:

```bash
python src/scraper.py