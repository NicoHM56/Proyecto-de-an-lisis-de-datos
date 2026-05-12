# Sobreviví al Cancer...ahora que?
El proyecto analiza, con herramientas de datos, los niveles de radiación en pacientes tras radioterapia. Busca evaluar efectos biológicos posteriores y su relación con la dosis y el tiempo, para mejorar el seguimiento clínico, la seguridad radiológica y la toma de decisiones basada en evidencia.


> ⚠️ **Proyecto Universitario** Este modelo en ninguna manera busca ser utilizado
> por profesionales de la salud. El proposito de este modelo es meramente ilustrar
> los conocimientos obtenidos durante el curso de Análisis de Datos y como el uso
> de base de datos, sql, y modelos de datos puede ser trascendental en el ambiente
> médico. Como el entrenar correctamente modelos de datos puede ayudar a determinar
> relaciones que antes no se habían visto o considerado y como esto puede impactar
> positivamente no solo en el cuidado de los pacientes sino en la carga de los
> trabajadores médicos

## Contexto del Patient Journey

**Post alta o post tratamiento:**¿qué pasa con los niveles de radiación en pacientes que recibieron quimioterapia después del tratamiento?

**Pregunta clínica:** Determinar si los pacientes que han recibido radioterapia requieren un seguimiento clínico más intensivo o personalizado, en función de la dosis acumulada y el riesgo de desarrollar efectos biológicos tardíos, con el fin de intervenir de manera temprana y prevenir complicaciones a largo plazo.

¿Existe una relación estadísticamente significativa entre la dosis acumulada de radioterapia recibida por pacientes oncológicos y la presencia de niveles detectables de radiación o efectos biológicos posteriores al tratamiento durante el seguimiento clínico? 

**¿Que es lo que informa el modelo de datos?:** El modelo de datos tras su analisis determina cual es la probabilidad de que el paciente pueda recaer en tener cáncer debido a los niveles residuales de radiación en su cuerpo. En base a esto lo clasifica para indicarle al usuario que tan frecuentemente deberia de ir al hospital este paciente por chequeos medicos. Este modelo en ningun modo pretende reemplazar el criterio médico, más ayudar al doctor a tener en la periferia los casos más delicados, con la mayor probabilidad de recaer.

**Dataset:** Para alimentar este modelo de datos se utilizo la base de datos **CCSS: Childhood Cancer Survivor Study**. Esta base de datos se enfoca en niños que sobrevivieron al cancer, lo que hace su base de datos ideal para este modelo que se plantea puesto que en niños hay más tiempo para que se presenten compplicaciones por acumulación de radiación. Especialmente si el cáncer regresa cuando el tratamiento no fue 100% efectivo. En este caso la acumulación es doble si logran vencer el cáncer nuevamente. 

## Resultados

| Métrica | Valor |
|---|---|
| AUC cross-validation (5-fold) | 0.901 ± 0.047 |
| AUC test (hold-out, n=75) | 0.914 |
| Sensibilidad | 80.0% |
| Especificidad | 87.5% |
| Modelo | Regresión logística regularizada |
| N tras curación | 297 (de 303 — se descartaron 6 con datos faltantes) |

Detalle completo en [notebooks/dashboard.ipynb](notebooks/dashboard.ipynb).

## Arquitectura: notebook delgado, módulos gordos

El notebook se lee como un **dashboard ejecutivo**: solo importa funciones
del paquete `src/`, las llama, y narra los resultados. Toda la lógica vive
en módulos `.py` testables. Esto es importante porque:

- Notebooks grandes son imposibles de revisar y reproducir.
- La lógica en módulos se puede testear (`tests/`), reusar y versionar bien.
- Los gráficos se rinden cuando el notebook los muestra, no cuando la
  función los crea — se puede cambiar el orden o suprimir sin reorganizar.

```
proyecto/
├── README.md                 ← este archivo
├── docker-compose.yml        ← mismo stack que los labs (jupyter/scipy-notebook)
├── requirements.txt
├── Makefile                  ← orquestación: `make all` desde cero a HTML
├── .gitignore                ← db/, processed/, html, checkpoints
├── data/
│   ├── README.md             ← procedencia, licencia, diccionario
│   └── raw/heart.csv         ← dataset UCI commiteado (público, 20KB)
├── db/                       ← SQLite generada (gitignored)
├── src/
│   ├── ingest.py             ← CSV → SQLite (raw_patients)
│   ├── curate.py             ← raw_patients → curated_patients
│   ├── eda.py                ← funciones que retornan Figure
│   ├── model.py              ← pipeline de regresión logística
│   └── validate.py           ← CV, ROC, métricas clínicas
├── notebooks/
│   └── dashboard.ipynb       ← orquesta todo, narra los resultados
└── tests/
    └── test_curate.py        ← smoke tests de curación
```

## Cómo correrlo

### Opción 1: Docker (recomendado, igual que los labs)

```bash
docker compose up
```

Abre http://localhost:8888 → `notebooks/dashboard.ipynb` → Run All.

### Opción 2: Pipeline completo automatizado (Makefile)

```bash
make all      # setup + ingest + curate + tests + render HTML
```

`make all` deja `notebooks/dashboard.html` listo para abrir.

> **El Makefile es un complemento, no un reemplazo.** Los labs usan
> `docker-compose` y eso aprendiste. Este proyecto muestra que puedes
> sumar herramientas (orquestación con Make, tests con pytest) cuando el
> proyecto crece. No estás obligado a usar Make — pero tampoco estás
> obligado a quedarte solo con lo que vimos en clase.

### Opción 3: Pipeline paso a paso

```bash
make ingest   # data/raw/heart.csv → db/heart.db (raw_patients)
make curate   # raw_patients → curated_patients
make test     # pytest
make render   # ejecuta el notebook y genera HTML
```

## Reproducibilidad

- `make all` desde un repo recién clonado debe producir el HTML sin
  intervención manual.
- La base SQLite (`db/heart.db`) **no** se commitea: es regenerable. Esto
  garantiza que cualquier resultado se reconstruye desde el CSV crudo.
- Versiones pinneadas en `requirements.txt`.

## Lo que este ejemplo intenta enseñar

1. **Trazabilidad clínica.** Cada decisión técnica está atada a una
   pregunta clínica concreta y un costo de error explícito.
2. **Honestidad metodológica.** Excluir `num` para evitar leakage,
   reportar sensibilidad/especificidad (no solo accuracy), documentar
   limitaciones reales.
3. **Reproducibilidad.** De CSV a HTML con un comando, sin estado mágico
   en notebooks.
4. **Estructura.** Notebook como dashboard, lógica en módulos, tests
   sobre lo que importa.

## Lo que este ejemplo NO intenta enseñar

- Modelos sofisticados. La regresión logística está justificada por N=297.
  En tu proyecto, el modelo correcto depende de tu pregunta y tu dataset.
- Visualización exhaustiva. Los gráficos del notebook responden preguntas
  específicas; no es un tour de seaborn.
- Despliegue. Esto es un análisis reproducible, no un sistema en producción.
