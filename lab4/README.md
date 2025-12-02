# Практическая работа №4. Исследование метаданных DNS трафика
kobyakmihail@yandex.ru

## Цель работы

1.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания основных функций обработки данных экосистемы
    `tidyverse` языка R
3.  Закрепить навыки исследования метаданных DNS трафика

## Исходные данные

1.  Программное обеспечение Windows 11
2.  Интерпретатор языка R v4.5.1
3.  Rstudio IDE

## Общая ситуация

Вы исследуете подозрительную сетевую активность во внутренней сети
Доброй Организации. Вам в руки попали метаданные о DNS трафике в
исследуемой сети. Исследуйте файлы, восстановите данные, подготовьте их
к анализу и дайте обоснованные ответы на поставленные вопросы
исследования.

## Задание

Используя программный пакет `dplyr`, освоить анализ DNS логов с помощью
языка программирования R.

## Подготовка к выполнению задания

Произведем загрузку библиотек:

``` r
library(tidyverse)
library(readr)

library(dplyr)

library(httr)
library(jsonlite)
```

## Шаги

### Подготовка данных

#### 1. Импортируйте данные DNS https://storage.yandexcloud.net/dataset.ctfsec/dns.zip

Произведем загрузку архива с лоагми. Загружать все данные будем в
диркеторию `data` (ее нужно создать, если не существует):

``` r
if (!dir.exists("data")) {
  dir.create("data")
}

file_url <- "https://storage.yandexcloud.net/dataset.ctfsec/dns.zip"
zip_filename <- "data/dns.zip"

download.file(
  url = file_url,
  destfile = zip_filename,
  mode = "wb",
  quiet = FALSE
)
```

Распакуем загруженный архив:

``` r
unzip(zip_filename,  exdir = "data")
```

Загрузим данные из лога:

``` r
dns_log <- read_delim("data/dns.log", 
                        delim = "\t",
                        col_names = FALSE,
                        col_types = cols(.default = col_character()),
                        trim_ws = TRUE,
                        na = c("", "NA", "-"))
dns_log
```

    # A tibble: 427,935 × 23
       X1    X2    X3    X4    X5    X6    X7    X8    X9    X10   X11   X12   X13  
       <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr>
     1 1331… CWGt… 192.… 45658 192.… 137   udp   33008 "*\\… 1     C_IN… 33    SRV  
     2 1331… C36a… 192.… 137   192.… 137   udp   57402 "HPE… 1     C_IN… 32    NB   
     3 1331… C36a… 192.… 137   192.… 137   udp   57402 "HPE… 1     C_IN… 32    NB   
     4 1331… C36a… 192.… 137   192.… 137   udp   57402 "HPE… 1     C_IN… 32    NB   
     5 1331… C36a… 192.… 137   192.… 137   udp   57398 "WPA… 1     C_IN… 32    NB   
     6 1331… C36a… 192.… 137   192.… 137   udp   57398 "WPA… 1     C_IN… 32    NB   
     7 1331… C36a… 192.… 137   192.… 137   udp   57398 "WPA… 1     C_IN… 32    NB   
     8 1331… ClEZ… 192.… 137   192.… 137   udp   62187 "EWR… 1     C_IN… 32    NB   
     9 1331… ClEZ… 192.… 137   192.… 137   udp   62187 "EWR… 1     C_IN… 32    NB   
    10 1331… ClEZ… 192.… 137   192.… 137   udp   62187 "EWR… 1     C_IN… 32    NB   
    # ℹ 427,925 more rows
    # ℹ 10 more variables: X14 <chr>, X15 <chr>, X16 <chr>, X17 <chr>, X18 <chr>,
    #   X19 <chr>, X20 <chr>, X21 <chr>, X22 <chr>, X23 <chr>

Видим, что в данных отсутствуют названия столбцов

#### 2. Добавьте пропущенные данные о структуре данных (назначении столбцов)

Обозначим названия столбцов:

