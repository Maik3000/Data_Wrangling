Lab3
================
Mayco
2024-09-05

``` r
#imports
actors <- read_csv("actors.csv")
directors_genres <- read_csv("directors_genres.csv")
directors <- read_csv("directors.csv")
movies_directors <- read_csv("movies_directors.csv")
movies_genres <- read_csv("movies_genres.csv")
movies <- read_csv("movies.csv")
rol <- read_csv("roles.csv")
```

``` r
#PROBLEMA 1
#Numero de pelis
num_peliculas <- movies %>%
  summarise(total_peliculas = n_distinct(id))
num_peliculas
```

    ## # A tibble: 1 × 1
    ##   total_peliculas
    ##             <int>
    ## 1          388269

``` r
#numero directores
num_directores <- directors %>%
  summarise(total_directores = n_distinct(id))

num_directores
```

    ## # A tibble: 1 × 1
    ##   total_directores
    ##              <int>
    ## 1            86880

``` r
#PROBLEMA 2
#Numero promedio de generos por director
generos_por_director <- directors_genres %>%
  group_by(director_id) %>%
  summarise(num_generos = n_distinct(genre))

promedio_generos_por_director <- generos_por_director %>%
  summarise(promedio_generos = mean(num_generos))

promedio_generos_por_director
```

    ## # A tibble: 1 × 1
    ##   promedio_generos
    ##              <dbl>
    ## 1             2.41

``` r
#PROBLEMA 3

#nuevo reporte por Role

# Calcular el número de películas por rol
peliculas_por_rol <- rol %>%
  group_by(role) %>%
  summarise(num_peliculas = n_distinct(movie_id))

#actores x rol
actores_por_rol <- rol %>%
  left_join(actors, by = c("actor_id" = "id")) %>%
  filter(gender == 'M') %>%  
  group_by(role) %>%
  summarise(num_actores = n_distinct(actor_id))

# numero de actrices  por rol
actrices_por_rol <- rol %>%
  left_join(actors, by = c("actor_id" = "id")) %>%
  filter(gender == 'F') %>%  
  group_by(role) %>%
  summarise(num_actrices = n_distinct(actor_id))

# directores por rol
directores_por_rol <- rol %>%
  left_join(movies_directors, by = "movie_id") %>% 
  group_by(role) %>%   
  summarise(num_directores = n_distinct(director_id))

reporte_por_rol <- peliculas_por_rol %>%
  left_join(actores_por_rol, by = "role") %>%
  left_join(actrices_por_rol, by = "role") %>%
  left_join(directores_por_rol, by = "role")

# Llenamos los valores NA con 0
reporte_por_rol <- reporte_por_rol %>%
  mutate(
    num_actores = ifelse(is.na(num_actores), 0, num_actores),
    num_actrices = ifelse(is.na(num_actrices), 0, num_actrices),
    num_directores = ifelse(is.na(num_directores), 0, num_directores)
  )

reporte_por_rol
```

    ## # A tibble: 1,174,613 × 5
    ##    role                    num_peliculas num_actores num_actrices num_directores
    ##    <chr>                           <int>       <dbl>        <dbl>          <int>
    ##  1 "\"Abe Lincoln\""                   1           1            0              1
    ##  2 "\"American Eagle\""                1           1            0              1
    ##  3 "\"Astoria\" Owner"                 1           0            1              1
    ##  4 "\"Bar From Hell\" Pat…             1           1            0              1
    ##  5 "\"Bellucci\" Grabowsk…             1           1            0              3
    ##  6 "\"Betsy Ross\""                    1           0            1              1
    ##  7 "\"Betty Bumcakes\" Ac…             1           1            0              1
    ##  8 "\"Betty Bumcakes\" Bo…             1           1            0              1
    ##  9 "\"Betty Bumcakes\" DP"             1           1            0              1
    ## 10 "\"Betty Bumcakes\" Di…             1           1            0              1
    ## # ℹ 1,174,603 more rows

