# Практическая работа №2. Основы обработки данных с помощью R и Dplyr
kobyakmihail@yandex.ru

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета `dplyr` – функции
    `select(), filter(), mutate(), arrange(), group_by()`

## Исходные данные

1.  Программное обеспечение Windows 11
2.  Интерпретатор языка R v4.5.1
3.  Rstudio IDE

## Подготовка к выполнению задания

Произведем загрузку библиотеки `dplyr`:

``` r
library(dplyr)
```

## Шаги

### 1. Сколько строк в датафрейме?

Воспользуемся функцией `nrow`:

``` r
starwars %>% nrow()
```

    [1] 87

### 2. Сколько столбцов в датафрейме?

Воспользуемся функцией `ncol`:

``` r
starwars %>% ncol()
```

    [1] 14

### 3. Как просмотреть примерный вид датафрейма?

Воспользуемся функцией `glimpse`:

``` r
starwars %>% glimpse()
```

    Rows: 87
    Columns: 14
    $ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Or…
    $ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 2…
    $ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.…
    $ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", N…
    $ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "…
    $ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue",…
    $ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, …
    $ sex        <chr> "male", "none", "none", "male", "female", "male", "female",…
    $ gender     <chr> "masculine", "masculine", "masculine", "masculine", "femini…
    $ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "T…
    $ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Huma…
    $ films      <list> <"A New Hope", "The Empire Strikes Back", "Return of the J…
    $ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imp…
    $ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1",…

### 4. Сколько уникальных рас персонажей (species) представлено в данных?

Воспользуемся функцией `distinct` для выбора уникальных значений и
`nrow` для последующего подсчета количества строк:

``` r
starwars %>% 
    distinct(species) %>% 
    nrow()
```

    [1] 38

### 5. Найти самого высокого персонажа

Воспользуемся функциями `filter` и `max`:

``` r
starwars %>% 
    filter(height == max(height, na.rm = TRUE))
```

    # A tibble: 1 × 14
      name      height  mass hair_color skin_color eye_color birth_year sex   gender
      <chr>      <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
    1 Yarael P…    264    NA none       white      yellow            NA male  mascu…
    # ℹ 5 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>

### 6. Найти всех персонажей ниже 170

Воспользуемся функциями `filter` и `arrange`:

``` r
starwars %>% 
    filter(height < 170) %>% 
    arrange(height)
```

    # A tibble: 22 × 14
       name     height  mass hair_color skin_color eye_color birth_year sex   gender
       <chr>     <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
     1 Yoda         66    17 white      green      brown            896 male  mascu…
     2 Ratts T…     79    15 none       grey, blue unknown           NA male  mascu…
     3 Wicket …     88    20 brown      brown      brown              8 male  mascu…
     4 Dud Bolt     94    45 none       blue, grey yellow            NA male  mascu…
     5 R2-D2        96    32 <NA>       white, bl… red               33 none  mascu…
     6 R4-P17       96    NA none       silver, r… red, blue         NA none  femin…
     7 R5-D4        97    32 <NA>       white, red red               NA none  mascu…
     8 Sebulba     112    40 none       grey, red  orange            NA male  mascu…
     9 Gasgano     122    NA none       white, bl… black             NA male  mascu…
    10 Watto       137    NA black      blue, grey yellow            NA male  mascu…
    # ℹ 12 more rows
    # ℹ 5 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>

### 7. Подсчитать ИМТ (индекс массы тела) для всех персонажей. ИМТ подсчитать по формуле `I = m / h ^ 2`, где `m` – масса (weight), а `h` – рост (height).

Сохраним в поле `height_m` значение роста в метрах, далее для
модификации исходного датафрейма воспользуемся функцией `mutate`:

