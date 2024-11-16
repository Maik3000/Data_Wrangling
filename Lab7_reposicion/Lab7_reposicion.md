lab7 reposicion
================
Mayco
2024-11-16

## 1) Cuántos productos contienen reviews con las palabras “love”, “recommend” y “enjoy”?

``` r
words <- c("love", "recommend", "enjoy" )
reviews_filtrados <- na.omit(reviews)

reviews_filtrados <- reviews %>% 
  filter(str_detect(tolower(text), paste(words, collapse = "|")))

num_productos <- reviews_filtrados %>% 
  distinct(product_id) %>% 
  count()

num_productos
```

    ## # A tibble: 1 × 1
    ##       n
    ##   <int>
    ## 1 26509

## 2. De los reviews de la pregunta 1, encuentre el top 5 de las tiendas que los venden?

``` r
reviews_tiendas <- reviews_filtrados %>%
  left_join(metadata, by = "parent_id")

top_5_tiendas <- reviews_tiendas %>%
  filter(!is.na(store)) %>%       
  group_by(store) %>%  
  summarise(count = n()) %>%
  arrange(desc(count)) %>%
  slice(1:5)

top_5_tiendas
```

    ## # A tibble: 5 × 2
    ##   store      count
    ##   <chr>      <int>
    ## 1 ASUTRA      1271
    ## 2 US Organic   669
    ## 3 Purple       582
    ## 4 JUNP         580
    ## 5 Fitbit       545

## 3. Genere un wordcloud sin stopwords de los reviews de la pregunta 1.

``` r
#procesar el texto en palabras
frecuencias <- reviews_filtrados %>%
  unnest_tokens(word, text) %>%
  anti_join(stop_words, by = "word") %>%
  filter(word != "br") %>%  
  count(word, sort = TRUE)

# Wordcloud
set.seed(123)
wordcloud(
  words = frecuencias$word,
  freq = frecuencias$n,
  min.freq = 2,
  max.words = 100,
  random.order = FALSE,
  colors = brewer.pal(8, "Set1")
)
```

![](Lab7_reposicion_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

## 4. Wordcloud de los reviews de las 5 tiendas encontradas en la pregunta 2.

``` r
frecuencias_top <- merge(reviews, metadata[, c("parent_id", "store")], by = "parent_id", all.x = TRUE) %>%
  filter(store %in% top_5_tiendas$store) %>%
  mutate(text = as.character(text)) %>%

  unnest_tokens(word, text) %>%
  anti_join(stop_words, by = "word") %>%
  filter(word != "br") %>%  

  count(word, sort = TRUE)

# Generar el wordcloud
set.seed(123)
wordcloud(words = frecuencias_top$word,
          freq = frecuencias_top$n,
          min.freq = 2,
          max.words = 100,
          random.order = FALSE,
          colors = brewer.pal(8, "Set1"))
```

![](Lab7_reposicion_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

## 5) 25 palabras más frecuentes de los reviews

``` r
top_25_palabras <- reviews %>%
  unnest_tokens(word, text) %>%
  anti_join(stop_words, by = "word") %>%
  count(word, sort = TRUE) %>%
  filter(word != "br") %>% 
  slice_head(n = 25)

top_25_palabras
```

    ## # A tibble: 25 × 2
    ##    word           n
    ##    <chr>      <int>
    ##  1 product   105919
    ##  2 love       53987
    ##  3 time       40077
    ##  4 easy       35437
    ##  5 price      27180
    ##  6 nice       26551
    ##  7 bought     26309
    ##  8 recommend  25255
    ##  9 quality    24684
    ## 10 day        24556
    ## # ℹ 15 more rows
