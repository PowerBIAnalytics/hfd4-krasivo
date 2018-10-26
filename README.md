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
На первой вкладке отчета расположены визуализации активности пользователей. Отсчет активности пользователей начинается с момента первой публикации поста / комментария.

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
Переключение метрик на когортной матрице происходит с помощью конструкции SWITCH:
```
Metrics Selected = 
SWITCH( TRUE();
    [Selected] = "Пользователи"; [USERS];
    [Selected] = "Посты и комментарии"; [Посты];
'Users (Posters)'[USERS])
```

### Контент
На второй вкладке мы выводим график кол-ва постов в динамике, рейтинг доменов в тексте постов и таблицу с постами и комментариями. (достаточно бесполезная вкладка)

### Вопросы
На вкладке "Вопросы" выводится самая функциональная часть нашей BI-системы. Тут мы выводим список вопросов, которые пользователи задавали в исследуемой группе. Определение типа поста происходит при помощи простой скоринговой модели - каждому посту начисляются баллы по вхождению "вопросительных" слов (подробнее об этом дальше). А с помощью фильтров можно сделать выборку по скорингу.

Кроме того, простым алгоритмом определяется, решен ли вопрос - если среди комментариев к вопросу есть те, в которых используются слова благодарности, вопрос помечается как решенный.

Скоринг и определение комментариев-благодарностей происходит на уровне Power Query. С помощью DAX вопрос помечается решенным:

```
//Определяем количество комментариев со словами благодарности
Thank You = 
CALCULATE([Comments];
    FILTER('Comments';'Comments'[Благодарочка] = "TRUE")
)
```
```
//Если "благодарственных" комментариев больше 0, помечаем вопрос решенным
Вопрос решен = 
IF([Thank You]>0;
    "Решен";
    BLANK()
)
```

### Статьи
На вкладке "Статьи" выводим посты, в которых пользователь делится полезными материалами.

Определение статей происходит по схожему алгоритму с определением постов-благодарностей. На уровне Power Query определяется, есть ли вхождения слов из словаря. 

На этой вкладке в таблице выводится кол-во комментариев, реакций и перепостов.

### Дополнительно 
+ На всех графиках есть возможность переключения детализации по дням, неделям и месяцам
+ На вкладках "Вопросы" и "Статьи" есть фильтр-поиск по постам


## Данные и Power Query

(Мы сделали отдельный параметр 'SourceFolder', чтобы вы могли указать в нем путь до папки с данными хакатона на вашем компьютере)

### Функции
+ Функция 'Поиск слов' - определение вхождений слов из словаря:
```
(keys as list, text as text) as logical =>
let
    Source = List.AnyTrue(List.Transform(keys, (key)=>Text.Contains(text, key, Comparer.OrdinalIgnoreCase)))
in
    Source
```
1. Возвращает TRUE в столбце  'Статья' запроса 'GroupPostsComments' в случае, если в тексте поста ('Message') содержится хотя бы одно слово из запроса 'Словарь - Статьи'
2. Возвращает TRUE в столбце "Благодарочка" запроса 'Comments' в случае, если в тексте поста ('Message') содержится хотя бы одно слово из запроса 'Словарь - Благодарность'

+ Функция 'Скоринг' - определение **количества** вхождений слов из словаря

```
= (keys as list, text as text) as any =>
let
    Source = List.Count(List.FindText(List.Transform(keys, (key)=>Logical.ToText(Text.Contains(text, key, Comparer.OrdinalIgnoreCase))),"true"))
in
    Source
```
1. Возвращает в стлбце 'Скоринг 1' запроса 'GroupPostsComments' количество вхождений слов из словаря 'Словарь - вопрос'
2. Возвращает в стлбце 'Скоринг 2' запроса 'GroupPostsComments' количество вхождений слов из словаря 'Словарь - вопрос - type 2'

### Расчет скоринга
Смысл создания двух словарей в том, чтобы можно было учесть как очевидно вопросительный слова - "Вопрос", "Вопросик", "Подскажите" и т.д., так и менее очевидно вопросительных - "Кто-нибудь", "Нужен", "Не могу" и т.д.

После определения количества вхождений, столбцы 'Скоринг 1' и 'Скоринг 2' умножаются на 15 и 5 соответственно. Это сделано для того, чтобы дифференцировать влияние вхождения слов из этих словарей. Значения коэффициентов можно легко поменять с помощью параметров 'score type 1' и 'score type 2'.

### Пополнение и корректировка словарей
Словари созданы с помощью 'Enter Data' внутри Power Query, так что их достаточно просто пополнять и корректировать.

## Итог
В рамках выполнения задания для хакатона мы попытались с помощью достаточно простого алгоритма определить посты из группы к статьям или вопросам. В условиях ограниченного времени мы сосредоточились на создании алгоритма и удобного (KRACIVOго) отчета для пользователя, а не на пополнении словарей. Это не законченная BI-система, подразумевается, что словари будут пополняться.

Также важным моментом для нас была возможность применения этого алгоритма к другим набрам данных. Мы считаем эту задачу выполненной, как и задачу от организаторов - извлечение знаний из группы.


Спасибо ¯\\_(ツ)_/¯
