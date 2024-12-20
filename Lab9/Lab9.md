Missing Data and Feature Engineering
================
Mayco
2024-11-15

``` r
titanic_md <- read.csv("titanic_MD.csv", stringsAsFactors = FALSE)
titanic <- read.csv("titanic.csv", stringsAsFactors = FALSE)
```

### Parte 1

#### 1) Reporte de missing Values

``` r
missing_data_report <- sapply(titanic_md, function(x) sum(is.na(x) | x == "" | x== "?"))

missing_data_report
```

    ## PassengerId    Survived      Pclass        Name         Sex         Age 
    ##           0           0           0           0          51          25 
    ##       SibSp       Parch      Ticket        Fare       Cabin    Embarked 
    ##           3          12           0           8           0          12

#### 2) Tipo de modelo por columna

Sex: Imputación por proporciones. Ya que se dividieron los datos
faltantes en proporcion al porcentaje que hay de hombres y mujeres.

Age: Imputación por la mediana para evitar valores extremos que puedan
afectar el valor.

SibSp y Parch: Imputación por moda porque son variables discretas.

Embarked: Imputación por moda ya que son variables categoricas nominales
sin orden alguno.

Fare: Imputación por la moda, segmentada por clase ya que la tarifa
depende del tipo de clase.

#### 3) Reporte de filas completas

``` r
complete_rows_report <- titanic_md[complete.cases(titanic_md), ]
head(complete_rows_report)
```

    ##    PassengerId Survived Pclass
    ## 1            2        1      1
    ## 2            4        1      1
    ## 3            7        0      1
    ## 6           22        1      2
    ## 8           28        0      1
    ## 10          55        0      1
    ##                                                   Name    Sex Age SibSp Parch
    ## 1  Cumings, Mrs. John Bradley (Florence Briggs Thayer)      ?  38     1     0
    ## 2         Futrelle, Mrs. Jacques Heath (Lily May Peel) female  35     1     0
    ## 3                              McCarthy, Mr. Timothy J   male  54     0     0
    ## 6                                Beesley, Mr. Lawrence   male  34     0     0
    ## 8                       Fortune, Mr. Charles Alexander      ?  19     3     2
    ## 10                      Ostby, Mr. Engelhart Cornelius   male  65     0     1
    ##      Ticket     Fare       Cabin Embarked
    ## 1  PC 17599  71.2833         C85        C
    ## 2    113803  53.1000        C123        S
    ## 3     17463  51.8625         E46        S
    ## 6    248698  13.0000         D56        S
    ## 8     19950 263.0000 C23 C25 C27        S
    ## 10   113509  61.9792         B30        C

#### 4) Imputacion de Missing Values

``` r
#sustituir ? y " " por NA
titanic_md$Sex[titanic_md$Sex == "?"] <- NA
titanic_md$Embarked[titanic_md$Embarked == ""] <- NA
```

##### SEX:

``` r
#Imputacion general

get_mode <- function(x) {
  x <- na.omit(x)
  uniqx <- unique(x)
  uniqx[which.max(tabulate(match(x, uniqx)))]
}
sex_mode <- get_mode(titanic_md$Sex)

titanic_md$Sex_imputed <- titanic_md$Sex
titanic_md$Sex_imputed[is.na(titanic_md$Sex_imputed)] <- sex_mode

cat("Imputacion por moda:", sex_mode)
```

    ## Imputacion por moda: male

##### AGE:

``` r
# mEDIA
age_mean <- mean(titanic_md$Age, na.rm = TRUE)

titanic_md$Age_mean <- titanic_md$Age
titanic_md$Age_mean[is.na(titanic_md$Age_mean)] <- age_mean

# Imputacion con la mediana
age_median <- median(titanic_md$Age, na.rm = TRUE)

titanic_md$Age_median <- titanic_md$Age
titanic_md$Age_median[is.na(titanic_md$Age_median)] <- age_median

# Moda

age_mode <- get_mode(titanic_md$Age)

titanic_md$Age_mode <- titanic_md$Age
titanic_md$Age_mode[is.na(titanic_md$Age_mode)] <- age_mode
```

