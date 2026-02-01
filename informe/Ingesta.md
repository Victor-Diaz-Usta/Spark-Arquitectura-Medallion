# Capa Bronze

Lo primero fue extraer los datos de SECOP para asi mismo crear la ingesta con una data eal y en linea, para este ejercicio se extrajeron los ultimos 100.000 datos para asi mirar como a estado actualmente moviendose el SECOP.

La capa Bronze corresponde al primer nivel del Lakehouse y contiene los datos en su estado más cercano al origen. En este proyecto, la capa Bronze se construyó a partir de un archivo CSV de contratos del SECOP II, el cual fue transformado únicamente a nivel técnico para poder ser almacenado en formato Delta Lake.

La ingesta se realizó mediante el archivo `ingesta.ipynb`, donde se creó una sesión de Spark conectada al clúster definido en Docker y se leyó el archivo CSV original sin aplicar filtros ni reglas de negocio. Para cumplir con las restricciones del formato Delta, se normalizaron los nombres de las columnas, eliminando caracteres no permitidos, sin modificar el contenido de los datos.

Durante el proceso se agregaron columnas de control (`fecha_ingesta` y `archivo_origen`) con el fin de garantizar trazabilidad y permitir auditoría y reprocesos posteriores. Los datos resultantes fueron almacenados en la ruta correspondiente a la capa Bronze del Lakehouse. Finalmente, se validó la creación de la tabla Delta mediante la consulta del historial de versiones, confirmando que la capa Bronze quedó correctamente construida y lista para ser utilizada como fuente de entrada para la capa Silver.