``` r
starwars %>% 
    mutate(
        height_m = height / 100,
        bmi = mass / (height_m ^ 2)
    ) %>% 
    arrange(desc(bmi)) %>%
    select(name, mass, height, height_m, bmi)
```

    # A tibble: 87 × 5
       name                   mass height height_m   bmi
       <chr>                 <dbl>  <int>    <dbl> <dbl>
     1 Jabba Desilijic Tiure  1358    175     1.75 443. 
     2 Dud Bolt                 45     94     0.94  50.9
     3 Yoda                     17     66     0.66  39.0
     4 Owen Lars               120    178     1.78  37.9
     5 IG-88                   140    200     2     35  
     6 R2-D2                    32     96     0.96  34.7
     7 Grievous                159    216     2.16  34.1
     8 R5-D4                    32     97     0.97  34.0
     9 Jek Tono Porkins        110    180     1.8   34.0
    10 Darth Vader             136    202     2.02  33.3
    # ℹ 77 more rows

### 8. Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по отношению массы (mass) к росту (height) персонажей.

Воспользуемся функцией `filter` для отбора существующих значений,
функцию `mutate` для добавления поля “вытянутости” `elongation`, а также
`slice_max` для отбора максимпольных значений поля “вытянутости”:

``` r
starwars %>% 
    filter(!is.na(mass), !is.na(height)) %>%
    mutate(
        elongation = mass / (height / 100)
    ) %>% 
    select(name, mass, height, elongation) %>%
    slice_max(elongation, n = 10)
```

    # A tibble: 10 × 4
       name                   mass height elongation
       <chr>                 <dbl>  <int>      <dbl>
     1 Jabba Desilijic Tiure  1358    175      776  
     2 Grievous                159    216       73.6
     3 IG-88                   140    200       70  
     4 Owen Lars               120    178       67.4
     5 Darth Vader             136    202       67.3
     6 Jek Tono Porkins        110    180       61.1
     7 Bossk                   113    190       59.5
     8 Tarfful                 136    234       58.1
     9 Dexter Jettster         102    198       51.5
    10 Chewbacca               112    228       49.1

### 9. Найти средний возраст персонажей каждой расы вселенной Звездных войн

Воспользуемся функциями `filter` для отбора существующих значений,
`group_by`, `summarise`, `mean` и `arrange`:

``` r
starwars %>% 
    filter(!is.na(species), !is.na(birth_year)) %>%
    group_by(species) %>%
    summarise(
        count = n(),
        avg_birth_year = mean(birth_year, na.rm = TRUE)
    ) %>% 
    arrange(desc(avg_birth_year))
```

    # A tibble: 15 × 3
       species        count avg_birth_year
       <chr>          <int>          <dbl>
     1 Yoda's species     1          896  
     2 Hutt               1          600  
     3 Wookiee            1          200  
     4 Cerean             1           92  
     5 Zabrak             1           54  
     6 Human             26           53.7
     7 Droid              3           53.3
     8 Trandoshan         1           53  
     9 Gungan             1           52  
    10 Mirialan           2           49  
    11 Twi'lek            1           48  
    12 Rodian             1           44  
    13 Mon Calamari       1           41  
    14 Kel Dor            1           22  
    15 Ewok               1            8  

### 10. Найти средний возраст персонажей каждой расы вселенной Звездных войн

Воспользуемся функцией `count`, её атрибутом `sort` и функцией `head`:

``` r
starwars %>%
    count(eye_color, sort = TRUE) %>%
    head(1)
```

    # A tibble: 1 × 2
      eye_color     n
      <chr>     <int>
    1 brown        21

### 11. Подсчитать среднюю длину имени в каждой расе вселенной Звездных войн

Воспользуемся функциями `filter`, `mutate`, `group_by` и `summarise`:

``` r
starwars %>%
    filter(!is.na(species), !is.na(name)) %>%
    mutate(name_length = nchar(name)) %>%
    group_by(species) %>%
    summarise(
        count = n(),
        avg_length = mean(name_length)
    )
```

    # A tibble: 37 × 3
       species   count avg_length
       <chr>     <int>      <dbl>
     1 Aleena        1      12   
     2 Besalisk      1      15   
     3 Cerean        1      12   
     4 Chagrian      1      10   
     5 Clawdite      1      10   
     6 Droid         6       4.83
     7 Dug           1       7   
     8 Ewok          1      21   
     9 Geonosian     1      17   
    10 Gungan        3      11.7 
    # ℹ 27 more rows

## Вывод

В данной работе мы познакомились с пакетом `dplyr` на примере анализа
датасета `starwars`
