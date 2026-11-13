# Comparativo: reglas de `main`, ML de regresión y ML de boosting

Rama `dev-ml-boosting` (y la misma copia en `dev-ml-llm`). Responde a la tarea 2 de Notion («Incluir modelo ML pequeño para puntuar pares duplicados»), que menciona «regresión logística o gradient boosting».
**Solo cifras agregadas e importancias de señales:** ningún registro, descripción ni número de SAP.

## Lo primero: qué hace y qué NO hace cada uno hoy

| | Reglas de `main` | ML de regresión (`A.L.P.E.K. Dev Regresion`, rama `dev-ml-llm`) | ML de boosting (`A.L.P.E.K. Dev Boosting`, rama `dev-ml-boosting`) |
|---|---|---|---|
| **Detecta duplicados (puntaje, banda, vetos)** | **Sí, es lo único que lo hace** | No | No |
| **Ordena la cola de revisión** | Por categoría y puntaje | Sí: reordena dentro de cada nivel por probabilidad | Sí: igual, con otro modelo |
| **Dónde actúa** | Todas las herramientas | Materiales, Clientes y proveedores, Equipos (Ubicaciones técnicas no tiene modelo) | Igual |
| **Puede cambiar un resultado** | — | No: nunca toca puntaje, banda, vetos, memoria ni estatus | No: igual |

Esto es lo que ya está integrado en las dos copias. Lo que la tarea 2 imagina **y todavía no existe en ninguna**: que Estricto, Normal y Flexible sean umbrales de la probabilidad, que el modelo siga aprendiendo de lo que el usuario confirma o descarta dentro de la aplicación, y las funciones de explicar coincidencias y sugerir qué registro conservar.

## Cómo piensa cada uno

**Reglas de `main`.** Un puntaje armado a mano por herramienta, con vetos que valen cero y bandas fijas:

- *Materiales:* 0.20 de parecido de texto (rapidfuzz) + 0.30 TF-IDF + 0.50 significado (e5), multiplicado por factores; medidas, atributo o número de parte distintos son veto (puntaje 0); «seguro» ≥ 0.90, «por revisar» ≥ 0.75, y los niveles 90 / 75 / 60.
- *Clientes y proveedores:* una cascada ID fiscal → nombre y dirección → nombre y ciudad (×0.96) → nombre y país (×0.92, solo para revisar); ID distinto nunca es «seguro»; una sucursal nunca es «seguro».
- *Equipos:* reglas duras R1 (serie + fabricante + modelo) y R2 (TAG con contexto) → «seguro»; reglas suaves con pesos (E2 0.80, E5 0.70, E6 y E10 0.60…) y castigos (TAG distinto −0.90, series distintas −0.90, instalados a la vez −0.50, centros −0.30).
- *Ubicaciones técnicas:* R3 es la única «seguro»; S1, S2, S3 y T1 con pesos fijos.
- **Cada peso lo eligió una persona y no cambia solo.** Cada decisión se puede explicar con la señal que la disparó.

**Regresión logística.** Una suma ponderada de las señales que las reglas ya calculan (el puntaje, si es «seguro» y cada señal) pasada por una curva S: `probabilidad = sigmoide(sesgo + Σ peso·señal)`. Los pesos salen de los ejemplos de decisiones. Un JSON de ~1 KB (un número por señal). Cada señal empuja siempre en un solo sentido (el del signo de su peso): una señal de parecido con peso positivo nunca baja la probabilidad al subir.

**Gradient boosting.** Un conjunto de hasta 100 árboles de decisión pequeños (profundidad máxima 3), cada uno corrige lo que los anteriores dejaron mal: `probabilidad = sigmoide(sesgo + tasa·Σ valor de la hoja de cada árbol)`. Puede aprender **cortes** («si el parecido de calle pasa de 0.62…») e **interacciones** («solo cuenta la calle si el ID fiscal coincide»). JSON de 12 a 33 KB. En esta rama se entrena con scikit-learn y se puntúa solo con numpy.

## En qué se comparan

