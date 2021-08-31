---
layout: post_es
categories: "es"
title: "Abrir Shapefile en R "
date: 2017-11-10
desc: "Hay muchas opciones para importar archivos shapefile en R, creo que la más fácil es a través de la librería rgdal..."
tags: R
---

Hay muchas opciones para importar archivos shapefile en R, creo que la más fácil es a través de la librería [rgdal](#https://cran.r-project.org/web/packages/rgdal/rgdal.pdf),  con la función readOGR para leer mapas como objetos espaciales.

<!--more-->

`library(rgdal)`

Recomiendo estar trabajando en el mismo working directory en donde se halla el archivo, así se puede verificar que el mismo esté completo y su nombre bien escrito.

`setwd("C:/Users/miusuario/Documents/esri")`

Un shapefile se compone **al menos de 3 archivos**, de extensión: `.shp .shx` y `.dbf`. Sin estos tres no se va a poder abrir el shapefile, y los tres tienen que tener exactamente el mismo nombre.

`.shp` es el archivo que almacena las entidades geométricas de los objetos.
`.shx` es el archivo que almacena el índice de las entidades geométricas.
`.dbf` es la base de datos, en formato dBASE, donde se almacena la información de los atributos de los objetos.

Opcionalmente se pueden utilizar otros archivos con extensión .cpg .prj y .qpj para especificar el encoding o incluir información sobre la proyección cartográfica.

Al correr `list.files()` en R, se muestra en la consola el contenido de la carpeta del directorio de trabajo, para este shapefile consiste de seis archivos

<pre>
[1] "mi_shape.cpg" "mi_shape.dbf" "mi_shape.prj" "mi_shape.qpj" "mi_shape.shp" "mi_shape.shx"
</pre>

En R se abre el shapefile al escribir su nombre sin ninguna extensión:

<pre>
sp_df <- readOGR(dsn = ".", layer = "mi_shape")
</pre>

Y ahora sí, con la ayuda de la librería leaflet,  disponer de todas las herramientas para hacer gráficos muy lindos:

<pre>
library(leaflet)

pal <- colorNumeric( palette = "YlGnBu", domain = shape2$AREA)

leaflet(shape2) %>%
    addProviderTiles("CartoDB.Positron") %>%
    addPolygons(color = ~pal(shape2$AREA),
    stroke = FALSE, fillOpacity = 0.6)

</pre>

<img src="/images/abrir-shapefile-R.png" class="postimg" >
