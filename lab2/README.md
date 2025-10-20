# Основы обработки данных с помощью R и Dplyr
artogal@yandex.ru

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета `dplyr` – функции
    `select(), filter(), mutate(), arrange(), group_by()`

## Исходные данные

1.  Программное обеспечение Windows 10 Pro
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.1
4.  Пакет dplyr

## План

1.  Проанализировать встроенный в пакет dplyr набор данных starwars
2.  Выполнить аталитические задачи с помощью языка R

##Шаги

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
     [5] htmltools_0.5.8.1 yaml_2.3.10       rmarkdown_2.29    knitr_1.50       
     [9] jsonlite_2.0.0    xfun_0.53         digest_0.6.37     rlang_1.1.6      
    [13] evaluate_1.0.5   

###Шаг 1. Анализ встроенного в пакет `dplyr` набор данных `starwars` с
помощью языка R.

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
starwars
```

    # A tibble: 87 × 14
       name     height  mass hair_color skin_color eye_color birth_year sex   gender
       <chr>     <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
     1 Luke Sk…    172    77 blond      fair       blue            19   male  mascu…
     2 C-3PO       167    75 <NA>       gold       yellow         112   none  mascu…
     3 R2-D2        96    32 <NA>       white, bl… red             33   none  mascu…
     4 Darth V…    202   136 none       white      yellow          41.9 male  mascu…
     5 Leia Or…    150    49 brown      light      brown           19   fema… femin…
     6 Owen La…    178   120 brown, gr… light      blue            52   male  mascu…
     7 Beru Wh…    165    75 brown      light      blue            47   fema… femin…
     8 R5-D4        97    32 <NA>       white, red red             NA   none  mascu…
     9 Biggs D…    183    84 black      light      brown           24   male  mascu…
    10 Obi-Wan…    182    77 auburn, w… fair       blue-gray       57   male  mascu…
    # ℹ 77 more rows
    # ℹ 5 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>

###Шаг 2. Ответы на вопросы

1.Сколько строк в датафрейме?

``` r
starwars %>% nrow()
```

    [1] 87

2\. Сколько столбцов в датафрейме?

``` r
starwars %>% ncol()
```

    [1] 14

3\. Как просмотреть примерный вид датафрейма?

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

4\. Сколько уникальных рас персонажей (species) представлено в данных?

``` r
length(unique(starwars$species))
```

    [1] 38

5\. Найти самого высокого персонажа.

``` r
starwars %>% 
  arrange(desc(height)) %>% 
  slice(1)
```

    # A tibble: 1 × 14
      name      height  mass hair_color skin_color eye_color birth_year sex   gender
      <chr>      <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
    1 Yarael P…    264    NA none       white      yellow            NA male  mascu…
    # ℹ 5 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>

6\. Найти всех персонажей ниже 170

``` r
starwars %>% 
  filter(height<170) %>%
  select(name, height, species)
```

    # A tibble: 22 × 3
       name                  height species       
       <chr>                  <int> <chr>         
     1 C-3PO                    167 Droid         
     2 R2-D2                     96 Droid         
     3 Leia Organa              150 Human         
     4 Beru Whitesun Lars       165 Human         
     5 R5-D4                     97 Droid         
     6 Yoda                      66 Yoda's species
     7 Mon Mothma               150 Human         
     8 Wicket Systri Warrick     88 Ewok          
     9 Nien Nunb                160 Sullustan     
    10 Watto                    137 Toydarian     
    # ℹ 12 more rows

7\. Подсчитать ИМТ (индекс массы тела) для всех персонажей. ИМТ
подсчитать по формуле

``` r
starwars %>% 
  mutate("IMT" = mass/(height*height)) %>% 
  select(name,IMT)
```

    # A tibble: 87 × 2
       name                   IMT
       <chr>                <dbl>
     1 Luke Skywalker     0.00260
     2 C-3PO              0.00269
     3 R2-D2              0.00347
     4 Darth Vader        0.00333
     5 Leia Organa        0.00218
     6 Owen Lars          0.00379
     7 Beru Whitesun Lars 0.00275
     8 R5-D4              0.00340
     9 Biggs Darklighter  0.00251
    10 Obi-Wan Kenobi     0.00232
    # ℹ 77 more rows

8\. Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по
отношению массы (mass) к росту (height) персонажей.

``` r
starwars %>% 
  mutate("MH" = mass/height) %>% 
  arrange(desc(MH)) %>%
  slice(1:10)
```

    # A tibble: 10 × 15
       name     height  mass hair_color skin_color eye_color birth_year sex   gender
       <chr>     <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
     1 Jabba D…    175  1358 <NA>       green-tan… orange         600   herm… mascu…
     2 Grievous    216   159 none       brown, wh… green, y…       NA   male  mascu…
     3 IG-88       200   140 none       metal      red             15   none  mascu…
     4 Owen La…    178   120 brown, gr… light      blue            52   male  mascu…
     5 Darth V…    202   136 none       white      yellow          41.9 male  mascu…
     6 Jek Ton…    180   110 brown      fair       blue            NA   <NA>  <NA>  
     7 Bossk       190   113 none       green      red             53   male  mascu…
     8 Tarfful     234   136 brown      brown      blue            NA   male  mascu…
     9 Dexter …    198   102 none       brown      yellow          NA   male  mascu…
    10 Chewbac…    228   112 brown      unknown    blue           200   male  mascu…
    # ℹ 6 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>, MH <dbl>

9\. Найти средний возраст персонажей каждой расы вселенной Звездных войн

``` r
starwars %>% 
  group_by(species) %>% 
  filter(!is.na(birth_year)) %>% 
  summarise(avg = mean(birth_year)) %>% 
  select(species, avg)
```

    # A tibble: 15 × 2
       species          avg
       <chr>          <dbl>
     1 Cerean          92  
     2 Droid           53.3
     3 Ewok             8  
     4 Gungan          52  
     5 Human           53.7
     6 Hutt           600  
     7 Kel Dor         22  
     8 Mirialan        49  
     9 Mon Calamari    41  
    10 Rodian          44  
    11 Trandoshan      53  
    12 Twi'lek         48  
    13 Wookiee        200  
    14 Yoda's species 896  
    15 Zabrak          54  

10\. Найти самый распространенный цвет глаз персонажей вселенной
Звездных

войн.

``` r
starwars %>% 
  group_by(eye_color) %>%
  filter(!is.na(eye_color)) %>%
  summarise(k = n()) %>%
  arrange(desc(k)) %>%
  slice(1)
```

    # A tibble: 1 × 2
      eye_color     k
      <chr>     <int>
    1 brown        21

``` r
starwars %>% 
  group_by(species) %>%
  filter(!is.na(name)) %>%
  summarise(name_length = mean(nchar(name)))
```

    # A tibble: 38 × 2
       species   name_length
       <chr>           <dbl>
     1 Aleena          12   
     2 Besalisk        15   
     3 Cerean          12   
     4 Chagrian        10   
     5 Clawdite        10   
     6 Droid            4.83
     7 Dug              7   
     8 Ewok            21   
     9 Geonosian       17   
    10 Gungan          11.7 
    # ℹ 28 more rows

## Оценка результата

В результате лабораторной работы мы:

1.  Получили практические навыки использования языка программирования R
    для обработки данных

2.  Закрепили знания базовых типов данных языка R

3.  Получили практические навыки использования функции обработки данных
    пакета dplyr

## Вывод

В результате выполнения работы были освоены основные инструменты
обработки данных пакета dplyr языка R.