``` r
#Regresion Lineal

age_model_data <- titanic_md %>%
  filter(!is.na(Age), !is.na(Sex_imputed), !is.na(Pclass), !is.na(SibSp), !is.na(Parch), !is.na(Fare))

age_lm <- lm(Age ~ Pclass + Sex_imputed + SibSp + Parch + Fare, data = age_model_data)

titanic_md$Age_regression <- titanic_md$Age
missing_age <- which(is.na(titanic_md$Age) & !is.na(titanic_md$Sex_imputed))
titanic_md$Age_regression[missing_age] <- predict(age_lm, newdata = titanic_md[missing_age, ])
```

``` r
#Manejo de Outliers
age_mean_all <- mean(titanic_md$Age, na.rm = TRUE)
age_sd_all <- sd(titanic_md$Age, na.rm = TRUE)

age_lower_limit <- age_mean_all - 3 * age_sd_all
age_upper_limit <- age_mean_all + 3 * age_sd_all

age_outliers <- which(titanic_md$Age < age_lower_limit | titanic_md$Age > age_upper_limit)

titanic_md$Age_outliers <- titanic_md$Age
titanic_md$Age_outliers[age_outliers] <- NA

titanic_md$Age_outliers[is.na(titanic_md$Age_outliers)] <- age_median

cat("Manejo de outliers:", age_outliers)
```

    ## Manejo de outliers:

##### SIBSP:

``` r
# media
sibsp_mean <- mean(titanic_md$SibSp, na.rm = TRUE)


titanic_md$SibSp_mean <- titanic_md$SibSp
titanic_md$SibSp_mean[is.na(titanic_md$SibSp_mean)] <- sibsp_mean

# Imputacion por mediana
sibsp_median <- median(titanic_md$SibSp, na.rm = TRUE)

titanic_md$SibSp_median <- titanic_md$SibSp
titanic_md$SibSp_median[is.na(titanic_md$SibSp_median)] <- sibsp_median

# moda
sibsp_mode <- get_mode(titanic_md$SibSp)

titanic_md$SibSp_mode <- titanic_md$SibSp
titanic_md$SibSp_mode[is.na(titanic_md$SibSp_mode)] <- sibsp_mode
```

``` r
#Regresion Lineal
sibsp_model_data <- titanic_md %>%
  filter(!is.na(SibSp), !is.na(Pclass), !is.na(Parch))

sibsp_lm <- lm(SibSp ~ Pclass + Parch, data = sibsp_model_data)

titanic_md$SibSp_regression <- titanic_md$SibSp
missing_sibsp <- which(is.na(titanic_md$SibSp))
titanic_md$SibSp_regression[missing_sibsp] <- round(predict(sibsp_lm, newdata = titanic_md[missing_sibsp, ]))
```

``` r
#Manejo de Outliers
sibsp_mean_all <- mean(titanic_md$SibSp, na.rm = TRUE)
sibsp_sd_all <- sd(titanic_md$SibSp, na.rm = TRUE)

sibsp_lower_limit <- sibsp_mean_all - 3 * sibsp_sd_all
sibsp_upper_limit <- sibsp_mean_all + 3 * sibsp_sd_all

sibsp_outliers <- which(titanic_md$SibSp < sibsp_lower_limit | titanic_md$SibSp > sibsp_upper_limit)

titanic_md$SibSp_outliers <- titanic_md$SibSp
titanic_md$SibSp_outliers[sibsp_outliers] <- NA
titanic_md$SibSp_outliers[is.na(titanic_md$SibSp_outliers)] <- sibsp_median
```

##### PARCH:

``` r
# media
parch_mean <- mean(titanic_md$Parch, na.rm = TRUE)

titanic_md$Parch_mean <- titanic_md$Parch
titanic_md$Parch_mean[is.na(titanic_md$Parch_mean)] <- parch_mean

#  mediana
parch_median <- median(titanic_md$Parch, na.rm = TRUE)

titanic_md$Parch_median <- titanic_md$Parch
titanic_md$Parch_median[is.na(titanic_md$Parch_median)] <- parch_median

# moda
parch_mode <- get_mode(titanic_md$Parch)

titanic_md$Parch_mode <- titanic_md$Parch
titanic_md$Parch_mode[is.na(titanic_md$Parch_mode)] <- parch_mode
```

