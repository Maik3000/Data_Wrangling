dw-2024-parcial-1
================
Tepi
25/09/2024

# Examen parcial

Indicaciones generales:

- Usted tiene el período de la clase para resolver el examen parcial.

- La entrega del parcial, al igual que las tareas, es por medio de su
  cuenta de github, pegando el link en el portal de MiU.

- Pueden hacer uso del material del curso e internet (stackoverflow,
  etc.). Sin embargo, si encontramos algún indicio de copia, se anulará
  el exámen para los estudiantes involucrados. Por lo tanto, aconsejamos
  no compartir las agregaciones que generen.

## Sección 0: Preguntas de temas vistos en clase (20pts)

- Responda las siguientes preguntas de temas que fueron tocados en
  clase.

1.  ¿Qué es una ufunc y por qué debemos de utilizarlas cuando
    programamos trabajando datos?

R/: es una funcion universal que se utiliza en los elementos de un
vector para no usar loops. Esto es muy util al momento de trabajar con
datasets grandes, ya que esto eliminara cierta complejidad.

2.  Es una técnica en programación numérica que amplía los objetos que
    son de menor dimensión para que sean compatibles con los de mayor
    dimensión. Describa cuál es la técnica y de un breve ejemplo en R.

R/: Broadcasting. Esta es una tecnica que permite realizar operaciones
entre vectores de diferentes dimensiones. Los elementos del vector más
pequeño se expanden automáticamente para ser compatibles con el array
más grande.

ejemplo en R, si tenemos 2 vectores de diferente tamaño : x \<- c(1,2) y
\<- c(1,2,3,4,5)

Lo que hace R es reciclar el vector x de la siguiente manera
c(1,2,1,2,1) z \<- x + y

print(z) El resultado seria: 2,4,4,6,6

3.  ¿Qué es el axioma de elegibilidad y por qué es útil al momento de
    hacer análisis de datos?

R/: Es un axioma que establece que se pueden tomar desiciones dentro de
un dataset solo si existe cierto conjunto de criterios claros, es util
para evitar cualquier sesgo y facilitar el proceso de analisis.

4.  Cuál es la relación entre la granularidad y la agregación de datos?
    Mencione un breve ejemplo. Luego, exploque cuál es la granularidad o
    agregación requerida para poder generar un reporte como el
    siguiente:

| Pais | Usuarios |
|------|----------|
| US   | 1,934    |
| UK   | 2,133    |
| DE   | 1,234    |
| FR   | 4,332    |
| ROW  | 943      |

R/: La granularidad se refiere al nivel de detalle con el que se
representan los datos, por consiguiente la relacion con la agregacion de
datos da como resultado una disminucion de la granularidad.

Ejemplo: si tenemos datos de ventas por municipios y se decide
cambiarlas a nivel de departamentos. En este caso se ha reducido la
granularidad de municipio a departamento al sumar todas las ventas de
todos los municipios dentro de cada departamento.

Para poder generar un reporte como el anterior la granularidad de la
data deberia estar relacionada a un nivel alto como lo es el pais ya que
la data esta agrupada por pais.

## Sección I: Preguntas teóricas. (50pts)

- Existen 10 preguntas directas en este Rmarkdown, de las cuales usted
  deberá responder 5. Las 5 a responder estarán determinadas por un
  muestreo aleatorio basado en su número de carné.

- Ingrese su número de carné en `set.seed()` y corra el chunk de R para
  determinar cuáles preguntas debe responder.

``` r
#set.seed("20210273") 
v<- 1:10
preguntas <-sort(sample(v, size = 5, replace = FALSE ))

paste0("Mis preguntas a resolver son: ",paste0(preguntas,collapse = ", "))
```

    ## [1] "Mis preguntas a resolver son: 2, 4, 5, 6, 8"

\[1\] “Mis preguntas a resolver son: 2, 4, 5, 6, 8”

### Listado de preguntas teóricas

2.  Al momento de filtrar en SQL, ¿cuál keyword cumple las mismas
    funciones que el keyword `OR` para filtrar uno o más elementos una
    misma columna?

R/: El keyword que podria cumplir con dichas funciones seria IN ya que
asume que uno o mas valores deben estar dentro de un conjunto de valores

4.  ¿Cuál es la diferencia entre utilizar `==` y `=` en R?

R/: ‘==’ sirve para hacer una comparacion entre dos valores, mientras
que ‘=’ solo asigna valor

5.  ¿Cuál es la forma correcta de cargar un archivo de texto donde el
    delimitador es `:`?

