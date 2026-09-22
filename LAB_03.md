---

### Archivo 2: `LAB_03.md`

Para crearlo en GitHub:
1. Pulsa en **Add file** > **Create new file**[cite: 9].
2. En el nombre escribe `LAB_03.md`.
3. Pega el siguiente bloque:

```markdown
# Laboratorio 03: Escriba consultas T-SQL avanzadas

**Tiempo estimado:** 30 minutos  
**Autor:** Eric Emmanuel Ramírez Duanca  

En este ejercicio, practica el uso de funciones JSON para crear y consultar datos JSON desde la base de datos AdventureWorksLT. También combina la salida JSON con un CTE y una función de ventana para crear un informe práctico.

Eres desarrollador de bases de datos para una empresa de comercio electrónico. El equipo de marketing necesita datos de productos en formato JSON para un catálogo web y es necesario crear informes que clasifiquen los productos dentro de las categorías.

---

## 1. Conéctese a AdventureWorksLT

Asegúrese de que la base de datos de muestra de AdventureWorksLT esté restaurada y disponible en su instancia SQL. Verificar la conectividad:

Copie y pegue el siguiente código T-SQL en una nueva ventana de consulta. Seleccione Ejecutar para ejecutar esta consulta:

```sql
-- Verify key tables in AdventureWorksLT
SELECT TOP (5) ProductID, Name, ListPrice 
FROM SalesLT.Product;

SELECT TOP (5) ProductCategoryID, Name 
FROM SalesLT.ProductCategory;
Cada consulta debe devolver hasta cinco filas de datos de muestra. Si alguna consulta no devuelve filas o falla, confirme que la base de datos AdventureWorksLT esté restaurada correctamente y que tenga acceso de lectura.

2. Generar salida JSON a partir de datos del producto
El equipo de marketing necesita información del producto en formato JSON para un catálogo web. Comience creando un objeto JSON simple a partir de la tabla Producto.

2.1. Crea un objeto JSON para cada producto
Úselo FOR JSON PATH para convertir filas de productos a JSON.

Copie y pegue el siguiente código T-SQL en una nueva ventana de consulta. Seleccione Ejecutar para ejecutar esta consulta:

SQL
-- 2.1. Generación de un array JSON plano a partir de la tabla Product
SELECT 
    ProductID,
    Name,
    Color,
    ListPrice
FROM SalesLT.Product
WHERE Color IS NOT NULL
ORDER BY ListPrice DESC
FOR JSON PATH;
Esta consulta selecciona productos con un valor de color y formatea los resultados como una matriz JSON. Cada fila se convierte en un objeto JSON con propiedades que coinciden con los nombres de las columnas. La cláusula FOR JSON PATH maneja la conversión automáticamente.

2.2. Crear JSON anidado con categorías de productos
Agregue información de categoría como un objeto anidado.

Copie y pegue el siguiente código T-SQL en una nueva ventana de consulta. Seleccione Ejecutar para ejecutar esta consulta:

SQL
-- 2.2. Construcción de objetos JSON anidados con JSON_OBJECT()
SELECT 
    p.ProductID,
    p.Name AS ProductName,
    p.ListPrice,
    JSON_OBJECT(
        'CategoryID': pc.ProductCategoryID,
        'CategoryName': pc.Name
    ) AS Category
FROM SalesLT.Product AS p
INNER JOIN SalesLT.ProductCategory AS pc
    ON p.ProductCategoryID = pc.ProductCategoryID
ORDER BY p.ListPrice DESC
FOR JSON PATH;
Esta consulta utiliza JSON_OBJECT para construir una estructura anidada. La propiedad Categoría contiene su propio objeto JSON con CategoryID y CategoryName. Este enfoque mantiene los datos relacionados agrupados en el resultado.

3. Combine JSON con una función CTE y de ventana
Ahora cree un informe más útil que clasifique los productos por precio dentro de cada categoría y genere el resultado como JSON.

3.1. Escriba un CTE con clasificación de funciones de ventana
Primero, construya la lógica de consulta usando un CTE y ROW_NUMBER().

Copie y pegue el siguiente código T-SQL en una nueva ventana de consulta. Seleccione Ejecutar para ejecutar esta consulta:

SQL
-- 3.1. Ranking por partición con Common Table Expression (CTE) y ROW_NUMBER()
WITH RankedProducts AS (
    SELECT 
        p.ProductID,
        p.Name AS ProductName,
        pc.Name AS CategoryName,
        p.ListPrice,
        ROW_NUMBER() OVER (
            PARTITION BY pc.ProductCategoryID 
            ORDER BY p.ListPrice DESC
        ) AS PriceRank
    FROM SalesLT.Product AS p
    INNER JOIN SalesLT.ProductCategory AS pc
        ON p.ProductCategoryID = pc.ProductCategoryID
    WHERE p.ListPrice > 0
)
SELECT 
    ProductID,
    ProductName,
    CategoryName,
    ListPrice,
    PriceRank