``` r
#Regresion lineal
parch_model_data <- titanic_md %>%
  filter(!is.na(Parch), !is.na(Pclass), !is.na(SibSp))

parch_lm <- lm(Parch ~ Pclass + SibSp, data = parch_model_data)

titanic_md$Parch_regression <- titanic_md$Parch
missing_parch <- which(is.na(titanic_md$Parch))
titanic_md$Parch_regression[missing_parch] <- round(predict(parch_lm, newdata = titanic_md[missing_parch, ]))
```

``` r
# manejo de outliers
parch_mean_all <- mean(titanic_md$Parch, na.rm = TRUE)
parch_sd_all <- sd(titanic_md$Parch, na.rm = TRUE)

parch_lower_limit <- parch_mean_all - 3 * parch_sd_all
parch_upper_limit <- parch_mean_all + 3 * parch_sd_all

parch_outliers <- which(titanic_md$Parch < parch_lower_limit | titanic_md$Parch > parch_upper_limit)

titanic_md$Parch_outliers <- titanic_md$Parch
titanic_md$Parch_outliers[parch_outliers] <- NA

titanic_md$Parch_outliers[is.na(titanic_md$Parch_outliers)] <- parch_median
```

##### EMBARKED:

``` r
# Moda 
embarked_mode <- get_mode(titanic_md$Embarked)

titanic_md$Embarked_imputed <- titanic_md$Embarked
titanic_md$Embarked_imputed[is.na(titanic_md$Embarked_imputed)] <- embarked_mode
```

##### FARE:

``` r
# media
fare_mean <- mean(titanic_md$Fare, na.rm = TRUE)

titanic_md$Fare_mean <- titanic_md$Fare
titanic_md$Fare_mean[is.na(titanic_md$Fare_mean)] <- fare_mean
```

``` r
#mediana
fare_median <- median(titanic_md$Fare, na.rm = TRUE)

titanic_md$Fare_median <- titanic_md$Fare
titanic_md$Fare_median[is.na(titanic_md$Fare_median)] <- fare_median
```

``` r
# moda
fare_mode <- get_mode(titanic_md$Fare)

titanic_md$Fare_mode <- titanic_md$Fare
titanic_md$Fare_mode[is.na(titanic_md$Fare_mode)] <- fare_mode
```

``` r
# Regresion Lineal

fare_model_data <- titanic_md %>%
  filter(!is.na(Fare), !is.na(Pclass), !is.na(SibSp), !is.na(Parch))

fare_lm <- lm(Fare ~ Pclass + SibSp + Parch, data = fare_model_data)

titanic_md$Fare_regression <- titanic_md$Fare
missing_fare <- which(is.na(titanic_md$Fare))
titanic_md$Fare_regression[missing_fare] <- predict(fare_lm, newdata = titanic_md[missing_fare, ])
```

``` r
#Outliers
fare_mean_all <- mean(titanic_md$Fare, na.rm = TRUE)
fare_sd_all <- sd(titanic_md$Fare, na.rm = TRUE)

fare_lower_limit <- fare_mean_all - 3 * fare_sd_all
fare_upper_limit <- fare_mean_all + 3 * fare_sd_all
fare_outliers <- which(titanic_md$Fare < fare_lower_limit | titanic_md$Fare > fare_upper_limit)

titanic_md$Fare_outliers <- titanic_md$Fare
titanic_md$Fare_outliers[fare_outliers] <- NA
titanic_md$Fare_outliers[is.na(titanic_md$Fare_outliers)] <- fare_median
```

#### 5) Que metodo se acerca mas a la realidad?

