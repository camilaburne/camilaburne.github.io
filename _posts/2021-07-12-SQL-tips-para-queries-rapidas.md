---
layout: post
categories: "es"
title: "SQL tips para correr queries rápido"
date: 2021-07-12
tags: sql blog
desc: "5 cosas a evitar al usar bases de big data"
---

Hacer queries con SQL es el primer paso de cualquier proyecto de ciencia de datos, y muchas veces, la parte más lenta. Para mejorar el rendimiento de la extracción de datos, comparto 5 errores para evitar en tu query.


<br />

Lo que _no_ hay que hacer:
1. [SELECT * FROM table](#t1)
2. [Cartesian Joins](#t2)
3. [Joins en tablas grandes](#t3)
4. [DISTINCT](#t4)
5. [WITH big tables](#t5)


<br />

---
### Lo que está mal con SELECT * FROM table <a name="t1"></a>


Hoy en día, los datos se guardan en big data warehouses que se organizan en columnas, como Redshift, Hadoop y Big-Query. En este tipo de almacenamiento, es más eficiente acceder a todas las filas desde una columna que acceder a todas las columnas de una sola fila. Por este motivo, hay que evitar usar el * al seleccionar todo, porque se está llamando a todas las columnas, usando el máximo de información de una tabla. Para que se corra más rápido, conviene nombrar solamente los campos que te interesen.

⛔️ `SELECT * FROM table ;`

✅ `SELECT col1, col2 FROM table ;`



**Notas sobre almacenamiento en columnas - columnar store - y filas - row store - **:

 En el row-store la información se guarda y recupera una fila a la vez, son fáciles de leer y escribir, pero no es fácil agregar o recuperar todas las filas al mismo tiempo. Este es el caso más tradicional, que se suele ver en tutoriales, de una "base de datos de estudiantes" en la que es de interés ver los datos de un individuo en particular. Anteriormente se usaba este tipo de almacenamiento, pero fue quedando desactualizado desde que la información se guarda masivamente, ocupando millones de filas.

 Para este caso se usa el columnar-store, almacenamiento de datos en columnas, que es común para información OLAP (online analytical processing). En este caso nos interesa realizar cálculos sobre campos, por ejemplo, obtener un número total de transacciones y  calcular su monto promedio.

En ambas situaciones, las tabla se ve exactamente iguales, pero el rendimiento es diferente según los tipos de consulta que ejecutamos. Para el almacenamiento orientado a filas, podríamos ver fácilmente toda la información de una unidad determinada, y en el almacenamiento orientado a columnas, podemos procesar agregados en columnas muy rápido. Como en aplicaciones de big data nos interesa realizar cálculos en columnas, hoy en día las bases de datos más comunes siguen este estilo. [Más sobre row y columnar stores.](https://medium.com/bluecore-engineering/deciding-between-row-and-columnar-stores-why-we-chose-both-3a675dab4087#:~:text=In%20row%20oriented%20databases%2C%20these,data%20is%20read%20at%20once.)

<br />

### Errores en Joins Cartesianos <a name="t2"></a>

Los joins cartersians, aka cross joins, devuelven todas las combinaciones posibles de los registros de dos tablas, por lo que si cruzás dos tablas con N filas, los resultados van a tener NxN filas. A menos que estés diseñando un experimento o creando una vista, en la vida real nunca vas a user esta combinación.

En algunos editores de SQL, este es el join predeterminado cuando no especificás el nombre, por lo que siempre, siempre, siempre, hay que escribir: `INNER JOIN` o` LEFT JOIN`. No es necesario ningún otro, ni siquiera `RIGHT JOIN` porque eso es para gente rara.

⛔️
<pre>
SELECT
 t1.col1,
 t2.col2
FROM table1 as t1, table2 as t2 ;
</pre>


✅
<pre>
SELECT
 t1.col1,
 t2.col2
 FROM table1 as t1
 LEFT JOIN table2 as t2 ON t1.id = t2.t1_id ;
</pre>


<br />

### Evitar joins con tablas enormes <a name="t3"></a>

Hablando de joins: no juntes tablas enormes entre sí. Es mejor hacer un proceso de dos pasos: primero reducí cada tabla original a través de una subquery, luego uní estas dos subqueries.
Aunque parece más complejo tener consultas anidadas, te va a ahorrar tiempo, ya que se reduce significativamente el tiempo de ejecución.

⛔️   
<pre>SELECT
  c.id,
  c.age,
  t.amt

FROM customers c                          
LEFT JOIN transactions t on c.id = t.cust_id
</pre>


✅
<pre>SELECT *    

FROM (
      SELECT id, age
      FROM customers
      ) c        
LEFT JOIN (
      SELECT cust_id, amt
      FROM transactions) t on c.id = t.cust_id
</pre>

### DISTINCT<a name="t4"></a>

La sentencia DISTINCT es esencialmente un grupo by, pero es más costoso porque uno se olvida cuánto se está agrupando. En general se usa para obtener registros únicos, generalmente cuando hay duplicados en tu tabla original o en cualquiera de las tablas que juntaste a través de un join. `DISTINCT` funciona ordenando primero todos los datos y luego agrupando por todos los campos incluidos en la instrucción SELECT. Es decir, es un group by de *todo*, por eso es lento.

Usar `DISTINCT` en varias columnas que no son índices, es costoso en tiempo y memoria, así que conviene evitarlo. En su lugar, se pueden usar tablas temporales con pre-agregados, y evitar uniones con tablas en crudo.

<br />

### WITH <a name="t5"></a>
WITH se usa para nombrar una subquery, para que puedas usarla en tu consulta principal. Esto podría ser útil para estructurar su código, cuando estás anidando consultas, pero puede ser costoso ya que estás ejecutando todas sus sub-queries a la vez.

Consultar muchas tablas grandes y llenas al mismo tiempo es una mala idea. A veces, cuando tengo que crear vistas o tablas que involucran muchas combinaciones, como más de 10, es mejor crear tablas temporales intermedias, e ir generándolas de una a la vez.

Por ejemplo, imaginemos que tenemos una tabla de clientes, una tabla de transacciones y una tabla de ciudad. Queremos tener el importe medio de la compra máxima que realiza un cliente en cada ciudad.


En lugar de ejecutar dos consultas al mismo tiempo:

⛔️   
<pre>
WITH my_big_table
  as (SELECT
      c.id,
      c.age,
      a.city,  
      max(t.amt) as max_amt      
      FROM customers c                          
      LEFT JOIN transactions t on c.id = t.cust_id
      LEFT JOIN cust_address a on c.id = a.cust_id
      group by 1,2,3
      )  

  SELECT a.city, avg(t.amt)
  group by 1 ;

</pre>

Es mejor crear una tabla temporal y hacerlo en dos pasos:

✅
<pre>
CREATE TEMPORARY TABLE AS max_amt_cust
  ( SELECT
      c.id,
      max(t.amt) as max_amt      
      FROM customers c                          
      LEFT JOIN transactions t on c.id = t.cust_id
      GROUP BY 1
  );

SELECT
  a.city(),
  max(m.amt) as max_amt      
FROM cust_address a
LEFT JOIN max_amt_cust m on m.id = a.cust_id
GROUP BY 1

</pre>

Recibí estos consejos de mi manager después de que ella identificara consultas que tardaban minutos en ejecutarse 😬. Como regla general, si una consulta tarda más de 30 segundos, es probable que haya alguno de estos cinco errores en tu SQL y que puedas solucionarlo limitando el número de columnas, corrigiendo joins, haciendo tablas intermedias y group by a mano; evitando los distinct, with y select *.  
