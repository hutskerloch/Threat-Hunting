# Исследование метаданных DNS трафика
artogal@yandex.ru

## Цель работы

1.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания основных функций обработки данных экосистемы
    `tidyverse` языка R
3.  Закрепить навыки исследования метаданных DNS трафика

## Исходные данные

1.  Компьютер
2.  ОС Windows
3.  Rstudio
4.  Библиотека `dply`
5.  Github

## План

1.  Импортируйте данные DNS –
    https://storage.yandexcloud.net/dataset.ctfsec/dns.zip
2.  Добавьте пропущенные данные о структуре данных (назначении столбцов)
3.  Преобразуйте данные в столбцах в нужный формат
4.  Просмотрите общую структуру данных с помощью функции `glimpse()`
5.  Сколько участников информационного обмена в сети Доброй Организации?
6.  Какое соотношение участников обмена внутри сети и участников
    обращений к внешним ресурсам?
7.  Найдите топ-10 участников сети, проявляющих наибольшую сетевую
    активность.
8.  Найдите топ-10 доменов, к которым обращаются пользователи сети и
    соответственное количество обращений
9.  Опеределите базовые статистические характеристики (функция
    `summary()` ) интервала времени между последовательными обращениями
    к топ-10 доменам.
10. Часто вредоносное программное обеспечение использует DNS канал в
    качестве канала управления, периодически отправляя запросы на
    подконтрольный злоумышленникам DNS сервер. По периодическим запросам
    на один и тот же домен можно выявить скрытый DNS канал. Есть ли
    такие IP адреса в исследуемом датасете?
11. Определите местоположение (страну, город) и организацию-провайдера
    для топ-10 доменов. Для этого можно использовать сторонние сервисы,
    например http://ip-api.com (API-эндпоинт – http://ip-api.com/json).

## Шаги 1-4. Подготовка данных

### Шаг 1.Установка библиотек

``` r
print(sessionInfo())
```

    R version 4.5.1 (2025-06-13 ucrt)
    Platform: x86_64-w64-mingw32/x64
    Running under: Windows 11 x64 (build 26100)

    Matrix products: default
      LAPACK version 3.12.1

    locale:
    [1] LC_COLLATE=Russian_Russia.utf8  LC_CTYPE=Russian_Russia.utf8   
    [3] LC_MONETARY=Russian_Russia.utf8 LC_NUMERIC=C                   
    [5] LC_TIME=Russian_Russia.utf8    

    time zone: Europe/Moscow
    tzcode source: internal

    attached base packages:
    [1] stats     graphics  grDevices utils     datasets  methods   base     

    loaded via a namespace (and not attached):
     [1] compiler_4.5.1    fastmap_1.2.0     cli_3.6.5         tools_4.5.1      
     [5] htmltools_0.5.8.1 rstudioapi_0.17.1 yaml_2.3.10       rmarkdown_2.29   
     [9] knitr_1.50        jsonlite_2.0.0    xfun_0.53         digest_0.6.37    
    [13] rlang_1.1.6       evaluate_1.0.5   

    install.packages("tidyverse")
    install.packages("lubridate")
    install.packages("jsonlite")
    install.packages("httr")

``` r
library(tidyverse)
```

    Warning: пакет 'tidyverse' был собран под R версии 4.5.2

    Warning: пакет 'ggplot2' был собран под R версии 4.5.2

    Warning: пакет 'tidyr' был собран под R версии 4.5.2

    Warning: пакет 'readr' был собран под R версии 4.5.2

    Warning: пакет 'purrr' был собран под R версии 4.5.2

    Warning: пакет 'forcats' был собран под R версии 4.5.2

    Warning: пакет 'lubridate' был собран под R версии 4.5.2

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.4     ✔ readr     2.1.6
    ✔ forcats   1.0.1     ✔ stringr   1.5.2
    ✔ ggplot2   4.0.1     ✔ tibble    3.3.0
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ✔ purrr     1.2.0     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(lubridate)
```

### Шаг 2-3. Импорт данных и добавление пропущенных данных

``` r
dns_data <- read_tsv("dns.log", 
                     col_names = FALSE, 
                     comment = "#", 
                     show_col_types = FALSE)
                    


