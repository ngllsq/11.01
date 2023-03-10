# Порядок выполнения предобработки и разведочного анализа данных
Удаление дубликатов
Дублирующие записи не только искажают статистические показатели 
датасета, но и снижают качество обучения модели, потому удалим полные 
дублирующие вхождения. Для начала уточним, сколько записей в датасете с 
помощью свойства Pandas.DataFrame.shape: 
```
df.shape
```
Удалим дублирующие записи с помощью Pandas.drop_duplicates() и обновим 
данные о размере данных: 
```
df = df.drop_dublicates()
```

Обработка пропусков
Стоит помнить, что в случае, если пропусков у признака слишком много 
(более 70%), такой признак удаляют. Проверим, насколько полны наши 
признаки: метод isnull() пройдется по каждой ячейке каждого столбца и 
определит, кто пуст, а кто нет, составив датафрейм такого же размера, 
состоящий из True / False. Метод mean() суммирует все значения True, 
определит концентрацию пропусков в каждом столбце. На 100 мы умножаем, 
чтобы получить значение в процентах: 
```
df.isnull().mean() * 100
```
Переменные удаляются с помощью drop()
Существует несколько способов обозначить пропуски, и зачастую создатели 
датасета не описывают данные в достаточной мере, и определять, как 
обозначены пропуски, приходится вручную. Из встреченных доселе 
обозначений приведу следующие:
 NaN / NaT (упрощенно: "не число" / "не время")
 Пустая ячейка
 Для числовых признаков – радикальный выброс. К примеру, для столбца 
"День" это число 999.
 Маркер или нестандартный символ
Встроенные методы Pandas позволяют с легкостью справиться с первыми 
двумя разновидностями таких пробелов. Разберемся для начала с 
категориальными переменными, объединив их в один вектор. Список 
получится совсем уж нелогичный, но это не столь важно в данной ситуации: 
мы лишь ищем способы обозначения пропуска: 

```
values = pd.unique(df)
```
Из общего списка уникальных значений этих переменных пропуски 
обозначаются словом "Неизвестно". Для числовых переменных пропуски –
число 999 или пустая ячейка.
Процесс обработки пропусков, к счастью, можно сократить с помощью 
sklearn.impute.SimpleImputer. Мы выбираем все категориальные переменные и 
применяем стратегию "[вставить вместо пропуска] самое распространенное 
значение".
Признаки, принадлежащие к булевому типу данных, обрабатываются 
алгоритмом тем же образом. Целевую переменную Y мы не обрабатываем 
(если в этом столбце есть пропуски, такие строки стоит удалить): >>from sklearn.impute import SimpleImputer 

```
imputer = SimpleImputer(missing_values = np.nan, strategy = 'most_frequent')
df['работа'] = imputer.fit_transform(df['работа'].values.reshape(-1,1)) [:, 0]
```
Подобным образом заполняются пустоты в числовых переменных, только 
стратегия теперь – "вставить среднее значение": 

```
imputer = SimpleImputer(missing_values = np.nan, strategy = 'mean')
```
Обнаружение аномалий
Самый легкий способ обнаружить выбросы – визуальный:

```
df.boxplot(column = ['возраст'], figsize = (2,7))
```
Скучковавшиеся окружности в верхней части изображения – и есть аномалии, 
и от них, как правило, избавляются с помощью квантилей:

