<h1>Содердание<span class="tocSkip"></span></h1>
<div class="toc"><ul class="toc-item"><li><span><a href="#Итоги-опроса-по-определению-NPS---постановка-задачи" data-toc-modified-id="Итоги-опроса-по-определению-NPS---постановка-задачи-1"><span class="toc-item-num">1&nbsp;&nbsp;</span>Итоги опроса по определению NPS - постановка задачи</a></span></li><li><span><a href="#Подключение-к-базе-и-выгрузка-данных" data-toc-modified-id="Подключение-к-базе-и-выгрузка-данных-2"><span class="toc-item-num">2&nbsp;&nbsp;</span>Подключение к базе и выгрузка данных</a></span></li><li><span><a href="#Создание-дашборда-в-Tableau" data-toc-modified-id="Создание-дашборда-в-Tableau-3"><span class="toc-item-num">3&nbsp;&nbsp;</span>Создание дашборда в Tableau</a></span></li><li><span><a href="#Создание-презентации-в-pdf" data-toc-modified-id="Создание-презентации-в-pdf-4"><span class="toc-item-num">4&nbsp;&nbsp;</span>Создание презентации в pdf</a></span></li></ul></div>

## Итоги опроса по определению NPS - постановка задачи

Заказчик данного исследования — большая телекоммуникационная компания, которая оказывает услуги на территории всего СНГ. Перед компанией стоит задача определить текущий уровень потребительской лояльности, или NPS (от англ. Net Promoter Score) среди клиентов из России. Компания провела опрос. 

Задача данного исследования - подготовка дашборда с итогами опроса. Данные опроса выгрузили в SQLite. Преобразовывать данные с помощью Python нельзя — мы должны использовать только SQL-запросы.

## Подключение к базе и выгрузка данных


```python
import pandas as pd
import numpy as np

from sqlalchemy import create_engine
```


```python
# Подключимся к базе данных:
path_to_db = '/datasets/telecomm_csi.db'
engine = create_engine(f'sqlite:///{path_to_db}', echo = False)
```


```python
query_user = '''
SELECT * FROM user;
'''
```


```python
df_user = pd.read_sql(query_user, engine)
df_user.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_id</th>
      <th>lt_day</th>
      <th>age</th>
      <th>gender_segment</th>
      <th>os_name</th>
      <th>cpe_type_name</th>
      <th>location_id</th>
      <th>age_gr_id</th>
      <th>tr_gr_id</th>
      <th>lt_gr_id</th>
      <th>nps_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>A001A2</td>
      <td>2320</td>
      <td>45.0</td>
      <td>1.0</td>
      <td>ANDROID</td>
      <td>SMARTPHONE</td>
      <td>55</td>
      <td>5</td>
      <td>5</td>
      <td>8</td>
      <td>10</td>
    </tr>
    <tr>
      <td>1</td>
      <td>A001WF</td>
      <td>2344</td>
      <td>53.0</td>
      <td>0.0</td>
      <td>ANDROID</td>
      <td>SMARTPHONE</td>
      <td>21</td>
      <td>5</td>
      <td>5</td>
      <td>8</td>
      <td>10</td>
    </tr>
    <tr>
      <td>2</td>
      <td>A003Q7</td>
      <td>467</td>
      <td>57.0</td>
      <td>0.0</td>
      <td>ANDROID</td>
      <td>SMARTPHONE</td>
      <td>28</td>
      <td>6</td>
      <td>9</td>
      <td>6</td>
      <td>10</td>
    </tr>
    <tr>
      <td>3</td>
      <td>A004TB</td>
      <td>4190</td>
      <td>44.0</td>
      <td>1.0</td>
      <td>IOS</td>
      <td>SMARTPHONE</td>
      <td>38</td>
      <td>4</td>
      <td>4</td>
      <td>8</td>
      <td>10</td>
    </tr>
    <tr>
      <td>4</td>
      <td>A004XT</td>
      <td>1163</td>
      <td>24.0</td>
      <td>0.0</td>
      <td>ANDROID</td>
      <td>SMARTPHONE</td>
      <td>39</td>
      <td>2</td>
      <td>6</td>
      <td>8</td>
      <td>10</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Соберем данные из разных витрин в одну таблицу:
query_result = '''
SELECT user.user_id,
       user.lt_day,
       user.age,
       user.gender_segment,
       user.os_name,
       user.cpe_type_name,
       location.country,
       location.city,
       age_segment.title as age_segment,
       traffic_segment.title as traffic_segment,
       lifetime_segment.title as lifetime_segment,
       user.nps_score,
       
       CASE
            WHEN lifetime_segment.title in ('08 36+', '06 13-24', '07 25-36') THEN 'True'
            ELSE 'False'
       END as is_new,
       
       
       CASE
            WHEN user.nps_score > 8 THEN 'cторонники'
            WHEN user.nps_score < 7 THEN 'критики'
            ELSE 'нейтралы'
       END as nps_group
           
       
FROM user

INNER JOIN location ON user.location_id = location.location_id
INNER JOIN age_segment ON user.age_gr_id = age_segment.age_gr_id
INNER JOIN traffic_segment ON user.tr_gr_id = traffic_segment.tr_gr_id
INNER JOIN lifetime_segment ON user.lt_gr_id = lifetime_segment.lt_gr_id
'''
```


