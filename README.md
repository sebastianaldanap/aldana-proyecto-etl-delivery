# 🍔 Pipeline ETL e Inteligencia de Mercado: Food Delivery

Proyecto final del curso **Python for Data Analyst** de la especialización en Data Analyst. Construí un pipeline ETL completo con Pandas y SQLite, y respondí cuatro preguntas de negocio con SQL y gráficos.

## 📑 Índice

- [Contexto de negocio](#-contexto-de-negocio)
- [Hallazgos clave](#-hallazgos-clave)
- [Recomendaciones para el CEO](#-recomendaciones-para-el-ceo)
- [Problemas encontrados en los datos y cómo los resolví](#-problemas-encontrados-en-los-datos-y-cómo-los-resolví)
- [Arquitectura del pipeline](#-arquitectura-del-pipeline)
- [Limitaciones](#-limitaciones)
- [Tecnologías](#-tecnologías)
- [Contenido del repositorio](#-contenido-del-repositorio)
- [Autor](#-autor)

## 🎯 Contexto de negocio

Una startup de food delivery quiere expandirse en Estados Unidos. Antes de lanzar la app, el directorio necesita saber cómo operan los competidores: en qué ciudades hay más restaurantes, cómo se distribuyen los precios y si un precio más alto se traduce en mejor calidad.

## 📊 Hallazgos clave

### 1. Geografía: Milwaukee y Seattle concentran la oferta

Milwaukee (123 restaurantes) y Seattle (119) lideran con amplia ventaja y reúnen el 36 % de los 669 restaurantes con ciudad. Lynnwood, Everett y Bellevue, que completan el top 5, también están en Washington.

![Top 5 ciudades con más restaurantes](screenshots/01_pregunta_1_top_ciudades.png)

### 2. Precios: el mercado no es caro

El 67 % de los restaurantes es Económico y el 23 % Moderadamente caro. Solo 4 son Caros y ninguno es Muy caro. 60 restaurantes (9 %) no tienen rango de precio.

![Distribución de restaurantes por rango de precios](screenshots/02_pregunta_2_rangos_precio.png)

### 3. Menú por ciudad: Seattle es la más cara para comer

El plato cuesta en promedio $11.98 en Seattle, $10.76 en Lynnwood y $9.52 en Milwaukee. La mediana confirma el mismo orden.

![Precio por plato en las 3 ciudades con más restaurantes](screenshots/03_pregunta_3_precio_por_ciudad.png)

### 4. Precio y calidad: más caro no significa mejor

Económico (4.62) y Moderadamente caro (4.64) tienen casi el mismo puntaje. Caro (4.45) se basa en solo 4 restaurantes, muy pocos para sacar una tendencia.

![Puntaje promedio por rango de precios](screenshots/04_pregunta_4_puntaje_por_rango.png)

## 💡 Recomendaciones para el CEO

- **Empezar por Washington y Milwaukee.** Las cuatro ciudades de Washington del top 5 suman 223 restaurantes, y Milwaukee tiene 123 más.
- **Posicionarse en precios Económico y Moderado.** El 90 % de los restaurantes está en esos dos rangos, así que ahí está la competencia.
- **Ajustar el precio por ciudad.** El plato en Seattle cuesta en promedio $2.46 más que en Milwaukee ($11.98 frente a $9.52).
- **Competir por calidad, no por precio.** Un precio más alto no se asocia a mejor puntaje, así que no hace falta subir precios para ofrecer una buena experiencia.
- **Confirmar antes de invertir.** Estos resultados salen de una muestra de 671 restaurantes (ver Limitaciones).

## 🔍 Problemas encontrados en los datos y cómo los resolví

| Problema | Cómo lo detecté | Cómo lo resolví |
|---|---|---|
| 28,167 restaurantes (44 %) sin puntaje | Data profiling (nulos por columna) | Los eliminé con el filtro de calidad |
| Un `price_range` con 17 símbolos `$` | `value_counts` | No lo mapeé: quedó como nulo |
| 1,861 direcciones (3 %) con comas de más, por ejemplo `Suite 4` | Conteo de comas por dirección | Corté desde la derecha con `rsplit`: las últimas tres partes son ciudad, estado y código postal |
| El menú viene en 10 archivos CSV | Exploración de la carpeta | Los leí con `glob`, verifiqué que tuvieran las mismas columnas y los uní con `concat` |
| `price` es texto (`15.99 USD`) | `.dtypes` | Quité " USD" y lo convertí a `float` |
| 37,551 platos con precio 0 y 2 con precio negativo | `describe()` del precio | Filtré con `price > 0` |
| `category` y `name` existen en ambas tablas | Preparación del cruce | Usé `suffixes` en el merge |
| Cada fila es un plato, no un restaurante | Diseño de `df_master` | Conté con `COUNT(DISTINCT id)`; para el puntaje me quedé primero con un registro por restaurante |
| Los menús solo cubren los `restaurant_id` del 1 al 10,082 | El cruce dejó 671 de 8,702 restaurantes | Lo documenté como limitación y trabajé con esa muestra |

## 🧱 Arquitectura del pipeline

| Fase | Qué hice |
|---|---|
| **Extract** | Lectura de `restaurants.csv` y de los 10 archivos de menús, y data profiling (dimensiones, tipos y nulos) |
| **Transform** | Mapeo de precios, separación de la dirección en calle, ciudad y estado, filtros de calidad, conversión del precio a número y cruce de tablas |
| **Load** | Base SQLite `delivery_insights.db` con la tabla `master_food_data`, creada con SQLAlchemy |
| **BI** | Cuatro consultas SQL leídas con `pd.read_sql` y un gráfico por pregunta, cada uno con su conclusión |

**Datos de partida:** `restaurants.csv` (63,469 restaurantes) y `restaurant-menus` (836,350 platos en 10 archivos CSV), entregados por el curso y no incluidos en este repositorio.

**Cómo se redujeron los datos:** primero filtré los restaurantes con más de 100 calificaciones y después hice el cruce, con `validate="one_to_many"`, para que el merge use menos memoria. La base se crea en el disco local de Databricks (`/tmp`) y se copia al Volume al final, porque SQLite necesita escribir y bloquear el archivo.

| Paso | Restaurantes | Platos |
|---|---|---|
| Datos originales | 63,469 | 836,350 |
| Con puntaje válido | 35,302 | n/a |
| Precio mayor a 0 | n/a | 798,797 |
| Más de 100 calificaciones | 8,702 | n/a |
| Tabla final después del cruce | 671 | 63,221 |

## ⚠️ Limitaciones

Los resultados describen una muestra de **671 restaurantes**, no todo el mercado de Estados Unidos: los menús entregados cubren solo los `restaurant_id` del 1 al 10,082. Los hallazgos son una primera señal para confirmar con datos más completos.

## 🛠️ Tecnologías

Python, Pandas, SQLAlchemy, SQLite, SQL, Matplotlib, Seaborn y Databricks (Free Edition).

## 📁 Contenido del repositorio

- `proyecto_etl_delivery.ipynb`: cuaderno con todo el desarrollo.
- `screenshots/`: capturas de los 4 gráficos del análisis.
- `README.md`: este documento.

## 👤 Autor

Sebastian Aldana Pachas