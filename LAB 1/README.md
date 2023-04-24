
# Задание 1: Надите утечку данных из Вашей сети
	Надите утечку данных из Вашей сети Важнейшие документы с результатми нашей исследовательской деятельности в области создания вакцин скачиваются в виде больших заархивированных дампов. Один из хостов в нашей сети используется для пересылки этой информации – он пересылает гораздо больше информации на внешние ресурсы в Интернете, чем остальные компьютеры нашей сети. Определите его IP-адрес.

```{r}
library(dplyr)
library(vroom)
library(arrow)
```


1. Импортируем датасет
```{r}
df_data <- arrow::read_csv_arrow("gowiththeflow_20190826.csv")
```

2. Дадим имена признакам
```{r}
colnames(df_data) <- c('timestamp','src','dst','port','bytes')
head(df_data,10)
```

3. Переведём милисекунды в удобный формат даты и времени
```{r}
df_data$timestamp <- as.POSIXct(df_data$timestamp/1000, origin = "1970-01-01", tz = "GMT")
head(df_data)
```

4. Очистим датасет, оставив в src ip-адреса, только нашего предприятия
```{r}
knitr::opts_chunk$set(
  df_sorted <- df_data
)
```

```{r}
knitr::opts_chunk$set(
  df_sorted <- df_sorted[df_sorted$src > 11 & df_sorted$src < 15 & df_sorted$dst < 11 | df_sorted$dst > 15, ]
)
```
4. Найдём ip-адрес и максимальное число передаваемых байтов(ответ кто злоумышленник в организации)
```{r}
knitr::opts_chunk$set(
 found_ip <- df_sorted %>%
            group_by(src) %>%
            summarise(bytes = mean(bytes)),
found_ip <- found_ip[which.max(found_ip$bytes),],
print(found_ip) 
)
```

Ответ: 13.37.84.125


# Задание 2: Надите утечку данных 2
Другой атакующий установил автоматическую задачу в системном планировщике cron для экспорта содержимого внутренней wiki системы. Эта система генерирует большое количество траффика в нерабочие часы, больше чем остальные хосты. Определите IP этой системы. Известно, что ее IP адрес отличается от нарушителя из предыдущей задачи
1. 
```{r}
library("stringr") 

df_sorted$hour <- with (df_sorted,format(as.POSIXct(df_sorted$timestamp), format = "%H"))
df_sorted$minutes <- with (df_sorted,format(as.POSIXct(df_sorted$timestamp), format = "%M"))
```

2.
```{r}
#Поиск рабочих часов
activhours <- df_sorted %>% group_by(hour) %>% summarise(N = n())
select(arrange(activhours,desc(N)),N,hour)
```

3.
```{r}
knitr::opts_chunk$set(
  df_sorted2 <- df_sorted,
  toMatch <- c("12.","13.","14."),
  df_sorted2$src_info <- with (df_sorted2, str_detect(df_sorted2$src, paste(toMatch,collapse="|"))),
  df_sorted2$dst_info <- with (df_sorted2, str_detect(df_sorted2$dst, paste(toMatch,collapse="|")))
)
```

4. 
```{r}

found_ip2 <- df_sorted2 %>% 
  filter(src != "13.37.84.125") %>% #не адрес из 1 пункта
  filter(src_info == TRUE) %>% #исходящий трафик
  filter(dst_info == FALSE) %>%
  filter(hour >= 0) %>% #нерабочие часы
  filter(hour < 16) %>%
  group_by(src) %>%
  summarise(bytes = mean(bytes))
select(arrange(found_ip2,desc(bytes)) %>% top_n(1),src)

```

Ответ: 12.55.77.96