R/: La forma correcta es usando la funcion read.csv()

6.  ¿Qué es un vector y en qué se diferencia en una lista en R?

R/: Un vector es una secuencia de datos que almacena datos del mismo
tipo, mientras que una lista almacena datos de diferente tipo

8.  Si en un dataframe, a una variable de tipo `factor` le agrego un
    nuevo elemento que *no se encuentra en los niveles existentes*,
    ¿cuál sería el resultado esperado y por qué?
    - El nuevo elemento
    - `NA`

R/: Asignaria NA al nuevo elemento ya que al ser una variable categorica
tiene un conjunto de niveles, R no podria manejarlo porque no se ha
definido dicho nivel.

Extra: ¿Cuántos posibles exámenes de 5 preguntas se pueden realizar
utilizando como banco las diez acá presentadas? (responder con código de
R.)

``` r
examenes_posibles <- choose(10,5)
examenes_posibles
```

    ## [1] 252

## Sección II Preguntas prácticas. (30pts)

- Conteste las siguientes preguntas utilizando sus conocimientos de R.
  Adjunte el código que utilizó para llegar a sus conclusiones en un
  chunk del markdown.

A. De los clientes que están en más de un país,¿cuál cree que es el más
rentable y por qué?

R/: El cliente mas rentable es a17a7558

B. Estrategia de negocio ha decidido que ya no operará en aquellos
territorios cuyas pérdidas sean “considerables”. Bajo su criterio,
¿cuáles son estos territorios y por qué ya no debemos operar ahí?

R/: Ya no se deberia contar con el territorio f7dfc635 ya que es el que
mas perdidas representa

## A

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
data <- readRDS("parcial_anonimo.rds")
str(data)
```

    ## Classes 'data.table' and 'data.frame':   226236 obs. of  11 variables:
    ##  $ DATE           : POSIXct, format: "2018-12-01" "2018-11-01" ...
    ##  $ Codigo Material: chr  "637caff5" "637caff5" "637caff5" "637caff5" ...
    ##  $ Descripcion    : chr  "0cf3ec3d" "0cf3ec3d" "0cf3ec3d" "0cf3ec3d" ...
    ##  $ Pais           : chr  "4046ee34" "4046ee34" "4046ee34" "4046ee34" ...
    ##  $ Distribuidor   : chr  "9a47575c" "9a47575c" "9a47575c" "9a47575c" ...
    ##  $ Territorio     : chr  "69c1b705" "69c1b705" "69c1b705" "69c1b705" ...
    ##  $ Cliente        : chr  "9d6e1d65" "9d6e1d65" "9d6e1d65" "9d6e1d65" ...
    ##  $ Marca          : chr  "61d7fbfc" "61d7fbfc" "61d7fbfc" "61d7fbfc" ...
    ##  $ Canal Venta    : chr  "7b48292e" "7b48292e" "7b48292e" "7b48292e" ...
    ##  $ Unidades plaza : num  2 0 3 3 8 3 0 3 6 10 ...
    ##  $ Venta          : num  26.5 0 39.8 39.8 106 ...
    ##  - attr(*, ".internal.selfref")=<externalptr>

``` r
#encontrar los clientes que estan en mas de un pais y ver las ganancias totales
cliente_multipais_rentabilidad <- data %>%
  group_by(Cliente) %>%
  filter(n_distinct(Pais) > 1) %>%
  summarise(ganancias_totales = sum(Venta), .groups = 'drop') %>%
  arrange(desc(ganancias_totales))

#cliente mas rentable
cliente_mas_rentable <- head(cliente_multipais_rentabilidad, 1)
cliente_mas_rentable
```

    ## # A tibble: 1 × 2
    ##   Cliente  ganancias_totales
    ##   <chr>                <dbl>
    ## 1 a17a7558            19818.

## B

``` r
#filtrar ventas negativas y suma de ventas negativas por territorio
territorios_con_ventas_negativas <- data %>%
  filter(Venta < 0) %>%
  group_by(Territorio) %>%
  summarise(Suma_Ventas_Negativas = sum(Venta), .groups = 'drop')

#Encontrar el territorio con suma de ventsa negativas mas baja
territorio_con_ventas_negativas <- territorios_con_ventas_negativas %>%
  slice_min(order_by = Suma_Ventas_Negativas)

territorio_con_ventas_negativas
```

    ## # A tibble: 1 × 2
    ##   Territorio Suma_Ventas_Negativas
    ##   <chr>                      <dbl>
    ## 1 f7dfc635                 -14985.
