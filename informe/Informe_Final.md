# Taller 3: Spark & Arquitectura Medallion

**Estudiantes:** Victor Hugo Díaz - Diego Alejandro Gomez

**Fecha:** 01-02-2026

**Repositorio:** (https://github.com/Victor-Diaz-Usta/Spark-Arquitectura-Medallion.git)

## Objetivo del taller

Desarrollar un flujo de datos tipo **Lakehouse** aplicando la **Arquitectura Medallion** (Bronce → Plata → Oro) sobre un clúster de **Spark** desplegado con **Docker**.  
La idea es construir un pipeline donde en **Bronce** se guarden los datos tal como llegan (en Delta Lake), en **Plata** queden ya limpios y validados, y en **Oro** se generen agregaciones útiles para análisis. Además, se implementa una **Puerta de Calidad (Quality Gate)** para separar los registros que cumplen las reglas y enviar los que no cumplen a una zona de **Cuarentena**, de forma que no se pierdan y puedan revisarse después.

## Entorno de trabajo

El taller se ejecutó de forma local usando un entorno con Spark y Docker. Para trabajar con los notebooks y revisar el estado del clúster se usaron dos accesos principales:

- **Jupyter Lab:** `http://localhost:8888`  
  (desde allí se ejecutaron los notebooks del proyecto)

- **Spark Master UI:** `http://localhost:8080`  
  (desde allí se verificó que el clúster estuviera activo y atendiendo los procesos)

Estos accesos permitieron correr el flujo completo y validar que las capas del Lakehouse se fueran generando correctamente.


## Capa Bronce (Bronze) — Ingesta

La capa **Bronce** representa el primer nivel del Lakehouse y está pensada para conservar los datos lo más cercanos posible a su origen. En este taller, la capa Bronce se construyó a partir de datos de **SECOP II** en formato **CSV**, con el objetivo de contar con una base “raw” y trazable que sirva como punto de partida para las siguientes capas.

Para esta práctica se trabajó con un volumen aproximado de **100.000 registros**, buscando tener una muestra amplia del comportamiento reciente de los contratos publicados. La ingesta se ejecutó desde el notebook de ingesta (`ingesta.ipynb`), levantando una sesión de **Spark** conectada al clúster definido en Docker y leyendo el archivo original **sin aplicar reglas de negocio** (es decir, sin filtrar ni “corregir” los registros).

En esta etapa, los ajustes fueron únicamente técnicos para garantizar compatibilidad con Delta Lake: se normalizaron los nombres de las columnas (por ejemplo, eliminando caracteres problemáticos) sin modificar el contenido de los datos. Además, se agregaron columnas de control como **`fecha_ingesta`** y **`archivo_origen`**, con el fin de mejorar la trazabilidad y facilitar auditorías o reprocesos posteriores.

Finalmente, los datos se almacenaron en formato **Delta Lake** en la ruta correspondiente a Bronce, y se validó la creación correcta de la tabla revisando la estructura generada (por ejemplo, presencia de `_delta_log`) y el historial de versiones.

**Ruta de salida (Delta):**
- `data/lakehouse/bronze/secop`

## Capa Plata (Silver) — Transformación y Quality Gate

En la capa **Plata** el objetivo fue dejar los datos de **Bronce** listos para usarlos: más ordenados, con formatos correctos (por ejemplo fechas como fechas) y, sobre todo, separando los registros que sí cumplen lo mínimo de los que vienen incompletos o inconsistentes.

### Qué se hizo en esta etapa

Se cargaron los datos de Bronce y se trabajó con las columnas necesarias para el ejercicio. Después se ajustaron algunos campos para que quedaran consistentes, principalmente en valores numéricos y fechas, para evitar problemas más adelante.

### Puerta de calidad (Quality Gate)

Luego se aplicó una revisión simple para decidir qué registros podían pasar a Plata. Las reglas fueron:

- El **Precio Base** debe ser **mayor a 0**  
- La **Fecha de Firma** **no puede estar vacía**

### Qué pasó con los registros

En vez de borrar los datos que no cumplen, se separaron en dos grupos:

- **Registros válidos:** se guardaron en `data/lakehouse/silver/secop`
- **Registros con problemas:** se enviaron a `data/lakehouse/quarantine/secop_errors`

A los registros que quedaron en cuarentena se les agregó la columna **`motivo_rechazo`**, para dejar claro por qué no pasaron la revisión.

### Resultado (conteos)

Para comprobar que la separación quedó bien, se contaron los registros en cada salida:

- Registros procesados: **100.000**
- Registros válidos en Plata: **7.942**
- Registros en Cuarentena: **92.058**

La suma coincide (**100.000 = 7.942 + 92.058**), así que el proceso separó todo sin dejar registros “perdidos”. En porcentaje, aproximadamente el **7,94%** pasó a Plata y el **92,06%** quedó en cuarentena.


## Capa Oro (Gold) — Analítica de negocio

En la capa **Oro** se buscó dejar un resultado final más fácil de interpretar, usando los datos que ya quedaron depurados en **Plata**. En esta etapa se generó un resumen por **departamento**, con el fin de identificar cuáles concentran el mayor valor de contratación en el conjunto analizado de SECOP II.

### Qué se hizo

Se cargaron los registros desde `data/lakehouse/silver/secop` y se preparó el campo `valor_del_contrato` para poder trabajarlo como número. Con esa base, se realizó la agregación por `departamento`, sumando el valor total y ordenando el resultado de mayor a menor para obtener un **Top 10**.

### Visualización

Para que el gráfico fuera más fácil de leer, se decidió no incluir los dos primeros departamentos del ranking, ya que sus valores son mucho más altos que el resto y terminaban “aplanando” las barras de los demás. 

Los departamentos excluidos en la visualización fueron:
- **Distrito Capital de Bogotá**
- **Caquetá**

El gráfico se construyó con los departamentos restantes para comparar mejor sus valores entre sí.

### Salida generada

El resultado completo del Top 10 (sin excluir departamentos) se guardó en la capa Oro en:

- `data/lakehouse/gold/top_deptos`

Esta salida corresponde al resultado final del flujo y queda lista para usarse en consultas, reportes o dashboards.

<img width="607" height="311" alt="image" src="https://github.com/user-attachments/assets/14f1c132-9c04-4af1-a689-bf1575963bc3" />

## Conclusión

Este trabajo consolidó un flujo de datos completo bajo un enfoque Lakehouse con arquitectura Medallion, llevando la información desde su estado crudo hasta una salida final lista para análisis. La separación por capas permitió mantener orden y control en cada etapa: conservar la fuente tal como llega (Bronce), estandarizar y validar lo esencial (Plata) y, en lugar de eliminar registros inconsistentes, aislarlos en una zona de cuarentena con trazabilidad del motivo de rechazo. Como resultado, se obtiene una vista agregada en la capa Oro que facilita la lectura del comportamiento de la inversión por departamento y deja el proceso preparado para ejecución repetible, auditoría y consumo en reportes o tableros. En conjunto, el pipeline no solo entrega resultados, sino que establece una base confiable para análisis posteriores y decisiones con respaldo en calidad de datos.

## Evidencias

Las capturas de soporte del proceso y de las salidas por capa se encuentran en la carpeta `imagenes_taller_3/`.


