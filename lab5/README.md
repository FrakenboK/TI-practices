# Практическая работа №5. Исследование информации о состоянии
беспроводных сетей
kobyakmihail@yandex.ru

## Цель работы

1.  Получить знания о методах исследования радиоэлектронной обстановки.
2.  Составить представление о механизмах работы Wi-Fi сетей на канальном
    и сетевом уровне модели OSI.
3.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    `tidyverse` языка R

## Исходные данные

1.  Программное обеспечение Windows 11
2.  Интерпретатор языка R v4.5.1
3.  Rstudio IDE

## Общая ситуация

Вы исследуете состояние радиоэлектронной обстановки с помощью журналов
программных средств анализа беспроводных сетей – `tcpdump` и
`airodump-ng`. Для этого с помощью сниффера (микрокомпьютера Raspberry
Pi и специализированного Wi-Fi адаптера, переведенного в режим
мониторинга) собирались данные. Сниффер беспроводного трафика был
установлен стационарно (не перемещался). Какой анализ можно провести с
помощью собранной информации

## Задание

Используя программный пакет `dplyr` языка программирования R провести
анализ журналов и ответить на вопросы

## Подготовка к выполнению задания

Произведем загрузку библиотек:

``` r
library(tidyverse)
library(readr)

library(httr)

library(dplyr)
```

## Шаги

### Подготовка данных

#### 1. Импортируйте данные

Для начала скачаем данные:

``` r
if (!dir.exists("data")) {
    dir.create("data")
}

wifi_csv_url <- "https://storage.yandexcloud.net/dataset.ctfsec/P2_wifi_data.csv"
csv_filename <- "data/P2_wifi_data.csv"

download.file(
    url = wifi_csv_url,
    destfile = csv_filename,
    mode = "wb",
    quiet = FALSE
)
```

Прочитаем данные двух датасетов из CSV-файла:

``` r
file_content <- read_lines(csv_filename)
second_table_start <- which(grepl("Station MAC", file_content))

wifi_data <- read_csv(
    csv_filename, 
    n_max = second_table_start[1] - 4
)
```

    Rows: 167 Columns: 15
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (6): BSSID, Privacy, Cipher, Authentication, LAN IP, ESSID
    dbl  (6): channel, Speed, Power, # beacons, # IV, ID-length
    lgl  (1): Key
    dttm (2): First time seen, Last time seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
head(wifi_data)
```

    # A tibble: 6 × 15
      BSSID     `First time seen`   `Last time seen`    channel Speed Privacy Cipher
      <chr>     <dttm>              <dttm>                <dbl> <dbl> <chr>   <chr> 
    1 BE:F1:71… 2023-07-28 09:13:03 2023-07-28 11:50:50       1   195 WPA2    CCMP  
    2 6E:C7:EC… 2023-07-28 09:13:03 2023-07-28 11:55:12       1   130 WPA2    CCMP  
    3 9A:75:A8… 2023-07-28 09:13:03 2023-07-28 11:53:31       1   360 WPA2    CCMP  
    4 4A:EC:1E… 2023-07-28 09:13:03 2023-07-28 11:04:01       7   360 WPA2    CCMP  
    5 D2:6D:52… 2023-07-28 09:13:03 2023-07-28 10:30:19       6   130 WPA2    CCMP  
    6 E8:28:C1… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
    # ℹ 8 more variables: Authentication <chr>, Power <dbl>, `# beacons` <dbl>,
    #   `# IV` <dbl>, `LAN IP` <chr>, `ID-length` <dbl>, ESSID <chr>, Key <lgl>