```
q = df['возраст'].quantile(0.99)
df[df['возраст'] < q
```
Описательная статистика
Раздел 
описательной статистики включает в себя проверку на нормальность 
распределения и определение прочих статистических метрик. С этим нам 
поможет замечательная библиотека pandas-profiling.
![image](https://user-images.githubusercontent.com/114596475/211725057-9f97e61a-63c5-4170-9575-aec34ab28560.png)
Профайлер высчитывает основные статистические метрики для каждой 
переменной и датасета в целом
Корреляции
Чем ярче 
(краснее / синее) ячейка, тем сильнее выражена корреляция между парой 
признаков.
Важность признаков
Прежде чем произвести инжиниринг признаков и сократить объем входных 
данных, стоит определить, какие признаки имеют первостепенную 
значимость, и в этом нам поможет Scikit-Learn и критерий Хи-квадрат (Chi-Squared Test)
![image](https://user-images.githubusercontent.com/114596475/211725624-0f431e78-03e5-4089-9f82-251a5e5e8118.png)
Рассмотрение парных особенностей
Чего только не создаст комьюнити в Науке о данных! Для нужд разведочного 
анализа крайне кстати будет и попарные графики, и здесь на помощь приходит 
другой великолепный класс - seaborn.pairplot(). Каждая из переменных ляжет 
в основу одной из осей двумерного точечного графика, и так, пока все пары 
признаков не будут отображены. Сократим названия длинных переменных, 
чтобы уместить их на скромном отведенном пространстве
![image](https://user-images.githubusercontent.com/114596475/211726104-3f654710-7548-4b4f-8d47-68f2ed3cb5b4.png)

Уменьшение размерности, стандартизация
Рассмотрев признаки по отдельности и попарно, мы пришли к выводу, что 
некоторые признаки могут быть как бы объединены с помощью специальной 
техники – Анализ главных компонент (PCA). Итак, давайте создадим 
заменяющий столбец, который представляет эти признаки в равной мере и тем 
самым уменьшим размер данных.
Выполняем Стандартизацию (Standartization) x, и это впоследствии станет 
частью тренировочных данных: 
![image](https://user-images.githubusercontent.com/114596475/211726157-86ae1c11-8e61-40a8-9ed5-1a52d898ae6e.png)
StandardScaler() на месте заменяет данные на их стандартизированную 
версию, и мы получаем признаки, где все значения как бы центрованы 
относительно нуля. Такое преобразование необходимо, чтобы сократить 
нагрузку на вычислительную систему компьютера, который будет обучать 
модель
Анализ главных компонент (Principal Component Analysis) представляет собой 
метод уменьшения размерности больших наборов данных путем 
преобразования большого набора переменных в меньший с минимальными 
потерями информативности
![image](https://user-images.githubusercontent.com/114596475/211726305-d90e9491-ae46-411e-b06a-9a20a07ddade.png)
Нормализация
Еще один шаг, не затронутый в примере выше, – это нормализация и порой 
приходится выбирать между ею и стандартизацией. Мы нормализуем те же 
признаки, характеризующие состояние экономики.
![image](https://user-images.githubusercontent.com/114596475/211726391-820a196d-1904-4d20-90fd-f7c41a0c89d3.png)

# Алгоритмы кодирования категориальных признаков. В каком случае какой алгоритм применять
LabelEncoder 
Первое (выбранное каким-то образом) уникальное значение кодируется нулем, второе единицей, и так далее, последнее кодируется числом, равным количеству уникальных значений минус единица.Реализация Label Encoder в sklearn прежде всего сортирует по алфавиту уникальные значения, потом присваивает им порядковый номер
```
from sklearn.preprocessing import LabelEncoder
labelencoder = LabelEncoder()
data_new = labelencoder.fit_transform(data.values)
data_new[:10]
```
One-Hot Encoder
Каждый столбец содержит «0» или «1», в зависимости от того, в какой столбец он был помещен. В этом процессе каждое целочисленное значение представляется как двоичный вектор, равный нулю, за исключением индекса целого числа, который помечен 1.
```
from sklearn.preprocessing import OneHotEncoder
enc = OneHotEncoder(handle_unknown='ignore')
X = [['Male', 1], ['Female', 3], ['Female', 2]]
enc.fit(X)
enc.categories_
enc.transform([['Female', 1], ['Male', 4]]).toarray()
enc.inverse_transform([[0, 1, 1, 0, 0], [0, 0, 0, 1, 0]])
```
# Алгоритм построения модели линейной регрессии
Простая линейная регрессия со scikit-learn
Начнём с простейшего случая линейной регрессии.
Следуйте пяти шагам реализации линейной регрессии:
1.Импортируйте необходимые пакеты и классы.
![image](https://user-images.githubusercontent.com/114596475/211726890-325a3e41-cfe4-4e4d-9096-cb0d27eeaca3.png)
Теперь у вас есть весь функционал для реализации линейной регрессии.
Фундаментальный тип данных NumPy – это тип массива numpy.ndarray. Далее 
под массивом подразумеваются все экземпляры типа numpy.ndarray.
Класс sklearn.linear_model.LinearRegression используем для линейной 
регрессии и прогнозов.
2.Предоставьте данные для работы и преобразования.
![image](https://user-images.githubusercontent.com/114596475/211726965-72ae8fe9-1e83-4c06-b415-a860de319757.png)
Теперь у вас два массива: вход x и выход y. Вам нужно вызвать 
.reshape()на x, потому что этот массив должен быть двумерным или более 
точным – иметь одну колонку и необходимое количество рядов. Это как раз 
то, что определяет аргумент (-1, 1
3.Создайте модель регрессии и приспособьте к существующим данным.
![image](https://user-images.githubusercontent.com/114596475/211727048-d8d4d4b0-7039-4e0c-91f5-74a0715a78cc.png)
Эта операция создаёт переменную model в качестве экземпляра 
LinearRegression. Вы можете предоставить несколько опциональных 
параметров классу LinearRegression:
fit_intercept – логический (True по умолчанию) параметр, который 
решает, вычислять отрезок b₀ (True) или рассматривать его как равный нулю 
(False).
normalize – логический (False по умолчанию) параметр, который решает, 
нормализовать входные переменные (True) или нет (False).
copy_X – логический (True по умолчанию) параметр, который решает, 
копировать (True) или перезаписывать входные переменные (False).
n_jobs – целое или None (по умолчанию), представляющее количество 
процессов, задействованных в параллельных вычислениях. None означает 
отсутствие процессов, при -1 используются все доступные процессоры.
![image](https://user-images.githubusercontent.com/114596475/211727240-26ad5381-9e40-4223-84ce-e1f2219bd575.png)
С помощью .fit() вычисляются оптимальные значение весов b₀ и b₁, 
используя существующие вход и выход (x и y) в качестве аргументов. Другими 
словами, .fit() совмещает модель. Она возвращает self - переменную model. 
4.Проверьте результаты совмещения и удовлетворительность модели.
![image](https://user-images.githubusercontent.com/114596475/211727356-2a80056a-c59b-4f0f-8fe2-0758cd2d273b.png)
.score() принимает в качестве аргументов предсказатель x и регрессор y, 
и возвращает значение R².
model содержит атрибуты .intercept_, который представляет собой 
коэффициент, и b₀ с .coef_, которые представляют b₁:
![image](https://user-images.githubusercontent.com/114596475/211727410-ccc19bb3-601e-49db-9864-2043f12066e9.png)
Код выше показывает, как получить b₀ и b₁. Заметьте, что .intercept_ –
это скаляр, в то время как .coef_ – массив
5.Примените модель для прогнозов.
![image](https://user-images.githubusercontent.com/114596475/211727456-f6fa9d39-072f-4bcf-90f3-06adfb1835c6.png)
Применяя .predict(), вы передаёте регрессор в качестве аргумента и 
получаете соответствующий предсказанный ответ.
# Алгоритм построения моделей кластеризации
Иерархическая кластеризация
В начале работы помещают каждый объект в отдельный кластер, а затем объединяют кластеры во все более крупные, пока все объекты выборки не будут содержаться в одном кластере. Таким образом строится система вложенных разбиений. Результаты таких алгоритмов обычно представляют в виде дерева – дендрограммы.
Импортируем нужные библиотеки:
```
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
```
Импортируем датасет
```
dataset = pd.read_csv('Mall_Customers.csv')
X = dataset.iloc[:, [3, 4]].values
```
Построим дендограмму для вычисления оптимального количества кластеров:
```
import scipy.cluster.hierarchy as sch
dendrogram = sch.dendrogram(sch.linkage(X, method = 'ward'))
plt.title('Dendrogram')
plt.xlabel('Customers')
plt.ylabel('Euclidean distances')
plt.show()
```
Метод K-means
Импортируем библиотеку:
```
from sklearn.cluster import KMeans
```
Импортируем наш датасет:
```
df2=pd.read_csv(r'C:\Users\veron\OneDrive\Рабочий стол\weight-height.csv')
X = df2.iloc[:, [1, 2]].values
```
Построим график для определения количества кластеров:
```
wcss = []
for i in range(1, 11):
    kmeans = KMeans(n_clusters = i, init = 'k-means++', random_state = 52)
    kmeans.fit(X)
    wcss.append(kmeans.inertia_)
plt.plot(range(1, 11), wcss)
plt.title('The Elbow Method')
plt.xlabel('Number of clusters')
plt.ylabel('WCSS')
plt.show()
```