``` r
column_names <- c("timestamp", "uid", "src_ip", "src_port", "dst_ip", "dst_port",
                    "proto", "trans_id", "query", "qclass", "qclass_name", "qtype", 
                    "qtype_name", "rcode", "rcode_name", "AA", "TC", "RD", "RA", 
                    "Z", "answers", "TTLs", "rejected")
```

Еще раз загрузим данные из файла с логом, но на этот раз с названиями
столбцов:

``` r
dns_log <- read_delim("data/dns.log",
                        delim = "\t",
                        col_names = column_names,
                        col_types = cols(.default = col_character()),
                        trim_ws = TRUE,
                        na = c("", "NA", "-"))

dns_log
```

    # A tibble: 427,935 × 23
       timestamp   uid   src_ip src_port dst_ip dst_port proto trans_id query qclass
       <chr>       <chr> <chr>  <chr>    <chr>  <chr>    <chr> <chr>    <chr> <chr> 
     1 1331901005… CWGt… 192.1… 45658    192.1… 137      udp   33008    "*\\… 1     
     2 1331901015… C36a… 192.1… 137      192.1… 137      udp   57402    "HPE… 1     
     3 1331901015… C36a… 192.1… 137      192.1… 137      udp   57402    "HPE… 1     
     4 1331901016… C36a… 192.1… 137      192.1… 137      udp   57402    "HPE… 1     
     5 1331901005… C36a… 192.1… 137      192.1… 137      udp   57398    "WPA… 1     
     6 1331901006… C36a… 192.1… 137      192.1… 137      udp   57398    "WPA… 1     
     7 1331901007… C36a… 192.1… 137      192.1… 137      udp   57398    "WPA… 1     
     8 1331901006… ClEZ… 192.1… 137      192.1… 137      udp   62187    "EWR… 1     
     9 1331901007… ClEZ… 192.1… 137      192.1… 137      udp   62187    "EWR… 1     
    10 1331901007… ClEZ… 192.1… 137      192.1… 137      udp   62187    "EWR… 1     
    # ℹ 427,925 more rows
    # ℹ 13 more variables: qclass_name <chr>, qtype <chr>, qtype_name <chr>,
    #   rcode <chr>, rcode_name <chr>, AA <chr>, TC <chr>, RD <chr>, RA <chr>,
    #   Z <chr>, answers <chr>, TTLs <chr>, rejected <chr>

Видим, что в данных неправильно указаны типы данных для столбцов

#### 3. Преобразуйте данные в столбцах в нужный формат

Еще раз загрузим данные из файла с логом, но на этот раз зададим каждом
столбцу название и тип. После загрузки преобразуем дату к формату
`datetime` и информацию о флагах к типу `boolean`:

``` r
dns_log <- read_delim("data/dns.log",
                        delim = "\t",
                        col_names = column_names,
                        col_types = cols(
                            timestamp = col_double(),

                            uid = col_character(),

                            src_ip = col_character(),
                            src_port = col_integer(),
                            dst_ip = col_character(),
                            dst_port = col_integer(),

                            proto = col_character(),
                            trans_id = col_integer(),

                            query = col_character(),

                            qclass = col_integer(),
                            qclass_name = col_character(),
                            qtype = col_integer(),
                            qtype_name = col_character(),

                            rcode = col_integer(),
                            rcode_name = col_character(),

                            AA = col_character(),
                            TC = col_character(),
                            RD = col_character(),
                            RA = col_character(),

                            Z = col_integer(),
                            answers = col_character(),
                            TTLs = col_character(),
                            rejected = col_character()
                        ),
                        trim_ws = TRUE,
                        na = c("", "NA", "-", "(empty)"))

dns_log <- dns_log %>%
    mutate(
        timestamp = as.POSIXct(timestamp, origin = "1970-01-01"),

        AA = AA == "T",
        TC = TC == "T",
        RD = RD == "T",
        RA = RA == "T",
        rejected = rejected == "T"
    )

dns_log
```

    # A tibble: 427,935 × 23
       timestamp           uid        src_ip src_port dst_ip dst_port proto trans_id
       <dttm>              <chr>      <chr>     <int> <chr>     <int> <chr>    <int>
     1 2012-03-16 15:30:05 CWGtK431H… 192.1…    45658 192.1…      137 udp      33008
     2 2012-03-16 15:30:15 C36a282Jl… 192.1…      137 192.1…      137 udp      57402
     3 2012-03-16 15:30:15 C36a282Jl… 192.1…      137 192.1…      137 udp      57402
     4 2012-03-16 15:30:16 C36a282Jl… 192.1…      137 192.1…      137 udp      57402
     5 2012-03-16 15:30:05 C36a282Jl… 192.1…      137 192.1…      137 udp      57398
     6 2012-03-16 15:30:06 C36a282Jl… 192.1…      137 192.1…      137 udp      57398
     7 2012-03-16 15:30:07 C36a282Jl… 192.1…      137 192.1…      137 udp      57398
     8 2012-03-16 15:30:06 ClEZCt3GL… 192.1…      137 192.1…      137 udp      62187
     9 2012-03-16 15:30:07 ClEZCt3GL… 192.1…      137 192.1…      137 udp      62187
    10 2012-03-16 15:30:07 ClEZCt3GL… 192.1…      137 192.1…      137 udp      62187
    # ℹ 427,925 more rows
    # ℹ 15 more variables: query <chr>, qclass <int>, qclass_name <chr>,
    #   qtype <int>, qtype_name <chr>, rcode <int>, rcode_name <chr>, AA <lgl>,
    #   TC <lgl>, RD <lgl>, RA <lgl>, Z <int>, answers <chr>, TTLs <chr>,
    #   rejected <lgl>

