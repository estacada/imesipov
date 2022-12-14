# Эффективный вывод чисел с плавающей точкой

## Как устроены числа с плавающей точкой в компьютере

По стандарту IEEE 754, число с плавающей точкой состоит из следующих частей:

 * знак мантиссы (указывает на отрицательность или положительность числа),
 * мантисса (выражает значение числа без учёта порядка),
 * порядок (выражает степень основания числа, на которое умножается мантисса).
![alt text](https://media.geeksforgeeks.org/wp-content/uploads/Double-Precision-IEEE-754-Floating-Point-Standard-1024x266.jpg)

Число $v$ можно представить как $v = f \times b^e$, где $f$ - мантисса (вместе со знаком), $b$ - основание, $e$ - экспонента (порядок).

## Проблема

Рассмотрим одну из главных проблем чисел с плавающей точкой --- округление. Число $\frac{3}{10}$ может быть записано компьютером, как $0.29999...$, когда на самом деле это конечная дробь, которую можно записать как $0.3$.
Другой пример --- число $10^{23}$, которое может быть записано компьютером как $9.999999999999999e22$. Эта проблема возникает из-за того, что в компьютере числа представлены в двоичной системе, тогда как при выводе этих чисел мы оперируем десятичной, 
и эти числа не могут быть представлены как целые дроби в системе с основанием $2$. На числовой прямой конечное число чисел, которые можно представить в таком формате, к примеру, числа $0.3$ просто нет.

Мы рассмотрим алгоритм, который будет корректно округлять число, делая его запись максимально короткой (то есть, отбрасывая лишние цифры после запятой). Будем искать число в форме $0.d_1d_2... \times B^k$, где $d_1, d_2,...$ - цифры выводящегося числа по основанию $B$.

Таким образом, мы получим обобщенный алгоритм, который может выводить числа с любым основанием в виде чисел с любым основанием, в том числе, выводить числа в компьютерной записи (с двоичным основанием) как числа с десятичным основанием.

## Алгоритм

Не ограничивая общности, считаем, что $v > 0$, случай с отрицательным числом абсолютно аналогичен.

Для начала введем несколько обозначений:

$v = f \times b^e$ - число, которое мы собираемся вывести, данное нам как тройка $f, b, e$.

$v^{+}$ - ближайшее сверху число к $v$, в смысле тройки $f, b, e$. Иными словами, это ближайшее сверху "в компьютерной записи" число к $v$.
Оно равно:
* $(f + 1) \times b^e$, если $f + 1$ не превосходит размера мантиссы
* $b^{p - 1} \times b^{e + 1}$, если $f + 1$ не умещается в мантиссу ( $b^p = f + 1$ ). Если $e + 1$ не умещается в экспоненту, то $v^{+} = +inf$

$v^{-}$ - ближайшее снизу число к $v$, в смысле тройки $f, b, e$.
Определяется как:
* $(f - 1) \times b^e$, если $f > b^{p - 1}$ (иначе говоря, $f$ не является степенью $b$)
* $(b^{p} - 1) \times b^{e + 1}$, иначе

$low = \frac{v^{-} + v}{2}$ - такое минимальное число, что все числа на промежутке $[low, v]$ будут округлены в сторону $v$.

$high = \frac{v + v^{+}}{2}$ - такое максимальное число, что все числа на промежутке $[v, high]$ будут округлены в сторону $v$.


Алгоритм на вход получает числа $B$, $v$ ( $f$, $b$, $e$ ).
На выходе - число $V = 0.d_1d_2...d_n \times B^k$, $n$ - такое минимальное число, что:
* $V \in (low, high)$ - это означает, что $V$ будет округлено к $v$ независимо от метода округления (к большему или к меньшему).
* $|V - v| < \frac{B^{k - n}}{2}$ - это означает, что $V$ будет корректно округлено к $v$.

Алгоритм:
1. Находим $v^{-}$, $v^{+}$, $low$, $high$.
2. Находим такое $k$, что $high$ < $B^k$, то есть $k = \lceil \log_B(high) \rceil$
3. Пусть $q_0 = \frac{v}{B^k}$

Ищем цифры:

$d_1 = \lfloor q_0 \times B \rfloor$

$q_1 = {q_0 \times B}$

$d_2 = \lfloor q_1 \times B \rfloor$

$q_2 = {q_1 \times B}$

...

Останавливаемся, когда достигнем такого минимального $n$, при котором выполнено одно из условий:
* $0.d_1d_2...d_n \times B^k > low$, то есть, $V$ округлилось бы вверх к $v$
* $0.d_1d_2...d_{n-1}[d_n + 1] \times B^k < high$, то есть $V$ округлилось бы внизу к $v$

Если выполнено первое условие, то выводим $0.d_1d_2...d_n \times B^k$.

Если второе, то выводим $0.d_1d_2...d_{n-1}[d_n + 1] \times B^k$.

Если оба, то пользуемся любым удобным методом округления.

## Асимптотика

Нас интересует работа этого алгоритма на компьютере, где операции сложения, вычитания, умножения и деления выполняются за константное время.
Тогда мы можем заметить, что первый шаг выполняется за $O(1)$. 

Третий шаг выполняется за $O(n)$, где $n$ будет длиной выводящегося числа, быстрее его мы никак не выполним.

Остается второй шаг, а именно, вычисление логарифма, которое быстрее, чем за $O(\log(high))$ не удастся выполнить. 
Однако, если $high$ достаточно маленькое число, то вычисление логарифма можно выполнить за $O(1)$ с помощью поиска первого значащего бита у числа.



Источник: https://dl.acm.org/doi/pdf/10.1145/249069.231397
