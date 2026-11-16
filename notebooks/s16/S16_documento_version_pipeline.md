# Documento de versión — Pipeline NovaMarket v1

**Archivo:** `novamarket_pipeline_v1.joblib`
**Fecha de esta versión:** 2026-11-16
**Notebook de origen:** `S16_pipeline_reproducible.ipynb`
**Datos de entrenamiento:** `novamarket_features_seleccionadas.csv` (semana 14), 570.976 filas
**Versiones de librerías:** scikit-learn `[COMPLETAR con sklearn.__version__]`, pandas `[COMPLETAR con pd.__version__]`

---

## 1. Qué es esta versión, en palabras simples
Es un único objeto `Pipeline` de scikit-learn que reúne las decisiones de preprocesamiento de las semanas anteriores. Recibe un lote de clientes ya codificado (con las mismas columnas que el dataset de la semana 14) y aplica, siempre en el mismo orden, la imputación de nulos y el escalado. Así, procesar un lote nuevo (por ejemplo, los 600 clientes de octubre) no depende de recordar el orden de los doce notebooks.

## 2. Qué transformaciones incluye
El pipeline tiene dos pasos: un `ColumnTransformer` (`preprocesador`) y una regresión logística (`modelo_validacion`) que solo se usa para medir la calidad del preprocesamiento.

### Columnas continuas (6): imputar con mediana y luego escalar con `StandardScaler`
`nivel_lealtad_cod` (ordinal de 0 a 3, semana 9), `precio_unitario`, `edad_cliente`, `monto_compra_log`, `dias_desde_ultima_compra`, `total_gastado`.

### Columnas binarias (1): pasan sin cambios (`passthrough`)
`canal_App` (one-hot de la semana 9). No se escala porque solo vale 0 o 1.

### Columnas que el pipeline descarta
`id_cliente` (identificador) y `fuga_cliente` (variable objetivo).

### Parámetros aprendidos con el conjunto de entrenamiento
| Columna | Mediana (imputación) | Media (escalador) | Desviación (escalador) |
|---|---|---|---|
| `nivel_lealtad_cod` | 1,0 | 1,4983 | 1,1179 |
| `precio_unitario` | 312.500 | 344.563,6 | 196.482,7 |
| `edad_cliente` | 47 | 46,5197 | 16,2429 |
| `monto_compra_log` | 13,0322 | 12,1511 | 3,5068 |
| `dias_desde_ultima_compra` | 234 | 262,4409 | 95,5169 |
| `total_gastado` | 2.776.800 | 3.075.738 | 1.860.830 |

Solo `dias_desde_ultima_compra` tiene nulos hoy (1.098 en el dataset completo, 0,19 %); en las demás columnas el imputador no modifica nada, pero se mantiene por si un lote nuevo trae nulos.

## 3. Qué tan bien funciona
Se midió el `recall` (qué fracción de los clientes que se fugan detecta el modelo) con validación cruzada de 5 particiones sobre el conjunto de entrenamiento, usando una regresión logística con `class_weight="balanced"`.

- **Recall promedio:** 0,648
- **Desviación estándar:** 0,004
- **Valores por partición:** 0,651; 0,650; 0,642; 0,652; 0,644
- **Recall en el conjunto de prueba:** `[COMPLETAR con recall_prueba]`
- **Recall con el pipeline cargado desde el archivo:** `[COMPLETAR]` (idéntico al anterior)

**Interpretación:** el modelo de validación detecta cerca de dos de cada tres clientes que se fugan, y el resultado es muy estable entre particiones. La regresión logística no es el modelo final del equipo de Machine Learning: es un termómetro para comprobar que el preprocesamiento funciona sin fuga.

## 4. Control de fuga de datos
- El preprocesamiento (imputación y escalado) está dentro del `Pipeline`, así que en cada partición de la validación cruzada las medianas, medias y desviaciones se recalculan solo con el entrenamiento de esa partición.
- El pipeline se ajustó (`fit`) **una sola vez**, únicamente con `X_entrenamiento` y `y_entrenamiento` (456.780 filas). **El conjunto de prueba (114.196 filas) nunca participó en el ajuste**; solo se usó con `.predict()` para evaluar.

## 5. Limitaciones conocidas
1. **Clientes repetidos entre entrenamiento y prueba.** El dataset tiene una fila por pedido (570.976 filas) y 151.117 clientes únicos. La división se hizo por filas: 76.519 de los 80.747 clientes del conjunto de prueba (94,8 %) también aparecen en entrenamiento. Por eso los `recall` anteriores pueden ser optimistas respecto al desempeño con clientes completamente nuevos. Pendiente: repetir la validación agrupando por `id_cliente` (`StratifiedGroupKFold`).
2. **Features calculadas fuera del pipeline.** `total_gastado` y `dias_desde_ultima_compra` se calcularon en la semana 13 sobre todo el dataset, antes de la división. Un lote nuevo debe calcularlas con la misma regla y el mismo corte de referencia (2026-10-19).
3. **`monto_compra_log` tiene una desviación amplia** (3,51) y media menor que la mediana, lo que sugiere una cola de valores bajos. Pendiente de revisar su construcción.
4. **El lote debe llegar ya codificado.** El pipeline no convierte `canal_compra` en texto; espera las 7 columnas con los mismos nombres y tipos de dato.

## 6. Cómo usarlo con un lote nuevo
```python
import joblib
pipeline = joblib.load("novamarket_pipeline_v1.joblib")
X_nuevo_listo = pipeline[:-1].transform(X_nuevo)   # solo preprocesamiento
```
`X_nuevo` debe tener las 7 columnas: `nivel_lealtad_cod`, `precio_unitario`, `edad_cliente`, `monto_compra_log`, `canal_App`, `dias_desde_ultima_compra`, `total_gastado`.

## 7. Historial de versiones
| Versión | Fecha | Cambios |
|---|---|---|
| v1 | 2026-11-16 | Primera versión: imputación con mediana y escalado en las 6 continuas; `canal_App` sin escalar. |

Si algo cambia (columnas, estrategia de imputación, nuevo ajuste), se guarda como `novamarket_pipeline_v2.joblib` y se agrega una fila a esta tabla, sin sobrescribir la v1.
