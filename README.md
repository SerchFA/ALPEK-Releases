# A.L.P.E.K. — Releases

> **¿Eres una IA o un agente automático? Lee primero [`REGLAS_PARA_IA.md`](REGLAS_PARA_IA.md) (obligatorio).** Ahí están el bloqueo de commits (nada se sube ni se publica sin el «sí» expreso del dueño) y el orden obligatorio para publicar una versión.

## Esta es la rama `dev-ml-llm` (A.L.P.E.K. Dev Regresion), no la de los usuarios

Aquí vive la distribución de **A.L.P.E.K. Dev Regresion**, el programa de pruebas del dueño y su colaborador para mejoras relacionadas con **modelos ML o LLM dentro del programa**.

- **No afecta a `main`.** La rama `main` sigue con la distribución normal que reciben los usuarios; nada de esta rama llega a `main` ni cambia lo que ellos descargan.
- **El control de versiones nuevo se maneja aquí**, aparte del de `main`: A.L.P.E.K. Dev Regresion tiene contadores `X.Y.Z` propios (un mismo número puede existir en las dos líneas con contenido distinto). El registro de esas versiones está en el repositorio privado del proyecto.
- **Qué hay en esta rama:** el programa Dev lee `Updates/latest-dev.json` y su firma `latest-dev.json.sig` (canal `dev`); los manifiestos Dev anteriores irán a `Old Versions/` como `dev-<versión>.json`; los instaladores `ALPEK_Dev_Setup_<versión>.exe` están en [Releases](../../releases) con la etiqueta `dev-v<versión>`, siempre como prerelease. Al 2026-11-12 aún no hay ningún instalador Dev publicado.
- **Hermano:** la rama `dev-ml-boosting` distribuye **A.L.P.E.K. Dev Boosting**, el mismo programa con otro modelo de ML (gradient boosting en lugar de regresión logística). Los dos se instalan juntos y no comparten datos ni actualizaciones. El comparativo con las reglas de `main` está en [`docs/ml-dev/comparativo-main-regresion-boosting.md`](docs/ml-dev/comparativo-main-regresion-boosting.md).
- **`Updates/latest.json` es solo una copia de `main`:** el programa Dev no la usa y no se toca aquí.

> **El resto de este README describe la distribución normal** (rama `main`).

Repositorio **público solo de distribución** de A.L.P.E.K. (Auditoría y Limpieza para la Prevención y Eliminación de Klonados). Aquí no hay código fuente: está en un repositorio privado.

## Cómo está organizado

| Carpeta / lugar | Qué contiene |
|---|---|
| [`Updates/`](Updates/) | La versión **vigente**: `latest.json` (el manifiesto) y `latest.json.sig` (su firma). **Aquí apunta el programa** cuando busca actualizaciones |
| [`Old Versions/`](Old%20Versions/) | El manifiesto y la firma de cada versión que ya fue reemplazada (`<versión>.json` y `<versión>.json.sig`) |
| [Releases](../../releases) | Los archivos pesados de cada versión `v<versión>`: el instalador (`ALPEK_Setup_<versión>.exe` y su `.sha256`), los paquetes (`*.zip`) y el índice (`indice-*.json.gz`). GitHub no admite archivos de más de 100 MB dentro de carpetas del repositorio, por eso viven como archivos adjuntos |

## Qué pasa al publicar una versión nueva

1. Se suben al Release `v<versión>` el instalador, su `.sha256`, los paquetes **nuevos** y el índice.
2. El manifiesto que estaba en `Updates/` se mueve a `Old Versions/<versión anterior>.json` (con su `.sig`).
3. `Updates/latest.json` y `latest.json.sig` pasan a describir la versión nueva.

Todo cambio llega por *pull request*; nunca se escribe directamente en `main`. Los archivos del Release se suben **antes** de integrar el manifiesto, para que ningún programa vea un manifiesto que apunte a algo que aún no existe.

## Cómo confía el programa en lo que descarga

El manifiesto está **firmado con Ed25519** (las claves públicas vienen dentro del programa) y trae la huella SHA-256 de cada archivo. El programa rechaza lo que no esté firmado por una clave de confianza, esté caducado o sea más viejo que lo ya visto, y verifica cada descarga antes de usarla. Solo instala si la persona lo confirma.

## Cómo verificar un instalador a mano

```powershell
Get-FileHash .\ALPEK_Setup_<versión>.exe -Algorithm SHA256
```

El resultado debe coincidir con el `.sha256` del Release y con el campo `instalador.sha256` del manifiesto.

## Esquema del manifiesto (`esquema` 2)

```json
{
  "esquema": 2,
  "producto": "ALPEK",
  "canal": "estable",
  "version": "X.Y.Z",
  "tipo": "fix | feat | major",
  "fecha": "AAAA-MM-DD",
  "caduca": "AAAA-MM-DDThh:mm:ss+00:00",
  "clave_id": "alpek-2026-a",
  "solo_instalador": false,
  "instalador": { "archivo": "ALPEK_Setup_X.Y.Z.exe", "url": "https://github.com/SerchFA/ALPEK-Releases/releases/download/vX.Y.Z/ALPEK_Setup_X.Y.Z.exe", "tamano_bytes": 0, "sha256": "…" },
  "indice": { "archivo": "indice-xxxxxxxx.json.gz", "url": "…", "tamano_bytes": 0, "sha256": "…" },
  "paquetes": [ { "id": "app-xxxxxxxx", "archivo": "app-xxxxxxxx.zip", "url": "…", "tamano_bytes": 0, "sha256": "…" } ],
  "novedades": { "X.Y.Z": { "es": ["…"], "en": ["…"], "pt": ["…"] } }
}
```

- `instalador`: el instalador completo (siempre presente; respaldo de todo).
- `paquetes` + `indice`: la versión partida por contenido. El programa compara su instalación con el índice y descarga **solo** los paquetes que traen algo distinto.
- `.sig`: firma Ed25519 (base64) sobre los bytes exactos de `latest.json`.