col_names <- c("ts", "uid", "id.orig_h", "id.orig_p", "id.resp_h", "id.resp_p", 
                  "proto", "trans_id", "query", "qclass", "qclass_name", "qtype", 
                  "qtype_name", "rcode", "rcode_name", "AA", "TC", "RD", "RA", 
                  "Z", "answers", "TTLs", "rejected")

colnames(dns_data) <- col_names
head(dns_data)
```

    # A tibble: 6 × 23
               ts uid   id.orig_h id.orig_p id.resp_h id.resp_p proto trans_id query
            <dbl> <chr> <chr>         <dbl> <chr>         <dbl> <chr>    <dbl> <chr>
    1 1331901006. CWGt… 192.168.…     45658 192.168.…       137 udp      33008 "*\\…
    2 1331901015. C36a… 192.168.…       137 192.168.…       137 udp      57402 "HPE…
    3 1331901016. C36a… 192.168.…       137 192.168.…       137 udp      57402 "HPE…
    4 1331901017. C36a… 192.168.…       137 192.168.…       137 udp      57402 "HPE…
    5 1331901006. C36a… 192.168.…       137 192.168.…       137 udp      57398 "WPA…
    6 1331901007. C36a… 192.168.…       137 192.168.…       137 udp      57398 "WPA…
    # ℹ 14 more variables: qclass <chr>, qclass_name <chr>, qtype <chr>,
    #   qtype_name <chr>, rcode <chr>, rcode_name <chr>, AA <lgl>, TC <lgl>,
    #   RD <lgl>, RA <lgl>, Z <dbl>, answers <chr>, TTLs <chr>, rejected <lgl>

### Шаг 4. Просмотр структуры данных

``` r
glimpse(dns_data)
```

    Rows: 427,935
    Columns: 23
    $ ts          <dbl> 1331901006, 1331901015, 1331901016, 1331901017, 1331901006…
    $ uid         <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jljz7Bs…
    $ id.orig_h   <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76", "19…
    $ id.orig_p   <dbl> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 1…
    $ id.resp_h   <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255", "1…
    $ id.resp_p   <dbl> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137…
    $ proto       <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp", "u…
    $ trans_id    <dbl> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 62187, 62…
    $ query       <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\…
    $ qclass      <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1"…
    $ qclass_name <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET", "C…
    $ qtype       <chr> "33", "32", "32", "32", "32", "32", "32", "32", "32", "32"…
    $ qtype_name  <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB…
    $ rcode       <chr> "0", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rcode_name  <chr> "NOERROR", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
    $ AA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ TC          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ RD          <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRU…
    $ RA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ Z           <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0…
    $ answers     <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ TTLs        <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rejected    <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…

## Шаги 5-10. Анализ данных

### Шаг 5. Сколько участников информационного обмена в сети Доброй Организации?

``` r
all_ips <- unique(c(dns_data$id.orig_h, dns_data$id.resp_h))
cat("Количество уникальных участников (IP-адресов):", length(all_ips), "\n")
```

    Количество уникальных участников (IP-адресов): 1359 

### Шаг 6. Какое соотношение участников обмена внутри сети и участников обращений к внешним ресурсам?

``` r
is_private <- function(ip) {
  # Проверка на частные адреса (RFC 1918)
  str_detect(ip, "^(10\\.|172\\.(1[6-9]|2[0-9]|3[0-1])\\.|192\\.168\\.)")
}

internal_ips <- all_ips[sapply(all_ips, is_private)]
external_ips <- setdiff(all_ips, internal_ips)


cat("Соотношение (внутр/внеш):", 
    round(length(internal_ips)/length(external_ips), 2), "\n\n")