#### 4. Просмотрите общую структуру данных с помощью функции `glimpse`

``` r
glimpse(dns_log)
```

    Rows: 427,935
    Columns: 23
    $ timestamp   <dttm> 2012-03-16 15:30:05, 2012-03-16 15:30:15, 2012-03-16 15:3…
    $ uid         <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jljz7Bs…
    $ src_ip      <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76", "19…
    $ src_port    <int> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 1…
    $ dst_ip      <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255", "1…
    $ dst_port    <int> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137…
    $ proto       <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp", "u…
    $ trans_id    <int> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 62187, 62…
    $ query       <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\…
    $ qclass      <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
    $ qclass_name <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET", "C…
    $ qtype       <int> 33, 32, 32, 32, 32, 32, 32, 32, 32, 32, 33, 33, 33, 12, 12…
    $ qtype_name  <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB…
    $ rcode       <int> 0, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    $ rcode_name  <chr> "NOERROR", NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    $ AA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ TC          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ RD          <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRU…
    $ RA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ Z           <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0…
    $ answers     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ TTLs        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ rejected    <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…

### Анализ

#### 5. Сколько участников информационного обмена в сети Доброй Организации?

Воспользуемся `pivot_longer` из пакета `tidyverse`

``` r
dns_log %>%
    pivot_longer(
        cols = c(src_ip, dst_ip), 
        values_to = "ip"
    ) %>%
    distinct(ip) %>%
    filter(!is.na(ip)) %>%
    count() %>%
    rename(ip_count = n)
```

    # A tibble: 1 × 1
      ip_count
         <int>
    1     1359

#### 6. Какое соотношение участников обмена внутри сети и участников обращений к внешним ресурсам?

К приватным сетям IPv4 относятся следующие: `10.0.0.0/8`,
`172.16.0.0/12` и `192.168.0.0/16`. Для решения задания воспользуемся
регулярными выражениями:

``` r
IPv4_regex <- "(^10\\.)|(^172\\.1[6-9]\\.)|(^172\\.2[0-9]\\.)|(^172\\.3[0-1]\\.)|(^192\\.168\\.)"

with(
    dns_log, 
    sum(!duplicated(c(
        src_ip[grepl(IPv4_regex, src_ip, perl = TRUE)],
        dst_ip[grepl(IPv4_regex, dst_ip, perl = TRUE)]
    ))) / sum(!duplicated(c(
        src_ip[!grepl(IPv4_regex, src_ip, perl = TRUE)],
        dst_ip[!grepl(IPv4_regex, dst_ip, perl = TRUE)]
    )))
)
```

    [1] 13.77174

