# A.L.P.E.K. Releases — instrucciones para agentes
Repositorio público solo de distribución de A.L.P.E.K. No contiene código fuente. Responde en español (es-MX).

## Qué hay aquí
- `Updates/`: la versión vigente (`latest.json` y su firma `latest.json.sig`).
- `Old Versions/`: manifiestos y firmas de versiones ya reemplazadas.
- Releases de GitHub: instaladores, paquetes e índices (los archivos pesados no van dentro del repo).
- `README.md`: cómo se organiza y cómo se verifica un instalador.

## Reglas fijas
- Sin secretos en archivos, commits, issues, PR ni respuestas. Este repo es público: nada de rutas locales, nombres de cuentas ni datos privados.
- Cambios solo por rama y PR; nunca commits directos en `main`.
- `Updates/latest.json` y `latest.json.sig` son un manifiesto firmado: no se editan a mano ni se regeneran aquí. Los publica el dueño con su flujo de publicación.
- Una versión publicada no se sobrescribe y no se borran Releases ni archivos de `Old Versions/`.
- No añadas código fuente, instaladores ni paquetes dentro del repo.
- Publicar o cambiar lo que descargan los usuarios solo con el visto bueno expreso del dueño.
- Afirma solo lo verificado; lo supuesto se marca `Assumption` y lo desconocido `TBD`.
