# Fuzzy time-series forecasting / Нечёткое прогнозирование временных рядов

This repository is a compact, reproducible tutorial on forecasting the University of Alabama enrollment series with fuzzy time series (FTS). The notebook is the project: it builds the model from first principles and compares it fairly with simple baselines at horizons of one, two, and three years.

Этот репозиторий — компактное воспроизводимое руководство по прогнозированию числа студентов Университета Алабамы с помощью нечётких временных рядов (FTS). Главный результат проекта — ноутбук: модель строится с нуля и корректно сравнивается с простыми базовыми методами на горизонтах один, два и три года.

## Run / Запуск

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
python -m pip install -r requirements.txt
jupyter lab example-Fuzzy-timeseries-forecast.ipynb
```

Run all cells from top to bottom. The notebook has no network or external-data dependency. It contains the 1971–1992 series directly so that results remain reproducible.

Запустите все ячейки по порядку. Ноутбук не требует сети или внешних данных: ряд за 1971–1992 годы хранится внутри него для полной воспроизводимости.

## What the notebook demonstrates / Что показывает ноутбук

- leakage-free rolling-origin evaluation rather than judging a model from one convenient year;
- forecasts for **1, 2, and 3 steps ahead**;
- trend-aware, recency-weighted first- and second-order FTS over annual changes;
- comparison with persistence (last value) and a linear-trend baseline;
- MAE, RMSE and MAPE calculated separately for every horizon;
- recursive FTS forecasting, including the fallback used for unseen higher-order rules.

- оценивание методом скользящего начала прогноза без утечки будущих данных;
- прогнозы на **1, 2 и 3 шага вперёд**;
- учитывающие тренд и давность наблюдений FTS первого и второго порядка для годовых изменений;
- сравнение с прогнозом последним значением и линейным трендом;
- MAE, RMSE и MAPE отдельно для каждого горизонта;
- рекурсивный FTS-прогноз и явное правило отката для невиданных правил высокого порядка.

## Theory (English)

### Fuzzy sets and fuzzification

A fuzzy set replaces a hard boundary with a membership function
`mu_A(x) in [0, 1]`. The model transforms levels into annual changes, pads the observed change range, and divides it into equal intervals. Their midpoints are the peaks of overlapping triangular fuzzy sets `D1, ..., Dk`. A numeric change is **fuzzified** by assigning it to the set with the greatest membership (equivalently, the nearest midpoint for these regular triangles). Modeling changes avoids the common level-state failure mode in which recursive forecasts settle on one interval midpoint and become horizontal.

### Fuzzy logical relationships

For first-order FTS, adjacent change states create rules `D(t-1) -> D(t)`. A second-order model uses `(D(t-2), D(t-1)) -> D(t)` and can represent short local patterns, but its rule table is sparser. Evidence is weighted by recency with exponential decay: recent transitions influence the forecast more strongly without discarding older history.

### Defuzzification and several steps ahead

The next crisp **change** is the recency-weighted mean of consequent-set midpoints and is added to the previous level. For an unseen second-order context, the implementation backs off to the learned first-order rule; if that too is unseen, it uses a recency-weighted average of the last three changes. Multi-step forecasts are recursive: fuzzify the predicted change, append the reconstructed level, and apply the rules again. A five-step example trained through 1987 makes the resulting non-horizontal trajectory explicit. Error normally grows with horizon because later forecasts depend on earlier forecasts.

### Honest evaluation

At each rolling origin, a model sees only the prefix ending at that origin. It then predicts the next three held-out observations. Metrics aggregate errors by horizon. MAPE is intuitive here because enrollment never approaches zero; MAE retains the original unit, while RMSE penalizes large misses more strongly. This small data set is educational, not evidence that one method is universally superior.

## Теория (Русский)

### Нечёткие множества и фаззификация

Нечёткое множество заменяет жёсткую границу функцией принадлежности
`mu_A(x) in [0, 1]`. Модель преобразует уровни в годовые изменения, расширяет диапазон изменений небольшим запасом и разбивает его на равные интервалы. Их середины становятся вершинами перекрывающихся треугольных множеств `D1, ..., Dk`. Изменение **фаззифицируется** выбором множества с максимальной степенью принадлежности. Моделирование изменений устраняет типичную проблему FTS уровней, когда рекурсивный прогноз попадает в одну середину интервала и превращается в горизонтальную линию.

### Нечёткие логические отношения

FTS первого порядка формирует из соседних состояний изменений правила `D(t-1) -> D(t)`. Модель второго порядка использует `(D(t-2), D(t-1)) -> D(t)`: она способна учитывать короткий локальный паттерн, но таблица правил получается более разреженной. Переходы взвешиваются экспоненциально по давности, поэтому недавняя динамика влияет сильнее, но старая история не отбрасывается.

### Дефаззификация и прогноз на несколько шагов

Числовой прогноз **изменения** — взвешенное по давности среднее середин множеств в правой части правила; изменение прибавляется к последнему уровню. Для неизвестного контекста второго порядка алгоритм использует правило первого порядка, а затем средневзвешенное трёх последних изменений. Многошаговый прогноз строится рекурсивно. Пример обучается по данным до 1987 года и прогнозирует пять следующих лет, явно демонстрируя негоризонтальную траекторию. С ростом горизонта ошибка обычно увеличивается.

### Корректное оценивание

В каждой точке скользящего теста модель получает только прошлый префикс ряда и прогнозирует три скрытых наблюдения. Метрики объединяются отдельно по каждому горизонту. MAPE здесь уместна, поскольку значения набора далеки от нуля; MAE измеряется в исходных единицах, а RMSE сильнее штрафует крупные ошибки. Маленький учебный набор не доказывает универсального превосходства какого-либо метода.

## Repository scope / Состав репозитория

The previous general-purpose fuzzy-regression package and demo entry point were unrelated to the time-series tutorial and have been removed. Keeping one executable notebook avoids two conflicting APIs and unnecessary `cvxopt`/packaging machinery.

Прежняя библиотека общей нечёткой регрессии и её демонстрационный entry point не использовались в примере временного ряда и удалены. Один исполняемый ноутбук устраняет две конкурирующие реализации и ненужные зависимости от `cvxopt` и упаковочного кода.

## References / Литература

1. Zadeh, L. A. (1965). *Fuzzy sets*. Information and Control, 8(3), 338–353.
2. Song, Q., & Chissom, B. S. (1993). *Fuzzy time series and its models*. Fuzzy Sets and Systems, 54(3), 269–277.
3. Tsaur, R.-C. (2012). *A fuzzy time series–Markov chain model with an application to forecasting the exchange rate between the Taiwan and US dollar*. IJICIC, 8(7B), 4931–4942.
