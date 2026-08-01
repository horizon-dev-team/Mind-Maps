# Code Guide

[ToC]

# Общее

## Операторы

- Не используйте `goto`. Это плохо.
- Не используйте оператор `:` для обхода проверок типобезопасности. Вместо этого приводите переменную к нужному типу.
- Не используйте `del`, это жутко медленно. Используйте `qdel()`.
- Не используйте `<>`, в мире SS13 это вообще не применяется. Используйте `!=`, это гораздо разумнее.

## Что следует использовать

- Используйте `SPAWN()`вместо `spawn()`
- Используйте `TIME` вместо `world.timeofday`
- Битовые флаги (Bitflags) (`&`) - пишите как `bitfield & bitflag`
- Используйте `'foo.ogg'` вместо `"foo.ogg"` для ресурсов, если только вам не нужно собирать строку (например, `"foo_[rand(2)].ogg"`).
- Используйте `FALSE` и `TRUE` вместо `0` и `1` для булевых значений.
- Используйте `x in y` вместо `y.Find(x)`, если только вам не нужен индекс.
  - Естественно, это не относится к регулярным выражениям.

# Синтаксис

## Комментирование

### Цель

По возможности мы всегда хотим, чтобы люди четко документировали свой код, чтобы другие (или вы сами в будущем) могли понять и узнать, почему был написан тот или иной кусок кода.

Unless the code is extremely complex, what one generally wants to comment is the _motivation_ behind a certain piece of code, or what it's supposed to fix - rather than what it's actually doing.

### Документирующие комментарии (Doc Comments)

Кроме того, у нас настроена система «документирующих комментариев». Она заключается в простом комментировании кода вот так:

```cs
/obj/item/clothing/suit
    /// If TRUE the suit will hide whoever is wearing it's hair
    var/over_hair = FALSE
```

Используя этот метод, при наведении на переменную или объект будет отображаться информация из комментария. Например:

![](https://i.imgur.com/IdKpEtf.png)

Более подробную информацию об этой системе см. на [DMByExample page](https://spacestation13.github.io/DMByExample/meta/dmdoc.html).

## Используемые дефайны (Defines)

### Дефайны времени

В кодовой базе есть некоторые дефайны (например, `SECONDS`), которые автоматически умножают число на правильное значение, чтобы получить число в децисекундах. Использовать их предпочтительнее, чем указывать буквальное значение в децисекундах.

### Единицы Измерения СИ (SI Units)

В кодовой базе также есть дефайны для других единиц СИ, таких как `WATTS`. Также доступны приставки единиц СИ, такие как `MILLI`. Их следует использовать всегда, когда вы имеете дело с величиной, являющейся единицей СИ. Если вы используете производную единицу, добавьте ее формулу в дефайны. Их можно объединять в цепочки вот так: `100 MILLI WATTS` или `1 KILO METER`.

## Никаких магических чисел

Не используйте числа, за которыми не стоит никакого объяснения. Вместо этого рекомендуется либо поместить их в константу, в локальный файловый #define, либо в глобальный #define.

<span style="color: red">Плохо:</span>

```csharp
proc/do_stuff(thing)
	switch(thing)
		if(0)
			stuff
		if(1)
			other stuff
```

<span style="color: green">Хорошо:</span>

```csharp
#define DO_THING_CORRECT 0
#define DO_THING_OTHER 1
proc/do_stuff(thing)
	switch(thing)
		if(DO_THING_CORRECT)
		    stuff
		if(DO_THING_OTHER)
			other stuff
```

Если вам не нужен этот дефайн нигде вне этого файла/процедуры, обязательно уберите его после использования:

```csharp
#undef DO_THING_CORRECT
#undef DO_THING_OTHER
```

## Используйте ранние возвраты (early returns)

Мы не хотим видеть десятки уровней вложенности, не заключайте процедуру в блок `if`, если вы можете просто сделать возврат при выполнении условия.

<span style="color: red">Плохо:</span>

```csharp
obj/test/proc/coolstuff()
    if (foo)
        if (!bar)
            if (baz == 420)
                do_stuff
```

<span style="color: green">Хорошо:</span>

```csharp
obj/test/proc/coolstuff()
    if (!foo || bar)
        return
    if (baz == "error_code")
        return
    do_stuff
```

## `foo.len` vs. `length(foo)`

В нашей кодовой базе используется последний вариант, синтаксис `length(foo)`.

Синтаксис `.len` вызывает ошибку времени выполнения (runtime) на нулевом `foo`, тогда как синтаксис `length()` - нет.
Он также быстрее (~6%) по причинам, связанным с внутренним байт-кодом (что на самом деле не имеет большого значения).

## Абстрактные types и typesof

Некоторые типы существуют только как родители и никогда не должны создаваться в игре (например, `/obj/item`). Помечайте их с помощью макроса `ABSTRACT_TYPE(/type)` Проверить, является ли тип абстрактным, можно с помощью макроса `IS_ABSTRACT(/type)`.

Чтобы получить список всех конкретных (не абстрактных) подтипов типа, вы должны использовать `concrete_typesof(type)`, результат кэшируется, поэтому нет необходимости сохранять его самостоятельно. (Как следствие, пожалуйста, делайте `.Copy` списка, если хотите изменить его локально). Правильное использование `ABSTRACT_TYPE` + `concrete_typesof` обычно предпочтительнее, чем использование `typesof` и `childrentypesof`, хотя бывают исключения.

Если вы хотите дополнительно отфильтровать результаты `concrete_typesof` (например, по значению переменной или по черному списку), рассмотрите возможность использования `filtered_concrete_typesof(type, filter)`. `filter` это процедура, которая должна возвращать 1, если вы хотите включить элемент. Опять же, результат кэшируется (поэтому процедура `filter` не должна зависеть от внешних переменных или случайности).

Пример:

```javascript
ABSTRACT_TYPE(/obj/item/hat)
/obj/item/hat
	var/is_cool = FALSE

/obj/item/hat/uncool
	name = "Uncool Hat"

/obj/item/hat/cool
	name = "Cool hat"
	is_cool = TRUE

proc/is_hat_cool(hat_type)
	var/obj/item/hat/hat = hat_type
	return initial(hat.is_cool)

proc/random_cool_hat()
	return pick(filtered_concrete_typesof(/obj/item/hat, /proc/is_hat_cool))
```

Подробности смотрите в `_stdlib/_types.dm`.

## Пробелы (Whitespace)

### Пробелы после управляющих операторов

Смотрите: `if(x)` vs `if (x)`

Никому до этого нет дела. Изменение этого без веской причины крайне не одобряется.

### Пробелы в списках, определениях/вызовах процедур и арифметике

Всегда ставьте пробелы после запятых или операторов и перед операторами:

<span style="color: red">Плохо:</span>

```csharp
var/whatever=list(1,2,3)
var/whatever2=multiply_some_numbers(2,2,2)
var/whatever3=3+4-whatever2
```

<span style="color: green">Хорошо:</span>

```csharp
var/whatever = list(1, 2, 3)
var/whatever2 = multiply_some_numbers(2, 2, 2)
var/whatever3 = 3 + 4 - whatever2
```

## Очень длинные строки

Для длинных строк используйте `\` aв конце строки для переноса на несколько строк.
С рекомендуемым пакетом расширений в редакторе появится вертикальная линия; если ваш код уходит далеко за эту линию, пришло время разбить код на несколько строк. Ставьте пробел перед `\`.

Делая это, делайте отступ каждой следующей строки на 1 таб больше, чем начало правой части определения (или что-то подобное — для вызовов процедур, на 1 таб дальше имени процедуры и т.д.).

<span style="color: red">Плохо:</span>

```csharp
var/moby_dick = "Call me Ishmael. Some years ago—never mind how long precisely—having little or no money in my purse, and nothing particular to interest me on shore, I thought I would sail about a little and see the watery part of the world. It is a way I have of driving off the spleen and regulating the circulation. ""
```

<span style="color: green">Хорошо:</span>

```javascript
var/moby_dick = "Call me Ishmael. Some years ago—never mind how long precisely \
                    —having little or no money in my purse, and nothing \
                    particular to interest me on shore, I thought I would sail \
                    about a little and see the watery part of the world. It is \
                    a way I have of driving off the spleen and regulating \
                    the circulation."
```

Списки следуют аналогичному шаблону, но `\` не требуется.

<span style="color: red">Плохо:</span>

```csharp
var/list/list_of_integers = list(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50)
```

<span style="color: green">Хорошо:</span>

```csharp
var/list/list_of_integers = list(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14,
                                15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26,
                                27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38,
                                39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50)
```

Для списков с длинными выражениями вполне допустимо выделять одно выражение на строку.

<span style="color: green">Хорошо:</span>

```csharp
var/list/dog_types = list(/mob/animal/dog/labrador,
                         /mob/animal/dog/chihuahua,
                         /mob/animal/dog/daschund,
                         /mob/animal/dog/retriever,
                         /mob/animal/dog/pug)
```

## Всегда используйте явный `src`

В процедурах, которые вызываются для объекта (то есть во всем, кроме глобальных процедур), `src` это объект, для которого вызывается процедура. Чтобы сослаться на переменную этого объекта, вы всегда должны использовать `src.varname`. Простое обращение к `varname` (оставляя `src` неявным) следует избегать, так как это менее понятно и приводит к ошибкам при рефакторинге.

То же самое касается процедур — чтобы вызвать процедуру для того же объекта, используйте `src.procname()`.

<span style="color: red">Плохо:</span>

```javascript
/datum/thing
    var/num = 10

/datum/thing/proc/add_one(var/add_another_time)
    num += 1
    if (add_another_time)
        add_one(FALSE)
```

<span style="color: green">Хорошо:</span>

```javascript
/datum/thing
    var/num = 10

/datum/thing/proc/add_one(var/add_another_time)
    src.num += 1
    if (add_another_time)
        src.add_one(FALSE)
```

## Имена переменных и аргументов

Очень короткие (1-2 символа) имена переменных/аргументов допустимы _только_ если они являются стандартными именами для общих типов.

- `/atom` - `A`
- `/atom/movable` - `AM`
- `/obj` - `O`
- `/obj/item` - `I`
- `/obj/item/grab` - `G`
- `/mob` - `M`
- `/mob/living` - `L`
- `/mob/living/critter` и `/obj/critter` - `C`
- `/mob/living/carbon/human` - `H`
- `/turf` - `T`

Имена должны быть описательными, если только контекст и тип не делают немедленно понятным, для чего используется переменная. Это касается и приведенных выше имен — используйте их только тогда, когда понятно, для чего нужна переменная.

<span style="color: red">Плохо:</span>

```csharp
var/obj/item/M
var/turf/J

proc/feed_person_food(mob/person_one, mob/person_two, obj/item/food/F)
```

<span style="color: green">Хорошо:</span>

```csharp
var/obj/item/I
var/turf/T

proc/feed_person_food(mob/person_feeding, mob/person_eating, obj/item/food/fed_food)
proc/move_ghost_to_turf(mob/dead/ghost/target, turf/T)
```

## Блок-схема: Проверка расстояния между объектами

![](https://mermaid.ink/img/pako:eNqVV21vGkcQ_iujkyxAxU7lSP1gNZYguClSQlobN0pNg5a7BTY-dvHunjG1_N_7zN4bbw0JH-DYm5l9duaZl32OYpPI6CI6OXlWWvkLem74uVzIv4RVYpJK18ASNaZG-xv1r8S_xvkvy6fGy8vLyclIT1OziufCehr2Rprwscb45l2ftJQJeUPxXMb35MxC-rnSs4ajRDkvdCz5La-TwY6Wwut_WrmVVHlpRVobmstyjazQM0mPIs1kKZ4oMWve9fBtNCQ-mEfpqJMCnExKGa9S2bwb4vu0K1y9rk3-ZmD06YG3B00PjN81L5Kv-8dWU_Irkx_NkbDVIdI1hLKY118lysrYYwUmRCy1rxCzxEGjghZmQrHQuUzt3lJ1Zrz5hubM8KIIPilVXDaZWbGck9JjpRll7MfB13ejaG-tKbxZvPLCzqRvs81XmZO2NYroNOBShP-8Q0CV75BHig-rjKbr9_Vqv3_dHEVvGaSDivAFTuWApgg4AJcQaKX8nI2H7c9qO504Npn2jqbGUmysBq9ihNPB7_cwIVN5r7R0yrXJqVTFRtfKC8HxkHZd7QOYEBQ6oYniXDgbRYWzpE52vPbuajju9W-GcFb5mPuoAwv8223BNRX3cSx2vhtFR53zrgffvJPehSzYs0AT6VdSauoEpN0NdwwhvzROBZNmSmbyFfYdu1U-ZAopBLaxY_2OYG2C3_jMTsNDrs_qRrexYk02Qxz0mnS2mMDXUE3FWlpXG2gKWgIc0AqaiBk0iVes4510wUE2GYs0zlLhQVjBhwJvWCAAgEhr41w9NZ1Ky9j_plQ-ytSRlUCpHfUHv_UH_eHnb4VKZnGqEik0YnVVPsNo4djTIg7gjK79_R2B2t9gzNoFI65u377v9646g8CNLXP8ueIoc8DkEzO8MlEHfGrNAkFGuLr16bZOuAXCPWQoOMn4IJibP28711e9Y6BuKlSFtUO43IP1zacv5_QTrb-ct7Zw_jqxl51HoxJUv8JG6BC0miuuW4KrKdwIQiBlF2yvzTouFE1Hbm6yFOknuZwkXMBAU8fJSE3HlQ9UweLaZA0rWbFkEbNHoHIgSMzKEn4J2rUCtB8i0par91kF8ChcPrCKXfyhM_i9MxwWrm2iDOxVgFj5NU1Sg6r3HfSCwS2KVBseocg-1O7H20HvpixXG_-OVKyleoKLvgNql2nzCXshrmh7qMkoTJ8_Dnpo_40J_3WBjQ2aZCr1p2zbmngjxa-LMHD618UlB4ATrgIt5qAPH3Zh8t-JrLoqddu7xnJKaDkDKR6LASJ06Llch94sUitFsiZYsymwQ2ED0SfUAqMbnoxNlMZ4hI4dBzUcyOfQan6hv2xRuLZjNBRLQletuej98XpjR0wZGMHWbZpmOm9GNOWSxM0t5dZHzfOn81Ad2_T66XXxJH3cKkv9NnsrAmyEYIz5h322Q4Scrpf0815N6PYGQ0S3zJPh9e0Ve7HoPsGRMFkNM3DXtkOPV65NdP-P7c2bg-COYfsxXPu5sxR-jigkwITZY8x_ATJPHPCQZ2b-lbZIonJEOjs7M0sOIoZILLnNUWmWD8IrjMHyaG790cEJO97LxZJHAlMhCjPTI0_SZrtt5xCarpV37gNZJtDrnOcMYyWX1xFWRTlNNywGQ_BcbaHW5F8XTMwDRRfQg-8XWerVMi2V8ynYLWWspkomG1j6U_AmHMbRCo6ADdCgXRVkeIqPvA4bHahu9fWDTsPnshy2t64UxC84UepbQVjiOb--TISlco7bkdS1qK5kq2K8v1l-wdi8bFBAV7XSLeSXzNDqSpFLbpB_Rzakx8ZtISzuDes7Snw5qK8JlPuqZFHUjhC5hVAJboXPLDaKwo1wFF3gMRH2fhSN9AvkRObNzVrH0cVUpE62o2yZYH7ju5IVi2L15T-fxKZS?bgColor=5B5F67)

## Блок-схема: Получение объектов в области вокруг объекта

![](https://mermaid.ink/img/pako:eNqlV19v2zYQ_yoHvdgGtAZD34Ihw4BkW4BlA-KgRTEPASWebK4S6ZGUPaPod-_dUbIk2w2CVg-xdP_vx7vj5VNWOo3ZdVbVbl9ulI_wdLuyQI93Ls7_XmX3YBE1RAdrjKCgNiGCqwB36A9xY-wajIXgGgTlUeWgytJ5zXTSEfoGW09aplxl_yyS9fR3Z4JxNnkpSRtU4drYkQfhUtkCA6JIPridKmoMEDcqMgsKBGZC5V3TBRJd81Phb-bL5Y9vKeL1JnI8WvmPFkMAE8C6KJG2NlJ2lfOLiT8yyM7uOEloXDF4I87gJOkkrQ0qf5YKE8n1WM6rvSY0kqiz9WEsT0xgrrIljpVi66swPg7-HguUSImIf7iHvbIRZok0A2U1mBigdPRt6YXOhTAztqxbTbbo-OIGwWNoa-LKAc9NRYQad2Rp8fMIGrH5fEAJ5gOqzRnPOmb9OeYorZ-F-ovWMHMzjoBdUp5eiok_tt6VYBUhO3c7g_scnFd2jTlgLBfjXENbrL3aboDFyCr_zHvU8i4O1kji_GjjsYxUVfD4x0CdGHp2xb_PjCtZTCa4TujEiX7F9IlBft69_-tpvsoekbg2jHqj6Wt0b7hDJL0-vrO6ndqUImb53usswNYFw7HnYo9KF8IWS6PqrlP4ZIPR6BVLhTdTix9cy9gWFNGBSj62qqYXKRFOG32YL96sssWghVZ_DSNqhSk83BwXgHn4Llym1qS5i4NIk7e8q1zuaGUPPQYedZtO2HlQ7bqhGAWPEzjuq8ElJeAaE2kC5KSfopW5Js3dO-zb_tUglbUh5yc4JeIFqLoh-CJegg2Hk6wMEU1sLU1jauX77qKJsqN2KGis0HyD_YZg3KNMm6le447zZ3_ZEQdguReM1WZnNNUQI_N90J7mchHd48sEYiraru-5fIfWZ9tXr-__d--XhPuSZ044ULn8DyqI_fki4eZPj2TQpS5iCML0ZpC8xP-Qz3kKMtcoAfn99vAffxtVzQjcobtGFcoTozI07o9H1M3drkhZ86WY-SZLsHdv3x7374-X4z4iyR5gkD-NclzoxxGWj44mhLbpu8Zt1X-tjFOKJEBRu_IjXeCt1S9lK2KUq_zOeQ5fyWWVy817RQqvSfTpV0r0iaLvG1qu7f7CfXvbRaOxMhZH4-R4CaQLktIkh53yXKZfMDtcjNpvKYImiBbtcgRVjRWh6bxFn8Md7wCJG90WPK9Ek_zTUjasfvADPTf9KnZK5gOaEG_6teZMNG0qZ0uf6BwXu0scop-sf5Aiwv14TTsSqQrGbrp4utioz3oyvSYXUlHj7Q361MRUHxPuJ-I9kWTO6J3uBfPpa6DzBvUVjnUn65SYkxVqsg0KWQooy7MGfaOMpk3-E4usMjrphjbIa3rltXeVrexnkmu3WkW80yY6n11Xqg6YZ6qNbnmwZXYdfYu90K1R1AvNUQpF6SH9vyD_Nnz-Aqqx9s0?bgColor=5B5F67)

# Whack BYOND shit

## Startup/Runtime trade-offs with lists and the "hidden" init() proc

Сначала прочитайте комментарии в [this BYOND thread](http://www.byond.com/forum/post/2086980?page=2#comment19776775).

В ней есть два ключевых момента:

- Определение списка в объявлении переменной вызывает скрытую процедуру: init(). Если вам нужно определить список при старте, делайте это в New(), чтобы избежать накладных расходов на второй вызов (Init(), а затем New()).
- Это также потребляет больше памяти вплоть до момента, когда список действительно требуется, даже если объект в вопросе может никогда его не использовать!

Помните: хотя этот компромисс имеет смысл во многих случаях, он не покрывает их все. Тщательно обдумывайте свое дополнение, прежде чем решать, нужно ли его использовать.

## Циклы for без проверки типа (typecheckless for-loops)

При переборе списков у вас обычно есть два случая: когда список содержит только один тип, и когда список содержит множество типов.

Для _первого случая_ мы можем сделать специальную оптимизацию, которая называется "цикл for без проверки типа".

Синтаксис выглядит так:

```csharp
for (var/obj/foo/bar as anything in my_list)
	bar.boogie()
```

В итоге это дает нам увеличение скорости на 50%, поскольку в обычном типизированном цикле for при каждой итерации выполняется проверка `istype(thing, obj/foo)`.

**Будьте осторожны:** Если что-то в списке не соответствует указанному типу, произойдет ошибка времени выполнения (runtime)! Это включает случай, когда вы пытаетесь получить доступ к значению у null.

_Дополнительное примечание_: Если вы используете `by_type[]`, существует макрос, который делает это автоматически:

```csharp
for_by_tcl(iterator, type)
	loop stuff
```

Пока вам не нужно фильтровать определенные дочерние типы внутри by_type, вы можете использовать эту конструкцию.

## Циклы for-in-to

`for (var/i = 1, i <= some_value, i++)` — стандартный способ записи цикла for в большинстве языков, но синтаксис DM `for(var/i in 1 to some_value)` на самом деле быстрее в своей реализации.

Поэтому, где это возможно, рекомендуется использовать синтаксис DM. (Примечание: ключевое слово to включительно, поэтому оно автоматически заменяет `<=`; если вам нужно `<` тогда пишите `1 to some_value-1`).

**Будьте осторожны:** Если либо `some_value`, либо `i` изменяется внутри тела цикла (под `for(...)`) или если вы перебираете список и меняете его длину, то вы не можете использовать этот тип цикла!

## Копирование в циклах for-in

Almost all of the time when iterating through lists, we use the `for (var/i in some_list)` syntax. However, in **some very few** cases, we run into a performance issue.

Внутренне BYOND копирует `some_list` для этой операции, чтобы при переборе списка вы не пропускали элементы или не натыкались на них дважды, если изменяете список внутри цикла.

Однако это может вызвать проблемы с производительностью при больших списках _сложных_ объектов, обычно более ~5000 элементов.
Существует оптимизация производительности, но имейте в виду, что **она применима только если вы проходите менее половины списка**.
Возможно, вы прерываете цикл (break) после найденного элемента, который случайно оказался в списке, или вы хотите обработать только первые 20 записей и тому подобное.

Код, избегающий этого копирования списка, будет выглядеть так:

```csharp
/proc/direct_iteration()
	var/list/some_list = list() // just say this has 10,000 objs in it

	for (var/i in 1 to length(some_list))
	var/obj/mine = some_list[i]

	// do stuff with this object
	if (condition)
		break
```

## Возврат по умолчанию (`.`)

Как и другие языки семейства C, DM имеет оператор `.` или "точка", используемый для доступа к переменным/членам/функциям экземпляра объекта. Например:example:

```javascript
var/mob/M = foo
M.gib()
```

Однако в DM также есть точечная переменная (dot variable), доступ к которой осуществляется просто как `.` сама по себе, со значением по умолчанию null. Теперь, что особенного в операторе точки, так это то, что он автоматически возвращается (как в операторе return) в конце процедуры, при условии, что процедура уже не делает явного возврата (например, `return x`)

Поскольку `.` присутствует в каждой процедуре, мы используем ее как временную переменную. Однако оператор `.` не может заменить типизированную переменную — он может хранить любые данные, как и любая другая переменная в DM, просто к нему нельзя обращаться как к типизированной, хотя оператор `.` совместим с несколькими операторами, которые выглядят странно, но работают отлично, например: `.++` для увеличения значения `.`.

## Ключевые слова переменных global и static

В DM есть ключевое слово переменной, называемое `global`. Это ключевое слово var используется для переменных внутри типов. Например:

```javascript
/mob/var/global/foo = TRUE
```

Это **не** означает, что вы можете обращаться к нему везде, как к глобальной переменной. Вместо это означает, что эта переменная будет существовать только один раз для всех экземпляров своего типа, в данном случае эта переменная будет существовать только один раз для всех мобов — она является общей для всего своего типа. (Это гораздо больше похоже на ключевое слово `static` в других языках, таких как PHP/C++/C#/Java)

Запутанно, правда?

Существует также недокументированное ключевое слово `static` , которое имеет то же поведение, что и `global`, но более правильно описывает поведение DM. Поэтому всегда используйте `static` вместо `global` в переменных, так как это снижает уровень неожиданности при чтении кода.

## Избегайте ненужных проверок типов и скрытых null'ов в списках

Приведение типов в циклах `for` несет в себе подразумеваемую проверку `istype()`, которая отфильтровывает неподходящие типы, включая null. Фраза `as anything` может использоваться для пропуска этой проверки.

Если мы знаем, что список должен содержать только нужный тип, то мы хотим пропустить проверку не только из-за небольшой оптимизации, но и чтобы поймать любые пустые записи (null), которые могли проникнуть в список.

Null'ы в списках обычно указывают на неправильно обработанные ссылки, что усложняет отладку жестких удалений (hard deletes). Вызов ошибки времени выполнения (runtime) в таких случаях чаще всего является позитивным моментом.

<span style="color: red">Плохо:</span>

```javascript
var/list/bag_of_atoms = list(new /obj, new /atom, new /atom/movable, new /atom/movable)
var/highest_alpha = 0
for(var/atom/thing in bag_of_atoms)
	if(thing.alpha <= highest_alpha)
		continue
	highest_alpha = thing.alpha
```

<span style="color: green">Хорошо:</span>

```javascript
var/list/bag_of_atoms = list(new /obj, new /atom, new /atom/movable, new /atom/movable)
var/highest_alpha = 0
for(var/atom/thing as anything in bag_of_atoms)
	if(thing.alpha <= highest_alpha)
		continue
	highest_alpha = thing.alpha
```

## Ключевое слово `usr`

`usr`в общем смысле — это "моб, который вызвал эту процедуру". Он сохраняется через произвольное количество вложенных вызовов процедур. Если что-то не было вызвано мобом,`usr` равен null.

`usr` требуется для команд (verbs), которые специально вызываются мобом, и нужен для применения эффектов к вызывающему мобу.

Вне команд (во всех остальных процедурах) `usr` is **_крайне ненадежен_**. Отличный пример этого: если кто-то подключит датчик давления к пушке, а затем вы наступите на пластину, _вы будете `usr` для этого выстрела_.

Вместо использования `usr`, передавайте моб пользователя в вашу процедуру в качестве аргумента.

<span style="color: red">Плохо:</span>

```csharp
proc/explode_user()
    usr.explode()
```

<span style="color: green">Хорошо:</span>

```csharp
proc/explode_user(mob/user)
    user.explode()

/mob/verb/explode_yourself()
    set name = "Explode Yourself"
    usr.explode()
```

## `as mob`, `as obj`, и т.д.

В командах (verbs), при вызове из командной строки, они позволяют пользователю автоматически заполнять результаты.

Вне команд они ничего не делают и должны быть удалены.

<span style="color: red">Плохо:</span>

```csharp
proc/give_mob_item(mob/person as mob, obj/item/gift as obj)
```

<span style="color: green">Хорошо:</span>

```csharp
proc/give_mob_item(mob/person, obj/item/gift)
mob/verb/get_mob_to_yourself(mob/target as mob)
```

_Дополнительное примечание_: Это относится в целом ко всему,
_используемому_ как команда (verb), что может быть любой процедурой, добавленной в атом
через `atom.verbs += /proc/x`.

Поэтому будьте осторожны при удалении `as x`, чтобы
убедиться, что оно не используется как команда где-то еще.

## `round()` иногда притворяется `floor()`

Согласно документации DM, использование `round(A)` только с одним аргументом считается устаревшим (deprecated). Правильный способ использовать `round()` - это `round(A,B)`, где B — ближайшее кратное. В большинстве случаев это будет `1`, хотя иногда вам может понадобиться округление до ближайших 10 или 5.

Что произойдет, если вы сделаете `round(A)` тогда? Как думаете? Оно _округляет вниз_ значение, и эквивалентно `floor(A)`. Оно **округляет в меньшую**. сторону. Спасибо, BYOND.

```csharp
usr << round(1.7)    // outputs 1
usr << round(1.7, 1) // outputs 2
```

Поэтому в общем случае используйте `round(A,1)` для ваших нужд округления.

# Полезные вещи

## `.git-blame-ignore-revs`

Выполните `git config blame.ignoreRevsFile .git-blame-ignore-revs` в корневой папке goonstation, чтобы игнорировать коммиты, которые затронули много файлов по причинам форматирования. Это помогает немного упростить работу с `git blame`, и вам нужно выполнить это только один раз.

## VSCode Debugger

Вы можете ознакомиться с руководством по использованию отладчика в [Developer Guide](https://hackmd.io/@goonstation/dev#How-to-use-the-VS-Code-Debugger).

## Локальный `__build.dm`, и другие локальные изменения кода

Если вы устали от необходимости постоянно добавлять и удалять `#define IM_REALLY_IN_A_FUCKING_HURRY_HERE` и подобное из `__build.dm`, вместе с прятанием и возвратом своих собственных инструментов разработки, есть решение!

Создайте файл с именем `__build.local.dm` прямо рядом с ним. Именовать нужно именно так.
Этот файл не будет отслеживаться Git, и позволит вам хранить там любые нужные вам дефайны.

Есть также поддержка файла `__development.local.dm`, который подключается в `goonstation.dme` после build-дефайнов, чтобы вы могли добавлять туда новые инструменты тестирования. _Убедитесь, что создаете этот файл в правильной директории!_

А именно, его нужно создать в `code/WorkInProgress/__development.local.dm`. Кроме того, существует флаг сборки `DISABLE_DEVFILE`, который остановит его загрузку. Используйте его перед тем, как просить о помощи, если ваше локальное окружение разработки ломается! И наоборот, если изменения вашего кода _не_ отображаются из этого файла, убедитесь, что этот флаг **отключен**.

И самое главное, не забудьте, что вы поместили свои конфиги и/или код в эти места! Пожалуйста, не приходите в #imcoder спрашивать, почему ваша локальная сборка всегда компилирует Nadir 😸, или почему ваша карта полна капибар.

## Debugging Overlays

Команда Debug-Overlays в игре — ваш друг. Она предлагает множество режимов для отладки многих вещей, таких как группы воздуха атмоса, надписи, области и многое другое.

## Profiler

Команда Open-Profiler в игре — тоже ваш друг. Обязательно введите буквально `.debug profile` во второе поле.
После обновления один раз вы получите подробные измерения производительности всех запущенных процедур.

Руководство по категориям:

- Self CPU: Затраты на код внутри самой процедуры.
- Total CPU: Общие затраты процессора — это затраты Self плюс всё, что вызывает процедура.
- Real Time: Сколько реального времени процедура фактически выполнялась.
- Overtime: Сколько времени было потрачено сверх 100 tick_usage. Это приводит к тому, что мы знаем как "лаг".

Если total cpu и real time одинаковы, процедура никогда не "спала" (не засыпала), в противном случае real time будет выше, так как он учитывает время ожидания процедуры.

## Еще лучший профайлер

Существует проект, предоставляющий невероятно продвинутый профайлер реального времени для DM, именуемый [byond-tracy](https://github.com/spacestation13/byond-tracy), способный обеспечивать невероятное разрешение.

![](https://i.imgur.com/1CEwo0g.png)

Чтобы заставить его работать, вам нужно сделать три вещи: скачать [the tracy 'viewer' application v0.13.x](https://github.com/wolfpld/tracy), и либо скомпилировать, либо скачать библиотеку byond-tracy.

- Первое можно скачать здесь: https://github.com/wolfpld/tracy/releases/tag/v0.13.1 (скачайте .zip и распакуйте, оно портативное)
- Второе можно скачать из [releases](https://github.com/spacestation13/byond-tracy/releases/latest) или скомпилировать из исходников на C. Файл .dll просто кладется в корневую папку игры.
- Раскомментируйте `#define TRACY_PROFILER_HOOK` в `_std/__build.dm`

Если вы на Linux, очевидно, вам нужно скомпилировать оба компонента вручную.

## Target Dummy

Вы можете заспавнить мишень (`/mob/living/carbon/human/tdummy`), чтобы легче тестировать вещи, наносящие урон — у них включен процент здоровья режима "ass day" и всплывающие окна урона видны, даже если ваша сборка не настроена на режим "ass day".

## Сигналы и компоненты (Signals and Components)

ninjanomnom из TG написал [полезное руководство](https://hackmd.io/@tgstation/SignalsComponentsElements) по сигналам и компонентам. Большая часть оттуда применима, хотя элементов (elements) в нашей кодовой базе не существует.

## Универсальная панель действий (Generic Action bar)

Ненавидите кодить панели действий? Создание нового определения для панели действий (datum) только ради того, чтобы иметь визуальную обратную связь при строительстве, кажется грубым? Ну не бойтесь! Теперь вы можете использовать макрос SETUP_GENERIC_ACTIONBAR()!

Для приватных панелей действий (видимых только владельцу) SETUP_GENERIC_PRIVATE_ACTIONBAR() делает то же самое.

Смотрите [\_std/macros/actions.dm](https://github.com/goonstation/goonstation/blob/master/_std/macros/actions.dm) для большей информации.

## Макрос определения турфов (Turf Define Macro)

Создание множества турфов иногда может быть настоящей болью. Если вы используете макрос `DEFINE_FLOORS()` как описано, он создаст симулированный, симулированный безвоздушный, несимулированный и несимулированный безвоздушный турф с указанным путем и переменными на этапе компиляции. Существует много вариаций этого определения, поэтому я рекомендую заглянуть в [\_std/macros/turf.dm](https://github.com/goonstation/goonstation/blob/master/_std/macros/turf.dm)

## Макросы отслеживания (Tracking Macros)

If you want to track everything of a type for whatever reason (need to iterate over all instances at some point, usually), use the tracking macros. Add `START_TRACKING` to `New()` (after the parent call), and `STOP_TRACKING` to `disposing()` (before the parent call). To get all tracked objects of a type, use `by_type[type]`.

To track a group of things which don't share the same type, define a tracking category and then use `START_TRACKING_CAT(CATEGORY)` and `STOP_TRACKING_CAT(CATEGORY)` in the same way as above. To get all tracked objects of a category use `by_cat[CATEGORY]`.

<span style="color: red">VERY VERY BAD:</span>

```javascript
for (var/mob/living/jellyfish in world)
    ...
```

<span style="color: green">Good:</span>

```javascript
/mob/living/jellyfish

    New()
        ..()
        START_TRACKING

    disposing()
		STOP_TRACKING
        ..()

for (var/mob/living/jellyfish/jelly in by_type[/mob/living/jellyfish])
        ...
```

## Git Hooks

The repository includes several Git hooks to automate common tasks:

### Installation

Run the installer to set up hooks at `tools/hooks/install`

### Available Hooks

- **Map merging** - Automatically reduces map file diffs on commit
- **Icon merging** - Resolves `.dmi` file conflicts during merges
- **TGUI building** - Automatically rebuilds TGUI bundles after merges/rebases and resolves bundle conflicts

**Note**: TGUI hooks require Node.js. If you don't have it installed or don't work on TGUI, you can safely skip these hooks.

See `tools/hooks/README.md` for more details.

# Проблемы с юнит-тестами (Unit Test Woes)

## Кэш проходимости (Passability Cache)

Юнит-тест кэша проходимости принудительно применяет правила, связанные с `pass_unstable`, переменной, которая используется для оптимизации поиска пути. Он предотвращает реализацию `Cross` и других коллбэков, основанных на столкновениях, для всех типов, которые объявляют `pass_unstable = FALSE`.

```
FAIL: /datum/unit_test/passability_cache 1.9s
	REASON #1: /obj/machinery/silly_doodad is stable and must not implement Cross
```

In this situation, the unit test detected a forbidden proc `Cross` on `/obj/machinery/silly_doodad`. Because `/obj/machinery` declares `pass_unstable = FALSE`, your subtype must also follow the rules. In this situation, you have two options:

- Remove the `Cross` implementation, or
- Set `pass_unstable = TRUE`, which may cause performance degredation, especially if the atom in question frequently appears in-game.

The same rules apply to any procs the test detects. For example, if it says `must not implement Enter`, then replace `Cross` with `Enter` in the above section.

### Cross vs. Crossed, etc.

Эти две процедуры делают два похожих, но разных вещи. `Cross` _возвращает_, может ли данный подвижный объект пересечь `src`, в то время как `Crossed` вызывается _всякий раз_ , когда данный подвижный объект пересекает `src`.

Если вам нужно только вызвать какое-то действие (то есть ваша машина лечит людей, когда кто-то на нее наступает), тогда вам следует использовать `Crossed`.

Вот быстрый список всех версий коллбэков движения, вызывающих побочные эффекты.

- `Cross` -> `Crossed`
- `Enter` -> `Entered`
- `Exit` -> `Exited`
- `Uncross` -> `Uncrossed`

### Cannot Possibly Be Stable

```
FAIL: /datum/unit_test/passability_cache 3.9s
	REASON #1: /obj/machinery/unstable_thingy/silly_doodad cannot possibly be stable because /obj/machinery/unstable_thingy implements Cross
```

Эта ошибка возникает, когда вы устанавливаете `pass_unstable = FALSE` для подтипа чего-то, что реализует запрещенную процедуру. В большинстве случаев вы ничего не можете сделать, кроме как следовать разделу выше для проблемного родительского типа. В этом случае вам следует посмотреть на `/obj/machinery/unstable_thingy`, чтобы увидеть, можете ли вы сделать его стабильным.