``` r
# AGE

missing_age_ids <- titanic_md$PassengerId[is.na(titanic_md$Age)]

actual_age <- titanic$Age[match(missing_age_ids, titanic$PassengerId)]

age_methods <- data.frame(
  Actual = actual_age,
  Mean = titanic_md$Age_mean[is.na(titanic_md$Age)],
  Median = titanic_md$Age_median[is.na(titanic_md$Age)],
  Mode = titanic_md$Age_mode[is.na(titanic_md$Age)],
  Regression = titanic_md$Age_regression[is.na(titanic_md$Age)],
  Outliers = titanic_md$Age_outliers[is.na(titanic_md$Age)]
)

age_mae <- sapply(age_methods[-1], function(x) mean(abs(x - age_methods$Actual), na.rm = TRUE))
age_mae
```

    ##       Mean     Median       Mode Regression   Outliers 
    ##   11.63690   11.66000   16.84000   11.01079   11.66000

El método de regresión lineal presenta el menor MAE, indicando que es el
más cercano a los valores reales.

``` r
# SIBSP

missing_sibsp_ids <- titanic_md$PassengerId[is.na(titanic_md$SibSp)]

actual_sibsp <- titanic$SibSp[match(missing_sibsp_ids, titanic$PassengerId)]

sibsp_methods <- data.frame(
  Actual = actual_sibsp,
  Mean = titanic_md$SibSp_mean[is.na(titanic_md$SibSp)],
  Median = titanic_md$SibSp_median[is.na(titanic_md$SibSp)],
  Mode = titanic_md$SibSp_mode[is.na(titanic_md$SibSp)],
  Regression = titanic_md$SibSp_regression[is.na(titanic_md$SibSp)],
  Outliers = titanic_md$SibSp_outliers[is.na(titanic_md$SibSp)]
)

sibsp_mae <- sapply(sibsp_methods[-1], function(x) mean(abs(x - sibsp_methods$Actual), na.rm = TRUE))
sibsp_mae
```

    ##       Mean     Median       Mode Regression   Outliers 
    ##  0.5129630  0.6666667  0.6666667  0.6666667  0.6666667

El método de la media presenta el menor MAE, indicando que es el más
cercano a los valores reales.

``` r
# PARCH

missing_parch_ids <- titanic_md$PassengerId[is.na(titanic_md$Parch)]

actual_parch <- titanic$Parch[match(missing_parch_ids, titanic$PassengerId)]

parch_methods <- data.frame(
  Actual = actual_parch,
  Mean = titanic_md$Parch_mean[is.na(titanic_md$Parch)],
  Median = titanic_md$Parch_median[is.na(titanic_md$Parch)],
  Mode = titanic_md$Parch_mode[is.na(titanic_md$Parch)],
  Regression = titanic_md$Parch_regression[is.na(titanic_md$Parch)],
  Outliers = titanic_md$Parch_outliers[is.na(titanic_md$Parch)]
)
parch_mae <- sapply(parch_methods[-1], function(x) mean(abs(x - parch_methods$Actual), na.rm = TRUE))
parch_mae
```

    ##       Mean     Median       Mode Regression   Outliers 
    ##  0.6666667  0.6666667  0.6666667  0.7500000  0.6666667

El método de la regresion presenta el mayor MAE, indicando que los demas
fueron mas efectivos.

``` r
#FARE


missing_fare_ids <- titanic_md$PassengerId[is.na(titanic_md$Fare)]

actual_fare <- titanic$Fare[match(missing_fare_ids, titanic$PassengerId)]

fare_methods <- data.frame(
  Actual = actual_fare,
  Mean = titanic_md$Fare_mean[is.na(titanic_md$Fare)],
  Median = titanic_md$Fare_median[is.na(titanic_md$Fare)],
  Mode = titanic_md$Fare_mode[is.na(titanic_md$Fare)],
  Regression = titanic_md$Fare_regression[is.na(titanic_md$Fare)],
  Outliers = titanic_md$Fare_outliers[is.na(titanic_md$Fare)]
)

fare_mae <- sapply(fare_methods[-1], function(x) mean(abs(x - fare_methods$Actual), na.rm = TRUE))
fare_mae
```

    ##       Mean     Median       Mode Regression   Outliers 
    ##   41.37000   40.72082   54.81668   30.23656   40.72082

El método de la regresion presenta el menor MAE, indicando que es el más
cercano a los valores reales.

#### Conclusiones

- Para variables numéricas como Age, SibSp, Parch y Fare, la imputacion
  por regresión lineal es el más preciso, ya que analiza la relación
  entre varias variables.
