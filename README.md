Resumen General del Proyecto
Este proyecto utiliza dbt para transformar datos almacenados en una base de datos Snowflake dentro del esquema tpch_sf1. Los datos provienen de un conjunto de tablas que representan órdenes (orders) y sus ítems asociados (lineitem). El proyecto implementa una arquitectura moderna de análisis de datos, siguiendo principios de ETL (Extract, Transform, Load), donde se prioriza:
•	Limpieza y transformación de datos para facilitar el análisis.
•	Enriquecimiento y agregación de métricas clave.
•	Validación de la calidad de datos mediante pruebas integradas de dbt.
El propósito principal es generar un modelo de datos organizado que permita el análisis de ventas, descuentos y métricas relacionadas a órdenes e ítems de manera eficiente.
 
Componentes Principales del Proyecto
1. Definición de las Fuentes de Datos
El archivo tpch_sources.yml define dos tablas base del esquema tpch_sf1:
•	orders: Contiene información sobre órdenes realizadas (como claves de cliente, precios totales, y fechas de orden).
•	lineitem: Contiene detalles de los ítems relacionados a cada orden (como clave del producto, cantidad, precio por unidad, descuentos y tasas).
Pruebas de Integridad (Tests)
Se aplican validaciones a las fuentes para garantizar la calidad de los datos:
•	unique: Verifica que no existan valores duplicados en columnas clave, como o_orderkey.
•	not_null: Garantiza que campos esenciales no contengan valores nulos.
•	relationships: Comprueba la consistencia referencial entre orders y lineitem, asegurando que toda clave en lineitem (como l_orderkey) exista en orders.
 
2. Modelos de Staging
Los modelos de staging preparan los datos brutos para análisis al realizar transformaciones iniciales. Estos modelos se encuentran en la carpeta models/staging.
2.1. Modelo stg_tpch_orders
Este modelo transforma la tabla orders al:
•	Renombrar columnas para estandarizar nombres (o_orderkey → order_key, o_custkey → customer_key).
•	Seleccionar únicamente las columnas relevantes para el análisis, como:
o	Clave de la orden (order_key).
o	Clave del cliente (customer_key).
o	Estado de la orden (status_code).
o	Precio total (total_price).
o	Fecha de la orden (order_date).
2.2. Modelo stg_tpch_line_items
Este modelo transforma la tabla lineitem al:
•	Generar una clave única (order_item_key) para cada ítem en base a las columnas l_orderkey y l_linenumber mediante la macro generate_surrogate_key.
•	Renombrar columnas para consistencia (l_orderkey → order_key, l_quantity → quantity, etc.).
•	Seleccionar datos como:
o	Claves de orden, producto y línea.
o	Información financiera (precio extendido, descuento, impuesto).
 
3. Modelo Intermedio de Combinación
El modelo int_order_items combina los datos de órdenes (stg_tpch_orders) con los ítems asociados (stg_tpch_line_items) para enriquecer la información disponible.
Transformaciones importantes:
•	Se unen ambas tablas mediante la columna order_key.
•	Se calcula el monto descontado por ítem usando una macro personalizada (discounted_amount), basada en el precio extendido y el porcentaje de descuento.
•	Este modelo incluye columnas clave como:
o	Detalles del ítem (clave, precio, descuento, etc.).
o	Información de la orden (clave, cliente, fecha, etc.).
 
4. Resumen Agregado de Ítems
El modelo int_order_items_summary genera métricas agregadas a nivel de orden:
•	gross_item_sales_amount: Suma del precio extendido de todos los ítems de una orden.
•	item_discount_amount: Suma de los montos descontados para todos los ítems de una orden.
Estos cálculos permiten comprender las ventas brutas y los descuentos aplicados a nivel de orden, consolidando información clave para el análisis.
 
5. Modelo Fact Table (fct_orders)
El modelo fct_orders es la tabla final (tabla de hechos) que integra toda la información disponible. Este modelo:
1.	Combina las órdenes (stg_tpch_orders) con el resumen agregado de sus ítems (int_order_items_summary).
2.	Incluye todas las columnas relevantes de las órdenes (order_key, customer_key, status_code, order_date) junto con las métricas calculadas (gross_item_sales_amount, item_discount_amount).
Pruebas Aplicadas:
•	Unicidad y no nulidad de order_key: Asegura que cada orden sea única y válida.
•	Valores aceptados en status_code: Valida que el estado de las órdenes sea uno de los permitidos (P, O, F).
 
Propósito General del Proyecto
El proyecto facilita:
1.	Limpieza y transformación: Estandariza nombres de columnas y selecciona datos relevantes.
2.	Enriquecimiento de datos: Calcula métricas importantes, como precios extendidos y descuentos.
3.	Creación de resúmenes: Genera tablas agregadas con información clave para análisis de ventas.
4.	Validación de calidad: Implementa pruebas automatizadas para asegurar la integridad y consistencia de los datos.
5.	Optimización para análisis: La estructura final (tabla fct_orders) permite realizar análisis avanzados de ventas, descuentos y comportamiento de órdenes.
 
Estructura del Proyecto
1.	Fuentes: Datos crudos (tpch.orders, tpch.lineitem).
2.	Staging: Limpieza y transformación inicial (stg_tpch_orders, stg_tpch_line_items).
3.	Intermedios: Modelos enriquecidos (int_order_items, int_order_items_summary).
4.	Tablas finales: Modelo listo para análisis (fct_orders).
Esta estructura modular y jerárquica de dbt asegura que cada etapa del pipeline de datos sea fácilmente comprensible, probada y reutilizable.

![image](https://github.com/user-attachments/assets/67c237c3-6e8a-437c-8f2b-4b2774e21ebf)
