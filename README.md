# Laboratorio 03 - Escriba consultas T-SQL avanzadas

**Autor:** Eric Emmanuel Ramírez Duanca

---

## Entorno de Desarrollo

* **Motor de base de datos:** Microsoft SQL Server 2022 o posterior (o Azure SQL Database)
* **Base de datos de ejemplo:** AdventureWorksLT
* **Cliente gráfico:** SQL Server Management Studio (SSMS)

---

## Instrucciones para reproducir el trabajo

Para preparar el entorno y ejecutar los ejercicios, sigue estos pasos desde SSMS:

1. **Restauración y conexión a la base de datos:** Asegúrate de tener restaurada la base de datos de muestra AdventureWorksLT. Conéctate a tu instancia local de SQL Server, abre una "Nueva Consulta" (New Query) y ejecuta los scripts asegurando el contexto en AdventureWorksLT.
2. **Verificación de tablas clave:** Ejecuta las consultas de verificación sobre `SalesLT.Product` y `SalesLT.ProductCategory` para validar el acceso a los datos.
3. **Generación y manipulación de datos JSON:**
   * **Objeto JSON simple:** Ejecuta la consulta usando `FOR JSON PATH` sobre la tabla de productos filtrando colores no nulos.
   * **JSON anidado:** Construye estructuras jerárquicas relacionando productos y categorías mediante `JSON_OBJECT()`.
4. **Consultas avanzadas con CTE y Funciones de Ventana:**
   * **Clasificación por categoría:** Implementa un Common Table Expression (CTE) con `ROW_NUMBER() OVER(PARTITION BY ... ORDER BY ...)` para extraer el Top 3 de productos por precio de cada categoría.
   * **Exportación con nodo raíz:** Encapsula el resultado estructurado anterior en un documento JSON formateado con la cláusula `ROOT('TopProducts')`.
5. **Procesamiento y parseo de JSON (`OPENJSON`):**
   * **Deserialización a formato tabular:** Convierte variables con arrays JSON en registros de tabla definiendo el esquema de tipos con `OPENJSON ... WITH`.
   * **Integración relacional:** Realiza un `INNER JOIN` entre la tabla física `SalesLT.Product` y la estructura devuelta por `OPENJSON` para contrastar precios actuales contra nuevos precios propuestos.

---

## Estructura del repositorio

```text
Lab_03/
├── README.md                   <-- (Este archivo explicativo del repositorio)
├── LAB_03.md                   <-- (Documentación completa en Markdown con imágenes)
├── LAB_03.pdf                  <-- (Documentación original en PDF con evidencias)
└── img/                        <-- (Carpeta con las capturas de pantalla de ejecución en SSMS)
    ├── AdventureWorksLT.png
    ├── Object-JSON.png
    ├── JSON-anidado.png
    ├── Funcion-CTE.png
    ├── Productos-JSON-CTE.png
    ├── JSON-filas.png
    └── Unir-JSON.png
