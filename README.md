# <center> <img src = https://raw.githubusercontent.com/AndreyRysistov/DatasetsForPandas/main/hh%20label.jpg alt="drawing" style="width:400px;">

# <center> Проект: Анализ резюме из HeadHunter

Скачать необходимые данные можно по ссылкам ([датасет](https://drive.google.com/file/d/1y8i16Y2LMnwPDjhyso8IkrNlEUBYBaR0/view?usp=drive_link) и [курсы валют](https://drive.google.com/file/d/1MIu3j5hSMsbucOtQeXuc7keIR1OLmhl_/view?usp=drive_link))

## Исследование структуры данных
1. Прочитайте данные с помощью библиотеки Pandas. Совет: перед чтением обратите внимание на разделитель внутри файла. 
```python
hh =pd.read_csv('/Users/macbook/Documents/GitHub/SF_Project_1/data/dst-3.0_16_1_hh_database.csv', sep=';')

hh_data = hh.copy()
hh_data.shape
```
Скачать файл можно по [ссылке](https://drive.google.com/file/d/1y8i16Y2LMnwPDjhyso8IkrNlEUBYBaR0/view?usp=drive_link)

2. Выведите несколько первых (последних) строк таблицы, чтобы убедиться в том, что ваши данные не повреждены. Ознакомьтесь с признаками и их структурой.
```python 
hh_data.head(2)
```
3. Выведите основную информацию о числе непустых значений в столбцах и их типах в таблице.
4. Обратите внимание на информацию о числе непустых значений.
```python
hh_data.info()
```
5. Выведите основную статистическую информацию о столбцах.

```python
hh_data.describe()
```

## Преобразование данных
1. Начнем с простого - с признака **"Образование и ВУЗ"**. Его текущий формат это: **<Уровень образования год выпуска ВУЗ специальность...>**. Например:
* Высшее образование 2016 Московский авиационный институт (национальный исследовательский университет)...
* Неоконченное высшее образование 2000  Балтийская государственная академия рыбопромыслового флота…
Нас будет интересовать только уровень образования.

Создайте с помощью функции-преобразования новый признак **"Образование"**, который должен иметь 4 категории: "высшее", "неоконченное высшее", "среднее специальное" и "среднее".

Выполните преобразование, ответьте на контрольные вопросы и удалите признак "Образование и ВУЗ".

Совет: обратите внимание на структуру текста в столбце **"Образование и ВУЗ"**. Гарантируется, что текущий уровень образования соискателя всегда находится в первых 2ух слов и начинается с заглавной буквы. Воспользуйтесь этим.

*Совет: проверяйте полученные категории, например, с помощью метода unique()*

```python
def get_education (data):

    #exclude_list = ["Высшее образование", "Неоконченное высшее", "Среднее специальное", "Среднее образование"]
    education_data = data.split(' ')
    education = ' '.join(education_data[0:2])
    return education

hh_data['Образование'] = hh_data['Образование и ВУЗ'].apply(get_education)

display(hh_data.groupby('Образование')['Образование'].count())

hh_data = hh_data.drop(['Образование и ВУЗ'], axis=1)
```
2. Теперь нас интересует столбец **"Пол, возраст"**. Сейчас он представлен в формате **<Пол , возраст , дата рождения >**. Например:
* Мужчина , 39 лет , родился 27 ноября 1979 
* Женщина , 21 год , родилась 13 января 2000
Как вы понимаете, нам необходимо выделить каждый параметр в отдельный столбец.

Создайте два новых признака **"Пол"** и **"Возраст"**. При этом важно учесть:
* Признак пола должен иметь 2 уникальных строковых значения: 'М' - мужчина, 'Ж' - женщина. 
* Признак возраста должен быть представлен целыми числами.

Выполните преобразование, ответьте на контрольные вопросы и удалите признак **"Пол, возраст"** из таблицы.

*Совет: обратите внимание на структуру текста в столбце, в части на то, как разделены параметры пола, возраста и даты рождения между собой - символом ' , '. 
Гарантируется, что структура одинакова для всех строк в таблице. Вы можете воспользоваться этим.*
```python
#Находим и выделяем пол соискателей
def get_sex(data):
    sex = data[0]
    return sex

hh_data['Пол'] = hh_data['Пол, возраст'].apply(get_sex)

women = round(hh_data['Пол'][hh_data['Пол'] == 'Ж'].count()/ hh_data.shape[0] *100, 2)
print(f'Процент женских резюме: {women}%')

#находим и выделяем возраст соискатилей
def get_age(data):
    age = int(data[11:13])
    return age

hh_data['Возраст'] = hh_data['Пол, возраст'].apply(get_age)

mean_age = round(hh_data['Возраст'].mean(),1)
print(f'Средний возраст соискателей: {mean_age}')

hh_data = hh_data.drop(['Пол, возраст'], axis=1)
hh_data.head(3)
```
3. Следующим этапом преобразуем признак **"Опыт работы"**. Его текущий формат - это: **<Опыт работы: n лет m месяцев, периоды работы в различных компаниях…>**. 

Из столбца нам необходимо выделить общий опыт работы соискателя в месяцах, новый признак назовем "Опыт работы (месяц)"

Для начала обсудим условия решения задачи:
* Во-первых, в данном признаке есть пропуски. Условимся, что если мы встречаем пропуск, оставляем его как есть (функция-преобразование возвращает NaN)
* Во-вторых, в данном признаке есть скрытые пропуски. Для некоторых соискателей в столбце стоит значения "Не указано". Их тоже обозначим как NaN (функция-преобразование возвращает NaN)
* В-третьих, нас не интересует информация, которая описывается после указания опыта работы (периоды работы в различных компаниях)
* В-четвертых, у нас есть проблема: опыт работы может быть представлен только в годах или только месяцах. Например, можно встретить следующие варианты:
    * Опыт работы 3 года 2 месяца…
    * Опыт работы 4 года…
    * Опыт работы 11 месяцев…
    * Учитывайте эту особенность в вашем коде

Учитывайте эту особенность в вашем коде

В результате преобразования у вас должен получиться столбец, содержащий информацию о том, сколько месяцев проработал соискатель.
Выполните преобразование, ответьте на контрольные вопросы и удалите столбец **"Опыт работы"** из таблицы.
```python
def get_experience(arg):

    if (arg is np.nan) or (arg == 'Не указано'):
        return None
    month_key_words =['месяц', 'месяцев', 'месяца']
    year_key_word = ['год', 'лет', 'года']
    args_splited = arg.split(' ')[:6]
    month = 0
    year = 0
    
    for i in range(len(args_splited)):
        if args_splited[i] in month_key_words:
            month = args_splited[i-1]
        if args_splited[i] in year_key_word:
            year = args_splited[i-1]
    return int(year)*12 + int(month)

hh_data['Опыт мес'] = hh_data['Опыт работы'].apply(get_experience)

hh_data = hh_data.drop(columns=['Опыт работы'])

print('Медианный опыт работы (в месяцах):', hh_data['Опыт мес'].median())
```
4. Хорошо идем! Следующий на очереди признак "Город, переезд, командировки". Информация в нем представлена в следующем виде: **<Город , (метро) , готовность к переезду (города для переезда) , готовность к командировкам>**. В скобках указаны необязательные параметры строки. Например, можно встретить следующие варианты:

* Москва , не готов к переезду , готов к командировкам
* Москва , м. Беломорская , не готов к переезду, не готов к командировкам
* Воронеж , готов к переезду (Сочи, Москва, Санкт-Петербург) , готов к командировкам

Создадим отдельные признаки **"Город"**, **"Готовность к переезду"**, **"Готовность к командировкам"**. При этом важно учесть:

* Признак **"Город"** должен содержать только 4 категории: "Москва", "Санкт-Петербург" и "город-миллионник" (их список ниже), остальные обозначьте как "другие".

    Список городов-миллионников:
    
   <code>million_cities = ['Новосибирск', 'Екатеринбург','Нижний Новгород','Казань', 'Челябинск','Омск', 'Самара', 'Ростов-на-Дону', 'Уфа', 'Красноярск', 'Пермь', 'Воронеж','Волгоград']
    </code>
    Инфорация о метро, рядом с которым проживает соискатель нас не интересует.
* Признак **"Готовность к переезду"** должен иметь два возможных варианта: True или False. Обратите внимание, что возможны несколько вариантов описания готовности к переезду в признаке "Город, переезд, командировки". Например:
    * … , готов к переезду , …
    * … , не готова к переезду , …
    * … , готова к переезду (Москва, Санкт-Петербург, Ростов-на-Дону)
    * … , хочу переехать (США) , …
    
    Нас интересует только сам факт возможности или желания переезда.
* Признак **"Готовность к командировкам"** должен иметь два возможных варианта: True или False. Обратите внимание, что возможны несколько вариантов описания готовности к командировкам в признаке "Город, переезд, командировки". Например:
    * … , готов к командировкам , … 
    * … , готова к редким командировкам , …
    * … , не готов к командировкам , …
    
    Нас интересует только сам факт готовности к командировке.
    
    Еще один важный факт: при выгрузки данных у некоторых соискателей "потерялась" информация о готовности к командировкам. Давайте по умолчанию будем считать, что такие соискатели не готовы к командировкам.
    
Выполните преобразования и удалите столбец **"Город, переезд, командировки"** из таблицы.

*Совет: обратите внимание на то, что структура текста может меняться в зависимости от указания ближайшего метро. Учите это, если будете использовать порядок слов в своей программе.*
``` python
def get_city (arg):
    million_cities = ['Новосибирск', 'Екатеринбург', 'Нижний Новгород', 'Казань', 'Челябинск', 'Омск', 'Самара', 'Ростов-на-Дону',
                       'Уфа', 'Красноярск', 'Пермь', 'Воронеж', 'Волгоград' ]
    
    city = arg.split(' ,')[0]
    if (city == 'Москва') or (city == 'Санкт-Петербург'):
        return city
    elif city in million_cities:
        return 'город-миллионник'
    else:
        return 'другие'

hh_data['Город'] = hh_data['Город, переезд, командировки'].apply(get_city)

percent_SP = round(hh_data['Город'].value_counts(normalize=True)['Санкт-Петербург']*100)
print(f'Процент соискателей живущих в Санкт-Петербурге: {percent_SP}%')

def get_ready_for_bisiness_trips(arg):
    if ('командировка' in arg):
        if ('не готов к командировкам' in arg) or('не готова к командировкам' in arg):
            return False
        else: 
            return True
    else:
        return False
    
hh_data['Готовность к командировкам'] = hh_data['Город, переезд, командировки'].apply(get_ready_for_bisiness_trips)

def get_relocation(arg):
    if ('не готов к переезду' in arg) or ('не готова к переезду' in arg):
        return False
    else:
        return True
    
hh_data['Готовность к переезду'] = hh_data['Город, переезд, командировки'].apply(get_relocation)

relocations_and_business = round(hh_data[['Готовность к переезду', 'Готовность к командировкам']].value_counts(normalize=True)[True, True] * 100)
print(f'Процентов соискателей готовы одновременно и к переездам, и к командировкам: {relocations_and_business}%')
hh_data = hh_data.drop(columns=['Город, переезд, командировки'])
```
5. Рассмотрим поближе признаки **"Занятость"** и **"График"**. Сейчас признаки представляют собой набор категорий желаемой занятости (полная занятость, частичная занятость, проектная работа, волонтерство, стажировка) и желаемого графика работы (полный день, сменный график, гибкий график, удаленная работа, вахтовый метод).
На сайте hh.ru соискатель может указывать различные комбинации данных категорий, например:
* полная занятость, частичная занятость
* частичная занятость, проектная работа, волонтерство
* полный день, удаленная работа
* вахтовый метод, гибкий график, удаленная работа, полная занятость

Такой вариант признаков имеет множество различных комбинаций, а значит множество уникальных значений, что мешает анализу. Нужно это исправить!

Давайте создадим признаки-мигалки для каждой категории: если категория присутствует в списке желаемых соискателем, то в столбце на месте строки рассматриваемого соискателя ставится True, иначе - False.

Такой метод преобразования категориальных признаков называется One Hot Encoding и его схема представлена на рисунке ниже:
<img src=https://raw.githubusercontent.com/AndreyRysistov/DatasetsForPandas/main/ohe.jpg>
Выполните данное преобразование для признаков "Занятость" и "График", ответьте на контрольные вопросы, после чего удалите их из таблицы
```python
def get_full_time_work (args):
    if ('полная занятость' in args): return True
    else: return False

def get_project_work (args):
    if ('проектная работа' in args): return True
    else: return False

def get_part_time_work (args):
    if ('частичная занятость' in args): return True
    else: return False

def get_internship (args):
    if ('стажировка' in args): return True
    else: return False

def get_volunteering (args):
    if ('волонтерство' in args): return True
    else: return False

hh_data['Полная занятость'] = hh_data['Занятость'].apply(get_full_time_work)
hh_data['Проектная работа'] = hh_data['Занятость'].apply(get_project_work)
hh_data['Частичная занятость'] = hh_data['Занятость'].apply(get_part_time_work)
hh_data['Стажировка'] = hh_data['Занятость'].apply(get_internship)
hh_data['Волонтерство'] = hh_data['Занятость'].apply(get_volunteering)

print(hh_data[['Проектная работа', 'Волонтерство']].value_counts()[True, True], 'людей ищут проектную работу и волонтёрство.')

hh_data['Полный день'] = hh_data['График'].apply(lambda x: True if 'полный день' in x else False)
hh_data['Вахтовый метод'] = hh_data['График'].apply(lambda x: True if 'вахтовый метод' in x else False)
hh_data['Гибкий график'] = hh_data['График'].apply(lambda x: True if 'гибкий график' in x else False)
hh_data['Удаленная работа'] = hh_data['График'].apply(lambda x: True if 'удаленная работа' in x else False)
hh_data['Сменный график'] = hh_data['График'].apply(lambda x: True if 'сменный график' in x else False)

print(hh_data[['Вахтовый метод', 'Гибкий график']].value_counts()[True, True], 'людей хотят работать вахтовым методом и с гибким графиком.')
hh_data = hh_data.drop(columns=['Занятость', 'График'])
```
6. (2 балла) Наконец, мы добрались до самого главного и самого важного - признака заработной платы **"ЗП"**. 
В чем наша беда? В том, что помимо желаемой заработной платы соискатель указывает валюту, в которой он бы хотел ее получать, например:
* 30000 руб.
* 50000 грн.
* 550 USD

Нам бы хотелось видеть заработную плату в единой валюте, например, в рублях. Возникает вопрос, а где взять курс валют по отношению к рублю?

На самом деле язык Python имеет в арсенале огромное количество возможностей получения данной информации, от обращения к API Центробанка, до использования специальных библиотек, например pycbrf. Однако, это не тема нашего проекта.

Поэтому мы пойдем в лоб: обратимся к специальным интернет-ресурсам для получения данных о курсе в виде текстовых файлов. Например, MDF.RU, данный ресурс позволяет удобно экспортировать данные о курсах различных валют и акций за указанные периоды в виде csv файлов. Мы уже сделали выгрузку курсов валют, которые встречаются в наших данных за период с 29.12.2017 по 05.12.2019. Скачать ее вы можете **на платформе**

Создайте новый DataFrame из полученного файла. В полученной таблице нас будут интересовать столбцы:
* "currency" - наименование валюты в ISO кодировке,
* "date" - дата, 
* "proportion" - пропорция, 
* "close" - цена закрытия (последний зафиксированный курс валюты на указанный день).


Перед вами таблица соответствия наименований иностранных валют в наших данных и их общепринятых сокращений, которые представлены в нашем файле с курсами валют. Пропорция - это число, за сколько единиц валюты указан курс в таблице с курсами. Например, для казахстанского тенге курс на 20.08.2019 составляет 17.197 руб. за 100 тенге, тогда итоговый курс равен - 17.197 / 100 = 0.17197 руб за 1 тенге.
Воспользуйтесь этой информацией в ваших преобразованиях.

<img src=https://raw.githubusercontent.com/AndreyRysistov/DatasetsForPandas/main/table.jpg>


Осталось только понять, откуда брать дату, по которой определяется курс? А вот же она - в признаке **"Обновление резюме"**, в нем содержится дата и время, когда соискатель выложил текущий вариант своего резюме. Нас интересует только дата, по ней бы и будем сопоставлять курсы валют.

Теперь у нас есть вся необходимая информация для того, чтобы создать признак "ЗП (руб)" - заработная плата в рублях.

После ответа на контрольные вопросы удалите исходный столбец заработной платы "ЗП" и все промежуточные столбцы, если вы их создавали.
Итак, давайте обсудим возможный алгоритм преобразования: 
1. Перевести признак "Обновление резюме" из таблицы с резюме в формат datetime и достать из него дату. В тот же формат привести признак "date" из таблицы с валютами.
2. Выделить из столбца "ЗП" сумму желаемой заработной платы и наименование валюты, в которой она исчисляется. Наименование валюты перевести в стандарт ISO согласно с таблицей выше.
3. Присоединить к таблице с резюме таблицу с курсами по столбцам с датой и названием валюты (подумайте, какой тип объединения надо выбрать, чтобы в таблице с резюме сохранились данные о заработной плате, изначально представленной в рублях). Значение close для рубля заполнить единицей 1 (курс рубля самого к себе)
4. Умножить сумму желаемой заработной платы на присоединенный курс валюты (close) и разделить на пропорцию (обратите внимание на пропуски после объединения в этих столбцах), результат занести в новый столбец "ЗП (руб)".

Скачать файл "ExchangeRates.csv" можно по [ссылке](https://drive.google.com/file/d/1MIu3j5hSMsbucOtQeXuc7keIR1OLmhl_/view?usp=drive_link)
```python
# Преобразовываю столбцы с датами в двух таблицах к одному формату
df_exchange_rate = pd.read_csv('data/ExchangeRates.csv', sep=',')
df_exchange_rate = df_exchange_rate.drop(['per', 'time', 'vol'], axis=1)
df_exchange_rate['date'] = pd.to_datetime(df_exchange_rate['date']).dt.date

hh_data['Обновление резюме'] = pd.to_datetime(hh_data['Обновление резюме'], dayfirst=True).dt.date

# выделяю из столбца "ЗП" сумму 
hh_data['Сумма'] = hh_data['ЗП'].apply(lambda x: float(x.split(' ')[0]))

# выделяю из столбца "ЗП" валюту и привожу ее к стандарту ISO
hh_data['Валюта'] = hh_data['ЗП'].apply(lambda x: x.split(' ')[1].replace('.', ''))

currency_dict = {
    'USD': 'USD', 'KZT': 'KZT','грн': 'UAH', 'белруб': 'BYN',
    'EUR': 'EUR', 'KGS': 'KGS','сум': 'UZS', 'AZN': 'AZN', 'руб': 'RUB'
    }

hh_data['Валюта'] = hh_data['Валюта'].apply(lambda x: currency_dict[x])

# объединяю две таблицы по сталбцам с датой и валютой, заполняю ячейки NaN на 1
merged = hh_data.merge(
    df_exchange_rate,
    left_on=['Обновление резюме', 'Валюта'],
    right_on=['date', 'currency'],
    how='left'
)
merged[['close', 'proportion']] = merged[['close', 'proportion']].fillna(1)

# создаю новый столбец с желаемой зарплатой в рублях
hh_data['ЗП (руб)'] = merged['close']/merged['proportion'] * merged['Сумма']

hh_data = hh_data.drop(columns=['ЗП', 'Валюта', 'Сумма'])

median_salary = round(hh_data['ЗП (руб)'].median()/1000)
print(f'Медианна желаемой заработной плата соискателей: {median_salary}тыс.руб.')
```
# Исследование зависимостей в данных
1. Постройте распределение признака **"Возраст"**. Опишите распределение, отвечая на следующие вопросы: чему равна мода распределения, каковы предельные значения признака, в каком примерном интервале находится возраст большинства соискателей? Есть ли аномалии для признака возраста, какие значения вы бы причислили к их числу?
*Совет: постройте гистограмму и коробчатую диаграмму рядом.*
```Python
fig = px.histogram(
    hh_data,
    x='Возраст',
    marginal='box',
    title='Распределение возраста для соискателей'
)
fig.show()
age_moda = (hh_data['Возраст'].mode())
print(f'Модальное значение возраста соискателей: {int(age_moda.iloc[0])} лет')
```
<img src=https://github.com/Long205sm/SF_Project_1/blob/master/graphs/1.png>

1. Модальное значение возраста соискателей: 30 лет
2. Предельные значения признака лежат от 10 до 77 лет, возраст большинства соискателей находится в интервале от 23 до 36 лет
3. 10 лет является анамалией т.к. трудовой договор можно заключать с 14 лет, соискатели 76-77 лет действительно могут искать работу скорее всего не является анамалией
2. Постройте распределение признака **"Опыт работы (месяц)"**. Опишите данное распределение, отвечая на следующие вопросы: чему равна мода распределения, каковы предельные значения признака, в каком примерном интервале находится опыт работы большинства соискателей? Есть ли аномалии для признака опыта работы, какие значения вы бы причислили к их числу?
*Совет: постройте гистограмму и коробчатую диаграмму рядом.*
```python
fig = px.histogram(
    hh_data,
    x='Опыт мес',
    marginal='box',
    title='Распределение опыта соискателей'
)
fig.show()
ex_moda = (hh_data['Опыт мес'].mode())
print(f'Модальное значение опыта соискателей: {int(ex_moda.iloc[0])} месяцев')
```
<img src=https://github.com/Long205sm/SF_Project_1/blob/master/graphs/2.png>

1. Модальное значение опыта соискателей: 81 месяцев
2. Предельные значения признака лежат от 1 до 1188 месяцев, опыт большинства соискателей находится в интервале от 5 до 154 месяцев
3. Опыт в 1188 месяцев (99 лет) вероятнее всего является аномальным
3. Постройте распределение признака **"ЗП (руб)"**. Опишите данное распределение, отвечая на следующие вопросы: каковы предельные значения признака, в каком примерном интервале находится заработная плата большинства соискателей? Есть ли аномалии для признака возраста? Обратите внимание на гигантские размеры желаемой заработной платы.
*Совет: постройте гистограмму и коробчатую диаграмму рядом.*
```python
mask1 = hh_data['ЗП (руб)'] < 400000
fig = px.histogram(
    hh_data[mask1],
    x='ЗП (руб)',
    marginal='box',
    title='Распределение желаемой зароботной платы соискателей'
)
fig.show()

salary_moda = (hh_data['ЗП (руб)'].mode())

mask2 =hh_data['ЗП (руб)'] > 1000000
print(hh_data[mask2]['ЗП (руб)'].count(), 'чел. с желаемой ЗП больше 1м руб.')

print(f'Модальное значение ЗП соискателей: {int(salary_moda.iloc[0])} руб')
```
<img src=https://github.com/Long205sm/SF_Project_1/blob/master/graphs/3.png>

1. Модальное значение ЗП соискателей: 50000 руб
2. Предельные значения признака лежат от 1 до 24.3 М руб., желаемая ЗП большинства соискателей находится в интервале от 24k до 90k руб.
3. Заработная плата в 7.5 и 24.3 М руб. скорее всего является аномальной но в теории возможной ()
4. Постройте диаграмму, которая показывает зависимость **медианной** желаемой заработной платы (**"ЗП (руб)"**) от уровня образования (**"Образование"**). Используйте для диаграммы данные о резюме, где желаемая заработная плата меньше 1 млн рублей.
*Сделайте выводы по представленной диаграмме: для каких уровней образования наблюдаются наибольшие и наименьшие уровни желаемой заработной платы? Как вы считаете, важен ли признак уровня образования при прогнозировании заработной платы?*
```python
data = hh_data.groupby('Образование', as_index=False)['ЗП (руб)'].mean()

fig = px.bar(
  data,
  x='Образование',
  y='ЗП (руб)',
  text_auto=True,
  title='Зависимость медианной желаемой заработной платы от уровня образования',
)
fig.update_layout(
    title_x =0.5
)
fig.show()
```
<img src=https://github.com/Long205sm/SF_Project_1/blob/master/graphs/4.png>

1. Для высшего оброзавания характерн наибольший уровнь желаемой заработной платы, а для средне специального наименьший.
2. Прослеживается четкая зависимость желаемой заработной платы от уровня образования, считаю что необходимо учитовать признак при прогнозировании заработной платы.
5. Постройте диаграмму, которая показывает распределение желаемой заработной платы (**"ЗП (руб)"**) в зависимости от города (**"Город"**). Используйте для диаграммы данные о резюме, где желая заработная плата меньше 1 млн рублей.
*Сделайте выводы по полученной диаграмме: как соотносятся медианные уровни желаемой заработной платы и их размах в городах? Как вы считаете, важен ли признак города при прогнозировании заработной платы?*
```python
mask = hh_data['ЗП (руб)'] < 1000000
fig = px.box(
    hh_data[mask],
    x='ЗП (руб)',
    color='Город',
    title='Распределение желаемой заработной платы в зависимости от города'
)
fig.update_layout(
    title_x =0.5,
)
fig.show()
```
<img src=https://github.com/Long205sm/SF_Project_1/blob/master/graphs/5.png>

1. Чем больше город тем больше медианные уровни желаемой заработной платы и их размах. (Москва - Питер - Город-миллионник - другие).
2. Так как прослеживается четкая зависимость ЗП от города необходимо этот признак учитывать при прогнозировании заработной платы.
6. Постройте **многоуровневую столбчатую диаграмму**, которая показывает зависимость медианной заработной платы (**"ЗП (руб)"**) от признаков **"Готовность к переезду"** и **"Готовность к командировкам"**. Проанализируйте график, сравнив уровень заработной платы в категориях.
```python
data = hh_data.groupby(['Готовность к переезду', 'Готовность к командировкам'])['ЗП (руб)'].median().unstack()
data
data = hh_data.groupby(['Готовность к переезду', 'Готовность к командировкам'], as_index=False)['ЗП (руб)'].median()

fig = px.bar(
    data,
    x='Готовность к переезду',
    y='ЗП (руб)',
    barmode='group',
    color ='Готовность к командировкам',
    title='Медианная ЗП по готовности к командировкам/переезду',
    text_auto=True
)
fig.show()
```
<img src=https://github.com/Long205sm/SF_Project_1/blob/master/graphs/6.png>

Медианная заработная плата соискателей, готовых и к переезду, и к командировкам равна 65.9k рублей.

7. Постройте сводную таблицу, иллюстрирующую зависимость **медианной** желаемой заработной платы от возраста (**"Возраст"**) и образования (**"Образование"**). На полученной сводной таблице постройте **тепловую карту**. Проанализируйте тепловую карту, сравнив показатели внутри групп.
```python
data = hh_data.pivot_table(
    values='ЗП (руб)',
    index='Образование',
    columns='Возраст',
    aggfunc='median',
)

fig = px.imshow(data, title= 'Зависимость медианной желаемой заработной платы от возраста')
fig.show()
```
<img src=https://github.com/Long205sm/SF_Project_1/blob/master/graphs/7.png>

Для высшего образования наблюдается самый быстрый карьерный рост

8. Постройте **диаграмму рассеяния**, показывающую зависимость опыта работы (**"Опыт работы (месяц)"**) от возраста (**"Возраст"**). Опыт работы переведите из месяцев в года, чтобы признаки были в едином масштабе. Постройте на графике дополнительно прямую, проходящую через точки (0, 0) и (100, 100). Данная прямая соответствует значениям, когда опыт работы равен возрасту человека. Точки, лежащие на этой прямой и выше нее - аномалии в наших данных (опыт работы больше либо равен возрасту соискателя)
```python
import plotly.graph_objects as go
hh_data['Опыт год'] = round(hh_data['Опыт мес'] / 12, 1)
fig = px.scatter(
    hh_data,
    x='Опыт год',
    y='Возраст',
    title = 'Зависимость опыта работы от возраста',
    height=550
    #trendline='ols',
)
reference_line = go.Scatter(
    x=[0, 100],
    y=[0, 100],
    mode="lines",
    line=go.scatter.Line(color="red"),
    showlegend=False
)

fig.add_trace(reference_line)
fig.show()
```
<img src=https://github.com/Long205sm/SF_Project_1/blob/master/graphs/8.png>

На графике мы видем 7 четких анамалий лежащих ниже или на вспомогательной прямой, говорящей нам что опыт работы больше либо равен возрасту соискателя.

**Дополнительные баллы**

Для получения 2 дополнительных баллов по разведывательному анализу постройте еще два любых содержательных графика или диаграммы, которые помогут проиллюстрировать влияние признаков/взаимосвязь между признаками/распределения признаков. Приведите выводы по ним. Желательно, чтобы в анализе участвовали признаки, которые мы создавали ранее в разделе "Преобразование данных".
```Python
data = hh_data.groupby('Город', as_index=False)['Город'].value_counts()

fig = px.pie(
    data, 
    values='count', 
    names='Город',
    hole=.2,
    title='Процентное соотношение резюме по городам',
    )
fig.show()
```
<img src=https://github.com/Long205sm/SF_Project_1/blob/master/graphs/9.png>

Согласно графику выше мы можем сделать следующий вывод: Наибольшее количество резюме было выставлено в Москве
```Python
mask = (hh_data['ЗП (руб)'] < 1000000)
fig = px.scatter(
    hh_data[mask],
    x='ЗП (руб)',
    y='Возраст',
    color='Пол',
    title = 'Зависимость зарплатных ожиданий от возраста с фильтром "Пол"',
    marginal_x="box", 
    marginal_y="histogram",
    height=550
)
fig.show()
```
<img src=https://github.com/Long205sm/SF_Project_1/blob/master/graphs/10.png>

Согласно графику выше мы можем сделать следующие выводы:
1. Пик зарплатных ожиданий приходится на возраст около 30 лет
2. В среднем зарплатные ожидания женщин ниже чем у мужчин

# Очистка данных
1. Начнем с дубликатов в наших данных. Найдите **полные дубликаты** в таблице с резюме и удалите их. 
```Python
mask = hh_data.duplicated()
hh_duplicates = hh_data[mask]
print(f'Число дубликатов: {hh_duplicates.shape[0]}')

hh_data= hh_data.drop_duplicates()
```
2. Займемся пропусками. Выведите информацию **о числе пропусков** в столбцах. 
```python
cols_null_sum = hh_data.isnull().sum()
cols_with_null = cols_null_sum[cols_null_sum>0].sort_values(ascending=False)
print('Количество пропусков в столбцах:')
display(cols_with_null)
```
3. Итак, у нас есть пропуски в 3ех столбцах: **"Опыт работы (месяц)"**, **"Последнее/нынешнее место работы"**, **"Последняя/нынешняя должность"**. Поступим следующим образом: удалите строки, где есть пропуск в столбцах с местом работы и должностью. Пропуски в столбце с опытом работы заполните **медианным** значением.
```py
values ={'Опыт мес': hh_data['Опыт мес'].median()}

hh_data = hh_data.fillna(values)
experience_mean = round(try_data['Опыт мес'].mean())
print(f'Cреднее значение в столбце «Опыт мес»: {experience_mean} мес')

hh_data = hh_data.dropna()
hh_data.shape
```
4. Мы добрались до ликвидации выбросов. Сначала очистим данные вручную. Удалите резюме, в которых указана заработная плата либо выше 1 млн. рублей, либо ниже 1 тыс. рублей.
```py
outliers1 = hh_data[hh_data['ЗП (руб)']>1000000]
outliers2 = hh_data[hh_data['ЗП (руб)']<1000]
total_outliers =outliers1.shape[0] + outliers2.shape[0]
print(f'{total_outliers}шт. выбросов обнаружено')

hh_data = hh_data.drop(outliers1.index, axis=0)
hh_data = hh_data.drop(outliers2.index, axis=0)
```
5. В процессе разведывательного анализа мы обнаружили резюме, в которых **опыт работы в годах превышал возраст соискателя**. Найдите такие резюме и удалите их из данных
```py
outliers = hh_data[hh_data['Опыт мес']/12 > hh_data['Возраст']]
print(f'{outliers.shape[0]}шт. выбросов обнаружено')

hh_data = hh_data.drop(outliers.index, axis=0)
```
6. В результате анализа мы обнаружили потенциальные выбросы в признаке **"Возраст"**. Это оказались резюме людей чересчур преклонного возраста для поиска работы. Попробуйте построить распределение признака в **логарифмическом масштабе**. Добавьте к графику линии, отображающие **среднее и границы интервала метода трех сигм**. Напомним, сделать это можно с помощью метода axvline. Например, для построение линии среднего будет иметь вид:

`histplot.axvline(log_age.mean(), color='k', lw=2)`

В какую сторону асимметрично логарифмическое распределение? Напишите об этом в комментарии к графику.
Найдите выбросы с помощью метода z-отклонения и удалите их из данных, используйте логарифмический масштаб. Давайте сделаем послабление на **1 сигму** (возьмите 4 сигмы) в **правую сторону**.

Выведите таблицу с полученными выбросами и оцените, с каким возрастом соискатели попадают под категорию выбросов?
```python
fig, ax =plt.subplots(1,1, figsize=(8,4))
log_age = np.log(hh_data['Возраст'])
hisplot = sns.histplot(log_age, ax=ax)
hisplot.axvline(log_age.mean(), color='k', lw=2)
hisplot.axvline(log_age.mean()+ 4  * log_age.std(), color='k', ls='--', lw=2)
hisplot.axvline(log_age.mean()- 3  * log_age.std(), color='k', ls='--', lw=2)
hisplot.set_title('Распределение возраста(log)')
```
<img src=https://github.com/Long205sm/SF_Project_1/blob/master/graphs/11.png>

Логарифмический график имеет левое распределение
```python
def outliers_z_score (data, feature, log_scale=False, left=3, right=3):
    if log_scale:
        x = np.log(data[feature] + 1)
    else:
        x = data[feature]
    mu = x.mean()
    sigma = x.std()
    lower_bound = mu - left * sigma
    upper_bound = mu + right * sigma
    outliers = data[(x < lower_bound) | (x > upper_bound)]
    cleaned = data[(x >= lower_bound) & (x <= upper_bound)]
    return outliers, cleaned

outliers, cleaned = outliers_z_score(hh_data, 'Возраст', log_scale=True, right=4)
print(f'Число выбросов по методу z-отклонения: {outliers.shape[0]}')
display(outliers['Возраст'])

hh_data = hh_data.drop(outliers.index, axis=0)
```
Обнаружено 3 выброса по методу z-отклонения (4 сигмы в правую сторону), под выбросы попали соискатели с возрастом 10 и 15 лет.