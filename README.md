# Прогнозирование загруженности автотерминала

## Оглавление
[1. Краткое описание проекта](#краткое-описание-проекта)  
[2. Этапы работы над проектом](#этапы-работы-над-проектом)  
[3. Результат](#результат)  
[4. Установка зависимостей](#установка-зависимостей)  
[5. Описание входных данных модели](#описание-входных-данных-модели)  
[6. Использование](#использование)  
[7. Вывод](#вывод)

## Краткое описание проекта

Проект посвящён получению прогноза количества автомобилей на автотерминале в пиковый час на следующий день с помощью методов машинного обучения. Реализовав данную модель, бизнес сможет анализировать ситуацию на последующие дни и обеспечивать бесперебойную доставку зерна от экспортёра к перегрузочному терминалу, тем самым минимизируя простои.

## Этапы работы над проектом:

1. **Сбор данных и подготовка признаков**
2. **Разведывательный анализ**
3. **Преобразование данных**
4. **Построение и отбор оптимальной модели**

## Результат
[Ссылка на Jupyter Notebook](https://github.com/postvlone/autoterminal_project/blob/main/prt.ipynb)

## Установка зависимостей

```bash
pip install -r requirements.txt
```

## Описание входных данных модели

Модель ожидает DataFrame со столбцами (типы указаны для примера):

| Колонка              | Тип   | Описание                                        |
| -------------------- | ----- | ----------------------------------------------- |
| `expected_autos`     | int   | Ожидаемое число машин по квотам                 |
| `part_day`           | float | Доля суток без приемки авто                     |
| `vessels`            | int   | Количество судов за день                        |
| `fact_autos`         | float | Фактическое число автомобилей                   |
| `fact_rail`          | float | Число вагонов за день                           |
| `loaded`             | float | Объём груза, погруженного на судно              |
| `warehouse_fullness` | float | Заполненность силосов на начало дня             |
| `day`                | int   | День месяца (1–31)                              |
| `month`              | int   | Месяц (1–12)                                    |
| `day_of_week`        | int   | День недели (0 – понедельник … 6 – воскресенье) |
| `num_autos_lag_1`    | float | Число автомобилей 1 день назад                  |
| `num_autos_lag_7`    | float | Число автомобилей 7 дней назад                  |
| `part_day_lag_1`     | float | Доля суток без приемки авто 1 день назад        |
| `is_holiday`         | int   | Флаг выходных дней (0 – рабочий, 1 – выходной)  |

## Использование

```python
import pandas as pd
from joblib import load

# 1. Загрузка обученного пайплайна из файла
pipeline = load('models/xgb_prod_pipeline_v1.joblib')

# 2. Подготовка данных для прогноза (пример):
new_data = pd.DataFrame([{
    'expected_autos': 455,
    'part_day': 0,
    'vessels': 2,
    'fact_autos': 359.0,
    'fact_rail': 28.0,
    'loaded': 50.0,
    'warehouse_fullness': 149153.262,
    'day': 9,
    'month': 6,
    'day_of_week': 3,
    'num_autos_lag_1': 42.0,
    'num_autos_lag_7': 50.0,
    'part_day_lag_1': 0,
    'is_holiday': 0
}])

# 3. Получение прогноза
prediction = pipeline.predict(new_data)
print(f"Прогноз числа автомобилей: {prediction[0]:.0f}")
```

## Вывод

Обученная модель (градиентный бустинг XGBoost) показала хорошее качество прогнозирования: она учитывает сезонные колебания и общую динамику загрузки автотерминала. На тестовой выборке средняя абсолютная ошибка (MAE) невелика, а предсказания в целом следуют за реальными значениями. Однако модель имеет тенденцию не до конца воспроизводить экстремальные всплески (очень высокие пиковые значения числа машин), что объясняется их ограниченным количеством в обучающих данных. Следует учитывать потенциальный заниженный прогноз.

Дальнейшие улучшения включают сбор дополнительных данных — например, учёт культуры зерна по каждому хранилищу, погодных условий, а также почасовой информации по каждому из признаков. Необходимо добиться более детализированных почасовых прогнозов.

