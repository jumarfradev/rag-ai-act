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
| Base de datos vectorial | Por decidir |
| Modelo de embeddings | Por decidir |
| Modelo de lenguaje (LLM) | Por decidir |
| Estrategia de troceado del texto | Por decidir |
| Interfaz de consulta | Por decidir |

## Estructura prevista

Orientativa; se ajustará cuando empiece el desarrollo.

```
.
├── data/
│   └── raw/            # Texto original del Reglamento
├── src/                # Código: ingesta, recuperación y generación
├── tests/              # Pruebas
├── docs/               # Documentación y decisiones
├── .env.example        # Plantilla de variables de entorno
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

# 2. Crear el archivo de variables de entorno y rellenarlo
cp .env.example .env

# 3. Activar las comprobaciones de seguridad antes de cada commit
pip install pre-commit
pre-commit install
```

## Seguridad

- Las claves y contraseñas van solo en `.env`, que no se sube al repositorio.
- Cada commit pasa por un detector de secretos (pre-commit + gitleaks) y GitHub lo vuelve a revisar en cada push.
- Las normas completas están en [SECURITY.md](SECURITY.md).

## Aviso

Este asistente es una herramienta de consulta y aprendizaje. Sus respuestas no constituyen asesoramiento jurídico; ante cualquier duda prevalece el texto oficial del Reglamento publicado en el Diario Oficial de la Unión Europea.