#### 7. Найдите топ-10 участников сети, проявляющих наибольшую сетевую активность

Воспользуемся `pivot_longer` из пакета `tidyverse`. Посчитем, какие
IP-адреса встречаются чаще всего:

``` r
dns_log %>%
    pivot_longer(
        cols = c(src_ip, dst_ip), 
        values_to = "ip"
    ) %>%
    count(ip, name = "activity_count") %>%
    arrange(desc(activity_count)) %>%
    slice_head(n = 10)
```

    # A tibble: 10 × 2
       ip              activity_count
       <chr>                    <int>
     1 192.168.207.4           266627
     2 10.10.117.210            75943
     3 192.168.202.255          68720
     4 192.168.202.93           26522
     5 172.19.1.100             25481
     6 192.168.202.103          18121
     7 192.168.202.76           16978
     8 192.168.202.97           16176
     9 192.168.202.141          14976
    10 192.168.202.110          14493

#### 8. Найдите топ-10 доменов, к которым обращаются пользователи сети и соответственное количество обращений

``` r
top_10_domains_list <- dns_log %>%
    count(query, name = "request_count") %>%
    arrange(
        desc(request_count)
    ) %>%
    slice_head(n = 10)

top_10_domains_list
```

    # A tibble: 10 × 2
       query                                                           request_count
       <chr>                                                                   <int>
     1 "teredo.ipv6.microsoft.com"                                             39273
     2 "tools.google.com"                                                      14057
     3 "www.apple.com"                                                         13390
     4 "time.apple.com"                                                        13109
     5 "safebrowsing.clients.google.com"                                       11658
     6 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00…         10401
     7 "WPAD"                                                                   9134
     8 "44.206.168.192.in-addr.arpa"                                            7248
     9 "HPE8AA67"                                                               6929
    10 "ISATAP"                                                                 6569

#### 9. Опеределите базовые статистические характеристики (функция `summary()`) интервала времени между последовательными обращениями к топ-10 доменам.

``` r
top_domains_log <- dns_log %>%
    filter(query %in% top_10_domains_list$query) %>%
    arrange(query, timestamp)

time_intervals <- top_domains_log %>%
    group_by(query) %>%
    mutate(
        time_diff = as.numeric(
            difftime(
                timestamp, 
                lag(timestamp), 
                units = "secs"
            )
        )
    ) %>%
    filter(!is.na(time_diff)) %>%
    ungroup()

summary(time_intervals$time_diff)
```

         Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
        0.000     0.000     0.750     8.758     1.740 52723.500 

#### 10. Часто вредоносное программное обеспечение использует DNS канал в качестве канала управления, периодически отправляя запросы на подконтрольный злоумышленникам DNS сервер. По периодическим запросам на один и тот же домен можно выявить скрытый DNS канал. Есть ли такие IP адреса в исследуемом датасете?

Будем фильтровать по стандартному отклонению (для выявления регулярных
запросов) и общему количеству запросов для определения подозрительной
активаности потенциальных C2-агентов, маскирующихся под DNS-активность

``` r
dns_log %>%
    filter(!is.na(src_ip), !is.na(query), 
            src_ip != "-", query != "-") %>%
    group_by(src_ip, query) %>%
    mutate(
            time_diff = as.numeric(
                difftime(
                    timestamp, 
                    lag(timestamp), 
                    units = "secs"
                )
            )
        ) %>%
    summarise(
        request_count = n(),
        mean_diff = mean(time_diff, na.rm = TRUE),
        sd_diff   = sd(time_diff, na.rm = TRUE),
    ) %>%
    filter(
        request_count > 35,
        mean_diff < 60,
        sd_diff < 1,
        !is.na(mean_diff)
    ) %>%
    arrange(mean_diff, desc(request_count))
```

    `summarise()` has grouped output by 'src_ip'. You can override using the
    `.groups` argument.

    # A tibble: 5 × 5
    # Groups:   src_ip [2]
      src_ip          query           request_count mean_diff sd_diff
      <chr>           <chr>                   <int>     <dbl>   <dbl>
    1 10.10.117.210   hq.h                      377   0        0     
    2 10.10.117.210   httphq.hec.net            102   0        0     
    3 10.10.117.210   www.h                      64   0        0     
    4 192.168.202.103 www.hkparts.net            36   0.00371  0.0124
    5 192.168.202.103 lifehacker.com             68   0.00612  0.0171

