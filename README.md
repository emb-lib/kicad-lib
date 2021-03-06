# Библиотека для САПР KiCad

## Назначение и состав

### Назначение

Проект имеет целью предоставить набор библиотек, выполненных максимально близко к требованиям стандартов ЕСКД в сочетании с возможностью описывать компоненты в простом и удобном для редактирования тектовом формате, пригодном для последующей автоматизированной генерации библиотек. Все схемные компоненты спроектированы на основе сетки 100 mils (2.54 мм).

### Состав

В настоящее время проект содержит только схемные библиотеки. Библиотеки можно условно разделить на две категории:

* созданные вручную;
* исходные описания компонентов-микросхем для последющей генерации файлов схемных библиотек.
 
#### Библиотеки, созданные вручную

| Имя | Описание |
|----------|-------------|
| `discrete.lib`| содержит резисторы, конденсаторы, индуктивные элементы, биполярные, полевые и МОП транзисторы, диоды, светодиоды, коммутационные элементы и т.д. |
| `conn.lib`    | несколько соединителей одиночного типа |
| `graphic.lib` | графические символы |
| `power.lib`   | символы цепей питания |



#### Библиотеки микросхем, создаваемые из описаний

Основная идея подхода основана на том, что символы микросхем в соответствии с требованиями ЕСКД не позволяют большого разнообразия исполнения и могут создаваться по заданному алгоритму. Для исходных описаний выбран YAML формат, который сочетает в себе простоту просмотра и редактирования средствами обычного (но лучше "программерского") текстового редактора с хорошей структурированностью, желательной для машинной обработки данных. 

### Создание библиотек микросхем из описаний

#### Формат описания компонента

Описание микросхемы очень простое и состоит из двух основных частей:

* общие свойства компонента;
* описания частей (parts) компонента.

В свою очередь описание каждой части состоит из:

* функциональной метки - то, что отображается на символе;
* описания секций (вертикальных отделов): количество и размеры;
* групп выводов.
 
Каждая группа выводов содержит следующие параметры:

* сторона размещения (левая или правая);
* флаг разделителя - указывает, надо ли отделять группу выводов от следующей группы горизонтальной линией;
* списки вывовов - каждый вывод описывается списком из:
  * номера вывода;
  * имени выввода;
  * расстоянием (в шагах вертикальной разметки) от предыдущего элемента (предыдущий элемент - это либо верх компонента, либо предыдущий вывод, либо разделитель групп).

Пример описания приведён ниже.

```YAML
#-------------------------------------------------------------------------------
#
#    Общие свойства компонента
#
Name:          LTC2256IUJ-14         # название компонента
NameOffset:    600                   # смещение названия по горизонтали (mils); опционально, значение по умолчанию: 1000
Description:   Cmp description       # описание; опционально; пример: 14-Bit, 25Msps Ultralow Power 1.8V ADC
Keywords:      Keywords list         # ключевые слова; опционально
Ref:           D                     # база позиционного обозначения
Footprint:     QFN-40                # посадочное место по умолчанию; опционально
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
    #    Pin groups definition
    PinGroup1:                       # имя группы выводов; задаётся по схеме PinGroupN, где N - целые числа
        Side:   Left                 # сторона размещения группы
        Sep:    Yes                  # флаг разделителя, величины Yes | No
        Height: 4                    # высота группы, заданная явно; опционально, по умолчанию высота вычисляется 
                                     # исходя из описания выводов (их количество и промежутки между ними)
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
### Сборка библиотек

Процесс сборки библиотек очень похож на процесс сборки программных продуктов, широко применямый в отрасли разработки программного обеспечения (ПО). Он включает в себя:
 
* компиляцию компонента (`.cmp` файл) из исходного описания (`.yml` файл) (аналог операции "компиляция `исходный файл` -> `объектный файл`");
* сборка файла библиотеки (`.lib` файл) из файлов компонетов (`.cmp` файлы) (аналог операции "сборка (линковка) `исполняемого файла` из `списка объектных файлов`
 
Особенностью является то, что при сборке программных продуктов как правило основная цель одна (хотя и может состоять из нескольких файлов), а в данном случае целей сразу много - по количеству библиотек. И хотя при разработке ПО целей тоже может быть не одна, тем не менее каждая такая цель собирается индивидуально (например, имеет отдельные `Makefile` или другие служебные файлы системы сборки). В даном же случае все цели (библиотеки) собираются за один проход в цикле. В результате генерируется по два файла на каждую библиотеку: 

* `.lib`: собственно файл библиотеки компонентов;
* `.dcm`: файл документации к библиотке, содержит текстовые описания компонентов, доступные из программ САПР `KiCad`, и ключевые слова.
 
#### Инструментарий

Сборка библиотек осуществляется на основе скриптов языка программирования (ЯП) `Python` и использует:

* утилиту `SCons`, которая, собственно, и осуществляет процесс сборки под управленем конфигурационного файла `SConstruct`;
* утилиту `icgen.py` конвертации `YAML` описания компонента в формат компонента `KiCad`. Для работы утилита требует `Python3` и пакет `python3-yaml`. Взять утилиту можно из [этого репозитория](https://github.com/emb-lib/kicad-tools).

#### Структура проекта и настройка окружения

Проект имеет следующую структуру (dcm и некоторые служебные файлы для краткости не показаны):

```YAML
../lib                   # корневая директория библиотеки (lib - имя для примера, имя может быть любым
     /sch                # директория схемных библиотек
       /src              # директория, содержащая директории с исходными описаниями компонентов
          /adi           # микросхемы Analog Devices Inc.
          /lt            # микросхемы Linear Technology
          /ti            # микросхемы Texas Instruments
          /<..>          # и т.д.
       /ic               # директория со сгенерированными библиотеками
          /cmp           # директория, куда помещаются файлы компонентов; 
             /adi        # файлы помещаются не в одну "кучу", а по директориям,
             /lt         # соответствующим структуре директорий исходных описаний
             /ti         #
             /<...>      #
          adi.lib        # сгенерированные библиотеки
          lt.lib         #
          ti.lib         #
          <...>.lib      #
       <libfiles>        # библиотеки созданные вручную
     SConstruct          # сборочный скрипт для утилиты Scons