```

    Соотношение (внутр/внеш): 13.77 

### Шаг 7. Найдите топ-10 участников сети, проявляющих наибольшую сетевую активность.

``` r
top_clients <- dns_data %>%
  count(id.orig_h, name = "request_count") %>%
  arrange(desc(request_count)) %>%
  head(10)

print(top_clients)
```

    # A tibble: 10 × 2
       id.orig_h       request_count
       <chr>                   <int>
     1 10.10.117.210           75943
     2 192.168.202.93          26522
     3 192.168.202.103         18121
     4 192.168.202.76          16978
     5 192.168.202.97          16176
     6 192.168.202.141         14967
     7 10.10.117.209           14222
     8 192.168.202.110         13372
     9 192.168.203.63          12148
    10 192.168.202.106         10784

### Шаг 8. Найдите топ-10 доменов, к которым обращаются пользователи сети и соответственное количество обращений

``` r
top_domains <- dns_data %>%
  filter(!is.na(query)) %>%
  count(query, name = "request_count") %>%
  arrange(desc(request_count)) %>%
  head(10)

print(top_domains)
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

### Шаг 9. Опеределите базовые статистические характеристики (функция summary() ) интервала времени между последовательными обращениями к топ-10 доменам.

``` r
top_domain_names <- top_domains$query

time_intervals <- dns_data %>%
  filter(query %in% top_domain_names) %>%
  arrange(query, ts) %>%
  group_by(query) %>%
  mutate(time_diff = as.numeric(ts - lag(ts))) %>%
  filter(!is.na(time_diff))


summary_stats <- time_intervals %>%
  group_by(query) %>%
  summarise(
    min = min(time_diff),
    q1 = quantile(time_diff, 0.25),
    median = median(time_diff),
    mean = mean(time_diff),
    q3 = quantile(time_diff, 0.75),
    max = max(time_diff)
  )
print(summary_stats)
```

    # A tibble: 10 × 7
       query                                    min    q1 median  mean     q3    max
       <chr>                                  <dbl> <dbl>  <dbl> <dbl>  <dbl>  <dbl>
     1 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00…     0 0.150  0.5   11.2   1.5   52724.
     2 "44.206.168.192.in-addr.arpa"              0 2.09   4     16.0  20.1   49680.
     3 "HPE8AA67"                                 0 0.75   0.75  16.6  25.5   50044.
     4 "ISATAP"                                   0 0.75   0.760 17.5   1.05  51998.
     5 "WPAD"                                     0 0.75   0.75  12.6   1.11  50049.
     6 "safebrowsing.clients.google.com"          0 0      1     10.0   2.01  49952.
     7 "teredo.ipv6.microsoft.com"                0 0      0      2.94  0.510 50388.
     8 "time.apple.com"                           0 0.370  1.76   8.67  4.72  50924.
     9 "tools.google.com"                         0 0      0      8.19  1     50365.
    10 "www.apple.com"                            0 0      1      8.61  3.01  50964.

### Шаг 10. Поиск IP-адресов с периодическими запросами на один домен (из топ-10 доменов)

