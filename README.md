# A.L.P.E.K. — Releases

Repositorio **público solo de distribución** de A.L.P.E.K. (Auditoría y Limpieza para la Prevención y Eliminación de Klonados). Aquí no hay código fuente: está en un repositorio privado.

## Cómo está organizado

| Carpeta / lugar | Qué contiene |
|---|---|
| [`Updates/`](Updates/) | La versión **vigente**: `latest.json` con el número de versión, el enlace al instalador, su tamaño y su huella SHA-256. **Aquí apunta el programa** cuando busca actualizaciones |
| [`Old Versions/`](Old%20Versions/) | Una ficha `<versión>.json` por cada versión que ya fue reemplazada (mismos datos que `latest.json`) |
| [Releases](../../releases) | Los instaladores (`ALPEK_Setup_<versión>.exe` y su `.sha256`). GitHub no admite archivos de más de 100 MB dentro de carpetas del repositorio, por eso los instaladores viven como archivos adjuntos de cada Release |

## Qué pasa al publicar una versión nueva

1. Se sube el instalador nuevo a un Release `v<versión>` (con su `.sha256`).
2. La ficha que estaba en `Updates/latest.json` se mueve a `Old Versions/<versión anterior>.json` y el Release anterior queda marcado como **anterior**.
3. `Updates/latest.json` pasa a describir la versión nueva.

Todo cambio llega por *pull request*; nunca se escribe directamente en `main`.

## Cómo verificar un instalador

```powershell
Get-FileHash .\ALPEK_Setup_<versión>.exe -Algorithm SHA256
```

El resultado debe coincidir con el `.sha256` del Release y con el campo `sha256` de la ficha.

## Esquema de la ficha

```json
{
  "esquema": 1,
  "version": "X.Y.Z",
  "fecha": "AAAA-MM-DD",
  "tipo": "fix | feat | major",
  "instalador": "ALPEK_Setup_X.Y.Z.exe",
  "url": "https://github.com/SerchFA/ALPEK-Releases/releases/download/vX.Y.Z/ALPEK_Setup_X.Y.Z.exe",
  "tamano_bytes": 0,
  "sha256": "…"
}
```

Aún no hay ninguna versión publicada aquí.
