# Проект по АвтОбрЕЯ Дарьи Матяш и Ксении Кошкиной
### Данные:
[Данные](https://drive.google.com/drive/folders/1glzkn6P8G-zh20TPEEdTr4W0uy-elWVJ)

Для BIO-разметки использовалась библиотека stanza. Для предсказания последовательностей использовался алгоритм CRF. 
Получившиеся метрики:
![Иллюстрация к проекту](https://github.com/kseniakoshkina/nlp-project/blob/main/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202021-12-28%20%D0%B2%2004.44.35.png)

### Aspect Based Sentiment Analysis

Описательная статистика датасета
Описательная статистика датасета будет фрагментарно даваться в разделах далее, здесь также кратко расскажем о датасете.

В файле train_aspects.txt лежит разметка каждого тоокена каждого предложения по тональности (neutral/positive/negative/both) и аскпекту (Food, Interior, Whole, Service), ниже приведен пример разметки некоторых токенов по обоим признакам.

Всего в корпусе 284 текста (train_reviews.txt)

Задача NER: предсказываем BIO-тег каждому токену

Для начала рассмотрим распределение тегов (без O):

Есть небольшой дисбаланс классов, можем предположить, что хуже всего будут предсказазываться самые малочисленные классы - весь домен Price (B-Price, I-Price), возможно чуть лучше I-Interior, I-Whole. Конечно же, идеально, чтобы «путались» такие классы внутри своего домена: например, вместе I-Price предсказывался B-Price - это как будто лучше B-Whole тега.
Для решения данной задачи мы попробовали следующие методы:
 • MultinomialNB
 • Conditional Random Fields
 • а также в планах было использовать tiny-RuBERT модель, но CRF нас вполне устроил своим результатом
Мы оставили два самых «успешных» эксперимента, в тетради представлен код с перебором гиперпараметров для CRF,  performance модели с лучшей метрикой (F1-score) со следующими параметрами:

c1 = 0.3196504797629
c2 = 0.00034474520532276035
algorithm='lbfgs'
max_iterations=100,
all_possible_transitions=True

Ниже classification report на тестовых данных:


Посмотрим на confusion matrix:
 

Интерпретация результатов

В большинстве случаев классы были предсказаны верно! Всего была допущена 131 ошибка из 11696 всех тегов. Если не считать ‘O’ класс (как традиционно делается), то была допущена 131 ошибка из 1228.
Анализ ошибок модели показал следующее:
 1. конечно, преобладание тегов ‘O’ несколько сместило распределение предсказаний модели, особенно, как и предполагалось ранее, небольших по количеству примеров классов
 2. приятно было, что случались внутридоменные миксы, однако все еще были ситуации «не очень»: когда первый токен последовательности аспектов предсказывался как inside-тег: например, все три слова одного аспекта как  I-класс, а еще хуже, когда первый токен выделялся как I, а последующий как B - это может говорить о переобучении модели на определенные слова, эту проблему также в дальнейшем может требовать решения при более частых случаях такого рода ошибок
 3. также видно, что самые маленькие классы смешивались с самыми объемными - но тут мы Америку не открыли и на confusion matrix это можно так же заметить

Дальнейшие шаги, возможности

 4. Можно попробовать на задаче NER-классификации предобученный BERT на русском языке, я бы даже брала его дистиллированную версию для более легкого и быстрого файн-тьюнинга. Там может помочь с проблемой дисбаланса класса взвешенная кросс-энтропия, возможно, маленький learning rate, scheduler также мог бы положительно повлиять на лучшую сходимость модели.
 5. Можно больше внимания уделить контексту аспектов: учитывать чуть больше контекста справа/слева, однако это идея может и негативно сказаться на качестве модели
 6. Можно подумать что-то про лемматизацию всего текста, однако также есть риски того, что модель по итогу будет иметь чуть хуже обобщающую способность



 ### Задача классификации: оценка тональности

Для начала посмотрим на распределение тегов на обучабщем датасете:

![Иллюстрация к проекту](https://github.com/kseniakoshkina/nlp-project/blob/main/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202021-12-28%20%D0%B2%2004.46.54.png)

Видим сильный «перевес» класса положительной тональности, понимаем, что возможен риск переобучения моделей в стороны полного предсказания только позитивного класса, а также возможно, что самый малочисленный класс может очень плохо предсказываться

Данные брались из текстов отзывов, которые были размечены. Каждому слову в отзывах приписывалась тональность. Сначала тональность бралась из трейнового размеченного датасета - train_aspects (если там присутствовало нужное слово). Если слова не оказывалось в трейне, то тональность слова бралась из тональных словарей: либо карта слов, либо rusentilex 2017. Таким же образом размечался тестовый датасет. Минус заключается в том, что модели подаются не изначальные данные, а размеченные. Метрики можно посмотреть ниже.

Однако мы также пробовали передать в классификатор обычные данные из трейнового и тестового датасета. Результаты получились нормальными, но непонятно как это будет работать на других данных.

Дальнейшие шаги, возможности:
1. Была попытка использования предобученных эмбеддингов Word2Vec, но она не привела к положительным результатам, способных улучшить качество, однако этот подход можно развивать далее: мир не сошелся на Word2Vec предобученных векторах - можно попробовать взять предобученный FastText или обучить на своем корпусе Word2vec/FastText (кажется более успешной первая идея с предобученный FastText’ом)
 2. Можно эмбеддить не только слово или словосочетание, но и +/- n соседей рядом, это может помочь повысить финальную метрику
 3. Можно создавать «костыли» с эмбеддингами, например, сделать какой-то датасет с парами «самых частотных» представителей каждого класса и на тестовой выборке искать косинусной близостью самый близкий эмбеддинг из сохраненный и приписывать класс сохраненного эмбеддинга. Это может иметь обратную сторону медали: так можно найти полные антонимы, а в задаче определения тональности это может быть опасно, поэтому для большей подстраховки стоит брать топ-n самых близких эмбеддингов. А еще можно задуматься, как в целом находить «самых типичных представителей класса» - можно попробовать сделать кластеризацию на 4 класса и взять центроиды и посмотреть, кто они, и их и сохранить как самых типичных и по ним определять классы на тестовой выборке.

В конечно итоге использовался классификатор PassiveAgressive. Также были затестированы наивный байес, логистическая регрессия и рэндом форест. 
Посмотрим на Confusion Matrix:

![Иллюстрация к проекту](https://github.com/kseniakoshkina/nlp-project/blob/main/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202021-12-28%20%D0%B2%2004.48.38.png)

Минус заключается в том, что различие между классами достаточно небольшое. Классификатор ошибается, потому что нейтральные и позитивные слова могут иметь мало различий.
