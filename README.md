## Библиотека для САПР KiCad

### Назначение и состав

#### Назначение

Проект имеет целью предоставить набор библиотек, выполненных максимально близко к требованиям стандартов ЕСКД в сочетании с возможностью описывать компоненты в простом и удобном для редактирования тектовом формате, пригодном для последующей автоматизированной генерации библиотек. Все схемные компоненты спроектированы на основе сетки 100 mils (2.54 мм).

#### Состав

В настоящее время проект содержит только схемные библиотеки. Библиотеки можно условно разделить на две категории:

* созданные вручную;
* исходные описания компонентов-микросхем для последющей генерации файлов схемных библиотек.
 
##### Библиотеки, созданные внучную

| Имя | Описание |
|----------|-------------|
| `discrete.lib`| содержит резисторы, конденсаторы, индуктивные элементы, биполярные, полевые и МОП транзисторы, диоды, светодиоды, коммутационные элементы и т.д. |
| `conn.lib`    | несколько соединителей одиночного типа |
| `graphic.lib` | графические символы |
| `power.lib`   | симолы цепей питания |



##### Генерирование библиотек микросхем из описаний

Основная идея подхода основана на том, что символы микросхем в соответствии с требованиями ЕСКД не позволяют большого разнообразия исполнения и могут создаваться по заданному алгоритму. Для исходных описаний выбран YAML формат, который сочетает в себе простоту просмотра и редакторования средствами обычного (но лучше "программерского") текстового редактора с хорошей структурированностью, желательной для машинной обработки данных. Описание микросхемы очень простое и состоит из двух основных частей:

* общие свойства компонента;
* описания частей (parts) компонента.

В свою очередь, каждая часть состоит из 

* функциональной метки - то, что отображается на символе;
* описания секций (вертикальных отделов): количество и размеры;
* групп выводов.
 
Каждая группа выводов содержит следующие поля:

* сторона размещения (левая или правая);
* флаг разделителя - указывает, надо ли отделать группу выводов от следующей горизонтальной линией;
* списки вывовов - каждый вывод описывается списком из:
  * номера вывода;
  * имени выввода;
  * расстоянием (в шагах вертикальной разметки) от предыдущего элемента.

Пример описания приведён ниже.

```YAML
#-------------------------------------------------------------------------------
#
#    Общие свойства компонента
#
Name:          LTC2256IUJ-14         # название компонента
NameOffset:    600                   # смещение названия по горизонтали (mils); опционально, значение по умолчанию: 1000
Description:   Cmp description       # описание; опциональлно; пример: High-speed low-power 14-bit ADC
Keywords:      Keywords list         # ключевые слова; опционально
Ref:           D                     # база позиционного обозначения
PinLen:        400                   # длина вывода (mils)
PinNameOffset: 50                    # смещение имени пина (mils)
Spacing:       200                   # шаг разметки по вертикали (mils)
Filled:        Yes                   # флаг, задающий, надо ли заполнять компонент цветом, значенияs: Yes | No

#-------------------------------------------------------------------------------
#
#    Описания частей
#
#    Компонент может иметь сколько угодно частей. Части именуются по схеме PartN, где N - целые числа, начиная с 1
#
Part1:                               # имя части
    Caption:  [ADC, 1500, -300]      # функциональная метка части; Текст и координаты размещения
    Sections: [1000, 1000, 1000]     # описание секций: количество и размеры (ширина). Секций может быть 1, 2 или 3. 
                                     # Например, [500] задаёт одну секцию шириной 500 mil, 
                                     # [600, 600. 600] - 3 секции по 600 mil и т.д.
                   
    #---------------------------------------------------------------------------
    #
    #    Pin groupы definition
    PinGroup1:                       # имя группы выводов; задаётся по схеме PinGroupN, где N - целые числа
        Side: Left                   # сторона размещения группы
        Sep:  Yes                    # флаг разделителя, величины Yes | No
        Pins:                        #
            - [1, AIN+, 1]           # описание вывода: [<номер>, <имя>, <расстояние от предыдущего объекта>]
            - [2, AIN-, 3]           # предыдущим объектом может быть верх части, вывод или разделитель
                                                         
    PinGroup2:
        Side: Left
        Sep:  Yes
        Pins:
            - [9,  VDD, 1]
            - [10, VDD, 1]
            - [40, VDD, 1]
            - [3,  VDD, 3]
    ...

    PinGroupN:
        Side: Right
        Sep:  No
        Pins:
            - [11, ENC+,    1]
            - [12, ENC-,    1]
            - [28, CLKOUT+, 1]
            - [27, CLKOUT-, 1]

Part2:
     ...
...

PartN:
     ...
````





