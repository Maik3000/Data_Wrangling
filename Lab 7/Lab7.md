Lab7
================
Mayco
2024-10-20

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
library(tm)
```

    ## Loading required package: NLP

``` r
library(wordcloud)
```

    ## Loading required package: RColorBrewer

``` r
library(ggplot2)
```

    ## 
    ## Attaching package: 'ggplot2'

    ## The following object is masked from 'package:NLP':
    ## 
    ##     annotate

``` r
library(slam)
```

``` r
# Cargar los datasets
metadata <- read.csv('Health_and_Personal_Care_metadata.csv', stringsAsFactors = FALSE)
reviews <- read.csv('Health_and_Personal_Care.csv', stringsAsFactors = FALSE)
```

## 1. Cuántos productos contienen reviews con las palabras “love”, “recommend” y “enjoy”?

``` r
keywords <- c('love', 'recommend', 'enjoy')

filtered_reviews <- reviews %>%
  filter(grepl(paste(keywords, collapse = '|'), text, ignore.case = TRUE))

unique_products <- n_distinct(filtered_reviews$parent_id)

cat('Número de productos con reviews que contienen las palabras "love", "recommend", o "enjoy":', unique_products, '\n')
```

    ## Número de productos con reviews que contienen las palabras "love", "recommend", o "enjoy": 25424

## 2. De los reviews de la pregunta 1, encuentre el top 5 de las tiendas que los venden?

``` r
filtered_reviews_metadata <- filtered_reviews %>%
  inner_join(metadata, by = 'parent_id')

top_stores <- filtered_reviews_metadata %>%
  group_by(store) %>%
  summarise(product_count = n_distinct(parent_id)) %>%
  arrange(desc(product_count)) %>%
  slice(1:5)  

cat('Top 5 tiendas por productos con reviews filtrados:\n')
```

    ## Top 5 tiendas por productos con reviews filtrados:

``` r
print(top_stores)
```

    ## # A tibble: 5 × 2
    ##   store       product_count
    ##   <chr>               <int>
    ## 1 ""                    914
    ## 2 "HAARBB"              212
    ## 3 "Eyekepper"           173
    ## 4 "Generic"              84
    ## 5 "Glade"                58

## 3. Genere un wordcloud sin stopwords de los reviews de la pregunta 1.

``` r
reviews_corpus <- Corpus(VectorSource(filtered_reviews$text))

reviews_corpus <- reviews_corpus %>%
  tm_map(content_transformer(tolower)) %>%
  tm_map(removePunctuation) %>%
  tm_map(removeNumbers) %>%
  tm_map(removeWords, stopwords("en")) %>%
  tm_map(stripWhitespace)
```

    ## Warning in tm_map.SimpleCorpus(., content_transformer(tolower)): transformation
    ## drops documents

    ## Warning in tm_map.SimpleCorpus(., removePunctuation): transformation drops
    ## documents

    ## Warning in tm_map.SimpleCorpus(., removeNumbers): transformation drops
    ## documents

    ## Warning in tm_map.SimpleCorpus(., removeWords, stopwords("en")): transformation
    ## drops documents

    ## Warning in tm_map.SimpleCorpus(., stripWhitespace): transformation drops
    ## documents

``` r
wordcloud(reviews_corpus, max.words = 100, random.order = FALSE, colors = brewer.pal(8, "Dark2"))
```

![](Lab7_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

## 4. Generar un wordcloud de los reviews de las 5 tiendas encontradas en la pregunta 2. Deberá de incluir todos los reviews de esas 5 tiendas.

``` r
top_stores_list <- top_stores$store

top_stores_reviews <- filtered_reviews_metadata %>%
  filter(store %in% top_stores_list)

top_stores_corpus <- Corpus(VectorSource(top_stores_reviews$text))

top_stores_corpus <- top_stores_corpus %>%
  tm_map(content_transformer(tolower)) %>%
  tm_map(removePunctuation) %>%
  tm_map(removeNumbers) %>%
  tm_map(removeWords, stopwords("en")) %>%
  tm_map(stripWhitespace)
```

    ## Warning in tm_map.SimpleCorpus(., content_transformer(tolower)): transformation
    ## drops documents

    ## Warning in tm_map.SimpleCorpus(., removePunctuation): transformation drops
    ## documents

    ## Warning in tm_map.SimpleCorpus(., removeNumbers): transformation drops
    ## documents

    ## Warning in tm_map.SimpleCorpus(., removeWords, stopwords("en")): transformation
    ## drops documents

    ## Warning in tm_map.SimpleCorpus(., stripWhitespace): transformation drops
    ## documents

``` r
wordcloud(top_stores_corpus, max.words = 100, random.order = FALSE, scale = c(2, 0.5), colors = brewer.pal(8, "Set1"))
```

![](Lab7_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

## 5. Cuáles son las 25 palabras más frecuentes de los reviews?

``` r
dtm <- TermDocumentMatrix(reviews_corpus)

word_freqs <- slam::row_sums(dtm)
word_freqs <- sort(word_freqs, decreasing = TRUE)

word_freqs_df <- data.frame(word = names(word_freqs), freq = word_freqs)

top_25_words <- head(word_freqs_df, 25)

cat('Top 25 palabras más frecuentes:\n')
```

    ## Top 25 palabras más frecuentes:

``` r
print(top_25_words)
```

    ##                word  freq
    ## love           love 53659
    ## product     product 28816
    ## recommend recommend 25083
    ## great         great 22508
    ## use             use 22097
    ## one             one 20063
    ## like           like 19440
    ## just           just 16578
    ## can             can 15130
    ## will           will 14014
    ## really       really 13708
    ## get             get 13318
    ## well           well 13295
    ## good           good 13180
    ## time           time 10787
    ## also           also 10661
    ## used           used 10459
    ## much           much 10054
    ## using         using 10041
    ## easy           easy  9633
    ## highly       highly  9452
    ## works         works  9008
    ## little       little  8779
    ## work           work  8316
    ## dont           dont  7826

``` r
ggplot(top_25_words, aes(x = reorder(word, freq), y = freq)) +
  geom_bar(stat = "identity", fill = "steelblue") +
  coord_flip() +
  labs(title = 'Top 25 Palabras Más Frecuentes en los Reviews',
       x = 'Palabra',
       y = 'Frecuencia') +
  theme_minimal()
```

![](Lab7_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->