``` r
#PROBLEMA 4

#Nuevo reporte por director

# peliculas que ha dirigido
peliculas_por_director <- movies_directors %>%
  group_by(director_id) %>%
  summarise(num_peliculas = n_distinct(movie_id))

# actores que han trabajado con el director
actores_por_director <- movies_directors %>%
  left_join(rol, by = "movie_id") %>%
  group_by(director_id) %>%
  summarise(num_actores = n_distinct(actor_id))

# género más común por director
genero_comun_por_director <- movies_directors %>%
  left_join(movies_genres, by = "movie_id") %>%
  group_by(director_id, genre) %>%
  summarise(num_peliculas_genero = n()) %>%
  slice_max(num_peliculas_genero, n = 1, with_ties = FALSE) %>% #evita duplicados
  ungroup() %>%
  select(director_id, genre)

# Unir todo
reporte_directores <- directors %>%
  left_join(peliculas_por_director, by = c("id" = "director_id")) %>%
  left_join(actores_por_director, by = c("id" = "director_id")) %>%
  left_join(genero_comun_por_director, by = c("id" = "director_id"))

reporte_directores
```

    ## # A tibble: 86,880 × 6
    ##       id first_name         last_name   num_peliculas num_actores genre      
    ##    <dbl> <chr>              <chr>               <int>       <int> <chr>      
    ##  1     1 Todd               1                       1           1 <NA>       
    ##  2     2 Les                12 Poissons             1           2 Short      
    ##  3     3 Lejaren            a'Hiller                2          15 Drama      
    ##  4     4 Nian               A                       1           1 <NA>       
    ##  5     5 Khairiya           A-Mansour               1           1 Documentary
    ##  6     6 Ricardo            A. Solla                1           3 Drama      
    ##  7     8 Kodanda Rami Reddy A.                     35          86 Action     
    ##  8     9 Nageswara Rao      A.                      1           1 <NA>       
    ##  9    10 Yuri               A.                      1           1 Comedy     
    ## 10    11 Swamy              A.S.A.                  1           2 Drama      
    ## # ℹ 86,870 more rows

``` r
#PROBLEMA 5

#Distribución de “Roles” por Pelicula 
hist_role_movie <- movies %>% 
  left_join(rol, by = c("id" = "movie_id")) %>% ##hacemos un join que une movies con roles por movie ID
  group_by(id) %>% ##primera agrupación para entender el número de roles por película
  summarise(n_roles = n_distinct(role)) %>% 
  ungroup() %>% ##utilizar esto cuando hago más de una agrupación o luego de usar un group_by.
  group_by(n_roles) %>% ##segunda agrupación para entender el número de películas por el número de roles
  summarise(n_movies = n()) %>% 
  arrange(n_roles)


hist_role_movie %>% 
  mutate(
    ratio_pct = round(100.0*n_movies/sum(n_movies),1), ##esto solo encuentra el % del total.
    ratio_cumulative = cumsum(ratio_pct) ##esto suma los porcentajes para ver el % acumulado.
  ) %>% 
  head(15)
```

    ## # A tibble: 15 × 4
    ##    n_roles n_movies ratio_pct ratio_cumulative
    ##      <int>    <int>     <dbl>            <dbl>
    ##  1       1   200569      51.7             51.7
    ##  2       2    26293       6.8             58.5
    ##  3       3    15283       3.9             62.4
    ##  4       4    11835       3               65.4
    ##  5       5    11509       3               68.4
    ##  6       6    10477       2.7             71.1
    ##  7       7    10043       2.6             73.7
    ##  8       8     9434       2.4             76.1
    ##  9       9     8724       2.2             78.3
    ## 10      10     8043       2.1             80.4
    ## 11      11     6953       1.8             82.2
    ## 12      12     6103       1.6             83.8
    ## 13      13     5381       1.4             85.2
    ## 14      14     4764       1.2             86.4
    ## 15      15     4451       1.1             87.5

``` r
# Distribución de “Roles” por Director

# Calculamos la distribución de roles por director
hist_role_director <- movies_directors %>%
  left_join(rol, by = "movie_id") %>%  
  group_by(director_id) %>%            
  summarise(n_roles = n_distinct(role)) %>%  
  ungroup() %>%
  group_by(n_roles) %>%                
  summarise(n_directors = n()) %>%      
  arrange(n_roles)                     

# Calculamos el porcentaje y el porcentaje acumulado
hist_role_director <- hist_role_director %>%
  mutate(
    ratio_pct = round(100.0 * n_directors / sum(n_directors), 1),   
    ratio_cumulative = cumsum(ratio_pct)                            
  )

# Imprimimos los primeros 15 registros
print(hist_role_director %>% head(15))
```

    ## # A tibble: 15 × 4
    ##    n_roles n_directors ratio_pct ratio_cumulative
    ##      <int>       <int>     <dbl>            <dbl>
    ##  1       1       29330      34.2             34.2
    ##  2       2        6196       7.2             41.4
    ##  3       3        4285       5               46.4
    ##  4       4        3203       3.7             50.1
    ##  5       5        2849       3.3             53.4
    ##  6       6        2444       2.8             56.2
    ##  7       7        1965       2.3             58.5
    ##  8       8        1767       2.1             60.6
    ##  9       9        1696       2               62.6
    ## 10      10        1679       2               64.6
    ## 11      11        1393       1.6             66.2
    ## 12      12        1255       1.5             67.7
    ## 13      13        1175       1.4             69.1
    ## 14      14        1031       1.2             70.3
    ## 15      15         983       1.1             71.4
