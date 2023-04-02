
# Задание 1: Надите утечку данных из Вашей сети

# Практика 1

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

3. Очистим датасет, оставив в src ip-адреса, только нашего предприятия
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