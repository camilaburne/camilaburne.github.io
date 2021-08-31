---
layout: post_es
categories: "es"
title: "De regresión a Machine Learning: Dos siglos de Estadística"
date: 2021-07-25
tags: stats
desc: "Hoy ML es ... pero empezó 200 años atrás con min cua"
---

🚧🚧 Sitio en construccion 🚧🚧

### Contenidos

| **Year**    | **Publication**                     | **Author**                         |
| 1805        | [Mínimos Cuadrados](#ls)            | Legendre & Gauss                   |
| 1812,1901   | [Teorema Central del Límite](#tcl)  | Laplace, Lyapunov                  |
| 18--        | [Regresion al promedio](#reg)       | Galton                             |
| 1907        | [T de Student](#t)                  | William Gosset, "Student"          |
| 1918        | [ANOVA](#fisher)                    | Fisher                             |
| 1936        | [Análisis Discriminante](#fisher)   | Fisher                             |
| 1940        | [Regresion Logistica](#logreg)      | AAVV                               |
| 1970        | [Modelos Lineales Gralizados](#glm) | Nelder & Wedderburn                |
| 1980        | [Arboles de decision](#tree)        | Breiman, Friedman, Olshen & Stone  |
| 1993        | [R software](#r)                    | Ross Ihaka & Robert Gentleman      |



Links con info:
[Gauss vs Lagrange](#https://projecteuclid.org/journals/annals-of-statistics/volume-9/issue-3/Gauss-and-the-Invention-of-Least-Squares/10.1214/aos/1176345451.full)
[History of Statistics - video](https://www.youtube.com/watch?v=zhLY4Vu_TEQ)
[Historia de R - video](https://www.youtube.com/watch?v=STihTnVSZnI)


## Mínimos Cuadrados <a name="ls"></a>

El primer hito en la historia es el más dramático, porque el desarrollo de mínimos cuadrados se disputa entre dos autores. Hoy se acepta que Gauss descubrió mínimos cuadrados primero, pero Lagendre le ganó en la publicación {1}. Una de las evidencias a favor de que Gauss lo inventó primero, fue que en 1799 utilizó el método de mínimos cuadrados para calcular la eliplicidad de la Tierra. De igual modo, fue gracias a Lagendre que se popularizó - tan rápidamente que en 10 años se estaba usando en varios países de Europa.

<img src="/images/gauss_legendre.jpg" class=rightimg>


Una primera demostración de la fuerza del método de Gauss se produjo cuando se utilizó para predecir la ubicación futura del asteroide recién descubierto Ceres. El 1 de enero de 1801, el astrónomo italiano Giuseppe Piazzi descubrió Ceres y pudo seguir su trayectoria durante 40 días antes de que se perdiera en el resplandor del sol. Basándose en estos datos, los astrónomos deseaban determinar la ubicación de Ceres después de que emergiera detrás del sol sin resolver las complicadas ecuaciones no lineales de Kepler del movimiento planetario. Las únicas predicciones que permitieron con éxito al astrónomo húngaro Franz Xaver von Zach reubicar Ceres fueron las realizadas por Gauss, de 24 años, utilizando análisis de mínimos cuadrados.

El método de mínimos cuadrados es uno de los hitos principales en la historia de la ciencia de datos porque por primera vez se acepta de manera masiva la idea de ajustar una función a datos observados. Además, este método podía aplicarse a cualquier disciplina, no estaba limitado a la astronomía y geodesia como muchos de los avances de los autores en esa época.


## Teorema Central del Límite <a name="tcl"></a>


At the beginning of the nineteenth century, Legendre and Gauss published papers on the method of least squares, which implemented the earliest form of what is now known as linear regression. The approach was first successfully applied to prob- lems in astronomy.

<img src="/images/laplace_lyapunov.jpg" class=rightimg>


Linear regression is used for predicting quantitative values, such as an individual’s salary. In order to predict qualitative values, such as whether a patient survives or dies, or whether the stock market in- creases or decreases, Fisher proposed linear discriminant analysis in 1936. In the 1940’s, various authors put forth an alternative approach, logistic regression. In the early 1970’s, Nelder and Wedderburn coined the term generalized linear models for an entire class of statistical learning methods that include both linear and logistic regression as special cases.
By the end of the 1970’s, many more techniques for learning from data were available. However, they were almost exclusively linear methods, be- cause fitting non-linear relationships was computationally infeasible at the time. By the 1980’s, computing technology had finally improved sufficiently that non-linear methods were no longer computationally prohibitive. In mid 1980’s Breiman, Friedman, Olshen and Stone introduced classification and regression trees, and were among the first to demonstrate the power of a detailed practical implementation of a method, including cross-validation for model selection. Hastie and Tibshirani coined the term generalized addi- tive models in 1986 for a class of non-linear extensions to generalized linear models, and also provided a practical software implementation.

## Regresion a la media <a name="reg"></a>
Aca va Galton

## T de Student <a name="t"></a>

<blockquote>
« Aunque es bien sabido que el método de usar la curva normal solo es confiable cuando la muestra es "grande", nadie nos ha dicho todavía muy claramente dónde debe trazarse el límite entre muestras "grandes" y "pequeñas" » Student, 1908
</blockquote>

Quiero decir que: cualquiera iniciando un curso en Estadística se topa con los test de media,
y se pregunta que por qué a veces usamos la T y a veces la Normal; y quién decide el tamaño de la muestra -- estas preguntas se las hizo Student hace 100 años, y al resolverla inventó una distribución, la T de Student. "Student" fue el pseudónimo que usó, ya que en Guinness no le dejaron usar su nombre, por medio a que la competencia usara sus análisis para su producción.

Las publicaciones de Student fueron valoradas por Fisher, con quien intercambió correspondencia en más de 150 cartas.

Again, although it is well known that the method of using the normal curve is only trustworthy when the sample is
"large," no one has yet told us very clearly where the limit between "l large " and " small " samples is to be drawn.

Esto es un hito también, desde el 3500 a.C. la estadística estuvo asociada a censos poblacionales, desde el siglo 19 a la astronomía y geodesia; pero a partir de T de Student, la teoría estadística se pone al servicio de la industria y de experimentos a pequeña escala.

## La era Fisher

<img src="/images/fisher_laurel.jpg" class=rightimg>


Fisher creó, casi que solo, las bases para la estadística moderna. Él fue el primero en hablar de Variancia, inventó el análisis de la variancia (ANOVA), el test de significancia! la hipótesis nula, la alternativa; la distribución y el test F, con su nombre, como el text exacto de Fisher, el modelo de efectos aleatorios, junto a otros 30 artículos de wikipedia de test, definiciones y métidos que llevan su nombre. Así que no es exagerado el título de "Estadístico del siglo 20"

## R  <a name="r"></a>
El último de la lista es R, y acá lo dejo porque esto sucedió después de que yo naciera y ya me sentí vieja.

## Polémica

- Eugenics: Fisher y Pearson defendían la "supremacía" de la raza blanca.




-------------
Nightingale died 10 years afeter the 20th centurt began, on the verge of a modern era.
The 20th century not only opened up an impressive consolidation of the mathematical theory of
statistics and probability theory, but it also totally transformed the amounts and kinds of numerical information available in virtually every area of human endeavor.
The rise of numbers had carried the world to the brink of a new digital age of information.