``` r
find_periodic_requests <- function(data, threshold = 0.9) {
  # Функция для поиска IP с периодическими запросами
  suspicious_ips <- data.frame()
  
  unique_ips <- unique(data$id.orig_h)
  
  for(ip in unique_ips) {
    ip_data <- data %>%
      filter(id.orig_h == ip) %>%
      arrange(ts)
    
    if(nrow(ip_data) > 10) {
      # Вычисляем интервалы между запросами
      ip_data <- ip_data %>%
        mutate(time_diff = c(NA, diff(as.numeric(ts))))
      
      # Ищем одинаковые домены с регулярными интервалами
      domain_patterns <- ip_data %>%
        filter(!is.na(time_diff)) %>%
        group_by(query) %>%
        summarise(
          request_count = n(),
          sd_interval = sd(time_diff),
          mean_interval = mean(time_diff)
        ) %>%
        filter(request_count > 5, sd_interval < mean_interval * 0.3)
      
      if(nrow(domain_patterns) > 0) {
        for(i in 1:nrow(domain_patterns)) {
          suspicious_ips <- bind_rows(suspicious_ips, 
            data.frame(
              ip = ip,
              domain = domain_patterns$query[i],
              requests = domain_patterns$request_count[i],
              mean_interval = round(domain_patterns$mean_interval[i], 2),
              regularity = "Высокая"
            ))
        }
      }
    }
  }
  return(suspicious_ips)
}

suspicious <- find_periodic_requests(dns_data)
if(nrow(suspicious) > 0) {
  cat("Найдены подозрительные активности:\n")
  print(suspicious)
} else {
  cat("Явных признаков скрытых DNS-каналов не обнаружено\n")
}
```

    Найдены подозрительные активности:
                              ip
    1            192.168.202.100
    2             192.168.202.76
    3             192.168.202.89
    4             192.168.202.97
    5  fe80::ba8d:12ff:fe53:a8d8
    6             192.168.202.79
    7             192.168.202.79
    8             192.168.202.79
    9             192.168.202.79
    10            192.168.202.79
    11            192.168.202.79
    12            192.168.202.79
    13            192.168.202.79
    14            192.168.202.79
    15            192.168.202.79
    16            192.168.202.79
    17            192.168.202.94
    18  fe80::223:dfff:fe97:4e12
    19   fe80::c62c:3ff:fe37:efc
    20            192.168.202.88
    21            192.168.202.88
    22  fe80::c62c:3ff:fe30:7333
    23            192.168.204.70
    24 fe80::d69a:20ff:fef9:b49c
    25           192.168.202.108
    26           192.168.202.112
    27           192.168.202.112
    28            192.168.204.45
    29            192.168.204.45
    30            192.168.204.45
    31            192.168.204.45
    32 fe80::3e07:54ff:fe41:3ed3
    33             192.168.25.25
    34             192.168.28.25
    35           192.168.202.118
    36           192.168.202.109
    37            192.168.25.254
    38           192.168.202.125
    39              172.16.42.42
    40              172.16.42.42
    41              172.16.42.42
    42              172.16.42.42
    43              172.16.42.42
    44              172.16.42.42
    45              172.16.42.42
    46              172.16.42.42
    47              172.16.42.42
                                                                        domain
    1                                                           svn.apache.org
    2                                                                   TIRANI
    3                                                                     CWFS
    4                                                             I2.YTIMG.COM
    5                                        c02gn35udjwr._sftp-ssh._tcp.local
    6                                                     bugzilla.mozilla.org
    7                                                      bugzilla.redhat.com
    8                                                               dd.cron.ru
    9                                                      developer.mimer.com
    10                                                        httpd.apache.org
    11                                                            pear.php.net
    12                                                            pecl.php.net
    13                                                                 php.net
    14                                                     wiki.phpmyadmin.net
    15                                                            www.fpdf.org
    16                                                       www.wikipedia.org
    17                                                            www.iana.org
    18                                                            enigma.local
    19                                                              os-x.local
    20                                                1.48.16.172.in-addr.arpa
    21                                               1.95.168.192.in-addr.arpa
    22                                                             staff.local
    23                                                          ntp.ubuntu.com
    24                                                          phearnet.local
    25                                                        httpd.apache.org
    26                                                             example.com
    27                                                         www.example.com
    28                                                          www.google.com
    29                              www.google.com.hil-wasllhx.dca.wayport.net
    30                                                          www.ubuntu.com
    31                              www.ubuntu.com.hil-wasllhx.dca.wayport.net
    32                  chase\\xe2\\x80\\x99s macbook pro._sftp-ssh._tcp.local
    33 *\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00
    34                                                                       -
    35                                           sb-ssl.google.com.localdomain
    36                               safebrowsing.clients.google.com.zyxel.com
    37                                                      openmedstation.org
    38                                                              XPBASELINE
    39                                                                  GOOGLE
    40                                                                      I2
    41                                                                LOCALLOW
    42                                                          LOGIN.LIVE.COM
    43                                                              MACROMEDIA
    44                                                                  NORTON
    45                                                                  OFFICE
    46                                                                    PCDR
    47                                                            UI.SKYPE.COM
       requests mean_interval regularity
    1        14          2.36    Высокая
    2         6          0.77    Высокая
    3         6          0.70    Высокая
    4         6          0.19    Высокая
    5        31          0.28    Высокая
    6        16          4.40    Высокая
    7        16          4.41    Высокая
    8        16          0.58    Высокая
    9        16          4.40    Высокая
    10       16          0.59    Высокая
    11       16          4.41    Высокая
    12       16          4.41    Высокая
    13       16          2.67    Высокая
    14       32          4.29    Высокая
    15       16          4.41    Высокая
    16       16          4.42    Высокая
    17       32          4.87    Высокая
    18        9          0.27    Высокая
    19       30          0.29    Высокая
    20       17          0.95    Высокая
    21       17          0.94    Высокая
    22        9          0.25    Высокая
    23        8          5.03    Высокая
    24        6          0.26    Высокая
    25       32          4.69    Высокая
    26       16          4.82    Высокая
    27       16          5.00    Высокая
    28       23          0.49    Высокая
    29       22          0.49    Высокая
    30       24          0.42    Высокая
    31       24          0.40    Высокая
    32       13          0.26    Высокая
    33        8          1.51    Высокая
    34        6          0.92    Высокая
    35        8          5.00    Высокая
    36        8          5.00    Высокая
    37        7          5.00    Высокая
    38        7          0.75    Высокая
    39      414          0.82    Высокая
    40      418          0.81    Высокая
    41      429          0.80    Высокая
    42        6          0.40    Высокая
    43      432          0.82    Высокая
    44      417          0.82    Высокая
    45      405          0.79    Высокая
    46      401          0.81    Высокая
    47        6          0.19    Высокая

