# Capa Oro – Analítica de Negocio

La capa Oro tiene como propósito generar **valor analítico** a partir de los datos ya saneados en la capa Plata. En esta etapa se construyen métricas agregadas que permiten responder preguntas de negocio, en este caso identificar los **departamentos con mayor inversión contractual** a partir de los datos de SECOP II.

Se inició activando una nueva SparkSession y leyendo la información desde la ruta `/app/data/lakehouse/silver/secop`. A partir de estos datos, se realizó un proceso de agregación donde se convirtió el campo valor_del_contrato a tipo numérico y se calculó la suma total de inversión por departamento, ordenando los resultados de mayor a menor y seleccionando el Top 10.

## Visualización
Para la visualización mediante diagrama de barras, se decidió **excluir los dos primeros departamentos** del ranking, ya que generan una alta asimetria que distorsiona la lectura del gráfico. 

Los departamentos excluidos fueron:
- **Distrito Capital de Bogotá**: presenta la mayor concentración de inversión del país, asociada a entidades nacionales, ministerios y grandes contratos centralizados.
- **Caquetá**: registra un volumen de inversión atípicamente alto en el periodo analizado, lo que provoca un efecto de escala que oculta el comportamiento del resto de departamentos.

El gráfico se construyó únicamente con los departamentos restantes para facilitar la comparación visual entre ellos.

## Conclusion
Finalmente, el resultado completo de la agregación (Top 10 real, sin exclusiones) se almacenó en formato **Delta Lake** en la ruta `/app/data/lakehouse/gold/top_deptos`, utilizando modo `overwrite`, ya que la capa Oro es **recalculable** y no requiere historial transaccional. Esta capa representa el nivel final del Lakehouse y está diseñada para ser consumida directamente por análisis, reportes o dashboards.

