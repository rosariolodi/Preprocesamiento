## Resumen general de features

Se construyeron **9 features** (más sus indicadores auxiliares) sobre el dataset integrado de ventas, bodega y CRM (570.976 pedidos, 151.117 clientes). Ninguna de las consignas se pudo ejecutar tal cual, porque usaban columnas que no existen en este dataset (`fecha_ultima_compra`, `numero_compras`, `monto`, `id_compra`, `fecha`, `canal_compra`, `df_transacciones`). En cada caso se derivó la variable desde columnas reales o se mapeó al nombre correcto.

### Features creadas

| Feature | Nivel | Dónde vive | Cómo se construyó |
|---|---|---|---|
| `dias_desde_ultima_compra` (+ `fecha_ultima_compra`, `dias_desde_ultima_compra_era_nulo`) | cliente | `df` y `resumen_cliente` | Máximo de `fecha_pedido` válida por `id_cliente`; días hasta el corte 2026-10-19. Los clientes sin fecha válida quedan en `NaN` con indicador. |
| `numero_compras` | cliente | `df` y `resumen_cliente` | Conteo de `id_pedido` distintos por `id_cliente` (1 a 14). |
| `es_cliente_recurrente` | cliente | `df` | `numero_compras > 1`. |
| `total_gastado`, `gasto_promedio` | cliente | `resumen_cliente` | Suma y promedio de `monto_compra` por `id_cliente`. |
| `score_R`, `score_F`, `score_M`, `rfm_score` | cliente | `resumen_cliente` | Cuartiles (`qcut` con `rank(method="first")`) de recencia (escala invertida), frecuencia y gasto; suma de 3 a 12. |
| `es_temporada_alta` | pedido | `df` | 1 si el mes de `fecha_pedido` es noviembre o diciembre; `NA` si la fecha es nula o futura. |
| `categoria_favorita` (+ `categoria_favorita_empate`) | cliente | `resumen_cliente` | Categoría con más pedidos por cliente, reconstruida antes desde las dummies `cat_*`. |
| `canal_principal` (+ `canal_principal_empate`) | cliente | `resumen_cliente` | Canal con más pedidos por cliente, reconstruido antes desde las dummies `canal_*`. |

**Feature descartada:** `compras_ultimos_30_dias`. La ventana de 30 días previa al corte (2026-09-19 a 2026-10-19) no contiene ningún pedido válido, porque los datos terminan el 2026-05-15. La feature sería constante en 0, o contaría solo fechas de 2027 (imposibles), por lo que no aporta información.

### Qué se usó para construirlas

- **Llave de agregación:** `id_cliente`, que no tiene nulos. No se usó `correo_cliente`, porque tiene faltantes (MNAR).
- **Mapeo de nombres:** `monto` → `monto_compra` (valor original, no las versiones escaladas, logarítmica o Box-Cox, que no tienen sentido al sumar); `id_compra` → `id_pedido`, que es único por fila (570.976 valores, sin nulos ni valores concatenados); `fecha` → `fecha_pedido`.
- **Reconstrucción desde dummies:** `categoria` y `canal_compra` se recuperaron con `idxmax` sobre las columnas `cat_*` y `canal_*`, verificando que cada fila tuviera exactamente una dummy activa.
- **Joins:** se hicieron con `how="left"` y `validate`, comprobando que el número de filas no cambiara.
- **Indicadores `*_era_nulo` y `*_empate`:** se conservó la trazabilidad en lugar de imputar valores inventados.

### Decisiones sobre la calidad de los datos

- **Fechas imposibles:** 14.694 pedidos con fecha en 2027 se trataron como inválidos (columna `fecha_pedido_futura`) y se excluyeron de la recencia y de `es_temporada_alta`. Además hay 20.288 fechas nulas.
- **Clientes sin recencia:** 993 clientes (1.098 filas) no tienen ninguna fecha válida; quedan sin `dias_desde_ultima_compra` ni `rfm_score`, sin imputar.
- **Empates:** el 45,1 % de los clientes tiene empate en la categoría favorita y el 36,6 % en el canal principal. El desempate es alfabético (para que sea reproducible) y se marca con indicadores.

### Limitaciones a tener en cuenta

1. **Desfase temporal:** las fechas válidas terminan el 2026-05-15 y el corte es el 2026-10-19, por lo que la recencia mínima es de 157 días. R conserva el orden relativo entre clientes, pero no indica abandono real. Del mismo modo, `es_temporada_alta` solo cubre noviembre y diciembre de 2025.
2. **Porcentajes por fila:** el 97,5 % de las filas pertenece a clientes recurrentes, pero ese número está inflado, porque los clientes recurrentes aportan más filas. La cifra por cliente es menor y es la que debe reportarse.
3. **Fuga de información:** las features por cliente (recencia, compras, gasto, RFM, categoría y canal favoritos) usan todo el historial, incluidos pedidos posteriores a cada fila. Si se usan para predecir `fuga_cliente` con la partición `conjunto`, deben recalcularse solo con datos de entrenamiento o con una ventana previa a cada pedido.
4. **Columnas contaminadas por fechas imposibles:** `mes_pedido`, `trimestre_pedido`, `dia_semana_pedido` y `es_fin_semana_pedido` se calcularon con las fechas de 2027 en esas 14.694 filas y no se corrigieron en este notebook.
5. **Desempate arbitrario:** `rank(method="first")` en RFM desempata por orden de aparición, lo que afecta sobre todo a `numero_compras`, que tiene pocos valores distintos.