---
title: "Project"
author: "Anna"
date: "2022-12-14"
output:
  html_document:
    df_print: paged
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
library (dplyr)
library (maditr)
library (flextable)
library (tidyr)
library (tibble)
library (stringr)
library (readr)
library (ggplot2)
library(ggpubr)
library (rstatix)

```
# Удаление случая, если есть пропуск в признаке

 

 

```{r}
df1 <- read_csv("df1_csv.csv")
summary (df1)
```


## Создадим новый датафрейм, удалив все случаи, где есть пропуск в признаке

```{r}
df1_na <- df1 %>% 
  filter(!is.na(`Длина тела`)& !is.na(`Масса тела`) & !is.na(`Мозг`)& !is.na(`Сердце`)& !is.na(`Лёгкие`)& !is.na(`Печень`)& !is.na(`Селезёнка`)& !is.na(`Почки`)& !is.na(`Тимус`)& !is.na(`Надпочечники`)& !is.na(`Поджелудочная железа`)& !is.na(`Плацента`)& !is.na(`Окружность головы`)& !is.na(`Окружность груди`))
summary (df1_na)
```

 
  
## Описательная статистика датафрейма df1_na

 
```{r}
statistics <- list(
      `Количество субъектов` = ~length(.x),
      `Количество (есть данные)` = ~sum(!is.na(.x)),
      `Нет данных` = ~sum(is.na(.x)),
      `Ср. знач.` = ~ifelse(sum(!is.na(.x)) == 0, "Н/П*", mean(.x, na.rm = TRUE) %>% round(2) %>% as.character()),
      `Станд. отклон.` = ~ifelse(sum(!is.na(.x)) < 3, "Н/П*", sd(.x, na.rm = TRUE) %>% round(2) %>% as.character()),
      `мин. - макс.` = ~ifelse(sum(!is.na(.x)) == 0, "Н/П*", paste0(min(.x, na.rm = TRUE) %>% round(2), " - ", max(.x, na.rm = TRUE) %>% round(2))),
      `Медиана` = ~ifelse(sum(!is.na(.x)) == 0, "Н/П*", median(.x, na.rm = TRUE) %>% round(2) %>% as.character()),
      `Q1 - Q3` = ~ifelse(sum(!is.na(.x)) == 0, "Н/П*", paste0(quantile(.x, 0.25, na.rm = TRUE) %>% round(2), " - ", quantile(.x, 0.75, na.rm = TRUE) %>% round(2)))
)
df1_na_all <- df1_na %>%
  mutate(`мёртвый/живой` = "Всего")
df1_na_work <- rbind(df1_na, df1_na_all)
stat_num_res <- df1_na_work %>%
  select(`мёртвый/живой`, where(is.numeric)) %>%
  group_by(`мёртвый/живой`) %>%
  summarise(across(where(is.numeric), statistics)) %>%
  mutate(across(everything(), as.character)) %>% 
  pivot_longer(!`мёртвый/живой`) %>%
  separate(name, into  = c("Переменная","Статистика"), sep = "_") %>% 
  dcast(`Переменная` + `Статистика` ~ `мёртвый/живой`, value.var = "value") 
stat_num_res %>% 
  flextable() %>%
  theme_box() %>%
  merge_v(c( "Переменная")) %>%
  set_caption("Описательная статистика количественных переменных") 
```

 
## сравниваем два датафрейма 
 
### Гестация
```{r}
res.length <- t.test(df1$`Гестация`, df1_na$`Гестация`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```
```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Гестация` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Гестация`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("гестация, нед.")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))
  
```
 
### Длина тела
```{r}
res.length <- t.test(df1$`Длина тела`, df1_na$`Длина тела`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Длина тела` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Длина тела`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
   theme_gray()+
  xlab("Датафрейм")+
  ylab("длина тела, см")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))
  
