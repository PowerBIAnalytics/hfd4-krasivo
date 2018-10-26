# BI-система команды KRACIVO для хакатона HFD4

## Команда
+ [Веретенников Павел](https://www.facebook.com/PavelVeret)
+ [Поклонский Артём](https://www.facebook.com/poclonsky)

## Цели и задачи
+ Извлечение знаний из группы FB
+ Создание масштабируемой системы, которую можно будет использовать в других проектах

# Описание BI-системы
## Визуализации и DAX
### Активность пользователей
На первой вкладке отчета расположены визуализации активности пользователей
+ График динамики новых пользователей
+ "Pie chart" с распределением по типу публикации
+ Накопительный график новых пользователей по дате первой активности
+ Распределение постов по пользователям
+ Матрица когорт, на которую можно вывести посты/комментарии и кол-во активных пользователей

На накопительном графике выводится мера, которая рассчитывается следующей формулой DAX:
```dax
Cumulative Users = 
CALCULATE (
    [USERS];
    FILTER (
        ALL ( 'Calendar_users'[Date] );
        'Calendar_users'[Date] <= MAX ( 'Calendar_users'[Date] )
    )
)
```

### Контент
На второй вкладке мы выводим график кол-ва постов в динамике, рейтинг доменов в тексте постов и таблицу с постами и комментариями. (достаточно бесполезная вкладка)

### Вопросы
На вкладке "Вопросы" выводится самая функциональная часть нашей BI-системы.
Тут мы выводим список вопросов, которые пользователи задавали в исследуемой группе.


### Статьи


### Дополнительно 
На всех графиках есть возможность переключения детализации по дням, неделям и месяцам


## Данные и Power Query