## Шаг 11. Обогащение данных

### Определите местоположение (страну, город) и организацию-провайдера для топ-10 доменов.

``` r
# Задача 9: Определите местоположение (страну, город) и организацию-провайдера для топ-10 доменов.
if (!exists("top_domains")) {
  top_domains <- dns_data %>%
    filter(!is.na(query), query != "") %>%
    count(query, name = "request_count") %>%
    arrange(desc(request_count)) %>%
    head(10)
}

top_10_domains <- top_domains$query
cat("Топ-10 доменов для геолокации:\n")
```

    Топ-10 доменов для геолокации:

``` r
print(top_10_domains)
```

     [1] "teredo.ipv6.microsoft.com"                                              
     [2] "tools.google.com"                                                       
     [3] "www.apple.com"                                                          
     [4] "time.apple.com"                                                         
     [5] "safebrowsing.clients.google.com"                                        
     [6] "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00"
     [7] "WPAD"                                                                   
     [8] "44.206.168.192.in-addr.arpa"                                            
     [9] "HPE8AA67"                                                               
    [10] "ISATAP"                                                                 

``` r
# Ваш улучшенный код функции get_geo_info
get_geo_info <- function(ip) {
  if (is.na(ip) || ip == "") {
     return(tibble(
      ip_address = NA_character_,
      country = "IP не определён",
      city = "IP не определён",
      isp = "IP не определён"
    ))
  }
  
  # Проверка на частные IP-адреса
  if (grepl("^(10\\.|192\\.168\\.|172\\.(1[6-9]|2[0-9]|3[0-1])\\.)", ip)) {
    return(tibble(
      ip_address = ip,
      country = "Частный IP",
      city = "Частный IP",
      isp = "Частный IP"
    ))
  }
  
  # Запрос к API
  url <- paste0("http://ip-api.com/json/", ip)
  
  response <- tryCatch({
    GET(url, timeout(10))
  }, error = function(e) {
    return(NULL)
  })
  
  if (is.null(response) || status_code(response) != 200) {
    return(tibble(
      ip_address = ip,
      country = "Ошибка API",
      city = "Ошибка API",
      isp = "Ошибка API"
    ))
  }
  
  data <- tryCatch({
    fromJSON(content(response, "text", encoding = "UTF-8"))
  }, error = function(e) {
    return(list(status = "fail", message = "JSON parse error"))
  })
  
  if (data$status == "success") {
    return(tibble(
      ip_address = ip,
      country = ifelse(is.null(data$country), "Неизвестно", data$country),
      city = ifelse(is.null(data$city), "Неизвестно", data$city),
      isp = ifelse(is.null(data$isp), "Неизвестно", data$isp),
      org = ifelse(is.null(data$org), "Неизвестно", data$org),
      lat = ifelse(is.null(data$lat), NA, data$lat),
      lon = ifelse(is.null(data$lon), NA, data$lon)
    ))
  } else {
    return(tibble(
      ip_address = ip,
      country = paste("API ошибка:", ifelse(is.null(data$message), data$status, data$message)),
      city = "Ошибка",
      isp = "Ошибка"
    ))
  }
}

# Функция для извлечения IP из столбца answers
extract_ip_from_answers <- function(answer_string) {
  if (is.na(answer_string) || answer_string == "" || answer_string == "-" || answer_string == "(empty)") {
    return(NA)
  }
  
  # Удаляем квадратные скобки и кавычки
  clean_string <- gsub("\\[|\\]|\"", "", answer_string)
  
  # Разделяем по запятой
  parts <- str_split(clean_string, ",")[[1]]
  
  # Паттерн для IPv4
  ipv4_pattern <- "\\b(?:[0-9]{1,3}\\.){3}[0-9]{1,3}\\b"
  
  for (part in parts) {
    part <- trimws(part)
    if (grepl(ipv4_pattern, part)) {
      ip_match <- regmatches(part, regexpr(ipv4_pattern, part))
      if (length(ip_match) > 0) {
        return(ip_match[1])
      }
    }
  }
  
  return(NA)
}

# Основная функция для получения геоинформации по домену
get_geo_by_domain <- function(domain) {
  cat("Обработка домена:", domain, "\n")
  
  # Пытаемся найти IP в столбце answers
  domain_data <- dns_data %>%
    filter(query == domain, !is.na(answers), 
           answers != "-", answers != "(empty)")
  
  if (nrow(domain_data) > 0) {
    # Берем первую запись с answers
    answer_string <- domain_data$answers[1]
    ip <- extract_ip_from_answers(answer_string)
    
    if (!is.na(ip)) {
      cat("  Найден IP в answers:", ip, "\n")
      geo_info <- get_geo_info(ip)
      
      return(tibble(
        domain = domain,
        ip_address = ip,
        country = geo_info$country,
        city = geo_info$city,
        isp = geo_info$isp,
        org = geo_info$org,
        source = "answers"
      ))
    }
  }
  
  # Если не нашли в answers, ищем в id.resp_h (DNS сервер)
  dns_server <- dns_data %>%
    filter(query == domain) %>%
    slice(1) %>%
    pull(id.resp_h)
  
  if (!is.na(dns_server) && dns_server != "") {
    cat("  Используем DNS сервер:", dns_server, "\n")
    geo_info <- get_geo_info(dns_server)
    
    return(tibble(
      domain = domain,
      ip_address = dns_server,
      country = geo_info$country,
      city = geo_info$city,
      isp = geo_info$isp,
      org = geo_info$org,
      source = "dns_server"
    ))
  }
  
  # Если ничего не нашли
  return(tibble(
    domain = domain,
    ip_address = NA_character_,
    country = "Не удалось определить IP",
    city = "Не удалось определить IP",
    isp = "Не удалось определить IP",
    org = "Не удалось определить IP",
    source = "none"
  ))
}

# Создаем список уникальных IP для геолокации
cat("\n=== ПОДГОТОВКА ДАННЫХ ДЛЯ ГЕОЛОКАЦИИ ===\n")
```


    === ПОДГОТОВКА ДАННЫХ ДЛЯ ГЕОЛОКАЦИИ ===