```
 
### Масса тела
```{r}
res.length <- t.test(df1$`Масса тела`, df1_na$`Масса тела`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Масса тела` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Масса тела`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("масса тела, г")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))
  
```
 
### Мозг
```{r }
res.length <- t.test(df1$`Мозг`, df1_na$`Мозг`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Мозг` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Мозг`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("мозг, г")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))
  
```

 
### Сердце
```{r }
res.length <- t.test(df1$`Сердце`, df1_na$`Сердце`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Сердце` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Сердце`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("сердце, г")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))+
  scale_y_continuous (breaks = c (0,20,40,60, 80), limits = c(0,70))
  
```
 
### Лёгкие
```{r}
res.length <- t.test(df1$`Лёгкие`, df1_na$`Лёгкие`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Лёгкие` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Лёгкие`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("лёгкие, г")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))
  
```
 
### Печень
```{r}
res.length <- t.test(df1$`Печень`, df1_na$`Печень`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Печень` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
      aes(x = `name`, y = `Печень`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("печень, г")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))
  
```
 
## Селезёнка
```{r}
res.length <- t.test(df1$`Селезёнка`, df1_na$`Селезёнка`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Селезёнка` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Длина тела`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("селезёнка, г")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))
  
```
 
### Почки
```{r}
res.length <- t.test(df1$`Почки`, df1_na$`Почки`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Почки` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Почки`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("почки, г")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))+
  scale_y_continuous (breaks = c (0,10,20,30,40,50,60), limits = c(0,60))
  
```
### Тимус
```{r}
res.length <- t.test(df1$`Тимус`, df1_na$`Тимус`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Тимус` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Тимус`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("тимус, г")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))
  
```
### Надпочечники
```{r}
res.length <- t.test(df1$`Надпочечники`, df1_na$`Надпочечники`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Надпочечники` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Надпочечники`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("Надпочечники, г")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))
  
```
### Поджелудочная железа
```{r}
res.length <- t.test(df1$`Поджелудочная железа`, df1_na$`Поджелудочная железа`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Поджелудочная железа` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
      aes(x = `name`, y = `Поджелудочная железа`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("Поджелудочная железа, г")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))

```
 
 
### Плацента
```{r}
res.length <- t.test(df1$`Плацента`, df1_na$`Плацента`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Плацента` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Плацента`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("плацента, г")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))

```
 
### Окружность головы
```{r}
res.length <- t.test(df1$`Окружность головы`, df1_na$`Окружность головы`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Окружность головы` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Окружность головы`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("окружность головы, см")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))
  
```
 
### Окружность груди
```{r}
res.length <- t.test(df1$`Окружность груди`, df1_na$`Окружность груди`, alternative="two.sided")
res.length$p.value
res.length$conf.int

```

```{r}

lst <- list(df1, df1_na)
lst <- lapply(1:length(lst), function(x){
  lst[[x]] <- lst[[x]] %>%
    dplyr::mutate(name = paste("DF",x))
})
whole <- do.call(rbind, lst)
whole$name <- as.factor(whole$name)
  
stat.test <- whole %>% t_test(`Окружность груди` ~ name, ref.group = "DF 1")
stat.test <- stat.test %>% add_xy_position (x = "name")
ggplot(data = whole, 
       aes(x = `name`, y = `Окружность груди`, fill=`name`,alpha=0.9)) + 
  geom_violin()+ 
  stat_summary(fun.data=mean_sdl, geom="pointrange", color="red")+
  theme_gray()+
  xlab("Датафрейм")+
  ylab("окружность груди, см")+
  theme(axis.title.y = element_text (size=12, margin = margin(l = 10, r = 10)))+
  theme(axis.title.x = element_text (size=12, margin = margin(t = 10, b = 10)))+
  theme(axis.text.x = element_text(size=10, ))+
  theme(legend.position = "none")+
  scale_x_discrete(labels=c('Исходный', 'Удаление всех пропусков'))
  
```



