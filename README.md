# Решение задачи с семантическим разрывом
- ## Постановка задачи
На вход подается изображение со стулом А и столом Б.
Требуется установить, помещается ли стул А под стол Б, если их перемещать параллельным переносом и не отрывая от пола. Под словом «помещается» в данном контексте подразумевается полное погружении _сидушки_ стула под пролет стола со стороны, ближайшей к камере.
- ## Требования
1. Общие к созданию фотографий для датасета:
    1. сделаны с одного и того же устройства
    2. выполнены с одной и той же точки в пространстве (подлокотник дивана), наклон устройства по оси z отсутствует или незначителен (+- 5
градусов)
    3. выполнены при одинаковом освещении: темнота за окном + искусственный комнатный свет
    4. на фотографии, помимо фоновых, должны быть только объекты А и Б
2. Общие к объектам А и Б:
    1. должны находиться на поверхности пола
    2. должны стоять в естественном положении на 4-х ножках
    3. ни одна часть одного объекта не должна полностью перекрывать ни одну часть другого объекта
    4. должны располагаться на фото целиком
3. К объекту Б (столу)
    1. стол неподвижен и всегда стоит на своем месте (придвинут к стене)
4. Дополнительные
    1. для отработки алгоритма нахождения особых точек необходимо наличие шаблонной фотографии стола, сделанной по тем же правилам, но без стула в кадре.


P.S. Примеры корректных и некорректный входных данных можно найти в папке  `tz`.
- ## Алгоритм решения задачи
1. На исходном изображении по заданному шаблону с помощью дескриптора `SIFT` обнаруживаются особые точки. По точкам определяется матрица гомографии и вычисляются координаты пролета стола
2. Далее к изображению применяется уникальная маска  brown  для отделения интересующих объектов A и B. Применяется морфологическая операция  `binary_closing`  для улучшения результата
3. После этого на полученном изображении выполняется обнаружение всех линий при помощи преобразования Хафа
4. Обнаруживается точка пересечения линий, проходящих через пары верхних и нижних особых точек стола (далее называется _фокусом_)
5. **Ключевой шаг для принятия решения:** Находится количество линий, обнаруженных в п.3, проходящих в заданной окрестности от фокуса.
6. Если число линий, найденных в п.5, больше эмпирически подобранного числа  10 , то считается, что сидушка стула полностью помещается под стол.

- ## Обоснование алгоритма
Для понимания идеи, заложенной в шаги 5-6 алгоритма, предлагаю представить упрощение, что фотография выполняется не под углом к сцене, а с позиции, перпендикулярной пролету стола. Тогда бы для получения положительного ответа достаточно было бы обнаружить на картинке несколько линий, параллельных горизонтальным линиям через особые точки стола (линии сидушки, перегородок спинки стула).

При фотографии под углом возникают перспективные искажения (верхняя горизонтальная линия через особые точки не параллельна нижней, аналогично со стулом). В этом случае из-за оптического устройства глаза возникает следующий эффект: все параллельные линии сходятся в одной точке. А значит понять, насколько стул ровно стоит по отношению к пролету, можно оценив расстояние от линий, образованных стулом, до найденного фокуса (расстояния по сути является значением толерантности к погрешности положения стула и ошибкам алгоритмов обнаружения линий). При этом фактически не требуется каким-то образом отделять линии, образованные стулом, т.к. на изображении по требованиям могут быть только объекты A и B.

- ## Использование
1. Получить исходный код программы можно склонировав проект и перейдя на ветку `develop`:
```
$ git clone https://github.com/KoDim97/Computer-Vision.git
$ git checkout develop
```
2. Далее необходимо установить все требуемые зависимости:
```
$ pip install -r requirements.txt
```
3. После шагов 1-2 можно приступать к использованию программы. Для поведения «из коробки» необходимо загрузить [датасет](https://drive.google.com/drive/folders/1JKaPlvEHKXMJtWJ7OTF-BAKhe6CkSRt0?usp=sharing) и положить его в корневую директорию проекта. В противном случае пользователю предлагается настроить глобальные переменные в файле  `config.py`. Параметры в представлении не нуждаются, разве что отдельно отмечу, что переменная  `SHOW_STEPS`  отвечает за отрисовку всех промежуточных шагов алгоритма.

Две основных функции для тестирования алгоритма:  `predict(img_path)`  и  `estimate_model(image_name_list, correct_ans_list, silent_mode=False)`. Название и сигнатуры говорят сами за себя.

**Attention!** Для фотографии разрешения 3024x4032 полный прогон алгоритма занимает порядка 40 секунд, т.е. выполнение  `estimate_model`  на всем датасете занимает почти 30 минут.
