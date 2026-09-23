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
- frequency-weighted first- and second-order FTS;
- comparison with persistence (last value) and a linear-trend baseline;
- MAE, RMSE and MAPE calculated separately for every horizon;
- recursive FTS forecasting, including the fallback used for unseen higher-order rules.

- оценивание методом скользящего начала прогноза без утечки будущих данных;
- прогнозы на **1, 2 и 3 шага вперёд**;
- частотно-взвешенные FTS первого и второго порядка;
- сравнение с прогнозом последним значением и линейным трендом;
- MAE, RMSE и MAPE отдельно для каждого горизонта;
- рекурсивный FTS-прогноз и явное правило отката для невиданных правил высокого порядка.

## Theory (English)

### Fuzzy sets and fuzzification

A fuzzy set replaces a hard boundary with a membership function
`mu_A(x) in [0, 1]`. The observed range is padded and divided into equal intervals. Their midpoints are the peaks of overlapping triangular fuzzy sets `A1, ..., Ak`. A numeric observation is **fuzzified** by assigning it to the set with the greatest membership (equivalently, the nearest midpoint for these regular triangles).

### Fuzzy logical relationships

For first-order FTS, adjacent states create rules `A(t-1) -> A(t)`. A second-order model uses `(A(t-2), A(t-1)) -> A(t)` and can represent short local patterns, but its rule table is sparser. The notebook retains transition frequencies. Thus, if `Ai` was followed by `Aj` three times and `Ak` once, their midpoint weights are 3/4 and 1/4 rather than 1/2 and 1/2.

### Defuzzification and several steps ahead

The next crisp forecast is the frequency-weighted mean of consequent-set midpoints. For an unseen second-order context, the implementation backs off to the learned first-order rule; if that too is unseen, it uses the current state's midpoint. Multi-step forecasts are recursive: fuzzify the predicted value, append that predicted state, and apply the rules again. Error normally grows with horizon because later forecasts depend on earlier forecasts.

### Honest evaluation

At each rolling origin, a model sees only the prefix ending at that origin. It then predicts the next three held-out observations. Metrics aggregate errors by horizon. MAPE is intuitive here because enrollment never approaches zero; MAE retains the original unit, while RMSE penalizes large misses more strongly. This small data set is educational, not evidence that one method is universally superior.

## Теория (Русский)

### Нечёткие множества и фаззификация

Нечёткое множество заменяет жёсткую границу функцией принадлежности
`mu_A(x) in [0, 1]`. Диапазон наблюдений расширяется небольшим запасом и разбивается на равные интервалы. Их середины становятся вершинами перекрывающихся треугольных множеств `A1, ..., Ak`. Число **фаззифицируется** выбором множества с максимальной степенью принадлежности (для таких равномерных треугольников — ближайшей середины).

### Нечёткие логические отношения

FTS первого порядка формирует из соседних состояний правила `A(t-1) -> A(t)`. Модель второго порядка использует `(A(t-2), A(t-1)) -> A(t)`: она способна учитывать короткий локальный паттерн, но таблица правил получается более разреженной. В ноутбуке сохраняются частоты переходов. Поэтому если после `Ai` состояние `Aj` встречалось три раза, а `Ak` один раз, их середины получают веса 3/4 и 1/4, а не 1/2 и 1/2.

### Дефаззификация и прогноз на несколько шагов

Числовой прогноз — взвешенное по частотам среднее середин множеств в правой части правила. Для неизвестного контекста второго порядка алгоритм использует изученное правило первого порядка, а при отсутствии и такого правила — середину текущего множества. Многошаговый прогноз строится рекурсивно: предсказанное число фаззифицируется, состояние добавляется в историю, и правило применяется снова. Поэтому с ростом горизонта ошибка обычно увеличивается.

### Корректное оценивание

В каждой точке скользящего теста модель получает только прошлый префикс ряда и прогнозирует три скрытых наблюдения. Метрики объединяются отдельно по каждому горизонту. MAPE здесь уместна, поскольку значения набора далеки от нуля; MAE измеряется в исходных единицах, а RMSE сильнее штрафует крупные ошибки. Маленький учебный набор не доказывает универсального превосходства какого-либо метода.

## Repository scope / Состав репозитория

The previous general-purpose fuzzy-regression package and demo entry point were unrelated to the time-series tutorial and have been removed. Keeping one executable notebook avoids two conflicting APIs and unnecessary `cvxopt`/packaging machinery.

Прежняя библиотека общей нечёткой регрессии и её демонстрационный entry point не использовались в примере временного ряда и удалены. Один исполняемый ноутбук устраняет две конкурирующие реализации и ненужные зависимости от `cvxopt` и упаковочного кода.

## References / Литература

1. Zadeh, L. A. (1965). *Fuzzy sets*. Information and Control, 8(3), 338–353.
2. Song, Q., & Chissom, B. S. (1993). *Fuzzy time series and its models*. Fuzzy Sets and Systems, 54(3), 269–277.
3. Tsaur, R.-C. (2012). *A fuzzy time series–Markov chain model with an application to forecasting the exchange rate between the Taiwan and US dollar*. IJICIC, 8(7B), 4931–4942.
