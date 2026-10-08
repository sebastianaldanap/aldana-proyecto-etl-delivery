# 🍔 Pipeline ETL e Inteligencia de Mercado: Food Delivery

Proyecto final del curso **Python for Data Analyst** de la especialización en Data Analyst. Construí un pipeline ETL completo con Pandas y SQLite, y respondí cuatro preguntas de negocio con SQL y gráficos.

## 🎯 Contexto de negocio

Una startup de food delivery quiere expandirse en Estados Unidos. Antes de lanzar la app, el directorio necesita saber cómo operan los competidores: en qué ciudades hay más restaurantes, cómo se distribuyen los precios y si un precio más alto se traduce en mejor calidad.

## 🧱 Arquitectura del pipeline

| Fase | Qué hice |
|---|---|
| **Extract** | Lectura de `restaurants.csv` y de los 10 archivos de menús, y data profiling (dimensiones, tipos y nulos) |
| **Transform** | Mapeo de precios, separación de la dirección en calle, ciudad y estado, filtros de calidad, conversión del precio a número y cruce de tablas |
| **Load** | Base SQLite `delivery_insights.db` con la tabla `master_food_data`, creada con SQLAlchemy |
| **BI** | Cuatro consultas SQL leídas con `pd.read_sql` y un gráfico por pregunta, cada uno con su conclusión |

## 📦 Datos

- `restaurants.csv`: 63,469 restaurantes (puntaje, calificaciones, categoría, rango de precios y dirección).
- `restaurant-menus`: 836,350 platos, entregados en 10 archivos CSV.

Los archivos son los entregados por el curso y no están incluidos en este repositorio.

## 🔧 Decisiones técnicas

- **Dirección:** el 3 % de las direcciones tiene comas de más (por ejemplo, `Suite 4`), así que un `split` normal desordenaba la ciudad. Corté desde la derecha con `rsplit`: las últimas tres partes son ciudad, estado y código postal.
- **Precios en cero y negativos:** filtré con `price > 0`, que quita ceros, nulos y también 2 precios negativos que no tienen sentido.
- **Orden de los filtros:** primero filtré los restaurantes con más de 100 calificaciones y después hice el cruce, para que el merge use menos memoria.
- **Merge validado:** usé `validate="one_to_many"` para evitar duplicar platos por ids repetidos.
- **Conteo de restaurantes:** cada fila de la tabla final es un plato, así que conté con `COUNT(DISTINCT id)` y no con `COUNT(*)`.
- **Puntaje por restaurante:** el `score` pertenece al restaurante y se repite en cada plato, así que en la pregunta 4 me quedé primero con un registro por restaurante y después promedié.
- **SQLite en Databricks:** la base se crea en el disco local (`/tmp`) y se copia al Volume al final, porque SQLite necesita escribir y bloquear el archivo.

## 🔻 Cómo se redujeron los datos

| Paso | Restaurantes | Platos |
|---|---|---|
| Datos originales | 63,469 | 836,350 |
| Con puntaje válido | 35,302 | n/a |
| Precio mayor a 0 | n/a | 798,797 |
| Más de 100 calificaciones | 8,702 | n/a |
| Tabla final después del cruce | 671 | 63,221 |

## 📊 Resultados

1. **Geografía:** Milwaukee (123 restaurantes) y Seattle (119) concentran la oferta. Lynnwood, Everett y Bellevue, que completan el top 5, también están en Washington.
2. **Precios:** el mercado no es caro. El 67 % de los restaurantes es Económico y el 23 % Moderadamente caro; solo 4 son Caros y ninguno es Muy caro. 60 restaurantes no tienen rango de precio.
3. **Menú por ciudad:** Seattle es la más cara para comer ($11.98 por plato en promedio), seguida de Lynnwood ($10.76) y Milwaukee ($9.52). La mediana confirma el mismo orden.
4. **Precio y calidad:** un precio más alto no garantiza mejor experiencia. Económico (4.62) y Moderadamente caro (4.64) tienen casi el mismo puntaje, y Caro (4.45) se basa en solo 4 restaurantes.

## ⚠️ Limitaciones

Los resultados describen una muestra de **671 restaurantes**, no todo el mercado de Estados Unidos: los menús entregados cubren solo los `restaurant_id` del 1 al 10,082. Los hallazgos son una primera señal para confirmar con datos más completos.

## 🛠️ Tecnologías

Python, Pandas, SQLAlchemy, SQLite, SQL, Matplotlib, Seaborn y Databricks (Free Edition).

## 📁 Contenido del repositorio

- `proyecto_etl_delivery.ipynb`: cuaderno con todo el desarrollo.
- `README.md`: este documento.

## 👤 Autor

Sebastian Aldana Pachas