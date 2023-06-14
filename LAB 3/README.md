
# Задание 3: Найдите утечку данных


Нужно найти только те порты, на которые отправлено меньше всего данных

dataset %>%
  select(src, dst, bytes,port) %>%
  mutate(outside_traffic = (str_detect(src,"^((12|13|14)\\.)") & !str_detect(dst,"^((12|13|14)\\.)"))) %>%
  filter(outside_traffic == TRUE) %>%
  group_by(port) %>%
  summarise(total_data=sum(bytes)) %>%
  filter(total_data < 5*10^9) %>%
  select(port) %>%
  collect() -> ports

ports <- unlist(ports)
ports <- as.vector(ports,'numeric')

Выбираем данные с нужными номерами портов
dataset %>%
  select(src, dst, bytes,port) %>%
  mutate(outside_traffic = (str_detect(src,"^((12|13|14)\\.)") & !str_detect(dst,"^((12|13|14)\\.)"))) %>%
  filter(outside_traffic == TRUE) %>%
  filter(port %in% ports) %>%
  group_by(src,port) %>%
  summarise(total_bytes=sum(bytes)) %>%
  arrange(desc(port)) %>%
  collect() -> df


Порты с маскимальным кол-вом данных
df %>%
  group_by(src, port) %>%
  summarise(total_data=sum(total_bytes)) %>%
  arrange(desc(total_data)) %>%
  head(10) %>%
  collect()

Количество хостов к портам
df %>%
  group_by(port) %>%
  summarise(hosts=n()) %>%
  arrange(hosts) %>%
  head(10) %>%
  collect()

Из предыдущих шагов следует вывод, что ip-адрес злоумышленника 12.55.77.96, а порт 31, т.к. из таблицы в 5 пункте видно, что 31 порт использовал только 1 хост и в тоже время из таблицы в 4 пункте видно, что больше всего данных было передано именно по этому порту
df %>%
  filter(port == 31) %>%
  group_by(src) %>%
  summarise(total_data=sum(total_bytes)) %>%
  collect()
Ответ: 12.55.77.96
