
<div style="background-color: #f0f8ff; padding: 20px; border-radius: 10px; font-family: Arial, sans-serif; color: #333;">
    <h1 style="color: #d9534f; text-align: center;">Proyecto de Consultoría BI para Pizzatopia</h1>
    <p>En este proyecto, asumo el papel de <span style="background-color: #d9534f; color: white; padding: 3px 5px; border-radius: 5px;">Consultor de BI</span> para <strong>Pizzatopia</strong>, una pizzería (ficticia) comprometida con la excelencia en la satisfacción del cliente y la eficiencia operativa. Aunque Pizzatopia ha estado recopilando datos transaccionales durante el último año, hasta ahora no ha podido utilizarlos de manera efectiva para impulsar las ventas y optimizar las operaciones.</p>
    
   <h2 style="color: #d9534f; text-align: center;">Objetivo del Proyecto</h2>
  <p>El objetivo de este proyecto es analizar los datos existentes para identificar oportunidades de mejora en las ventas y la eficiencia operativa. Utilizando una base de datos <strong>MySQL</strong> alojada en <strong>Google Cloud</strong>, llevaré a cabo un análisis exhaustivo y desarrollaré visualizaciones interactivas mediante <strong>Looker Studio</strong>. Estas herramientas permitirán transformar los datos en insights accionables y facilitarán la toma de decisiones informadas.</p>

  El conjunto de datos, que tiene cuatro tablas con un total de 48.620 registros y doce campos, se puede encontrar en https://www.mavenanalytics.io/data-playground con el nombre Pizza Place Sales.

  <h2 style="color: #d9534f; text-align: center;">Consultas SQL</h2>
</div>

 <pre style="white-space: pre; overflow-x: auto;">
<code>     
-- 1. ¿Cuál es el número promedio de clientes al día?
SELECT 
  ROUND(COUNT(order_id)/COUNT(DISTINCT date),0) AS avg_clientes_por_dia
FROM pizzadb.orders;

-- 2. ¿Cuántas pizzas suele haber en un pedido?
SELECT 
  ROUND(SUM(od.quantity)/COUNT(DISTINCT od.order_id),2) AS avg_pizza_por_pedido
FROM pizzadb.order_details od; 

-- 3. ¿Cuál es el total de pizzas vendidad y el ingreso total?
--    ¿Cuál es el ingreso medio por pedido?

SELECT
  SUM(od.quantity) AS pizzas_vendidas,
  SUM(od.quantity*p.price) AS ingreso_total,
  ROUND(SUM(od.quantity*p.price)/COUNT(DISTINCT od.order_id),2) AS avg_venta_por_pedido   
FROM pizzadb.order_details od
JOIN pizzadb.pizza p ON  p.pizza_id =  od.pizza_id;

-- 4. ¿Qué cantidad promedio de pizzas se vende por hora y cuál es el ingreso generado por hora?

SELECT 
  EXTRACT(Hour FROM o.time) AS Hora,
  ROUND(SUM(od.quantity)/COUNT(DISTINCT o.date),0) AS avg_cantidad,
  ROUND(COUNT(distinct o.order_id)/COUNT(DISTINCT o.date),0) AS pedidos_totales,
  ROUND(SUM(od.quantity*p.price),0) AS ingreso_hora
FROM pizzadb.orders o
JOIN pizzadb.order_details od ON od.order_id = o.order_id
JOIN pizzadb.pizza p ON p.pizza_id = od.pizza_id
GROUP BY Hora
ORDER BY avg_cantidad desc;

-- 5. ¿Qué mes experimentó el mayor número de pedidos de ingresos?}
SELECT
  MONTH(o.date) AS mes_num,
  CASE
    WHEN MONTHNAME(o.date) = 'January' THEN 'Enero'
    WHEN MONTHNAME(o.date) = 'February' THEN 'Febrero'
    WHEN MONTHNAME(o.date) = 'March' THEN 'Marzo'
    WHEN MONTHNAME(o.date) = 'April' THEN 'Abril'
    WHEN MONTHNAME(o.date) = 'May' THEN 'Mayo'
    WHEN MONTHNAME(o.date) = 'June' THEN 'Junio'
    WHEN MONTHNAME(o.date) = 'July' THEN 'Julio'
    WHEN MONTHNAME(o.date) = 'August' THEN 'Agosto'
    WHEN MONTHNAME(o.date) = 'September' THEN 'Septiembre'
    WHEN MONTHNAME(o.date) = 'October' THEN 'Octubre'
    WHEN MONTHNAME(o.date) = 'November' THEN 'Noviembre'
    WHEN MONTHNAME(o.date) = 'December' THEN 'Diciembre'
  END AS mes,
  COUNT(DISTINCT o.order_id) AS pedidos_totales,
  ROUND(SUM(p.price*od.quantity),0) AS ingreso