```python
result = pd.read_sql(query_result, engine)
result.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_id</th>
      <th>lt_day</th>
      <th>age</th>
      <th>gender_segment</th>
      <th>os_name</th>
      <th>cpe_type_name</th>
      <th>country</th>
      <th>city</th>
      <th>age_segment</th>
      <th>traffic_segment</th>
      <th>lifetime_segment</th>
      <th>nps_score</th>
      <th>is_new</th>
      <th>nps_group</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>A001A2</td>
      <td>2320</td>
      <td>45.0</td>
      <td>1.0</td>
      <td>ANDROID</td>
      <td>SMARTPHONE</td>
      <td>Россия</td>
      <td>Уфа</td>
      <td>05 45-54</td>
      <td>04 1-5</td>
      <td>08 36+</td>
      <td>10</td>
      <td>True</td>
      <td>cторонники</td>
    </tr>
    <tr>
      <td>1</td>
      <td>A001WF</td>
      <td>2344</td>
      <td>53.0</td>
      <td>0.0</td>
      <td>ANDROID</td>
      <td>SMARTPHONE</td>
      <td>Россия</td>
      <td>Киров</td>
      <td>05 45-54</td>
      <td>04 1-5</td>
      <td>08 36+</td>
      <td>10</td>
      <td>True</td>
      <td>cторонники</td>
    </tr>
    <tr>
      <td>2</td>
      <td>A003Q7</td>
      <td>467</td>
      <td>57.0</td>
      <td>0.0</td>
      <td>ANDROID</td>
      <td>SMARTPHONE</td>
      <td>Россия</td>
      <td>Москва</td>
      <td>06 55-64</td>
      <td>08 20-25</td>
      <td>06 13-24</td>
      <td>10</td>
      <td>True</td>
      <td>cторонники</td>
    </tr>
    <tr>
      <td>3</td>
      <td>A004TB</td>
      <td>4190</td>
      <td>44.0</td>
      <td>1.0</td>
      <td>IOS</td>
      <td>SMARTPHONE</td>
      <td>Россия</td>
      <td>РостовнаДону</td>
      <td>04 35-44</td>
      <td>03 0.1-1</td>
      <td>08 36+</td>
      <td>10</td>
      <td>True</td>
      <td>cторонники</td>
    </tr>
    <tr>
      <td>4</td>
      <td>A004XT</td>
      <td>1163</td>
      <td>24.0</td>
      <td>0.0</td>
      <td>ANDROID</td>
      <td>SMARTPHONE</td>
      <td>Россия</td>
      <td>Рязань</td>
      <td>02 16-24</td>
      <td>05 5-10</td>
      <td>08 36+</td>
      <td>10</td>
      <td>True</td>
      <td>cторонники</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Cохраним таблицу как csv-файл:
df_user.to_csv('telecomm_csi_tableau.csv', index=False)
```

## Создание дашборда в Tableau

Мы построили дашборд, который представил информацию о текущем уровне NPS среди клиентов и показал, как этот уровень меняется в зависимости от пользовательских признаков.
Из дашборда понятно, какие группы пользователей участвовали в опросе и как преобладание пользователей какой-то группы отражается на общем уровне NPS.
В дашборде применены фильтры — они сделали дашборд интерактивным и упростили менеджерам работу с данными опроса.

Ниже ссылка на дашборд:

https://public.tableau.com/views/NPS_16450175105840/Dashboard2_NPS?:language=en-US&publish=yes&:display_count=n&:origin=viz_share_link

## Создание презентации в pdf

Используя дашборд, мы ответили на следующие вопросы:
- Как распределены участники опроса по возрасту, полу и возрасту? Каких пользователей больше: новых или старых? Пользователи из каких городов активнее участвовали в опросе?
- Какие группы пользователей наиболее лояльны к сервису? Какие менее?
- Какой общий NPS среди всех опрошенных?
- Как можно описать клиентов, которые относятся к группе cторонников (англ. promoters)?

Ответы на данные вопросы были оформлены ввиде презентации. ССылка на презентацию ниже:

https://drive.google.com/file/d/1nJj0sg_feehdE21Xlfdcj8kiGF4WvdLP/view?usp=sharing