- Para variables Sex y Embarked es mejor usar la imputación por moda, ya
  que son variables categoricas y hay categorias predominantes

### Parte 2

#### 1) Normalizacion de columnas

``` r
titanic_md$Age_final <- titanic_md$Age_regression
titanic_md$SibSp_final <- titanic_md$SibSp_mean
titanic_md$Parch_final <- titanic_md$Parch_mean
titanic_md$Fare_final <- titanic_md$Fare_regression

titanic$Age_final <- titanic$Age
titanic$SibSp_final <- titanic$SibSp
titanic$Parch_final <- titanic$Parch
titanic$Fare_final <- titanic$Fare
```

``` r
#normalizacion titanic_md post procesamietno

# a) Standardization
titanic_md <- titanic_md %>%
  mutate(
    Age_std = scale(Age_final),
    SibSp_std = scale(SibSp_final),
    Parch_std = scale(Parch_final),
    Fare_std = scale(Fare_final)
  )

# b) Minmax Scaling
min_max_scale <- function(x) {
  (x - min(x)) / (max(x) - min(x))
}

titanic_md <- titanic_md %>%
  mutate(
    Age_minmax = min_max_scale(Age_final),
    SibSp_minmax = min_max_scale(SibSp_final),
    Parch_minmax = min_max_scale(Parch_final),
    Fare_minmax = min_max_scale(Fare_final)
  )

# c) MaxabsScaler
max_abs_scale <- function(x) {
  x / max(abs(x))
}

titanic_md <- titanic_md %>%
  mutate(
    Age_maxabs = max_abs_scale(Age_final),
    SibSp_maxabs = max_abs_scale(SibSp_final),
    Parch_maxabs = max_abs_scale(Parch_final),
    Fare_maxabs = max_abs_scale(Fare_final)
  )
```

``` r
#Normalizacion titanic.csv

# a) Standarization
titanic <- titanic %>%
  mutate(
    Age_std = scale(Age_final),
    SibSp_std = scale(SibSp_final),
    Parch_std = scale(Parch_final),
    Fare_std = scale(Fare_final)
  )

# b) MinMax Scaling
titanic <- titanic %>%
  mutate(
    Age_minmax = min_max_scale(Age_final),
    SibSp_minmax = min_max_scale(SibSp_final),
    Parch_minmax = min_max_scale(Parch_final),
    Fare_minmax = min_max_scale(Fare_final)
  )

# c) MaxAbsScaler
titanic <- titanic %>%
  mutate(
    Age_maxabs = max_abs_scale(Age_final),
    SibSp_maxabs = max_abs_scale(SibSp_final),
    Parch_maxabs = max_abs_scale(Parch_final),
    Fare_maxabs = max_abs_scale(Fare_final)
  )
```

#### 2) Comparacion de estadisticos y conclusion

``` r
calculate_stats <- function(data, variable) {
  stats <- data.frame(
    Variable = variable,
    Mean = mean(data[[variable]], na.rm = TRUE),
    SD = sd(data[[variable]], na.rm = TRUE),
    Min = min(data[[variable]], na.rm = TRUE),
    Max = max(data[[variable]], na.rm = TRUE),
    Median = median(data[[variable]], na.rm = TRUE)
  )
  return(stats)
}
```

AGE

``` r
# Comparacion de normalizacion para AGE

stats_age_std_md <- calculate_stats(titanic_md, "Age_std")
stats_age_std_complete <- calculate_stats(titanic, "Age_std")

stats_age_minmax_md <- calculate_stats(titanic_md, "Age_minmax")
```

    ## Warning in min(data[[variable]], na.rm = TRUE): ningún argumento finito para
    ## min; retornando Inf

    ## Warning in max(data[[variable]], na.rm = TRUE): ningun argumento finito para
    ## max; retornando -Inf

``` r
stats_age_minmax_complete <- calculate_stats(titanic, "Age_minmax")

stats_age_maxabs_md <- calculate_stats(titanic_md, "Age_maxabs")
```

    ## Warning in min(data[[variable]], na.rm = TRUE): ningún argumento finito para
    ## min; retornando Inf

    ## Warning in min(data[[variable]], na.rm = TRUE): ningun argumento finito para
    ## max; retornando -Inf