``` r
# Создаем таблицу домен->IP
domain_ip_map <- tibble()

for (domain in top_10_domains) {
  # Ищем IP в answers
  ip_candidates <- dns_data %>%
    filter(query == domain, !is.na(answers), 
           answers != "-", answers != "(empty)") %>%
    slice(1) %>%
    pull(answers)
  
  if (length(ip_candidates) > 0) {
    ip <- extract_ip_from_answers(ip_candidates[1])
    if (!is.na(ip)) {
      domain_ip_map <- bind_rows(domain_ip_map, 
                                 tibble(domain = domain, 
                                        destination_ip = ip,
                                        source = "answers"))
    }
  }
  
  # Если не нашли, берем DNS сервер
  if (!domain %in% domain_ip_map$domain) {
    dns_ip <- dns_data %>%
      filter(query == domain) %>%
      slice(1) %>%
      pull(id.resp_h)
    
    if (!is.na(dns_ip) && dns_ip != "") {
      domain_ip_map <- bind_rows(domain_ip_map,
                                 tibble(domain = domain,
                                        destination_ip = dns_ip,
                                        source = "dns_server"))
    }
  }
}
```

    Warning: Unknown or uninitialised column: `domain`.

``` r
cat("\nСопоставление доменов и IP:\n")
```


    Сопоставление доменов и IP:

``` r
print(domain_ip_map)
```

    # A tibble: 10 × 3
       domain                                                  destination_ip source
       <chr>                                                   <chr>          <chr> 
     1 "teredo.ipv6.microsoft.com"                             fec0:0:0:ffff… dns_s…
     2 "tools.google.com"                                      192.168.207.4  dns_s…
     3 "www.apple.com"                                         127.0.0.1      answe…
     4 "time.apple.com"                                        172.19.1.100   dns_s…
     5 "safebrowsing.clients.google.com"                       192.168.207.4  dns_s…
     6 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\… 192.168.27.203 dns_s…
     7 "WPAD"                                                  192.168.202.2… dns_s…
     8 "44.206.168.192.in-addr.arpa"                           192.168.207.4  dns_s…
     9 "HPE8AA67"                                              192.168.202.2… dns_s…
    10 "ISATAP"                                                192.168.202.2… dns_s…

``` r
# Запрашиваем геоданные для уникальных IP
cat("\n=== ЗАПРОС ГЕОДАННЫХ ===\n")
```


    === ЗАПРОС ГЕОДАННЫХ ===

``` r
geo_results_df <- tibble(
  ip_address = character(),
  country = character(),
  city = character(),
  isp = character(),
  org = character()
)

unique_ips_to_check <- unique(na.omit(domain_ip_map$destination_ip))
cat("Уникальных IP для проверки:", length(unique_ips_to_check), "\n")
```

    Уникальных IP для проверки: 6 

``` r
# Добавляем задержку между запросами, чтобы не превысить лимит API
for (i in seq_along(unique_ips_to_check)) {
  ip <- unique_ips_to_check[i]
  cat(sprintf("[%d/%d] Запрос геоданных для IP: %s\n", i, length(unique_ips_to_check), ip))
  
  geo_info_row <- get_geo_info(ip)
  geo_results_df <- bind_rows(geo_results_df, geo_info_row)
  
  # Пауза между запросами (чтобы не блокировали API)
  if (i < length(unique_ips_to_check)) {
    Sys.sleep(1.2)
  }
}
```

    [1/6] Запрос геоданных для IP: fec0:0:0:ffff::2
    [2/6] Запрос геоданных для IP: 192.168.207.4
    [3/6] Запрос геоданных для IP: 127.0.0.1
    [4/6] Запрос геоданных для IP: 172.19.1.100
    [5/6] Запрос геоданных для IP: 192.168.27.203
    [6/6] Запрос геоданных для IP: 192.168.202.255

``` r
cat("\nРезультаты геолокации IP:\n")
```


    Результаты геолокации IP:

