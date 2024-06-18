
# Python
## Что такое PEP 8?
[[PEP8]]
## Scope в Python – какие бывают?
- Local scope: in current function
- Global scope: available while running code
- Module-level scope: global objects in current module
- Outermost: for all imported names in called program, *looked up last*

## List и Tuples – зачем нужны и какие отличия?
[[list vs tuple]]

## Что такое pass в Python?
Pass is like dummy or stub, to not implement 


## Что такое модули и пакеты в Python?
[[modules]] and [[packages]]

##  Какие отличия между массивом и списком в Python?
[[python list vs array]]

## Отрицательное значение индексов в Python
Searches from the end in reverse


## Как управляется память в Python?
[[shallow vs deep copy]]


## Как делается перегрузка операторов в Python?
[[operator overloading]]

## Как передаются аргументы – значения или ссылка?
Always by reference to the variable

## Что такое *args и kwargs?*
[[args and kwargs]]
## Пустой класс и как он используется?
Just to temporary put something

##  Используются ли в python спецификаторы доступа?
[[naming styles]]


##  Написать алгоритм (несколько примеров)
If you need to count or find encounters in some way, just use dict/hashmap or set to check because of fast look up via hash function ([[Data structures]]), and logical binding


## == vs is
[[is vs ==]]


## self
[[self]]

## Threads vs Processes
[[multiprocessing vs multithreading]]


## For what threading is perfect fit
> NOTE: for CPU bound multiprocessing is better

Good fit:
- Threading is perfect for CPU bound processes (a lot of computations, processing, ML)
Bad fit:
- Not good if system has low memory resources
- if it has a lot of I/O bound operations (waiting a lot from outside)
- if it has a lot of async operations
- if reactive approach


## python interpreter
[[python interpreter]]



## asynchronous
[[concurrency in python]]


## event loop
[[Event loop]]