Видим 5 доменных записей, к которым просходило множество обращений с
большой частотой. Они могли быть использованны злоумышленником для
коммуникации С2-агентов или проброса reverse shell

### Обогащение данных

#### 11. Определите местоположение (страну, город) и организацию-провайдера для топ-10 доменов. Для этого можно использовать сторонние сервисы, например http://ip-api.com (API-эндпоинт http://ip-api.com/json)

Обогатим информацию о доменах, используя библиотеки `jsonlite` и `httr`:

``` r
get_domain_info <- function(domain) {
    if(is.na(domain) || domain == "") return(NULL)
    api_url <- paste0("http://ip-api.com/json/", domain)

    tryCatch({
        response <- GET(api_url)

        if(status_code(response) == 200) {
            data <- fromJSON(content(response, "text"))
            return(
                tibble(
                    domain = domain,
                    country = ifelse(!is.null(data$country), data$country, NA),
                    countryCode = ifelse(!is.null(data$countryCode), data$countryCode, NA),
                    region = ifelse(!is.null(data$region), data$region, NA),
                    regionName = ifelse(!is.null(data$regionName), data$regionName, NA),
                    city = ifelse(!is.null(data$city), data$city, NA),
                    zip = ifelse(!is.null(data$zip), data$zip, NA),
                    lat = ifelse(!is.null(data$lat), data$lat, NA),
                    lon = ifelse(!is.null(data$lon), data$lon, NA),
                    timezone = ifelse(!is.null(data$timezone), data$timezone, NA),
                    isp = ifelse(!is.null(data$isp), data$isp, NA),
                    org = ifelse(!is.null(data$org), data$org, NA),
                    as = ifelse(!is.null(data$as), data$as, NA),
                    query = ifelse(!is.null(data$query), data$query, NA),
                    stringsAsFactors = FALSE
                )
            )
        } else {
            return(NULL)
        }
    }, error = function(e) {
        return(NULL)
    })
}

bind_rows(
    map(
        top_10_domains_list$query, 
        get_domain_info
    )
)
```

    # A tibble: 10 × 15
       domain         country countryCode region regionName city  zip     lat    lon
       <chr>          <chr>   <chr>       <chr>  <chr>      <chr> <chr> <dbl>  <dbl>
     1 "teredo.ipv6.… <NA>    <NA>        <NA>   <NA>       <NA>   <NA>  NA     NA  
     2 "tools.google… United… US          CA     California Moun… "940…  37.4 -122. 
     3 "www.apple.co… United… US          VA     Virginia   Ashb… "201…  39.0  -77.5
     4 "time.apple.c… United… US          GA     Georgia    Atla… ""     33.7  -84.4
     5 "safebrowsing… United… US          CA     California Moun… "940…  37.4 -122. 
     6 "*\\x00\\x00\… <NA>    <NA>        <NA>   <NA>       <NA>   <NA>  NA     NA  
     7 "WPAD"         <NA>    <NA>        <NA>   <NA>       <NA>   <NA>  NA     NA  
     8 "44.206.168.1… <NA>    <NA>        <NA>   <NA>       <NA>   <NA>  NA     NA  
     9 "HPE8AA67"     <NA>    <NA>        <NA>   <NA>       <NA>   <NA>  NA     NA  
    10 "ISATAP"       <NA>    <NA>        <NA>   <NA>       <NA>   <NA>  NA     NA  
    # ℹ 6 more variables: timezone <chr>, isp <chr>, org <chr>, as <chr>,
    #   query <chr>, stringsAsFactors <lgl>

## Вывод

В данной работе мы познакомились с экосистемой `tidyverse` на примере
анализа DNS-трафика. Мы также научились коммуникации по протоколу HTTP,
используя язык R
