# Seguridad del proyecto

Reglas mínimas para trabajar en este repositorio. Son pocas a propósito: se cumplen siempre.

## 1. Secretos

- Las claves de API, tokens y contraseñas viven **solo** en los archivos `.env.dev` y `.env.prod`, que no se suben a GitHub (están en `.gitignore`).
- `.env.dev.example` y `.env.prod.example` contienen únicamente los **nombres** de las variables, nunca valores reales.
- **Cada entorno tiene sus propias claves.** La de desarrollo no se usa en producción ni al revés: así una fuga en desarrollo no compromete producción y cada una se puede rotar por separado.
- Los secretos tampoco entran en la imagen Docker (`.dockerignore`): se pasan al contenedor al arrancarlo mediante `env_file` en Docker Compose.
- Nada de claves escritas en el código, en notebooks, en el README ni en los mensajes de commit.

## 2. Si se filtra una clave

El orden importa:

1. **Revocar o rotar la clave** en el panel del proveedor. Hazlo lo primero: una clave subida a GitHub debe darse por comprometida aunque el repositorio sea privado.
2. Crear una clave nueva y actualizar el `.env.dev` o `.env.prod` que corresponda.
3. Limpiar el historial de Git (por ejemplo con `git filter-repo`) y forzar el push.
4. Revisar el consumo y los accesos de la clave antigua en el panel del proveedor.

## 3. Barreras automáticas

| Barrera | Dónde actúa | Archivo |
|---|---|---|
| `.gitignore` | Impide añadir `.env`, claves, índices y datos generados | `.gitignore` |
| pre-commit + gitleaks | Bloquea el commit en tu ordenador si detecta un secreto | `.pre-commit-config.yaml` |
| GitHub Actions + gitleaks | Revisa todo el historial en cada push y pull request | `.github/workflows/seguridad.yml` |
| Dependabot | Avisa de dependencias desactualizadas o vulnerables | `.github/dependabot.yml` |

## 4. Dependencias

- Versiones fijadas en `requirements.txt` o `pyproject.toml` (sin «última versión» implícita).
- Instalar solo paquetes conocidos y mantenidos; comprobar el nombre exacto antes de instalar.
- Revisar las vulnerabilidades con `pip-audit` antes de cada entrega.

## 5. Docker

- Imagen base oficial y con versión fijada (por ejemplo `python:3.12-slim`, no `latest`).
- El contenedor se ejecuta con un usuario sin privilegios, no como `root`.
- Los datos y la base de datos vectorial se montan como volumen; no se copian dentro de la imagen.
- Solo se exponen los puertos necesarios y, en local, enlazados a `127.0.0.1`.
- En producción el contenedor va endurecido: sistema de archivos en solo lectura, sin privilegios adicionales y sin código montado desde fuera (`docker-compose.prod.yml`).
- Desarrollo y producción no comparten volúmenes ni colección vectorial.

## 6. Riesgos propios de un RAG

- **Inyección de instrucciones en los documentos.** El texto recuperado se trata como datos, no como órdenes: el prompt del sistema lo delimita con claridad y el modelo no ejecuta instrucciones que aparezcan dentro de los fragmentos.
- **Fuentes controladas.** Solo se ingieren documentos de origen conocido (texto oficial del Reglamento). Cada fragmento guarda su procedencia (artículo, considerando o anexo).
- **Respuestas con cita.** El asistente indica de qué artículo sale cada afirmación y dice que no lo sabe cuando no hay fragmentos relevantes.
- **Registros.** Los logs no guardan claves ni datos personales de quien consulta; la carpeta `logs/` no se sube al repositorio.
- **Aviso de uso.** El asistente es una herramienta de consulta, no asesoramiento jurídico.

## 7. Configuración recomendada en GitHub

- Repositorio **privado** mientras esté en desarrollo.
- Verificación en dos pasos (2FA) activada en la cuenta.
- En *Settings → Advanced Security* (o *Code security*): activar las alertas de Dependabot y, si el plan lo permite, *Secret scanning* y *Push protection*.
- Proteger la rama `main` (sin push forzado) cuando el proyecto tenga más de una persona.