| Criterio | Reglas de `main` | Regresión | Boosting |
|---|---|---|---|
| Aprende de decisiones | No | Sí | Sí |
| Relaciones entre señales | Las que se escribieron (vetos, factores) | Solo suma de efectos; no ve interacciones | **Cortes e interacciones** sin escribirlas |
| Explicabilidad | **Máxima:** cada señal y su peso | **Alta:** un peso por señal | Media: importancia por señal, pero la probabilidad de un par depende de un camino por muchos árboles |
| Riesgo de memorizar | Ninguno | Bajo | **Mayor**, sobre todo con pocos ejemplos |
| Pocos ejemplos (cientos de pares) | Funciona sin ejemplos | Aguanta | **Frágil**: puede quedarse con una sola señal |
| Nuevos datos (otro cliente, otro formato) | Estable | Se degrada, pero con aviso suave | **Se degrada más y a veces de golpe** (ver abajo) |
| Peso del modelo | — | 0.5 a 5 KB | 12 a 33 KB |
| Puntuar 60 000 pares | — | 0.5 a 0.8 s | 0.7 a 0.9 s |
| Entrenar con todos los datos | — | segundos (no se midió aparte) | 3 a 14 s por conjunto (el sintético, con 181 000 pares de Materiales, 14 s) |
| Dependencias al usarlo | ninguna nueva | numpy | numpy (scikit-learn solo para entrenar) |
| Monotonía | Sí | Sí (más parecido, más probabilidad) | **No garantizada**: un corte puede invertir el orden en una zona |
| Cuándo gana en teoría | Cuando las reglas ya capturan todo | Datos pocos y relaciones simples | Muchos ejemplos de negocio con interacciones reales |

## Resultados medidos (mismos datos, misma partición, mismas métricas)

Se entrenaron los dos modelos con **exactamente los mismos pares** de entrenamiento y se midieron sobre los mismos pares de **auditoría que ningún modelo vio**. «Revisión al 90 %» es la fracción de los pares que hay que revisar de arriba abajo para encontrar el 90 % de los duplicados (menos es mejor). Las etiquetas son de una IA (Claude Haiku 5.5) o de un generador sintético, **no del negocio**.

### Con sus propios datos (cada modelo en el conjunto con el que se entrenó)

| Conjunto | Herramienta | Pares entren. / audit. | AP reglas · regresión · boosting | AUC reglas · regresión · boosting | Revisión al 90 %: reglas · regresión · boosting |
|---|---|---|---|---|---|
| Fase 1 (planillas reales + Haiku) | Materiales | 1 865 / 345 | 0.999 · 0.999 · **1.000** | 0.973 · 0.981 · **0.991** | 86 % · 86 % · **86 %** |
| | Clientes y proveedores | 231 / 83 | 0.518 · 1.000 · **1.000** | 0.364 · 1.000 · **1.000** | 92 % · 54 % · **54 %** |
| | Equipos | 650 / 275 | 0.939 · 0.994 · **0.979** | 0.960 · 0.996 · **0.983** | 34 % · 33 % · **33 %** |
| Fases 2 y 3 (sintético) | Materiales | 181 081 / 78 350 | 1.000 · 1.000 · **1.000** | 0.999 · 1.000 · **1.000** | 89 % · 89 % · **89 %** |
| | Clientes y proveedores | 7 626 / 2 743 | 0.961 · 1.000 · **1.000** | 0.913 · 1.000 · **1.000** | 74 % · 57 % · **57 %** |
| | Equipos | 10 680 / 3 170 | 0.981 · 1.000 · **1.000** | 0.894 · 1.000 · **1.000** | 92 % · 74 % · **74 %** |
| | Ubicaciones técnicas | 1 783 / 1 382 | 1.000 · 1.000 · **1.000** | 1.000 · 1.000 · **1.000** | 90 % · 90 % · **90 %** |
| Tanda 2 (~55 000 registros, alto índice de duplicados) | Materiales | 666 / 186 | 1.000 · 1.000 · **1.000** | 1.000 · 1.000 · **1.000** | 83 % · 83 % · **83 %** |
| | Clientes y proveedores | 1 053 / 346 | 0.994 · 1.000 · **1.000** | 0.988 · 1.000 · **1.000** | 59 % · 58 % · **58 %** |
| | Equipos | 408 / 119 | 1.000 · 1.000 · **1.000** | 1.000 · 1.000 · **1.000** | 79 % · 79 % · **79 %** |

Ubicaciones técnicas en la tanda 2 (1 no duplicado entre 216 pares) y en la fase 1 (0 duplicados) **no tiene modelo en ninguno de los dos**. El boosting además se niega a entrenar con menos de 5 pares de una clase.

**Lectura:** en datos como estos las reglas ya ordenan casi perfecto, así que los dos modelos las igualan o las mejoran un poco (Clientes y proveedores y Equipos en sintético), y **entre sí empatan**. Con esto no se puede decir que uno sea mejor.

### Con datos que no vieron (la prueba que sí los distingue)

Modelos entrenados con **otros** datos (planillas reales + Haiku, o sintético), medidos sobre la auditoría de la tanda 2. Es lo que pasa cuando el modelo llega a un cliente o a un formato nuevo.

