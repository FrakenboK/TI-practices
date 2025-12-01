# Практическая работа №3. Основы обработки данных с помощью R и Dplyr
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

Установим пакет `nycflights13`:

    > install.packages('nycflights13')

    trying URL 'https://cran.rstudio.com/bin/windows/contrib/4.5/nycflights13_1.0.2.zip'
    Content type 'application/zip' length 4511564 bytes (4.3 MB)
    downloaded 4.3 MB

    package ‘nycflights13’ successfully unpacked and MD5 sums checked

    The downloaded binary packages are in
        C:\Users\FrakenboK\AppData\Local\Temp\RtmpkrieeV\downloaded_packages

Произведем загрузку библиотеки `nycflights13` и `dplyr`:

``` r
library(nycflights13)
library(dplyr)
```

## Шаги

### 1. Сколько встроенных в пакет `nycflights13` датафреймов?

``` r
dataframes <- data(package = "nycflights13")$results[, "Item"]

dataframes
```

    [1] "airlines" "airports" "flights"  "planes"   "weather" 

``` r
length(dataframes)
```

    [1] 5

### 2. Сколько строк в каждом датафрейме?

``` r
airlines %>% 
    nrow()
```

    [1] 16

``` r
airports %>% 
    nrow()
```

    [1] 1458

``` r
flights %>% 
    nrow()
```

    [1] 336776

``` r
planes %>% 
    nrow()
```

    [1] 3322

``` r
weather %>% 
    nrow()
```

    [1] 26115

### 3. Сколько столбцов в каждом датафрейме?

``` r
airlines %>% 
    ncol()
```

    [1] 2

``` r
airports %>% 
    ncol()
```

    [1] 8

``` r
flights %>% 
    ncol()
```

    [1] 19

``` r
planes %>% 
    ncol()
```

    [1] 9

``` r
weather %>% 
    ncol()
```

    [1] 15

### 4. Как просмотреть примерный вид датафрейма?

``` r
glimpse(airlines)
```

    Rows: 16
    Columns: 2
    $ carrier <chr> "9E", "AA", "AS", "B6", "DL", "EV", "F9", "FL", "HA", "MQ", "O…
    $ name    <chr> "Endeavor Air Inc.", "American Airlines Inc.", "Alaska Airline…

``` r
glimpse(airports)
```

    Rows: 1,458
    Columns: 8
    $ faa   <chr> "04G", "06A", "06C", "06N", "09J", "0A9", "0G6", "0G7", "0P2", "…
    $ name  <chr> "Lansdowne Airport", "Moton Field Municipal Airport", "Schaumbur…
    $ lat   <dbl> 41.13047, 32.46057, 41.98934, 41.43191, 31.07447, 36.37122, 41.4…
    $ lon   <dbl> -80.61958, -85.68003, -88.10124, -74.39156, -81.42778, -82.17342…
    $ alt   <dbl> 1044, 264, 801, 523, 11, 1593, 730, 492, 1000, 108, 409, 875, 10…
    $ tz    <dbl> -5, -6, -6, -5, -5, -5, -5, -5, -5, -8, -5, -6, -5, -5, -5, -5, …
    $ dst   <chr> "A", "A", "A", "A", "A", "A", "A", "A", "U", "A", "A", "U", "A",…
    $ tzone <chr> "America/New_York", "America/Chicago", "America/Chicago", "Ameri…

``` r
glimpse(flights)
```

    Rows: 336,776
    Columns: 19
    $ year           <int> 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2…
    $ month          <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
    $ day            <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
    $ dep_time       <int> 517, 533, 542, 544, 554, 554, 555, 557, 557, 558, 558, …
    $ sched_dep_time <int> 515, 529, 540, 545, 600, 558, 600, 600, 600, 600, 600, …
    $ dep_delay      <dbl> 2, 4, 2, -1, -6, -4, -5, -3, -3, -2, -2, -2, -2, -2, -1…
    $ arr_time       <int> 830, 850, 923, 1004, 812, 740, 913, 709, 838, 753, 849,…
    $ sched_arr_time <int> 819, 830, 850, 1022, 837, 728, 854, 723, 846, 745, 851,…
    $ arr_delay      <dbl> 11, 20, 33, -18, -25, 12, 19, -14, -8, 8, -2, -3, 7, -1…
    $ carrier        <chr> "UA", "UA", "AA", "B6", "DL", "UA", "B6", "EV", "B6", "…
    $ flight         <int> 1545, 1714, 1141, 725, 461, 1696, 507, 5708, 79, 301, 4…
    $ tailnum        <chr> "N14228", "N24211", "N619AA", "N804JB", "N668DN", "N394…
    $ origin         <chr> "EWR", "LGA", "JFK", "JFK", "LGA", "EWR", "EWR", "LGA",…
    $ dest           <chr> "IAH", "IAH", "MIA", "BQN", "ATL", "ORD", "FLL", "IAD",…
    $ air_time       <dbl> 227, 227, 160, 183, 116, 150, 158, 53, 140, 138, 149, 1…
    $ distance       <dbl> 1400, 1416, 1089, 1576, 762, 719, 1065, 229, 944, 733, …
    $ hour           <dbl> 5, 5, 5, 5, 6, 5, 6, 6, 6, 6, 6, 6, 6, 6, 6, 5, 6, 6, 6…
    $ minute         <dbl> 15, 29, 40, 45, 0, 58, 0, 0, 0, 0, 0, 0, 0, 0, 0, 59, 0…
    $ time_hour      <dttm> 2013-01-01 05:00:00, 2013-01-01 05:00:00, 2013-01-01 0…

