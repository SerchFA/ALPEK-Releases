# Modelo de ML de A.L.P.E.K. Dev: pruebas y evolución

Rama `dev-ml-llm`. Resumen de lo que se corrió para entrenar y medir el modelo (regresión logística sobre las señales de las reglas). **Solo cifras agregadas y coeficientes**: ningún registro, descripción ni número de SAP.
El detalle visual (gráficas de líneas de cada corrida) está en [`reporte_evolucion_modelo.html`](reporte_evolucion_modelo.html). Cómo está construido: `docs/dev-ml-llm.md` del repositorio privado.

## Límite que no se puede olvidar

Las etiquetas **no son del negocio**: en la fase 1 las puso una IA (Claude Haiku 5.5) y en las fases 2 y 3 la «verdad» de un generador de datos sintéticos. Todo lo que se mide es acuerdo con esas etiquetas.
Hasta que personas del negocio decidan en Resultados (el almacén las guarda con `origen` = `persona` y se entrenan aparte), no se sabe si el modelo mejora el trabajo real.

## Qué se corrió

- **Fase 1 · planillas reales de SAP, decide Claude Haiku 5.5.** 12 corridas (4 herramientas × Estricta, Normal y Flexible), 4 rondas cada una (190 grupos por corrida), con las planillas reales de Materiales (ECC), Equipos y Ubicaciones técnicas (S/4HANA). Clientes y proveedores no tiene planilla real a la mano: se usó un conjunto sintético de unas 3 000 empresas. La IA vio los grupos como en Resultados pero **sin puntaje ni banda**.
- **Fase 2 · datos sintéticos de unos 26 000 registros por corrida**, con verdad conocida, hasta 12 rondas por corrida.
- **Fase 3 · datos sintéticos de unos 40 000 registros por corrida**, otra semilla, hasta 12 rondas.

**Fase 1** (planillas reales; Clientes y proveedores sintético de unas 3 000 empresas):

| Corrida | Registros leídos | Grupos | Rondas | Grupos decididos |
|---|---:|---:|---:|---:|
| Clientes y proveedores · Estricta | 6,734 | 724 | 4 | 189 |
| Clientes y proveedores · Flexible | 6,734 | 777 | 4 | 190 |
| Clientes y proveedores · Normal | 6,734 | 740 | 4 | 190 |
| Equipos · Estricta | 90,718 | 270 | 4 | 190 |
| Equipos · Flexible | 90,718 | 3,288 | 4 | 190 |
| Equipos · Normal | 90,718 | 822 | 4 | 190 |
| Materiales · Estricta | 19,393 | 272 | 4 | 190 |
| Materiales · Flexible | 19,393 | 993 | 4 | 190 |
| Materiales · Normal | 19,393 | 525 | 4 | 190 |
| Ubicaciones técnicas · Estricta | 17,517 | 13 | 4 | 13 |
| Ubicaciones técnicas · Flexible | 17,517 | 1,549 | 4 | 190 |
| Ubicaciones técnicas · Normal | 17,517 | 1,485 | 4 | 190 |

**Fase 2** (sintético, semilla 11):

| Corrida | Registros leídos | Grupos | Rondas | Grupos decididos |
|---|---:|---:|---:|---:|
| Clientes y proveedores · Estricta | 28,600 | 2,782 | 12 | 2,661 |
| Clientes y proveedores · Flexible | 28,600 | 3,988 | 12 | 3,622 |
| Clientes y proveedores · Normal | 28,600 | 3,571 | 12 | 3,285 |
| Equipos · Estricta | 26,250 | 5,125 | 12 | 4,432 |
| Equipos · Flexible | 26,250 | 5,500 | 12 | 4,765 |
| Equipos · Normal | 26,250 | 5,500 | 12 | 4,737 |
| Materiales · Estricta | 26,130 | 3,468 | 12 | 3,170 |
| Materiales · Flexible | 26,130 | 5,262 | 12 | 4,549 |
| Materiales · Normal | 26,130 | 3,517 | 12 | 3,184 |
| Ubicaciones técnicas · Estricta | 27,874 | 0 | 12 | 0 |
| Ubicaciones técnicas · Flexible | 27,874 | 1,439 | 12 | 1,439 |
| Ubicaciones técnicas · Normal | 27,874 | 1,439 | 12 | 1,439 |

**Fase 3** (sintético, semilla 23):