| Auditoría (pares) | Entrenado con | AP regresión → boosting | AUC regresión → boosting | Revisión al 90 % regresión → boosting |
|---|---|---|---|---|
| Materiales (186) | planillas reales + Haiku | 0.998 → **0.996** | 0.971 → **0.960** | 83 % → **84 %** |
| Materiales (186) | sintético | 0.999 → **0.998** | 0.988 → **0.985** | 83 % → **83 %** |
| Equipos (119) | planillas reales + Haiku | 0.956 → **1.000** | 0.654 → **1.000** | 92 % → **79 %** |
| Equipos (119) | sintético | 1.000 → **0.905** | 1.000 → **0.474** | 79 % → **86 %** |
| Clientes (187) | planillas reales + Haiku | 0.713 → **0.538** | 0.672 → **0.473** | 88 % → **91 %** |
| Clientes (187) | sintético | 0.998 → **0.988** | 0.997 → **0.988** | 50 % → **50 %** |
| Proveedores (159) | planillas reales + Haiku | 0.941 → **0.791** | 0.816 → **0.593** | 85 % → **90 %** |
| Proveedores (159) | sintético | 1.000 → **1.000** | 1.000 → **1.000** | 68 % → **68 %** |

**Lectura:** el boosting **generaliza peor en 6 de 8 casos** (en 3 por unas milésimas o centésimas: Materiales dos veces y Clientes con sintético; en 3 por mucho: Equipos con sintético, Clientes y Proveedores con planillas reales + Haiku), mejor en 1 y empata en 1. En dos casos llega a un AUC menor que 0.5 (ordena peor que el azar): Equipos entrenado con sintético (0.474) y Clientes entrenado con planillas reales + Haiku (0.473). Gana claramente en un caso (Equipos entrenado con planillas reales: 1.000 contra 0.654). La regresión se degrada, pero con menos sobresaltos.

### Por qué el boosting se pasa de listo

La importancia que cada modelo da a las señales (modelos entregados, tanda 2):

| Herramienta | Regresión (mayores pesos) | Boosting (importancia, suma 1) |
|---|---|---|
| Materiales | texto 3.77, puntaje 2.75, factor 2.41 | **puntaje 1.00**, todo lo demás 0 |
| Clientes y proveedores | calle 3.30, puntaje 1.89, ID distinto −1.72 | **calle 0.987**, lo demás ≈ 0 |
| Equipos | puntaje 2.07, banda 1.62, «sin serie» −1.61 | **puntaje 1.00**, todo lo demás 0 |

Con ejemplos fáciles de separar el boosting encuentra **una sola señal que ya separa todo** y construye todo sobre ella: si esa señal cambia de comportamiento en otros datos, el modelo se cae. La regresión reparte el peso entre varias. Un ejemplo extremo: entrenado con las decisiones de Haiku en Clientes y proveedores (donde la IA aplicó «ID fiscal distinto ⇒ no son duplicados»), el boosting puso **toda** su importancia en `id_distinto` (1.00): aprendió la regla de la IA, no la señal del dato, y por eso saca AUC 0.47 en otro conjunto.

## Conclusiones (provisionales)

1. **Hoy ninguno de los dos mejora lo que hacen las reglas de `main` de forma que el negocio note:** con datos sintéticos fáciles las reglas ya ordenan casi perfecto; los modelos las igualan, y en Clientes y proveedores y Equipos bajan la revisión necesaria (del 74 % al 57 % y del 92 % al 74 % con los datos sintéticos de las fases 2 y 3).
2. **Entre los dos, la regresión es la opción más segura con pocos datos y con etiquetas de una IA:** más estable en datos nuevos, explicable con un peso por señal y de 1 KB. El boosting no demostró ventaja que compense su fragilidad.
3. **El boosting solo debería ganar con muchas decisiones de personas del negocio** (miles de pares, de origen `persona`) donde haya interacciones reales entre señales. Esa prueba **no existe todavía**: es lo que desempataría. Mientras tanto, **no hay base para pasar ninguno a `main`**.
4. **Cuidados si se decide quedarse con el boosting:** restringir la monotonía (que más parecido no pueda bajar la probabilidad), exigir más ejemplos por clase, medirlo siempre contra datos de otro origen y no solo contra su auditoría, y desconfiar de un modelo cuya importancia cae en una sola señal.
5. **Límites de esta comparación:** etiquetas de una IA o de un generador (no del negocio); auditorías de cientos de pares en la tanda 2 (las diferencias de unas décimas no son concluyentes); hiperparámetros del boosting fijos y no afinados (a propósito, para no usar la auditoría); Ubicaciones técnicas sin modelo en ambos; no se probó un instalador Dev real.