``` r
stats_age_maxabs_complete <- calculate_stats(titanic, "Age_maxabs")

age_stats_md <- rbind(stats_age_std_md, stats_age_minmax_md, stats_age_maxabs_md)
age_stats_complete <- rbind(stats_age_std_complete, stats_age_minmax_complete, stats_age_maxabs_complete)

age_stats_md$Dataset <- "titanic_MD"
age_stats_complete$Dataset <- "titanic.csv"

age_stats <- rbind(age_stats_md, age_stats_complete)
age_stats
```

    ##     Variable          Mean        SD       Min      Max      Median     Dataset
    ## 1    Age_std -7.223954e-17 1.0000000 -2.283929 2.958861 0.008631506  titanic_MD
    ## 2 Age_minmax           NaN        NA       Inf     -Inf          NA  titanic_MD
    ## 3 Age_maxabs           NaN        NA       Inf     -Inf          NA  titanic_MD
    ## 4    Age_std -1.873122e-16 1.0000000 -2.221601 2.833416 0.020811593 titanic.csv
    ## 5 Age_minmax  4.394844e-01 0.1978233  0.000000 1.000000 0.443601416 titanic.csv
    ## 6 Age_maxabs  4.459303e-01 0.1955483  0.011500 1.000000 0.450000000 titanic.csv

SIBSP

``` r
stats_sibsp_std_md <- calculate_stats(titanic_md, "SibSp_std")
stats_sibsp_std_complete <- calculate_stats(titanic, "SibSp_std")

stats_sibsp_minmax_md <- calculate_stats(titanic_md, "SibSp_minmax")
stats_sibsp_minmax_complete <- calculate_stats(titanic, "SibSp_minmax")

stats_sibsp_maxabs_md <- calculate_stats(titanic_md, "SibSp_maxabs")
stats_sibsp_maxabs_complete <- calculate_stats(titanic, "SibSp_maxabs")

sibsp_stats_md <- rbind(stats_sibsp_std_md, stats_sibsp_minmax_md, stats_sibsp_maxabs_md)
sibsp_stats_complete <- rbind(stats_sibsp_std_complete, stats_sibsp_minmax_complete, stats_sibsp_maxabs_complete)

sibsp_stats_md$Dataset <- "titanic_MD"
sibsp_stats_complete$Dataset <- "titanic.csv"

sibsp_stats <- rbind(sibsp_stats_md, sibsp_stats_complete)
sibsp_stats
```

    ##       Variable          Mean        SD        Min      Max     Median
    ## 1    SibSp_std -3.221028e-17 1.0000000 -0.7196151 3.962218 -0.7196151
    ## 2 SibSp_minmax  1.537037e-01 0.2135916  0.0000000 1.000000  0.0000000
    ## 3 SibSp_maxabs  1.537037e-01 0.2135916  0.0000000 1.000000  0.0000000
    ## 4    SibSp_std -3.873268e-17 1.0000000 -0.7210661 3.936172 -0.7210661
    ## 5 SibSp_minmax  1.548270e-01 0.2147195  0.0000000 1.000000  0.0000000
    ## 6 SibSp_maxabs  1.548270e-01 0.2147195  0.0000000 1.000000  0.0000000
    ##       Dataset
    ## 1  titanic_MD
    ## 2  titanic_MD
    ## 3  titanic_MD
    ## 4 titanic.csv
    ## 5 titanic.csv
    ## 6 titanic.csv

PARCH

