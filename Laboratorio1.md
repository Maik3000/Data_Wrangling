Lab1
================
Mayco
2024-08-03

``` r
knitr::opts_chunk$set(echo = TRUE)
library(dplyr)
library(tidyr)
library(readr)
library(readxl)
```

``` r
#Problema1
data1 <- read_excel("01-2023.xlsx")
data2 <- read_excel("02-2023.xlsx")
data3 <- read_excel("03-2023.xlsx")
data4 <- read_excel("04-2023.xlsx")
data5 <- read_excel("05-2023.xlsx")
data6 <- read_excel("06-2023.xlsx")
data7 <- read_excel("07-2023.xlsx")
data8 <- read_excel("08-2023.xlsx")
data9 <- read_excel("09-2023.xlsx")
data10 <- read_excel("10-2023.xlsx")
data11 <- read_excel("11-2023.xlsx")
```

``` r
file_names <- c("01-2023.xlsx", "02-2023.xlsx", "03-2023.xlsx", 
                "04-2023.xlsx", "05-2023.xlsx", "06-2023.xlsx", 
                "07-2023.xlsx", "08-2023.xlsx", "09-2023.xlsx", 
                "10-2023.xlsx", "11-2023.xlsx")
lista_data <- list(data1, data2, data3, data4, data5, data6, data7, data8, data9, data10, data11)

#Agregar una columna adicional que identifique al mes y año
lista_data <- mapply(function(data, file) {
  fecha <- sub(".xlsx", "", file)
  data <- data %>% mutate(Fecha = fecha)
  return(data)
}, lista_data, file_names, SIMPLIFY = FALSE)
```

``` r
#Unificar todos los archivos
datosagrupados <- bind_rows(lista_data)
datosagrupados <- datosagrupados %>% select(-TIPO, -...10)
```

``` r
#Exportar ese archivo en formato csv o Excel.
write_csv(datosagrupados, "datosagrupados.csv")
```

``` r
#Problema 2
moda <- function(v) {
  uniqv <- unique(v)
  uniqv[which.max(tabulate(match(v, uniqv)))]
}
lista_vectores <- list(
  c(1, 2, 2, 3, 4),
  c(5, 6, 7, 7, 7, 8),
  c(9, 10, 10, 11, 11, 11)
)

modas <- lapply(lista_vectores, moda)
modas
```

    ## [[1]]
    ## [1] 2
    ## 
    ## [[2]]
    ## [1] 7
    ## 
    ## [[3]]
    ## [1] 11

``` r
#Problema 3
datos_txt <- read_delim("INE_PARQUE_VEHICULAR_080219.txt", delim = ",")
```

    ## Warning: One or more parsing issues, call `problems()` on your data frame for details,
    ## e.g.:
    ##   dat <- vroom(...)
    ##   problems(dat)

    ## Rows: 2435294 Columns: 1
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): ANIO_ALZA|MES|NOMBRE_DEPARTAMENTO|NOMBRE_MUNICIPIO|MODELO_VEHICULO|...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