```

При запуске сборки автоматически создаётся директория `ic` и всё её содержимое. В конфигурации САПР `KiCad` необходимо указать пути к библиоткам, для приведённого выше примера это два пути:

* `<path-to-lib>/`
* `<path-to-lib/ic`

Для того, чтобы `scons` "нашёл" утилиту конвертации `icgen.py`, необходимо в системе создать переменную окружения `KICAD_TOOLS`, которая будет указывать путь до директории с `icgen.py`, например: 

`export KICAD_TOOLS="$CAD/kicad/tools"`

#### Рабочий процесс

Рабочий процесс сводится к созданию файлов описания в `YAML` формате, помещения их в директориях `src/<имя библиотеки>`и запуска `scons`, например, вот так:

`scons -Q -s -D all`

или так, если нужно пересобрать все библиотеки:

`scons -Q -s -D rebuild all`

Разумеется, в первом случае генерируются только изменённые компоненты и пересобирается соответствующая библиотека.

### Примеры

#### LTC2256, 14-разрядный высокоскоростной малопотребляющий АЦП

##### Описание
```YAML
Name:          LTC2256IUJ-14
Description:   14-Bit, 25Msps Ultralow Power 1.8V ADC
Keywords:      Ultralow Power ADC Linear Techology
Ref:           D
Footprint:     QFN-40
PinLen:        400  # mils
PinNameOffset: 50
Spacing:       200  # mils
Filled:        Yes

Part1: 
    Caption:  [ADC, 1500, -300]
    Sections: [1000, 1000, 1000]
                                       
    PinGroup1:
        Side: Left
        Sep:  Yes
        Pins:
            - [1, AIN+, 1]
            - [2, AIN-, 3]
                                                         
    PinGroup2:
        Side: Left
        Sep:  Yes
        Pins:
            - [9,  VDD, 1]
            - [10, VDD, 1]
            - [40, VDD, 1]
            - [3,  VDD, 3]

    PinGroup3:
        Side: Right
        Sep:  Yes
        Pins:
            - [26, OVDD, 1]
            - [25, OGND, 3]

    PinGroup4:
        Side: Left
        Sep:  Yes
        Pins:
            - [37, VCM,  1]
            - [4,  REFH, 2]
            - [5,  REFH, 1]
            - [6,  REFL, 3]
            - [7,  REFL, 1]

    PinGroup5:
        Side: Left
        Sep:  No
        Pins:
            - [39, SENSE, 1]
            - [38, VREF,  1]


    PinGroup6:
        Side: Right
        Sep:  Yes
        Pins:
            - [18, D0_1,   1]
            - [20, D2_3,   1]
            - [22, D4_5,   1]
            - [24, D6_7,   1]
            - [30, D8_9,   1]
            - [32, D10_11, 1]
            - [34, D12_13, 1]
            - [36, OF,     2]

    PinGroup7:
        Side: Right
        Sep:  Yes
        Pins:
            - [8,  PAR/~SER, 1]
            - [13, ~CS,      1]
            - [14, SCK,      1]
            - [15, SDI,      1]
            - [16, SDO,      1]

    PinGroup8:
        Side: Right
        Sep:  No
        Pins:
            - [11, ENC+,    1]
            - [12, ENC-,    1]
            - [28, CLKOUT+, 1]
            - [27, CLKOUT-, 1]

```

##### Результат
![ltc2256](https://cloud.githubusercontent.com/assets/14235855/15468736/adc111d8-2107-11e6-88c5-8113eb2e7557.png)

#### Сдвоенный операционный усилитель

Компонент разбит на три части: две - "половинки" усилителя, третья - питание.

```YAML

#-------------------------------------------------------------------------------
Name:          LT6234IDD
NameOffset:    600
Description:   Low-noise low-power dual OpAmp
Keywords:      Dual Low-noise Linear Techology
Ref:           D
PinLen:        200  # mils
PinNameOffset: 50
Spacing:       200  # mils
Filled:        Yes

#-------------------------------------------------------------------------------
Part1: 
    Caption:  ['|>', 700, -250]
    Sections: [500, 500, 500]
                                       
    PinGroup1:
        Side: Left
        Sep:  No
        Pins:
            - [3, IN+, 1]
            - [2, IN-, 3]
                                                         
    PinGroup2:
        Side: Right
        Sep:  No
        Pins:
            - [1,  OUT, 1]

#-------------------------------------------------------------------------------    
Part2: 
    Caption: ['|>', 700, -250]
    Sections: [500, 500, 500]
                                       
    PinGroup1:
        Side: Left
        Sep:  No
        Pins:
            - [5, IN+, 1]
            - [6, IN-, 3]
                                                         
    PinGroup2:
        Side: Right
        Sep:  No
        Pins:
            - [7,  OUT, 1]

#-------------------------------------------------------------------------------
Part3: 
    Caption: 
    Sections: [800]
                                       
    PinGroup1:
        Side: Left
        Sep:  No
        Pins:
            - [8, V+, 1]
            - [4, V-, 3]
            
#-------------------------------------------------------------------------------
```

##### Результат

![lt6234](https://cloud.githubusercontent.com/assets/14235855/15468744/b612f3ba-2107-11e6-8b7d-e314618acead.png)