FROM RankedProducts
WHERE PriceRank <= 3
ORDER BY CategoryName, PriceRank;
El CTE calcula un rango de precios para cada producto dentro de su categoría. La cláusula PARTITION BY reinicia la numeración de cada categoría y ORDER BY ListPrice DESC asigna el rango 1 al producto más caro. Los filtros de consulta externos muestran solo los 3 productos principales por categoría.

3.2. Genere los productos clasificados como JSON
Agregar FOR JSON PATH para formatear los resultados de una API.

Copie y pegue el siguiente código T-SQL en una nueva ventana de consulta. Seleccione Ejecutar para ejecutar esta consulta:

SQL
-- 3.2. Exportar el Top 3 clasificado a JSON con nodo raíz ROOT()
WITH RankedProducts AS (
    SELECT 
        p.ProductID,
        p.Name AS ProductName,
        pc.Name AS CategoryName,
        p.ListPrice,
        ROW_NUMBER() OVER (
            PARTITION BY pc.ProductCategoryID 
            ORDER BY p.ListPrice DESC
        ) AS PriceRank
    FROM SalesLT.Product AS p
    INNER JOIN SalesLT.ProductCategory AS pc
        ON p.ProductCategoryID = pc.ProductCategoryID
    WHERE p.ListPrice > 0
)
SELECT 
    ProductID,
    ProductName,
    CategoryName,
    ListPrice,
    PriceRank
FROM RankedProducts
WHERE PriceRank <= 3
ORDER BY CategoryName, PriceRank
FOR JSON PATH, ROOT('TopProducts');
La adición ROOT('TopProducts') envuelve toda la matriz JSON en un objeto con una propiedad con nombre. Esto hace que sea más fácil trabajar con el resultado en aplicaciones que esperan un elemento raíz.

4. Analizar datos JSON con OPENJSON
Ahora practique la lectura de datos JSON nuevamente en filas usando OPENJSON.

4.1. Analizar una matriz JSON en filas
Supongamos que recibe actualizaciones de productos como JSON. Úselo OPENJSON para convertirlo en una tabla.

Copie y pegue el siguiente código T-SQL en una nueva ventana de consulta. Seleccione Ejecutar para ejecutar esta consulta:

SQL
-- 4.1. Deserializar un string JSON en filas tabulares usando OPENJSON y WITH
DECLARE @ProductUpdates NVARCHAR(MAX) = N'[
    {"ProductID": 680, "NewPrice": 1250.00},
    {"ProductID": 706, "NewPrice": 1450.00},
    {"ProductID": 707, "NewPrice": 38.99}
]';

SELECT 
    ProductID,
    NewPrice
FROM OPENJSON(@ProductUpdates)
WITH (
    ProductID INT '$.ProductID',
    NewPrice DECIMAL(10,2) '$.NewPrice'
);
La cláusula WITH define el esquema para la salida. Cada propiedad JSON se asigna a una columna con un tipo de datos específico. La sintaxis $.PropertyName le dice a SQL Server qué ruta JSON leer para cada columna.

4.2. Unir JSON analizado con datos existentes
Combine los datos JSON con la tabla Producto para ver los precios actuales y nuevos.

Copie y pegue el siguiente código T-SQL en una nueva ventana de consulta. Seleccione Ejecutar para ejecutar esta consulta:

SQL
-- 4.2. JOIN entre una tabla relacional existente y datos procesados con OPENJSON
DECLARE @ProductUpdates NVARCHAR(MAX) = N'[
    {"ProductID": 680, "NewPrice": 1250.00},
    {"ProductID": 706, "NewPrice": 1450.00},
    {"ProductID": 707, "NewPrice": 38.99}
]';

SELECT 
    p.ProductID,
    p.Name,
    p.ListPrice AS CurrentPrice,
    updates.NewPrice,
    updates.NewPrice - p.ListPrice AS PriceDifference
FROM SalesLT.Product AS p
INNER JOIN OPENJSON(@ProductUpdates)
WITH (
    ProductID INT '$.ProductID',
    NewPrice DECIMAL(10,2) '$.NewPrice'
) AS updates
    ON p.ProductID = updates.ProductID;
Esta consulta une el JSON analizado directamente con la tabla Producto. La función OPENJSON con una cláusula WITH actúa como una tabla, por lo que puedes unirla como cualquier otra fuente de datos. El resultado muestra el precio actual de cada producto junto con el nuevo precio propuesto.