``` r
stats_parch_std_md <- calculate_stats(titanic_md, "Parch_std")
stats_parch_std_complete <- calculate_stats(titanic, "Parch_std")

stats_parch_minmax_md <- calculate_stats(titanic_md, "Parch_minmax")
stats_parch_minmax_complete <- calculate_stats(titanic, "Parch_minmax")

stats_parch_maxabs_md <- calculate_stats(titanic_md, "Parch_maxabs")
stats_parch_maxabs_complete <- calculate_stats(titanic, "Parch_maxabs")

parch_stats_md <- rbind(stats_parch_std_md, stats_parch_minmax_md, stats_parch_maxabs_md)
parch_stats_complete <- rbind(stats_parch_std_complete, stats_parch_minmax_complete, stats_parch_maxabs_complete)

parch_stats_md$Dataset <- "titanic_MD"
parch_stats_complete$Dataset <- "titanic.csv"

parch_stats <- rbind(parch_stats_md, parch_stats_complete)
parch_stats
```

    ##       Variable         Mean        SD        Min      Max     Median
    ## 1    Parch_std 1.581750e-17 1.0000000 -0.6344486 4.858752 -0.6344486
    ## 2 Parch_minmax 1.154971e-01 0.1820432  0.0000000 1.000000  0.0000000
    ## 3 Parch_maxabs 1.154971e-01 0.1820432  0.0000000 1.000000  0.0000000
    ## 4    Parch_std 5.580975e-17 1.0000000 -0.6300014 4.670700 -0.6300014
    ## 5 Parch_minmax 1.188525e-01 0.1886543  0.0000000 1.000000  0.0000000
    ## 6 Parch_maxabs 1.188525e-01 0.1886543  0.0000000 1.000000  0.0000000
    ##       Dataset
    ## 1  titanic_MD
    ## 2  titanic_MD
    ## 3  titanic_MD
    ## 4 titanic.csv
    ## 5 titanic.csv
    ## 6 titanic.csv

FARE

``` r
stats_fare_std_md <- calculate_stats(titanic_md, "Fare_std")
stats_fare_std_complete <- calculate_stats(titanic, "Fare_std")

stats_fare_minmax_md <- calculate_stats(titanic_md, "Fare_minmax")
```

    ## Warning in min(data[[variable]], na.rm = TRUE): ningún argumento finito para
    ## min; retornando Inf

    ## Warning in max(data[[variable]], na.rm = TRUE): ningun argumento finito para
    ## max; retornando -Inf

``` r
stats_fare_minmax_complete <- calculate_stats(titanic, "Fare_minmax")

stats_fare_maxabs_md <- calculate_stats(titanic_md, "Fare_maxabs")
```

    ## Warning in min(data[[variable]], na.rm = TRUE): ningún argumento finito para
    ## min; retornando Inf

    ## Warning in min(data[[variable]], na.rm = TRUE): ningun argumento finito para
    ## max; retornando -Inf

``` r
stats_fare_maxabs_complete <- calculate_stats(titanic, "Fare_maxabs")

fare_stats_md <- rbind(stats_fare_std_md, stats_fare_minmax_md, stats_fare_maxabs_md)
fare_stats_complete <- rbind(stats_fare_std_complete, stats_fare_minmax_complete, stats_fare_maxabs_complete)

fare_stats_md$Dataset <- "titanic_MD"
fare_stats_complete$Dataset <- "titanic.csv"

fare_stats <- rbind(fare_stats_md, fare_stats_complete)

# Mostramos los resultados
fare_stats
```

    ##      Variable          Mean        SD       Min      Max     Median     Dataset
    ## 1    Fare_std -3.374728e-18 1.0000000 -1.489232 5.690813 -0.2720424  titanic_MD
    ## 2 Fare_minmax           NaN        NA       Inf     -Inf         NA  titanic_MD
    ## 3 Fare_maxabs           NaN        NA       Inf     -Inf         NA  titanic_MD
    ## 4    Fare_std -9.183015e-17 1.0000000 -1.030579 5.679882 -0.2839958 titanic.csv
    ## 5 Fare_minmax  1.535780e-01 0.1490211  0.000000 1.000000  0.1112566 titanic.csv
    ## 6 Fare_maxabs  1.535780e-01 0.1490211  0.000000 1.000000  0.1112566 titanic.csv

### Conclusion

Age: Las diferencias en la media sugieren que la imputación tuvo un
impacto marginal en estas variables

SibSp y Parch: Los estadísticos son similares entre ambos datasets, lo
que indica que la imputación de los valores faltantes de Age en
titanic_md no alteró significativamente la distribución general.

Fare: Nuevamente se evidencia que hay mayor diferencia en la media entre
ambos datasets
