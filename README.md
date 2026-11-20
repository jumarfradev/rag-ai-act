# Asistente RAG sobre el AI Act

> **Estado:** fase inicial. Este README describe lo que queremos construir y se irá completando a medida que avance el proyecto.

## Qué queremos hacer

Vamos a construir un **RAG** (*Retrieval-Augmented Generation*, generación aumentada por recuperación): un asistente al que se le pueden hacer preguntas en lenguaje natural sobre el **AI Act**, el Reglamento Europeo de Inteligencia Artificial (Reglamento (UE) 2024/1689), y que responde apoyándose en el texto de la propia norma.

El asistente no contesta «de memoria». Antes de responder, busca en una **base de datos vectorial** los fragmentos de la legislación más relacionados con la pregunta y redacta la respuesta a partir de ellos, indicando de qué artículo, considerando o anexo sale cada afirmación.

## Cómo funciona un RAG

```
  INGESTA (una vez)                          CONSULTA (cada pregunta)

  Texto del AI Act                           Pregunta del usuario
        │                                            │
        ▼                                            ▼
  Troceado en fragmentos                     Embedding de la pregunta
        │                                            │
        ▼                                            ▼
  Embeddings (vectores)  ──────────►  Base de datos vectorial
                                             │
                                             ▼
                                  Fragmentos más relevantes
                                             │
                                             ▼
                                  Modelo de lenguaje (LLM)
                                             │
                                             ▼
                                  Respuesta con citas a los artículos
```

1. **Ingesta.** Se carga el texto del Reglamento, se divide en fragmentos y cada fragmento se convierte en un vector (*embedding*) que se guarda en la base de datos vectorial junto con su procedencia.
2. **Recuperación.** La pregunta se convierte también en un vector y se buscan los fragmentos más parecidos.
3. **Generación.** El modelo de lenguaje recibe la pregunta y los fragmentos recuperados y redacta la respuesta citando las fuentes.

## Objetivos

- Consultar la legislación del AI Act en lenguaje natural.
- Respuestas basadas en el texto de la norma, con referencia al artículo, considerando o anexo.
- Que el asistente reconozca cuándo la norma no responde a la pregunta, en lugar de inventar.
- Proyecto reproducible con Docker.

## Decisiones

Las decisiones técnicas se documentan aquí según se van tomando.

| Decisión | Estado |
|---|---|
| Tema: asistente de consulta sobre el AI Act | Decidido |
| Lenguaje: Python, con Docker | Decidido |
| Dos entornos: desarrollo y producción | Decidido |
| Dónde se despliega producción | Por decidir |
| Base de datos vectorial | Por decidir |
| Modelo de embeddings | Por decidir |
| Modelo de lenguaje (LLM) | Por decidir |
| Estrategia de troceado del texto | Por decidir |
| Interfaz de consulta | Por decidir |

## Entornos

El proyecto tiene dos entornos separados. Comparten el código, pero no las claves ni los datos.

| | Desarrollo | Producción |
|---|---|---|
| Para qué | Probar y romper cosas sin riesgo | La versión estable que se enseña y se usa |
| Rama de Git | `dev` | `main` |
| Variables | `.env.dev` | `.env.prod` |
| Docker Compose | `docker-compose.dev.yml` | `docker-compose.prod.yml` |
| Código | Montado en vivo desde `src/` | Dentro de la imagen, sin montar |
| Índice vectorial | Volumen y colección `ai_act_dev` | Volumen y colección `ai_act_prod` |
| Logs | `DEBUG` | `INFO` |
| Puerto local | 8000 | 8080 |

Forma de trabajar: el día a día se hace en la rama `dev`; cuando algo está probado, se pasa a `main` con un pull request.

```bash
# Desarrollo
docker compose -f docker-compose.yml -f docker-compose.dev.yml up --build

# Producción
docker compose -f docker-compose.yml -f docker-compose.prod.yml up --build -d
```

Estos comandos funcionarán cuando exista el `Dockerfile`, todavía pendiente.

## Estructura prevista

Orientativa; se ajustará cuando empiece el desarrollo.

```
.
├── data/
│   └── raw/                  # Texto original del Reglamento
├── src/                      # Código: ingesta, recuperación y generación
├── tests/                    # Pruebas
├── docs/                     # Documentación y decisiones
├── .env.dev.example          # Plantilla de variables de desarrollo
├── .env.prod.example         # Plantilla de variables de producción
├── docker-compose.yml        # Base común
├── docker-compose.dev.yml    # Entorno de desarrollo
├── docker-compose.prod.yml   # Entorno de producción
├── .gitignore
├── .dockerignore
├── SECURITY.md
└── README.md
```

## Puesta en marcha

Pendiente de completar cuando exista código. Lo que ya aplica:

```bash
# 1. Clonar el repositorio
git clone <URL-del-repositorio>
cd <carpeta-del-repositorio>

# 2. Crear los archivos de variables de cada entorno y rellenarlos
cp .env.dev.example .env.dev
cp .env.prod.example .env.prod

# 3. Activar las comprobaciones de seguridad antes de cada commit
pip install pre-commit
pre-commit install
```

## Seguridad

- Las claves y contraseñas van solo en `.env.dev` y `.env.prod`, que no se suben al repositorio. Cada entorno usa sus propias claves.
- Cada commit pasa por un detector de secretos (pre-commit + gitleaks) y GitHub lo vuelve a revisar en cada push.
- Las normas completas están en [SECURITY.md](SECURITY.md).

## Aviso

Este asistente es una herramienta de consulta y aprendizaje. Sus respuestas no constituyen asesoramiento jurídico; ante cualquier duda prevalece el texto oficial del Reglamento publicado en el Diario Oficial de la Unión Europea.
