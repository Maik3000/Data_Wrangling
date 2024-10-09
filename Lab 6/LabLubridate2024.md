Lubridate Lab
================
Tepi
10/6/2024

## Laboratorio de la libreria Lubridate

``` r
# Instala los paquetes si no están instalados
if (!requireNamespace("dplyr", quietly = TRUE)) {
  install.packages("dplyr")
}
if (!requireNamespace("lubridate", quietly = TRUE)) {
  install.packages("lubridate")
}
if (!requireNamespace("nycflights13", quietly = TRUE)) {
  install.packages("nycflights13")
}

# Carga las librerías necesarias
library(dplyr)
library(lubridate)
library(nycflights13)
```

## Resuelva las siguientes preguntas:

### Ejercicio 1: Convertir columnas de hora en fecha-hora

Problema: Convierte las columnas dep_time (hora de salida) y arr_time
(hora de llegada) en objetos de tipo datetime usando make_datetime() de
lubridate. Recuerda que estas columnas están en formato militar (HHMM).

Ayuda: Investiga la funcion matematica de modulo de r.

``` r
flights_datetime <- flights %>%
  filter(!is.na(dep_time), !is.na(arr_time)) %>%  
  mutate(
    dep_hour = dep_time %/% 100,   
    dep_min = dep_time %% 100,     
    arr_hour = arr_time %/% 100,   
    arr_min = arr_time %% 100      
  )

flights_datetime <- flights_datetime %>%
  mutate(
    dep_datetime = make_datetime(year, month, day, dep_hour, dep_min),
    arr_datetime = make_datetime(year, month, day, arr_hour, arr_min)
  )

flights_datetime %>%
  select(dep_datetime, arr_datetime) %>%
  head()
```

    ## # A tibble: 6 × 2
    ##   dep_datetime        arr_datetime       
    ##   <dttm>              <dttm>             
    ## 1 2013-01-01 05:17:00 2013-01-01 08:30:00
    ## 2 2013-01-01 05:33:00 2013-01-01 08:50:00
    ## 3 2013-01-01 05:42:00 2013-01-01 09:23:00
    ## 4 2013-01-01 05:44:00 2013-01-01 10:04:00
    ## 5 2013-01-01 05:54:00 2013-01-01 08:12:00
    ## 6 2013-01-01 05:54:00 2013-01-01 07:40:00

## Ejercicio 2: Duracion del vuelo

Calcula el tiempo de vuelo total en minutos entre las columnas dep_time
y arr_time que calculaste en el primer Ejercicio.

``` r
flights_datetime <- flights_datetime %>%
  mutate(
    flight_duration = as.numeric(difftime(arr_datetime, dep_datetime, units = "mins"))
  )

flights_datetime %>%
  select(dep_datetime, arr_datetime, flight_duration) %>%
  head()
```

    ## # A tibble: 6 × 3
    ##   dep_datetime        arr_datetime        flight_duration
    ##   <dttm>              <dttm>                        <dbl>
    ## 1 2013-01-01 05:17:00 2013-01-01 08:30:00             193
    ## 2 2013-01-01 05:33:00 2013-01-01 08:50:00             197
    ## 3 2013-01-01 05:42:00 2013-01-01 09:23:00             221
    ## 4 2013-01-01 05:44:00 2013-01-01 10:04:00             260
    ## 5 2013-01-01 05:54:00 2013-01-01 08:12:00             138
    ## 6 2013-01-01 05:54:00 2013-01-01 07:40:00             106

## Ejercicio 3: Extraer componentes de fechas

Extrae el dia de la semana y la hora en que salieron los aviones y
guardalos en las variables `dep_day_of_week` y `dep_hour`.

``` r
flights_datetime <- flights_datetime %>%
  mutate(
    dep_day_of_week = wday(dep_datetime, label = TRUE, abbr = TRUE),  
    dep_hour = hour(dep_datetime)                                     
  )

flights_datetime %>%
  select(dep_datetime, dep_day_of_week, dep_hour) %>%
  head()
```

    ## # A tibble: 6 × 3
    ##   dep_datetime        dep_day_of_week dep_hour
    ##   <dttm>              <ord>              <int>
    ## 1 2013-01-01 05:17:00 "mar\\."               5
    ## 2 2013-01-01 05:33:00 "mar\\."               5
    ## 3 2013-01-01 05:42:00 "mar\\."               5
    ## 4 2013-01-01 05:44:00 "mar\\."               5
    ## 5 2013-01-01 05:54:00 "mar\\."               5
    ## 6 2013-01-01 05:54:00 "mar\\."               5

## Ejercicio 4: Crear nuevas columnas con el día de la semana y la semana del año

Problema: Usando la columna `time_hour`, crea una nueva columna que
indique el día de la semana y otra que indique la semana del año en la
que ocurrió el vuelo.

Ayuda: Invesitga la funcion wday de lubridate.

``` r
flights_new <- flights %>%
  mutate(
    day_of_week = wday(time_hour, label = TRUE, abbr = TRUE),  
    week_of_year = week(time_hour)                             
  )

flights_new %>%
  select(time_hour, day_of_week, week_of_year) %>%
  head()
```

    ## # A tibble: 6 × 3
    ##   time_hour           day_of_week week_of_year
    ##   <dttm>              <ord>              <dbl>
    ## 1 2013-01-01 05:00:00 "mar\\."               1
    ## 2 2013-01-01 05:00:00 "mar\\."               1
    ## 3 2013-01-01 05:00:00 "mar\\."               1
    ## 4 2013-01-01 05:00:00 "mar\\."               1
    ## 5 2013-01-01 06:00:00 "mar\\."               1
    ## 6 2013-01-01 05:00:00 "mar\\."               1

## Ejercicio 5: Encontrar los vuelos que salieron los fines de semana

Problema: Filtra los vuelos que despegaron un sábado o domingo y
devuelve el total de vuelos en fines de semana.

``` r
weekend_flights <- flights %>%
  mutate(day_of_week = wday(time_hour, week_start = 1)) %>%  
  filter(day_of_week %in% c(6, 7))

total_weekend_flights <- weekend_flights %>%
  summarise(total_flights = n())

total_weekend_flights
```

    ## # A tibble: 1 × 1
    ##   total_flights
    ##           <int>
    ## 1         85077