FROM pizzadb.orders o
JOIN pizzadb.order_details od ON od.order_id = o.order_id
JOIN pizzadb.pizza p ON p.pizza_id = od.pizza_id
GROUP BY mes, mes_num
ORDER BY mes_num asc;

-- 6. ¿Hay días de máxima afluencia?

SELECT 
  CASE 
    WHEN DAYNAME(o.date) = 'Sunday' THEN 'Domingo'
    WHEN DAYNAME(o.date) = 'Monday' THEN 'Lunes'
    WHEN DAYNAME(o.date) = 'Tuesday' THEN 'Martes'
    WHEN DAYNAME(o.date) = 'Wednesday' THEN 'Miércoles'
    WHEN DAYNAME(o.date) = 'Thursday' THEN 'Jueves'
    WHEN DAYNAME(o.date) = 'Friday' THEN 'Viernes'
    WHEN DAYNAME(o.date) = 'Saturday' THEN 'Sábado'
  END AS dia,
  ROUND(COUNT(o.order_id)/COUNT(DISTINCT o.date),0) AS avg_clientes
  SUM(od.quantity) AS ventas_totales
FROM pizzadb.orders o
JOIN pizzadb.order_details od ON od.order_id= o.order_id
GROUP BY dia;

-- 7. ¿Cuál es el tamaño de pizza más vendido?

SELECT
  p.size AS tamaño,
  SUM(od.quantity) AS cantidad_total
FROM pizzadb.pizza p
JOIN pizzadb.order_details od ON od.pizza_id = p.pizza_id
GROUP BY tamaño
ORDER BY cantidad_total desc;

-- 8. ¿Qué categoría de pizza es más ordenada?

SELECT
  pt.category AS categoria,
  SUM(od.quantity) AS cantidad_total
FROM pizzadb.pizza_type pt
JOIN pizzadb.pizza p ON p.pizza_type_id = pt.pizza_type_id
JOIN pizzadb.order_details od ON od.pizza_id = p.pizza_id
GROUP BY categoria
ORDER BY cantidad_total desc;

-- 9. ¿Cuáles son los ingredientes más ordenados?

SELECT 
    ingredient, 
    COUNT(ingredient) AS count
FROM (
    SELECT 
        TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(pt.ingredients, ',', numbers.n), ',', -1)) AS ingredient
    FROM pizzadb.pizza_type pt
    JOIN pizzadb.pizza p ON pt.pizza_type_id = p.pizza_type_id
    JOIN pizzadb.order_details od ON p.pizza_id = od.pizza_id
    JOIN (
        SELECT 1 n UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 
        UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9 UNION ALL SELECT 10
    ) numbers ON CHAR_LENGTH(pt.ingredients) - CHAR_LENGTH(REPLACE(pt.ingredients, ',', '')) >= numbers.n - 1
) ingredients_split
GROUP BY ingredient
ORDER BY count desc;

-- 10. Si el número promedio de pizzas es 2, podemos asumir 2 personas por orden
-- Entonces ¿Qué % del total de ordenes ha ordenado 2 pizas?

SELECT
  COUNT(*) AS pedidos_menores_a_3,
  COUNT(*)*100/ (SELECT COUNT(DISTINCT od.order_id) FROM pizzadb.order_details od) AS porcentaje_2_pizzas,
  (SELECT SUM(od.quantity) FROM pizzadb.order_details od) AS total_pizzas,
  (SELECT SUM(od.quantity)/COUNT(DISTINCT od.order_id) FROM pizzadb.order_details od) AS avg_pizzas
FROM ( SELECT
        od.order_id,
        SUM(od.quantity) AS pizzas
       FROM pizzadb.order_details od
       GROUP BY od.order_id
      ) subquery
WHERE subquery.pizzas <3;    
</code>
</pre>

## <b> Resultados: https://lookerstudio.google.com/s/jfai69kWXUg </b>
<div>
    <img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*kxfn2q_nUQo-EyiU7rnd7g.jpeg" alt="Dashboard" style="max-width: 100%; height: auto;">
</div>
