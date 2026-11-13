# Modelo de ML de A.L.P.E.K. Dev Boosting: entrenamiento y resultados

Rama `dev-ml-boosting`. Esta copia usa **gradient boosting** (árboles pequeños) donde `A.L.P.E.K. Dev Regresion` (rama `dev-ml-llm`) usa regresión logística. **Solo cifras agregadas e importancias de señales:** ningún registro, descripción ni número de SAP.

- **Comparativo completo (reglas de `main` contra regresión contra boosting):** [`comparativo-main-regresion-boosting.md`](comparativo-main-regresion-boosting.md).
- Cómo está construido el modelo: `docs/dev-ml-boosting.md` («Modelo de ML y pruebas con una IA»).

**Corrida comparativa ronda por ronda (2026-11-13):** reglas, regresión y boosting sobre las mismas decisiones de Claude Haiku 5.5 (esfuerzo muy alto) en los mismos escenarios de la tanda 2; reporte con las líneas de cada ronda en [`corrida-comparativa/reporte_algoritmo_regresion_boosting.html`](corrida-comparativa/reporte_algoritmo_regresion_boosting.html) y resumen en el comparativo.

## Límite que no se puede olvidar

Las etiquetas **no son del negocio**: en la fase 1 las puso una IA (Claude Haiku 5.5) y en las fases 2, 3 y la tanda 2 la «verdad» de un generador sintético o, otra vez, Haiku. Todo lo que se mide es acuerdo con esas etiquetas. Hasta que personas del negocio decidan en Resultados (el almacén las guarda con `origen` = `persona`), no se sabe si el modelo mejora el trabajo real.

## Qué se entrenó (mismos parámetros que la regresión)

Se reutilizaron **los mismos conjuntos de pares, la misma partición entrenamiento/auditoría y las mismas métricas** que en `dev-ml-llm`; solo cambió el modelo. Los modelos entregados en `config/ml/` se reentrenaron sobre lo ya decidido en la tanda 2; **después, en la corrida comparativa, Haiku decidió de nuevo** los mismos escenarios con el boosting reentrenándose en cada ronda (ver más abajo y el comparativo).

| Conjunto | Origen de las etiquetas | Qué es |
|---|---|---|
| `haiku` | Claude Haiku 5.5 | Fase 1: planillas reales de Materiales, Equipos y Ubicaciones técnicas; Clientes y proveedores, sintético de unas 3 000 empresas. 12 corridas, 190 grupos por corrida |
| `sintetico` | verdad del generador | Fases 2 y 3: sintético de ~26 000 y ~40 000 registros por corrida, hasta 12 rondas |
| `tanda2` | Claude Haiku 5.5 (esfuerzo muy alto) | Sintético con alto índice de duplicados, ~55 000 registros por corrida; los modelos que se entregan en `codigo/config/ml/` salen de aquí |

Hiperparámetros fijos, sin afinar con la auditoría: 100 árboles como máximo, profundidad 3, tasa 0.1, submuestra 0.8, mínimo 3 pares por hoja, parada temprana (10 rondas sin mejora sobre el 15 % de validación), clases balanceadas, semilla 0. No entrena con menos de 12 pares ni con menos de 5 de una clase.

Resultados (en `modelos/boosting/`): `<conjunto>/resumen.json` (por herramienta: pares de entrenamiento y auditoría, métricas de las reglas y del modelo, importancia de las señales, tamaño), `tanda2/<s3|cli|prov>/antes_despues.json` (los modelos de otros conjuntos medidos en la auditoría de la tanda 2), `comparativo.json` y `generaliza.json` (regresión contra boosting).

## Modelos entregados

| Herramienta | Pares de entrenamiento / auditoría | AP reglas → modelo | AUC reglas → modelo | Revisión al 90 % reglas → modelo | Peso |
|---|---:|---|---|---|---:|
| Materiales | 666 / 186 | 1.000 → **1.000** | 1.000 → **1.000** | 83 % → **83 %** | 12.9 KB |
| Clientes y proveedores | 1 053 / 346 | 0.994 → **1.000** | 0.988 → **1.000** | 59 % → **58 %** | 32.2 KB |
| Equipos | 408 / 119 | 1.000 → **1.000** | 1.000 → **1.000** | 79 % → **79 %** | 12.3 KB |
| Ubicaciones técnicas | 216 / 60 | sin modelo: 1 no duplicado entre 216 pares | | | — |

**Cuidado con cómo se lee:** con datos así de fáciles las reglas ya ordenan casi perfecto, de modo que igualarlas no demuestra nada; y con datos que no vio, el boosting resultó **menos estable que la regresión** (ver el comparativo).