``` r
glimpse(planes)
```

    Rows: 3,322
    Columns: 9
    $ tailnum      <chr> "N10156", "N102UW", "N103US", "N104UW", "N10575", "N105UW…
    $ year         <int> 2004, 1998, 1999, 1999, 2002, 1999, 1999, 1999, 1999, 199…
    $ type         <chr> "Fixed wing multi engine", "Fixed wing multi engine", "Fi…
    $ manufacturer <chr> "EMBRAER", "AIRBUS INDUSTRIE", "AIRBUS INDUSTRIE", "AIRBU…
    $ model        <chr> "EMB-145XR", "A320-214", "A320-214", "A320-214", "EMB-145…
    $ engines      <int> 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, …
    $ seats        <int> 55, 182, 182, 182, 55, 182, 182, 182, 182, 182, 55, 55, 5…
    $ speed        <int> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ engine       <chr> "Turbo-fan", "Turbo-fan", "Turbo-fan", "Turbo-fan", "Turb…

``` r
glimpse(weather)
```

    Rows: 26,115
    Columns: 15
    $ origin     <chr> "EWR", "EWR", "EWR", "EWR", "EWR", "EWR", "EWR", "EWR", "EW…
    $ year       <int> 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013,…
    $ month      <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
    $ day        <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
    $ hour       <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 13, 14, 15, 16, 17, 18, …
    $ temp       <dbl> 39.02, 39.02, 39.02, 39.92, 39.02, 37.94, 39.02, 39.92, 39.…
    $ dewp       <dbl> 26.06, 26.96, 28.04, 28.04, 28.04, 28.04, 28.04, 28.04, 28.…
    $ humid      <dbl> 59.37, 61.63, 64.43, 62.21, 64.43, 67.21, 64.43, 62.21, 62.…
    $ wind_dir   <dbl> 270, 250, 240, 250, 260, 240, 240, 250, 260, 260, 260, 330,…
    $ wind_speed <dbl> 10.35702, 8.05546, 11.50780, 12.65858, 12.65858, 11.50780, …
    $ wind_gust  <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, 20.…
    $ precip     <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,…
    $ pressure   <dbl> 1012.0, 1012.3, 1012.5, 1012.2, 1011.9, 1012.4, 1012.2, 101…
    $ visib      <dbl> 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10,…
    $ time_hour  <dttm> 2013-01-01 01:00:00, 2013-01-01 02:00:00, 2013-01-01 03:00…

### 5. Сколько компаний-перевозчиков (carrier) учитывают эти наборы данных (представлено в наборах данных)?

``` r
airlines %>%
    summarise(unique_carriers = n_distinct(carrier))
```

    # A tibble: 1 × 1
      unique_carriers
                <int>
    1              16

### 6. Сколько рейсов принял аэропорт John F Kennedy Intl в мае?

``` r
flights %>%
    filter(
        dest == "JFK",
        month == 5
    ) %>%
    summarise(total_flights = n())
```

    # A tibble: 1 × 1
      total_flights
              <int>
    1             0

### 7. Какой самый северный аэропорт?

``` r
airports %>%
    filter(lat == max(lat, na.rm = TRUE)) %>%
    select(name, lat)
```

    # A tibble: 1 × 2
      name                      lat
      <chr>                   <dbl>
    1 Dillant Hopkins Airport  72.3

### 8. Какой аэропорт самый высокогорный (находится выше всех над уровнем моря)?

``` r
airports %>%
    filter(alt == max(alt, na.rm = TRUE))%>%
    select(name, alt)
```

    # A tibble: 1 × 2
      name        alt
      <chr>     <dbl>
    1 Telluride  9078

### 9. Какие бортовые номера у самых старых самолетов?

``` r
planes %>%
    filter(year == min(year, na.rm = TRUE)) %>%
    select(tailnum, year) %>%
    arrange(tailnum)
```

    # A tibble: 1 × 2
      tailnum  year
      <chr>   <int>
    1 N381AA   1956

### 10. Какая средняя температура воздуха была в сентябре в аэропорту John F Kennedy Intl (в градусах Цельсия)

Воспользуемся формулой перевода градусов: `temp_c = (temp_f - 32) * 5/9`

``` r
jfk_september <- weather %>%
    filter(
        origin == "JFK",
        month == 9,
        !is.na(temp)
    )

jfk_september_celsius <- jfk_september %>%
    mutate(
        temp_c = (temp - 32) * 5/9
    )

jfk_september_celsius %>%
    summarise(
        avg_temp_c = mean(temp_c, na.rm = TRUE)
    )
```

    # A tibble: 1 × 1
      avg_temp_c
           <dbl>
    1       19.4

### 11. Самолеты какой авиакомпании совершили больше всего вылетов в июне?

``` r
flights %>% 
    filter(month == 6) %>% 
    count(carrier, sort = TRUE) %>% 
    slice(1) %>% 
    left_join(airlines, by = "carrier") %>% 
    rename(departures = n) %>% 
    select(name, departures)
```

    # A tibble: 1 × 2
      name                  departures
      <chr>                      <int>
    1 United Air Lines Inc.       4975

### 12. Самолеты какой авиакомпании задерживались чаще других в 2013 году?

``` r
flights %>%
    filter(
        year == 2013,
        dep_delay > 0,
        arr_delay > 0
    ) %>%
    group_by(carrier) %>%
    summarise(n = n()) %>%
    rename(count = n) %>% 
    arrange(desc(count)) %>% 
    left_join(airlines, by = "carrier")%>%
    head(1) %>%
    select(name)
```

    # A tibble: 1 × 1
      name                    
      <chr>                   
    1 ExpressJet Airlines Inc.

## Вывод

В данной работе мы познакомились с пакетом `dplyr` на примере анализа
датасетов из пакета `nycflights13`
