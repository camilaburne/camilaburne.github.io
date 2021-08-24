---
layout: post_es
categories: "es"
title: "Código resumen para el análisis de datos en R "
date: 2017-03-01
desc: "Les comparto un código que sirve de ayuda para limpiar y analizar datos en R en menos de 10 pasos. Yo lo tengo guardado como .txt..."
tags: R

---



Este código resumen se lo hice a una amiga para ayudarla en su primera entrevista de trabajo, apenas terminamos de estudiar la carrera, allá por 2017. Sirve de ayuda para limpiar y analizar datos en R en menos de 10 pasos, con un conjunto de datos inventado para revisar que funcione :)
<a name="p0"></a> Cada vez que empiezo un proyecto nuevo en R, abro esta guía txt para no tener que buscar ni escribir las mismas funciones que uso cada vez. Comparto el código en
<a href="/download/template-analisis-datos-R-codigo.txt" class="download">archivo txt </a> y la <a href="/download/base_ejemplo.csv" class="download">base csv de ejemplo</a> .


### Pasos:

1. [**Directorio**](#p1) - Elegir un directorio (carpeta) para tu código o verificar que estás en el correcto.
2. [**Abrir**](#p2) un conjunto de datos .csv en R como un dataframe (df).
3. [**Dimensionar**](#p3) la forma y tamaño del df, columnas, tipos de variables y estadísticas resumen para tener una idea de tus datos.
4. [**Seleccionar**](#p4) subgrupos de variables y/o filas.
5. [**Cambiar tipos**](#p5) de variable (de factor a texto, de decimal a entero).
6. [**Filtrar**](#p6) datos por condiciones o reglas.
7. [**Limpiar**](#p7) duplicados o imputar valores perdidos
8. [**Visualizar**](#p8) datos agregados y gráficos.
9. [**Exportar**](#p9) el dataframe como .csv

### Código:


<pre>
# 1 DIRECTORIO<a name="p1"></a>  

# getwd() para ver el directorio en el que estás ahora
# setwd() para cambiarlo si no era el correcto
# list.files() imprime lo que hay adentro

getwd()
setwd("/Users/camilaburne/folder")
list.files()

</pre>

Al correr list.files() podés ver los archivos en tu directorio. Están ordenados y podés acceder al primer elemento con `list.files()[1]`:

<img src="/images/template-analisis-datos-R-p1.png" class="postimg" >


<pre>
# 2 ABRIR<a name="p2"></a>    

# Abrir archivos que estén dentro del directorio
# poniendo el nombre en donde dice 'file'
# read.csv(file, header = TRUE, sep = ",")

df01 <- read.csv("base_ejemplo.csv")
df02 <- read.csv(list.files()[1])

</pre>

En el panel de la derecha, una vez que importás los archivos con `read_csv`, te aparecen el nombre y dimensión de cada objeto.

<img src="/images/template-analisis-datos-R-p2.png" class="postimg" >


<pre>
# 3 DIMENSIONAR<a name="p3"></a>  

# Funciones para ver qué son y qué tienen los df
# dim() devuelve el nro de filas y columnas
# name() muestra las columnas
# head() y tail() las primeras y últ filas

dim(df01)
names(df01)
head(df01, 10)
tail(df01,  2)

# Funciones para ver qué son y qué tienen las variables
# typeof() te dice qué tipo de objeto es

typeof(df01$grupo)
typeof(df01$gasto)
typeof(df01$nombre)

# Funciones para ver sus estadísticas resumen
# mean() y median() para ver promedio y mediana
# range() y sd() para ver rango y dispersión
# quantile() es para ver los percentiles

mean(df01$gasto)
median(df01$gasto)

range(df01$gasto)
sd(df01$gasto)

quantile(df01$gasto, 0.5)
summary(df01)

</pre>

Con names() se ve que el dataframe muestra valores de compra y venta, para ciertos nombres, grupos y mes. Una variable que puede analizarse es gasto, de la cual se calcula el promedio y los percentiles 25%, 50%, 75%, 90%.

<img src="/images/template-analisis-datos-R-p3.png" class="postimg" >


<pre>
# 4 SELECCIONAR<a name="p4"></a>    

# Seleccionar una variable solita, con $ o [ ]
df01$nombre
df01[ ,'nombre']

# Seleccionar un par de filas y de columnas con [ ]
# y pasando el nombre de columnas y número de filas
df01[1:10, ]
df01[c(1,10,20,30), c('nombre','gasto')]
df01[c(1,10,20,30), c(1,2)]

</pre>

<pre>
# 5 CAMBIAR TIPOS<a name="p5"></a>  

# Funciones para cambiar el tipo de variable
df01$nombre <- as.character(df01$nombre)
df01$gasto  <- as.numeric(df01$gasto)
df01$gasto_redondo <- as.integer(df01$gasto)

</pre>

<pre>
# 6 FILTRAR<a name="p6"></a>  

# Seleccionar filas usando filtros. Operadores útiles:
#  &     cumple TODAS las conduciones (Y)
#  |     cumple AL MENOS una condicion (O)
#  %in%  significa contenido en  

df01[df01$gasto < 70, ]
df01[df01$gasto < 100  & df01$grupo==1, ]

df01[!(df01$mes=='enero'),]
df01[(df01$mes=='febrero' | df01$mes=='marzo'),]

df01[df01$grupo %in% c(1,2,3), ]

grup12 = c(1,2)
vars   = c(1,4)
df01[df01$grupo %in% grup12, vars]

# Una vez que te interesa un subgrupo que filtraste
# Conviene guardarlo en un df o variable nueva

gastos    <- as.data.frame(df01$gasto)
meses     <- unique(df01$mes)
df01enero <- df01[df01$mes=='enero',]

</pre>

El operador `!` significa "que no sea", en este caso si se filtran meses que no son 'enero' se ven los restantes (febrero, marzo).

<img src="/images/template-analisis-datos-R-p6.png" class="postimg" >



<pre>
# 7 LIMPIAR<a name="p7"></a>  

# is.na() te dice TRUE si hay datos perdidos
# Para reemplazarlos:

df01$pct[is.na(df01$pct)] = 0

# duplicated() devuelve TRUE si hay repetidos
# Para encontrar duplicados:
df01$doble <- duplicated(df01)
table(df01$doble)

# Crear df nuevo sin duplicados
df01unico <- subset(df01, df01$doble==FALSE)

nrow(df01)
nrow(df01unico)

</pre>

La función duplicated() devuelve `TRUE` si la fila está repetida, y `FALSE` si es única. Con subset se puede obtener el subgrupo de filas que no están repetidas, generando un dataframe con menos filas que el original.

<img src="/images/template-analisis-datos-R-p7.png" class="postimg" >



<pre>
# 8 VISUALIZAR<a name="p8"></a>  

# Funciones para sacar conteos, porcentajes,
# y graficar scatterplots, gráfico de barras, histogramas

table(df01unico$tipo)
prop.table(table(df01unico$tipo))

hist(df01unico$pct, col = 'navyblue', border = 'white')
plot(df01unico$gasto, df01unico$pct, pch=19)
barplot(table(df01unico$grupo))

</pre>

En el panel a la derecha se visualizan los gráficos, ya sea que se hagan con R básico o  ggplot.

<img src="/images/template-analisis-datos-R-p8-2.png" class="postimg" >

Los gráficos básicos de R son sencillos pero aún así tienen varios parámetros para modificar, cómo main (título), xlab, ylab (nombre de ejes). Al correr `?hist`, se ven los parámetros de esta función.

<img src="/images/template-analisis-datos-R-p8-1.png" class="postimg" >



<pre>
# 9 EXPORTAR<a name="p9"></a>  

# revisar dónde lo estoy guardando

getwd()
write.csv(df01unico, 'baseok.csv', row.names = F)

</pre>

Con esta guía se pueden arrancar muchísimos proyectos de ciencias de datos, ya que entender, organizar, limpiar, resumir, visualizar la información cruda es la parte más importante, la que lleva más tiempo (el 80% al menos!) y la habilidad que le pidieron a mi amiga en su entrevista :) que al final se ganó el puesto y ya lleva +5 años ahí.

<br>
<br>
<br>
<br>
Volver al [principio del artículo](#p0)