| Corrida | Registros leídos | Grupos | Rondas | Grupos decididos |
|---|---:|---:|---:|---:|
| Clientes y proveedores · Estricta | 44,098 | 4,268 | 12 | 3,793 |
| Clientes y proveedores · Flexible | 44,098 | 6,349 | 12 | 5,393 |
| Clientes y proveedores · Normal | 44,098 | 5,503 | 12 | 4,682 |
| Equipos · Estricta | 40,320 | 7,875 | 12 | 6,415 |
| Equipos · Flexible | 40,320 | 8,451 | 12 | 6,893 |
| Equipos · Normal | 40,320 | 8,451 | 12 | 6,878 |
| Materiales · Estricta | 40,200 | 4,724 | 12 | 4,129 |
| Materiales · Flexible | 40,200 | 7,194 | 12 | 5,967 |
| Materiales · Normal | 40,200 | 4,818 | 12 | 4,225 |
| Ubicaciones técnicas · Estricta | 42,852 | 0 | 12 | 0 |
| Ubicaciones técnicas · Flexible | 42,852 | 2,201 | 12 | 2,201 |
| Ubicaciones técnicas · Normal | 42,852 | 2,201 | 12 | 2,201 |

## Modelo final por herramienta

Cada modelo se entrena con **todo** lo aprendido de su herramienta (los tres niveles y todas las semillas) y se mide sobre pares de **auditoría que nunca vio**. «Reglas» es el orden que dan hoy las reglas fijas; «Revisión al 90 %» es la fracción de los pares que hay que revisar, de arriba abajo, para encontrar el 90 % de los duplicados (menos es mejor).

### Entrenado con datos sintéticos (fases 2 y 3)

| Herramienta | Pares de entrenamiento | Pares de auditoría | AP reglas → modelo | AUC reglas → modelo | Revisión al 90 % reglas → modelo | Peso |
|---|---:|---:|---|---|---|---:|
| Materiales | 181,081 | 78,350 | 1.000 → **1.000** | 0.999 → **1.000** | 89 % → **89 %** | 0.7 KB |
| Clientes y proveedores | 7,626 | 2,743 | 0.961 → **1.000** | 0.913 → **1.000** | 74 % → **57 %** | 0.7 KB |
| Equipos | 10,680 | 3,170 | 0.981 → **1.000** | 0.894 → **1.000** | 92 % → **74 %** | 1.3 KB |
| Ubicaciones técnicas | 1,783 | 1,382 | 1.000 → **1.000** | 1.000 → **1.000** | 90 % → **90 %** | 0.6 KB |

### Entrenado con las decisiones de Haiku sobre planillas reales (fase 1)

| Herramienta | Pares de entrenamiento | Pares de auditoría | AP reglas → modelo | AUC reglas → modelo | Revisión al 90 % reglas → modelo | Peso |
|---|---:|---:|---|---|---|---:|
| Materiales | 1,865 | 345 | 0.999 → **0.999** | 0.973 → **0.981** | 86 % → **86 %** | 0.8 KB |
| Clientes y proveedores | 231 | 83 | 0.518 → **1.000** | 0.364 → **1.000** | 92 % → **54 %** | 0.7 KB |
| Equipos | 650 | 275 | 0.939 → **0.994** | 0.960 → **0.996** | 34 % → **33 %** | 5.0 KB |
| Ubicaciones técnicas | 905 | — | no se pudo entrenar: sin las dos clases o sin ejemplos suficientes (0 duplicados entre sus ejemplos) | | | |

## Lo que se aprendió de las pruebas

- **Clientes y proveedores y Equipos:** el modelo ordena mejor que las reglas (en los datos sintéticos baja la revisión necesaria para hallar el 90 % de los duplicados de 74 % a 57 % y de 92 % a 74 %).
- **Materiales y Ubicaciones técnicas:** las reglas ya ordenan casi perfecto estos datos; el modelo no aporta una mejora medible. En Ubicaciones casi todos los ejemplos son de una clase: con los datos sintéticos son duplicados (6 no duplicados entre 1 783 pares) y con las planillas reales la IA marcó todo como no duplicado (el generador y la IA etiquetan de forma opuesta el mismo patrón: misma descripción y mismo padre con otro código). **No es un modelo confiable.**
- **El modelo aprende la política de quien etiqueta.** En Clientes y proveedores la IA recibió el criterio «ID fiscal distinto ⇒ no son duplicados» y el generador marca como la misma empresa registros con ID distinto (variantes de captura); así, la IA coincide con la verdad del generador sólo en una parte de los grupos comparables:

| Corrida | Grupos comparables | Exactitud de la IA | Dijo «no duplicados» y lo eran | Dijo «duplicados» y no lo eran |
|---|---:|---:|---:|---:|
| bp_Estricta | 188 | 65 % | 66 | 0 |
| bp_Flexible | 97 | 73 % | 26 | 0 |
| bp_Normal | 108 | 69 % | 34 | 0 |

- **El sesgo de la IA importa más que el volumen:** más rondas y más registros no cambian lo que el modelo aprende de un criterio equivocado. Lo que lo mejora es decidir con personas del negocio.
- Se midió la concordancia entre los agentes de distintos niveles sobre los mismos pares (100 %), pero comparten el mismo criterio, así que no es una prueba de que sea correcto.
- En la fase 1 la salida de `aplicar` le mostraba a la IA las métricas agregadas de la auditoría (nunca puntajes por par); las IA reportaron no usarlas, y la salida ya no las muestra. Un par de errores de captura de una IA en un id de grupo quedaron sin registrar en una ronda (1 grupo) y se corrigieron.

## Qué falta

- Decisiones de personas del negocio en Resultados (origen `persona`) y reentrenar con ellas.
- Un criterio por escrito para los casos que la IA marcó como dudosos (ID fiscal distinto con nombre y dirección iguales; mismo TAG en otra ubicación; equipos idénticos sin serie).
- Si se integra el modelo a la pantalla: sólo como orden de revisión, con interruptor y sin tocar lo que las reglas dan por seguro.


## Tanda 2: alto índice de duplicados, ~55 000 registros por corrida

Datos sintéticos con 51 % a 65 % de los registros en un grupo de duplicados y con negativos difíciles, de unos 55 000 registros por corrida: Clientes (55 037), Proveedores (54 804), Materiales (55 275), Equipos (55 500) y Ubicaciones técnicas (56 115). Código de los generadores revisado por Claude Sonnet 5.5 (esfuerzo alto);
decisiones de Claude Haiku 5.5 (esfuerzo muy alto), 190 grupos por corrida en 4 rondas, sin ver puntajes. Reporte con las gráficas de líneas: [`reporte_antes_despues.html`](reporte_antes_despues.html). Los modelos nuevos están en [`modelos/tanda2/`](modelos/tanda2).

Orden de las reglas contra el del modelo nuevo, sobre pares de auditoría que ningún modelo vio:

| Conjunto | Pares de entrenamiento / auditoría | AP: reglas → modelo nuevo | AUC: reglas → modelo nuevo | Revisión al 90 %: reglas → modelo nuevo | Modelo anterior (planillas + Haiku), AP / AUC |
|---|---|---|---|---|---|
| Materiales | 666 / 186 | 1.000 → **1.000** | 1.000 → **1.000** | 83 % → **83 %** | 0.998 / 0.971 |
| Equipos | 408 / 119 | 1.000 → **1.000** | 1.000 → **1.000** | 79 % → **79 %** | 0.956 / 0.654 |
| Ubicaciones técnicas | 216 / 60 | — → **—** | — → **—** | — → **—** | — / — |
| Clientes | 431 / 187 | 0.994 → **1.000** | 0.993 → **1.000** | 50 % → **50 %** | 0.713 / 0.672 |
| Proveedores | 622 / 159 | 0.993 → **1.000** | 0.978 → **1.000** | 69 % → **68 %** | 0.941 / 0.816 |

- **Las reglas ya ordenan casi perfecto estos datos**: el modelo nuevo las iguala pero no las supera. Los modelos entrenados antes sí se degradan sobre estos pares (Equipos baja a un AUC de 0.65 y Clientes a un AP de 0.71).
- **Exactitud de Haiku contra la verdad del generador:** Clientes y Proveedores 99 % a 100 % (antes 65 % a 73 %, con un criterio que chocaba con el generador); Materiales 100 % (contando «ZZ» y «2Z» como el mismo sello, que en la vida real lo son); Ubicaciones 97 % a 99 %; Equipos 64 % en Normal y Flexible (100 % en Estricta): los 68 errores son un solo caso, «mismo TAG con otra descripción», que el generador cuenta como duplicado y el criterio dado exigía la misma descripción.
- **Ubicaciones técnicas** sigue sin ser entrenable con dos clases (0 y 1 no duplicados en Normal y Flexible); además la cola no muestra las unidades hijas de las áreas copiadas entre plantas.
- **Clientes y Proveedores:** entre 26 % y 53 % de los grupos quedaron dudosos porque mezclan un par duplicado con empresas de nombre parecido; decidir por par y no por grupo lo resolvería.
- Las etiquetas siguen siendo de una IA, no del negocio.