## Cómo se reprodujo

Desde `codigo/` con el entorno virtual del programa (los conjuntos de datos y sesiones viven en la PC del dueño del proyecto, fuera de los repos):

```powershell
python -m mldev.ml_verdad final <destino>\haiku <base-fase-1>
python -m mldev.ml_verdad final <destino>\sintetico <base-fase-2> <base-fase-3>
python -m mldev.ml_verdad final <destino>\tanda2 <base-s3> <base-s3-cli> <base-s3-prov>
python -m mldev.ml_verdad antes <base-s3> <destino>\tanda2\s3 haiku_v1=<destino>\haiku sintetico_v1=<destino>\sintetico
```

Los resultados del boosting (resúmenes y comparativos) están en [`modelos/boosting/`](modelos/boosting) de la rama `dev-ml-boosting`; los de la regresión, en `docs/ml-dev/modelos/` de la rama `dev-ml-llm`.


## Corrida comparativa ronda por ronda (2026-11-13)

Se repitió el protocolo de la tanda 2 con el modelo de boosting «alimentándose» de las decisiones de una IA: **los mismos escenarios** (datos sintéticos de unos 55 000 registros por corrida con alto índice de duplicados, generados con la misma semilla y los mismos generadores revisados por Claude Sonnet 5.5; coinciden los registros y los grupos de la tanda 2), **Claude Haiku 5.5 en esfuerzo muy alto decidiendo de nuevo** en Resultados (190 grupos por corrida en 4 rondas, sin ver puntajes), y el modelo reentrenándose en cada ronda. Después se repitieron **exactamente esas decisiones** con el modelo de regresión (`reaplicar` en la rama `dev-ml-llm`), de modo que reglas, regresión y boosting salen de las mismas etiquetas, de los mismos pares de entrenamiento y de la misma auditoría en cada ronda.

Reporte con las líneas de cada ronda: [`corrida-comparativa/reporte_algoritmo_regresion_boosting.html`](corrida-comparativa/reporte_algoritmo_regresion_boosting.html) (cifras en `corrida-comparativa/evolucion_*.json`). Se pudieron medir 9 de las 14 corridas; en las otras 5 casi todo lo que decidió la IA fue «duplicado» y la auditoría no trae las dos clases.

| Corrida | AP reglas · regresión · boosting | Revisión al 90 % reglas · regresión · boosting |
|---|---|---|
| Clientes Estricta | 0.990 · 1.000 · sin modelo | 92 % · 85 % · — |
| Clientes Normal | 1.000 · 1.000 · 1.000 | 73 % · 73 % · 73 % |
| Clientes Flexible | 0.999 · 0.994 · 0.922 | 27 % · 28 % · 28 % |
| Proveedores Normal | 1.000 · 1.000 · 1.000 | 56 % · 56 % · 56 % |
| Proveedores Flexible | 0.974 · 0.998 · 1.000 | 68 % · 57 % · 57 % |
| Materiales Flexible | 1.000 · 1.000 · 1.000 | 59 % · 59 % · 59 % |
| Equipos Estricta, Normal y Flexible | 1.000 · 1.000 · 1.000 | 88 %, 75 % y 78 % (los tres iguales) |

- **Coinciden en 6 de las 9 corridas medibles**: las reglas ya ordenan casi perfecto estos datos.
- **Donde difieren, la regresión gana 2 de 3 veces** (Clientes Estricta, donde el boosting ni entrena por tener solo 2 no duplicados, y Clientes Flexible, donde el boosting cae a AP 0.922); el boosting gana una (Proveedores Flexible, por 0.002 de AP).
- **Modelo con todo lo aprendido de cada herramienta:** Clientes y Proveedores, los dos modelos con AP 1.000 contra 0.994 y 0.995 de las reglas; Materiales y Equipos, los tres iguales; Ubicaciones técnicas, la regresión entrena con 1 solo no duplicado entre 216 pares y el boosting no entrena.
- **La réplica es fiel:** los pares de entrenamiento y auditoría del modelo conjunto coinciden con los de la tanda 2 en Clientes (431 / 187) y Equipos (408 / 119) y casi en Materiales (675 contra 666) y Proveedores (610 contra 622): Haiku decidió de nuevo y sus decisiones no son idénticas.
- **Conclusión:** con este volumen y estos escenarios no hay evidencia de que el boosting supere a la regresión ni a las reglas. Las etiquetas siguen siendo de una IA sobre datos sintéticos.
