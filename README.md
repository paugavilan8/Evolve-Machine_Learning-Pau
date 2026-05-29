# Modelo de probabilidad de recompra

Ejercicio práctico Machine Learning. Modelo supervisado que estima la
**probabilidad de que un cliente vuelva a comprar**, con el fin de dirigir las acciones
comerciales: **fidelización** para los clientes de alta probabilidad y **retención** para
los de baja.

## Objetivo de negocio

El dataset son transacciones, no clientes, y no existe una etiqueta de "recompra". La
construimos mediante un **corte temporal**: con las compras anteriores al corte calculamos
las variables de cada cliente (RFM), y con las posteriores observamos si volvió a comprar.

## Datos

- **Fuente:** Online Retail II (transacciones de un retailer online, 2009–2011).
- **Volumen:** 1.067.371 transacciones → **779.425** tras la limpieza.
- 5.878 clientes y 43 países (Reino Unido concentra ~92% del volumen).

### Exclusiones aplicadas (y por qué)

| Exclusión | Motivo |
|---|---|
| Filas sin `Customer ID` (22,8%) | El modelo es por cliente; sin ID no se pueden atribuir |
| Cancelaciones (facturas que empiezan por `C`, 1,83%) | No son compras |
| `Quantity` ≤ 0 / `Price` ≤ 0 | Devoluciones, regalos, errores de precio |
| Duplicados exactos | Ruido |

Se mantienen **todos los países** y se añade la variable `is_UK`, para no perder a los
mayoristas internacionales.

## Metodología

1. **Carga y EDA** — calidad de datos, estacionalidad, top de países, recurrencia de clientes.
2. **Limpieza y exclusiones.**
3. **Variable objetivo y features (RFM)** — corte temporal `2011-06-09`; ~6 meses de ventana
   para observar la recompra. Features por cliente: recencia, antigüedad, frecuencia, gasto,
   ticket medio, nº de productos, actividad reciente (90 días), país, etc.
4. **Modelos** — Regresión Logística (baseline), Random Forest y XGBoost, comparados por AUC
   con validación cruzada estratificada.
5. **Optimización** — `GridSearchCV` sobre XGBoost; curva ROC, matriz de confusión e
   importancia de variables.
6. **Aplicación de negocio** — segmentación de clientes por score (fidelización vs retención).

## Resultados

- Población de modelado: **4.966 clientes**, con un target **equilibrado (~52% recompra)**.
- **AUC ≈ 0,80** — capacidad de discriminación sólida entre quién recompra y quién no.
- Variables más relevantes: **actividad reciente (90 días)**, **recencia**, **frecuencia** y
  **gasto (monetary)**.

## Cómo ejecutarlo

```bash
python -m venv boost
# Activar el entorno:
#   Windows (PowerShell): .\boost\Scripts\Activate.ps1
#   Mac/Linux:            source boost/bin/activate
pip install -r requirements.txt
jupyter notebook
```

Coloca `online_retail_II.xlsx` en la misma carpeta que el notebook y ejecuta todas las
celdas en orden (Kernel → Restart & Run All). La primera carga del Excel tarda varios minutos.

## Contenido del repositorio

- `online_retail_II_modelo_recompra.ipynb` — notebook completo (EDA, limpieza, features,
  modelos, optimización y aplicación de negocio).
- `requirements.txt` — dependencias.

> El dataset (`online_retail_II.xlsx`) no se incluye en el repositorio por su tamaño.

## Próximos pasos

- Más variables: estacionalidad y categorías de producto.
- Enfoque de *time-to-next-purchase* (supervivencia) y cálculo de CLV.
- Validación con ventana de corte móvil y recalibración periódica.
- Puesta en producción: scoring mensual integrado con el CRM y medición del ROI.
