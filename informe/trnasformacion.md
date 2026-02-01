## Capa Plata (Silver) – Transformación y Quality Gate

La capa Plata tiene como objetivo transformar los datos crudos provenientes de la capa Bronze en un conjunto de información estructurada, tipada y validada, lista para análisis. En esta etapa se prioriza la calidad de los datos, la reducción de ruido y la preparación semántica de las variables clave del dominio contractual del SECOP II.

Como primer paso, se leyeron los datos almacenados en formato **Delta Lake** desde la capa Bronze y se realizó una **selección de columnas relevantes**, enfocándose únicamente en variables necesarias para el análisis contractual, financiero y administrativo. Esta reducción controlada del esquema permite mejorar el rendimiento, la claridad analítica y la mantenibilidad del modelo de datos.

Posteriormente, se aplicaron **conversiones de tipo (casting)** sobre campos críticos como fechas y valores monetarios, garantizando coherencia técnica y evitando errores en procesos posteriores. Sobre estos datos ya tipados, se implementó un **Quality Gate**, definiendo reglas de validación tales como: valor del contrato mayor a cero y fechas de inicio no nulas. Esta lógica permite evaluar la confiabilidad de cada registro de forma explícita.

Finalmente, los datos fueron **bifurcados en dos flujos**: los registros que cumplen todas las reglas de calidad se almacenaron en la ruta `/app/data/lakehouse/silver/secop`, mientras que los registros inválidos fueron enviados a `/app/data/lakehouse/quarantine/secop_errors`, junto con el motivo de rechazo. Este enfoque garantiza trazabilidad, auditoría y buenas prácticas de ingeniería de datos, evitando la pérdida de información y permitiendo análisis de calidad posteriores.