``` r
print(geo_results_df)
```

    # A tibble: 6 × 5
      ip_address       country    city       isp        org  
      <chr>            <chr>      <chr>      <chr>      <chr>
    1 fec0:0:0:ffff::2 Ошибка API Ошибка API Ошибка API <NA> 
    2 192.168.207.4    Частный IP Частный IP Частный IP <NA> 
    3 127.0.0.1        Ошибка API Ошибка API Ошибка API <NA> 
    4 172.19.1.100     Частный IP Частный IP Частный IP <NA> 
    5 192.168.27.203   Частный IP Частный IP Частный IP <NA> 
    6 192.168.202.255  Частный IP Частный IP Частный IP <NA> 

``` r
# Объединяем домены с геоданными
cat("\n=== ФИНАЛЬНЫЕ РЕЗУЛЬТАТЫ ГЕОЛОКАЦИИ ===\n")
```


    === ФИНАЛЬНЫЕ РЕЗУЛЬТАТЫ ГЕОЛОКАЦИИ ===

``` r
domain_geo_info_final <- domain_ip_map %>%
  left_join(geo_results_df, by = c("destination_ip" = "ip_address")) %>%
  rename(ip_address = destination_ip) %>%
  select(domain, ip_address, country, city, isp, org, source)

# Добавляем количество запросов для каждого домена
domain_geo_info_final <- domain_geo_info_final %>%
  left_join(top_domains %>% select(query, request_count), 
            by = c("domain" = "query")) %>%
  arrange(desc(request_count))

# Выводим результаты
cat("\nГеолокация топ-10 доменов:\n")
```


    Геолокация топ-10 доменов:

``` r
cat("============================================================================================================\n")
```

    ============================================================================================================

``` r
cat(sprintf("%-40s %-20s %-15s %-20s %-30s\n", 
            "Домен", "Страна", "Город", "Провайдер", "IP адрес"))
```

    Домен                               Страна         Город      Провайдер   IP адрес                 

``` r
cat("============================================================================================================\n")
```

    ============================================================================================================

``` r
for (i in 1:nrow(domain_geo_info_final)) {
  row <- domain_geo_info_final[i, ]
  domain_display <- ifelse(nchar(row$domain) > 38, 
                          paste0(substr(row$domain, 1, 35), "..."), 
                          row$domain)
  
  cat(sprintf("%-40s %-20s %-15s %-20s %-30s\n",
              domain_display,
              substr(row$country, 1, 18),
              substr(row$city, 1, 13),
              substr(row$isp, 1, 18),
              row$ip_address))
}
```

    teredo.ipv6.microsoft.com                Ошибка API     Ошибка API Ошибка API     fec0:0:0:ffff::2              
    tools.google.com                         Частный IP    Частный IP Частный IP    192.168.207.4                 
    www.apple.com                            Ошибка API     Ошибка API Ошибка API     127.0.0.1                     
    time.apple.com                           Частный IP    Частный IP Частный IP    172.19.1.100                  
    safebrowsing.clients.google.com          Частный IP    Частный IP Частный IP    192.168.207.4                 
    *\x00\x00\x00\x00\x00\x00\x00\x00\x...   Частный IP    Частный IP Частный IP    192.168.27.203                
    WPAD                                     Частный IP    Частный IP Частный IP    192.168.202.255               
    44.206.168.192.in-addr.arpa              Частный IP    Частный IP Частный IP    192.168.207.4                 
    HPE8AA67                                 Частный IP    Частный IP Частный IP    192.168.202.255               
    ISATAP                                   Частный IP    Частный IP Частный IP    192.168.202.255               

## Оценка результата

В рамках практческой работы была исследована подозрительная сетевая
активность во внутренней сети Доброй Организации. Были восстановлены
недостающие метаданные и подготовлены ответы на вопросы.

## Вывод

Работа показала, как с помощью R и dplyr можно эффективно обрабатывать и
исследовать данные DNS-трафика, выявлять активных участников сети и
потенциально подозрительные IP-адреса.