``` r
client_conn_data <- read_csv(
    csv_filename, 
    skip = second_table_start[1] - 2
)
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 12081 Columns: 7
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (3): Station MAC, BSSID, Probed ESSIDs
    dbl  (2): Power, # packets
    dttm (2): First time seen, Last time seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
head(client_conn_data)
```

    # A tibble: 6 × 7
      `Station MAC`  `First time seen`   `Last time seen`    Power `# packets` BSSID
      <chr>          <dttm>              <dttm>              <dbl>       <dbl> <chr>
    1 CA:66:3B:8F:5… 2023-07-28 09:13:03 2023-07-28 10:59:44   -33         858 BE:F…
    2 96:35:2D:3D:8… 2023-07-28 09:13:03 2023-07-28 09:13:03   -65           4 (not…
    3 5C:3A:45:9E:1… 2023-07-28 09:13:03 2023-07-28 11:51:54   -39         432 BE:F…
    4 C0:E4:34:D8:E… 2023-07-28 09:13:03 2023-07-28 11:53:16   -61         958 BE:F…
    5 5E:8E:A6:5E:3… 2023-07-28 09:13:04 2023-07-28 09:13:04   -53           1 (not…
    6 10:51:07:CB:3… 2023-07-28 09:13:05 2023-07-28 11:56:06   -43         344 (not…
    # ℹ 1 more variable: `Probed ESSIDs` <chr>

#### 2. Привести датасеты в вид “аккуратных данных”, преобразовать типы столбцов всоответствии с типом данных

Преобразуем данные

``` r
wifi_data <- wifi_data %>%

    rename_with(~ gsub("[.# ]", "_", .x)) %>%
    rename_with(~ gsub("_+", "_", .x)) %>%
    rename_with(~ tolower(.x))%>%
    rename_with(~ gsub("^_+", "", .x))%>%
    rename_with(~ gsub("-", "_", .x))%>%

    mutate(
        channel = as.integer(channel),
        speed = as.integer(speed),
        power = as.integer(power),
        beacons = as.integer(beacons),
        iv = as.integer(iv),
        id_length = as.integer(id_length),
        privacy = as.factor(privacy),
        cipher = as.factor(cipher),
        authentication = as.factor(authentication),
    )

client_conn_data <- client_conn_data %>%

    rename_with(~ gsub("[.# ]", "_", .x)) %>%
    rename_with(~ gsub("_+", "_", .x)) %>%
    rename_with(~ tolower(.x))%>%
    rename_with(~ gsub("^_+", "", .x))%>%
    rename_with(~ gsub("-", "_", .x))%>%

    mutate(
        power = as.integer(power),
        packets = as.integer(packets),
        
    )
```

#### 3. Просмотрите общую структуру данных с помощью функции `glimpse`

``` r
glimpse(wifi_data)
```

    Rows: 167
    Columns: 15
    $ bssid           <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9…
    $ first_time_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ last_time_seen  <dttm> 2023-07-28 11:50:50, 2023-07-28 11:55:12, 2023-07-28 …
    $ channel         <int> 1, 1, 1, 7, 6, 6, 11, 11, 11, 1, 6, 14, 11, 11, 6, 6, …
    $ speed           <int> 195, 130, 360, 360, 130, 130, 195, 130, 130, 195, 180,…
    $ privacy         <fct> WPA2, WPA2, WPA2, WPA2, WPA2, OPN, WPA2, WPA2, WPA2, W…
    $ cipher          <fct> CCMP, CCMP, CCMP, CCMP, CCMP, NA, CCMP, CCMP, CCMP, CC…
    $ authentication  <fct> PSK, PSK, PSK, PSK, PSK, NA, PSK, PSK, PSK, PSK, PSK, …
    $ power           <int> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42,…
    $ beacons         <int> 846, 750, 694, 510, 647, 251, 1647, 1251, 704, 617, 13…
    $ iv              <int> 504, 116, 26, 21, 6, 3430, 80, 11, 0, 0, 86, 0, 0, 0, …
    $ lan_ip          <chr> "0.  0.  0.  0", "0.  0.  0.  0", "0.  0.  0.  0", "0.…
    $ id_length       <int> 12, 4, 2, 14, 25, 13, 12, 13, 24, 12, 10, 0, 24, 24, 1…
    $ essid           <chr> "C322U13 3965", "Cnet", "KC", "POCO X5 Pro 5G", NA, "M…
    $ key             <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…

``` r
glimpse(client_conn_data)
```

    Rows: 12,081
    Columns: 7
    $ station_mac     <chr> "CA:66:3B:8F:56:DD", "96:35:2D:3D:85:E6", "5C:3A:45:9E…
    $ first_time_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ last_time_seen  <dttm> 2023-07-28 10:59:44, 2023-07-28 09:13:03, 2023-07-28 …
    $ power           <int> -33, -65, -39, -61, -53, -43, -31, -71, -74, -65, -45,…
    $ packets         <int> 858, 4, 432, 958, 1, 344, 163, 3, 115, 437, 265, 77, 7…
    $ bssid           <chr> "BE:F1:71:D5:17:8B", "(not associated)", "BE:F1:71:D6:…
    $ probed_essids   <chr> "C322U13 3965", "IT2 Wireless", "C322U21 0566", "C322U…

### Анализ точек доступа

#### 1. Определить небезопасные точки доступа (без шифрования – OPN)

``` r
wifi_data %>%
    filter(privacy == "OPN")
```

    # A tibble: 42 × 15
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     2 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:12       6   130 OPN     <NA>  
     3 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:11       6   130 OPN     <NA>  
     4 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:10       6    -1 OPN     <NA>  
     5 00:25:0… 2023-07-28 09:13:06 2023-07-28 11:56:21      44    -1 OPN     <NA>  
     6 E8:28:C… 2023-07-28 09:13:09 2023-07-28 11:56:05      11   130 OPN     <NA>  
     7 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:27:06       6   130 OPN     <NA>  
     8 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:39:43       6   130 OPN     <NA>  
     9 E8:28:C… 2023-07-28 09:13:17 2023-07-28 11:52:32       1   130 OPN     <NA>  
    10 E8:28:C… 2023-07-28 09:13:50 2023-07-28 11:43:39      11   130 OPN     <NA>  
    # ℹ 32 more rows
    # ℹ 8 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>

#### 2. Определить производителя для каждого обнаруженного устройства

Для этого воспользуемся запросами в API, а также настроим кэш, для
уменьшения количества сетевых запросов

``` r
manufacturer_cache <- new.env(hash = TRUE)

get_manufacturer <- function(mac_address) {
    clean_mac <- gsub("[:.-]", "", mac_address)
    clean_mac <- toupper(clean_mac)
    
    oui <- substr(clean_mac, 1, 6)
    
    if (exists(oui, envir = manufacturer_cache)) {
        return(get(oui, envir = manufacturer_cache))
    }
    
    url <- paste0("https://www.macvendorlookup.com/api/v2/", oui)
    response <- GET(url, timeout(3))
    
    if (status_code(response) == 200) {
        content_data <- content(response, "parsed")
        
        if (length(content_data) > 0) {
            manufacturer <- content_data[[1]]$company
            if (is.null(manufacturer) || manufacturer == "") {
                manufacturer <- "Unknown"
            }
        } else {
            manufacturer <- "Unknown"
        }
        
        assign(oui, manufacturer, envir = manufacturer_cache)
        return(manufacturer)
        
    } else {
        assign(oui, "Unknown", envir = manufacturer_cache)
        return("Unknown")
    }
}

wifi_data <- wifi_data %>%
mutate(
    manufacturer = sapply(bssid, get_manufacturer)
)

wifi_data %>% select(bssid, manufacturer)
```

    # A tibble: 167 × 2
       bssid             manufacturer           
       <chr>             <chr>                  
     1 BE:F1:71:D5:17:8B Unknown                
     2 6E:C7:EC:16:DA:1A Unknown                
     3 9A:75:A8:B9:04:1E Unknown                
     4 4A:EC:1E:DB:BF:95 Unknown                
     5 D2:6D:52:61:51:5D Unknown                
     6 E8:28:C1:DC:B2:52 Eltex Enterprise Ltd.  
     7 BE:F1:71:D6:10:D7 Unknown                
     8 0A:C5:E1:DB:17:7B Unknown                
     9 38:1A:52:0D:84:D7 Seiko Epson Corporation
    10 BE:F1:71:D5:0E:53 Unknown                
    # ℹ 157 more rows

#### 3. Выявить устройства, использующие последнюю версию протокола шифрования WPA3, и названия точек доступа, реализованных на этих устройствах

``` r
wifi_data %>%
    filter(str_detect(privacy, "WPA3")) %>%
    arrange(desc(power))
```

    # A tibble: 8 × 16
      bssid     first_time_seen     last_time_seen      channel speed privacy cipher
      <chr>     <dttm>              <dttm>                <int> <int> <fct>   <fct> 
    1 76:C5:A0… 2023-07-28 11:16:36 2023-07-28 11:16:38       6   130 WPA3 W… CCMP  
    2 BE:FD:EF… 2023-07-28 10:15:24 2023-07-28 10:15:28       6   130 WPA3 W… CCMP  
    3 CE:48:E7… 2023-07-28 09:59:20 2023-07-28 10:04:15      44   866 WPA3 W… CCMP  
    4 3A:DA:00… 2023-07-28 10:27:01 2023-07-28 10:27:10       6   130 WPA3 W… CCMP  
    5 8E:1F:94… 2023-07-28 10:08:32 2023-07-28 10:15:27      44   866 WPA3 W… CCMP  
    6 A2:FE:FF… 2023-07-28 09:41:52 2023-07-28 09:41:52       6   130 WPA3 W… CCMP  
    7 26:20:53… 2023-07-28 09:15:45 2023-07-28 09:33:10      44   866 WPA3 W… CCMP  
    8 96:FF:FC… 2023-07-28 09:52:54 2023-07-28 10:25:02      44   866 WPA3 W… CCMP  
    # ℹ 9 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>

#### 4. Отсортировать точки доступа по интервалу времени, в течение которого они находились на связи, по убыванию (Не забудьте склеить сессии! Сессии считаются независимыми если интервал времени между ними превышает 45 минут.)

Для этого реализуем функцию склейки:

``` r
merge_sessions <- function(data, max_gap_minutes = 45) {
  
    data_prepared <- data %>%
        mutate(
        first_time_seen = as.POSIXct(first_time_seen),
        last_time_seen = as.POSIXct(last_time_seen)
        ) %>%
        arrange(bssid, first_time_seen)
    
    merge_group_sessions <- function(group_data) {
        if (nrow(group_data) == 1) {
            return(group_data %>%
                    mutate(
                        session_id = 1,
                        session_duration_minutes = as.numeric(
                        difftime(last_time_seen, first_time_seen, units = "mins")
                        )
                    )
            )
        }
    
        result <- group_data %>%
            arrange(first_time_seen) %>%
            mutate(
                prev_last_time = lag(last_time_seen),
                gap_minutes = ifelse(
                    is.na(prev_last_time),
                    0,
                    as.numeric(difftime(first_time_seen, prev_last_time, units = "mins"))
                ),
                session_start = ifelse(row_number() == 1 | gap_minutes > max_gap_minutes, 
                                    TRUE, FALSE),
                session_id = cumsum(session_start)
            ) %>%
            group_by(session_id) %>%
            summarise(
                merged_first_time_seen = min(first_time_seen),
                merged_last_time_seen = max(last_time_seen),
                essid = first(essid),
                channel = first(channel),
                privacy = first(privacy),
                manufacturer = first(manufacturer),
                avg_power = mean(power, na.rm = TRUE),
                max_power = max(power, na.rm = TRUE),
                total_beacons = sum(beacons, na.rm = TRUE),
                total_iv = sum(iv, na.rm = TRUE),
                num_sessions_merged = n(),
                .groups = "drop"
            ) %>%
            mutate(
                session_duration_minutes = as.numeric(
                    difftime(merged_last_time_seen, merged_first_time_seen, units = "mins")
                )
            )
        
        return(result)
    }
  
    wifi_sessions <- data_prepared %>%
        group_by(bssid) %>%
        group_modify(~ merge_group_sessions(.x)) %>%
        ungroup() %>%
        mutate(
        session_duration_hours = round(session_duration_minutes / 60, 2),
        session_duration_days = round(session_duration_minutes / (60 * 24), 2)
        ) %>%
        arrange(desc(session_duration_minutes))
    
    return(wifi_sessions)
}

merge_sessions(wifi_data)
```

    # A tibble: 167 × 20
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 00:25:0… 2023-07-28 09:13:06 2023-07-28 11:56:21      44    -1 OPN     <NA>  
     2 E8:28:C… 2023-07-28 09:13:09 2023-07-28 11:56:05      11   130 OPN     <NA>  
     3 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     4 08:3A:2… 2023-07-28 09:13:27 2023-07-28 11:55:53      14    -1 WPA     <NA>  
     5 6E:C7:E… 2023-07-28 09:13:03 2023-07-28 11:55:12       1   130 WPA2    CCMP  
     6 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:12       6   130 OPN     <NA>  
     7 48:5B:3… 2023-07-28 09:13:06 2023-07-28 11:55:11       1   270 WPA2    CCMP  
     8 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:11       6   130 OPN     <NA>  
     9 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:10       6    -1 OPN     <NA>  
    10 8E:55:4… 2023-07-28 09:13:06 2023-07-28 11:55:09       6    65 WPA2    CCMP  
    # ℹ 157 more rows
    # ℹ 13 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>, session_id <dbl>, session_duration_minutes <dbl>,
    #   session_duration_hours <dbl>, session_duration_days <dbl>

#### 5. Обнаружить топ-10 самых быстрых точек доступа

``` r
wifi_data %>%
    filter(!is.na(speed) & speed > 0) %>%
    arrange(desc(speed)) %>%
    head(10)
```

    # A tibble: 10 × 16
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 26:20:5… 2023-07-28 09:15:45 2023-07-28 09:33:10      44   866 WPA3 W… CCMP  
     2 96:FF:F… 2023-07-28 09:52:54 2023-07-28 10:25:02      44   866 WPA3 W… CCMP  
     3 CE:48:E… 2023-07-28 09:59:20 2023-07-28 10:04:15      44   866 WPA3 W… CCMP  
     4 8E:1F:9… 2023-07-28 10:08:32 2023-07-28 10:15:27      44   866 WPA3 W… CCMP  
     5 9A:75:A… 2023-07-28 09:13:03 2023-07-28 11:53:31       1   360 WPA2    CCMP  
     6 4A:EC:1… 2023-07-28 09:13:03 2023-07-28 11:04:01       7   360 WPA2    CCMP  
     7 56:C5:2… 2023-07-28 09:17:49 2023-07-28 10:27:22       1   360 WPA2    CCMP  
     8 E8:28:C… 2023-07-28 09:18:16 2023-07-28 11:36:43      48   360 OPN     <NA>  
     9 E8:28:C… 2023-07-28 09:18:16 2023-07-28 11:51:48      48   360 OPN     <NA>  
    10 E8:28:C… 2023-07-28 09:18:30 2023-07-28 11:43:23      48   360 OPN     <NA>  
    # ℹ 9 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>

#### 6. Отсортировать точки доступа по частоте отправки запросов (beacons) в единицу времени по их убыванию

``` r
wifi_data %>%
    filter(!is.na(beacons) & beacons > 0 & 
            !is.na(first_time_seen) & !is.na(last_time_seen)) %>%

    mutate(
        first_time_seen = as.POSIXct(first_time_seen),
        last_time_seen = as.POSIXct(last_time_seen),
        duration_minutes = as.numeric(difftime(last_time_seen, first_time_seen, units = "mins")),
        duration_minutes = ifelse(duration_minutes <= 0, 0.1, duration_minutes),
        beacon_rate_per_min = beacons / duration_minutes,
        beacon_rate_per_sec = beacon_rate_per_min / 60,
        beacon_rate_per_hour = beacons / (duration_minutes / 60)
    ) %>%

    arrange(desc(beacon_rate_per_min)) 
```

    # A tibble: 102 × 20
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 F2:30:A… 2023-07-28 10:27:02 2023-07-28 10:27:09       1   130 WPA2    CCMP  
     2 B2:CF:C… 2023-07-28 10:40:54 2023-07-28 10:40:59       6   180 WPA2    CCMP  
     3 3A:DA:0… 2023-07-28 10:27:01 2023-07-28 10:27:10       6   130 WPA3 W… CCMP  
     4 02:BC:1… 2023-07-28 09:24:46 2023-07-28 09:24:48      11   130 OPN     <NA>  
     5 00:3E:1… 2023-07-28 10:34:03 2023-07-28 10:34:05      11   130 OPN     <NA>  
     6 76:C5:A… 2023-07-28 11:16:36 2023-07-28 11:16:38       6   130 WPA3 W… CCMP  
     7 D2:25:9… 2023-07-28 09:45:29 2023-07-28 09:45:42      12   180 WPA2    CCMP  
     8 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 11:50:44      11   195 WPA2    CCMP  
     9 76:E4:E… 2023-07-28 09:18:50 2023-07-28 09:18:50      13   180 WPA2    CCMP  
    10 C2:B5:D… 2023-07-28 09:32:42 2023-07-28 09:32:42       6   130 WPA2    CCMP  
    # ℹ 92 more rows
    # ℹ 13 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>, duration_minutes <dbl>, beacon_rate_per_min <dbl>,
    #   beacon_rate_per_sec <dbl>, beacon_rate_per_hour <dbl>

### Анализ клиентов

#### 1. Определить производителя для каждого обнаруженного устройства

Для этого воспользуемся функцией, написанной нами для поиска
производителей точек доступа:

``` r
client_conn_data <- client_conn_data %>%
    filter(!is.na(bssid) & bssid != "(not associated)") %>%
    mutate(
        manufacturer = sapply(bssid, get_manufacturer)
    )

client_conn_data %>% select(bssid, manufacturer)
```

    # A tibble: 186 × 2
       bssid             manufacturer                 
       <chr>             <chr>                        
     1 BE:F1:71:D5:17:8B Unknown                      
     2 BE:F1:71:D6:10:D7 Unknown                      
     3 BE:F1:71:D5:17:8B Unknown                      
     4 1E:93:E3:1B:3C:F4 Unknown                      
     5 E8:28:C1:DC:FF:F2 Eltex Enterprise Ltd.        
     6 00:25:00:FF:94:73 Apple, Inc.                  
     7 00:26:99:F2:7A:E2 Cisco Systems, Inc           
     8 0C:80:63:A9:6E:EE TP-LINK TECHNOLOGIES CO.,LTD.
     9 E8:28:C1:DD:04:52 Eltex Enterprise Ltd.        
    10 0A:C5:E1:DB:17:7B Unknown                      
    # ℹ 176 more rows

#### 2. Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес

``` r
rand_prefixes <- c("A", "a", "E", "e", "2", "6")

client_conn_data %>%
    filter(
        !substr(station_mac, 2, 2) %in% rand_prefixes
        )
```

    # A tibble: 59 × 8
       station_mac       first_time_seen     last_time_seen      power packets bssid
       <chr>             <dttm>              <dttm>              <int>   <int> <chr>
     1 5C:3A:45:9E:1A:7B 2023-07-28 09:13:03 2023-07-28 11:51:54   -39     432 BE:F…
     2 C0:E4:34:D8:E7:E5 2023-07-28 09:13:03 2023-07-28 11:53:16   -61     958 BE:F…
     3 68:54:5A:40:35:9E 2023-07-28 09:13:06 2023-07-28 11:50:50   -31     163 1E:9…
     4 74:4C:A1:70:CE:F7 2023-07-28 09:13:06 2023-07-28 09:20:01   -71       3 E8:2…
     5 4C:44:5B:14:76:E3 2023-07-28 09:13:09 2023-07-28 09:47:44    -1      71 E8:2…
     6 A0:E7:0B:AE:D5:44 2023-07-28 09:13:09 2023-07-28 11:34:42   -37     125 0A:C…
     7 48:68:4A:93:DF:B4 2023-07-28 09:13:13 2023-07-28 11:50:22   -48     122 1E:9…
     8 28:7F:CF:23:25:53 2023-07-28 09:13:14 2023-07-28 11:51:50   -37     156 9A:7…
     9 8C:55:4A:DE:F2:38 2023-07-28 09:13:17 2023-07-28 11:56:16   -65     117 8A:A…
    10 88:D8:2E:4F:9B:1A 2023-07-28 09:13:19 2023-07-28 11:51:24   -29     240 4A:E…
    # ℹ 49 more rows
    # ℹ 2 more variables: probed_essids <chr>, manufacturer <chr>

#### 3. Кластеризовать запросы от устройств к точкам доступа по их именам. Определить время появления устройства в зоне радиовидимости и время выхода его из нее.

``` r
client_data_processed <- client_conn_data %>%
    mutate(
        first_time_seen = as.POSIXct(first_time_seen),
        last_time_seen = as.POSIXct(last_time_seen),
        probed_essids = ifelse(is.na(probed_essids) | probed_essids == "", 
                            "Unknown", 
                            as.character(probed_essids))
    ) %>%
    filter(probed_essids != "Unknown")

requests <- client_data_processed %>%
    mutate(
        cluster = str_split(probed_essids, ",")
    ) %>%
    unnest(cluster) %>%
    mutate(
        cluster = str_trim(cluster),
        cluster = ifelse(cluster == "", "Unknown", cluster)
    ) %>%
    filter(cluster != "Unknown")

cl <- requests %>%
    group_by(station_mac, cluster) %>%
    summarise(
        first_appearance = min(first_time_seen, na.rm = TRUE),
        last_appearance = max(last_time_seen, na.rm = TRUE),
        duration_minutes = as.numeric(
        difftime(last_appearance, first_appearance, units = "mins")
        ),
        session_count = n(),
        avg_power = round(mean(power, na.rm = TRUE), 1),
        total_packets = sum(packets, na.rm = TRUE),
        packets_per_minute = ifelse(duration_minutes > 0,
                                round(total_packets / duration_minutes, 2),
                                0),
        .groups = "drop"
    ) %>%
    arrange(station_mac, first_appearance)

cl
```

    # A tibble: 120 × 9
       station_mac  cluster first_appearance    last_appearance     duration_minutes
       <chr>        <chr>   <dttm>              <dttm>                         <dbl>
     1 00:F4:8D:F7… Hornet… 2023-07-28 10:45:04 2023-07-28 11:43:26           58.4  
     2 00:F4:8D:F7… Redmi … 2023-07-28 10:45:04 2023-07-28 11:43:26           58.4  
     3 02:69:A5:29… Galaxy… 2023-07-28 10:52:35 2023-07-28 11:24:51           32.3  
     4 04:8C:9A:0B… MIREA_… 2023-07-28 10:27:47 2023-07-28 11:55:51           88.1  
     5 06:15:2E:12… MIREA_… 2023-07-28 10:27:47 2023-07-28 10:30:41            2.9  
     6 06:15:2E:12… MT_FREE 2023-07-28 10:27:47 2023-07-28 10:30:41            2.9  
     7 06:F2:A9:C1… MIREA_… 2023-07-28 09:31:40 2023-07-28 11:50:58          139.   
     8 0A:AB:49:39… MIREA_… 2023-07-28 10:32:39 2023-07-28 11:39:44           67.1  
     9 0A:C2:C3:08… MIREA_… 2023-07-28 10:14:39 2023-07-28 10:14:47            0.133
    10 0C:E4:41:E8… MIREA_… 2023-07-28 09:32:07 2023-07-28 10:21:40           49.6  
    # ℹ 110 more rows
    # ℹ 4 more variables: session_count <int>, avg_power <dbl>,
    #   total_packets <int>, packets_per_minute <dbl>

#### 4. Оценить стабильность уровня сигнала внури кластера во времени. Выявить наиболее стабильный кластер.

``` r
sustainability <- requests %>%
    group_by(cluster) %>%
    summarise(
        n_requests = n(),
        unique_devices = n_distinct(station_mac),
        mean_power = round(mean(power, na.rm = TRUE), 2),
        median_power = round(median(power, na.rm = TRUE), 2),
        sd_power = if (n() == 1) 0 else round(sd(power, na.rm = TRUE), 2),
        min_power = min(power, na.rm = TRUE),
        max_power = max(power, na.rm = TRUE),
        range_power = round(max_power - min_power, 2),
        cv_power = if (abs(mean_power) > 0) 
        round((sd_power / abs(mean_power)) * 100, 2) else 0,
        .groups = "drop"
    ) %>%
    filter(n_requests >= 5) %>%
    arrange(sd_power)

most_stable <- sustainability %>% slice_min(sd_power, n = 1)
most_stable
```

    # A tibble: 1 × 10
      cluster n_requests unique_devices mean_power median_power sd_power min_power
      <chr>        <int>          <int>      <dbl>        <dbl>    <dbl>     <int>
    1 GIVC            13             13      -62.7          -63     5.22       -71
    # ℹ 3 more variables: max_power <int>, range_power <dbl>, cv_power <dbl>

## Вывод

В данной работе мы научились анализировать логи Wi-Fi с использованием
языка R
