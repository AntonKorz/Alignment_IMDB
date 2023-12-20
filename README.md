# Alignment_IMDB

Весь проект содержится в ноутбуке main.


дальше будет дополнение последней части задания из main
# Level 3 P.S.

В последней части задания я писал о том, что можно поиграть с регуляризацией loss-функции. Обычно во всех методах используется KL-дивергенция или её обобщения. Минусом KL-дивергенции является то, что она игнорирует "геометрию" пространства, на котором задаются вероятностные распределения. Например, если SFT-политика дает высокую вероятность слову "прекрасный" и маленькую вероятность слову "шикарный",  наша ненешняя политика - наоборот, то KL-дивергенция будет высокой, хотя нам очевидно, что сильно штрафовать тут не надо. Вторым минусом является то, что она довольно нестабильна, если те или иные вероятности почти нулевые.

- Что можно сделать с этим?
- Попробовать использовать метрику Вассерштейна в качестве метрики!
- Но как?
- Берем словарь нашей модельки -> обучаем на этих токенах w2v -> получаем метрику на словаре, индуцированную метрикой латентного пространства -> с этой метрикой оптимизируем loss, где в качестве регуляризации берется расстояние Вассерштейна между SFT-распределением на словаре и нашим нынешним распределением.

В словаре GPT-2 50 257 различных токенов, в других различных открытых LLM до 250 000. Можно взять обычный w2v и отобразить весь наш словарь в латентное пространство небольшой размерности. Благодаря этому мы можем определить метрику на словаре, например, L2-normalized Euclidean distance. Или же можно учесть, что скорее всего наш словарь будет не тупым облаком точек, а каким-то интересным многообразием меньшей размерности, и использовать это для определения метрики. Короче, здесь есть простор для экспериментов.

Если у нас есть метрика, то можно определить расстояние Вассерштейна. Правда, если открыть его формулу, то можно быстро понять, что это расстояние довольно сложно считать, и еще сложнее считать субградиент. И это большая проблема. В этой статье (https://proceedings.neurips.cc/paper_files/paper/2015/file/a9eb812238f753132652ae09963a05e9-Paper.pdf) предлагают итерационный алгоритм для приближенного поиска градиента. Но проблему это вроде не решает, потому что придется хранить матрицу размера n_vocab^2 (хотя возможно этот метод можно еще как-то оптимизировать). С другой стороны, в RLHF обычно применяется не самый быстрый PPO, и возможно подсчет Вассерштейн лосс не будет bottleneck.

Короче говоря, метод может быть даже прикольным, но работать скорее всего будет медленно. Однако это не мешает подумать еще в эту сторону и придумать что-то похожее и эффективное.

P.P.S. Блин, понял что все таки фигню придумал, потому что размер словаря слишком большой ¯\_(ツ)_/¯
