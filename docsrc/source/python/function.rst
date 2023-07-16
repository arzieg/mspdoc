.. _functions:

##########
Functions 
##########


Callable
==========

..code-block:: python
    StrategyFunction = Callable[[list[int]], bool]      - Signatur einer Funktion, Input hier eine list von int, return ein bool

    def func_a (b: list[int]) -> bool:
        return 5*b

    def func_b (b: list[int]) -> bool:
        return 2^b

    buy_strategy: StrategyFunction
    sell_strategy: StrategyFunction

    ...
    im weiteren Verlauf kann dann auf func_a oder func_b zugewiesen werden.


closure
--------
Wraper um eine Callable Funktion. Prinzipiell ist mit einem Callable erst einmal eine Signatur vorgegeben, die nicht geändert werden kann. 
Wenn man nun aber weitere Parameter übergeben will, kann man die Funktion quasi mit einem Wraper erweitern (aka closure)

..code-block:: python

    def should_by_avg(prices: list[int]) -> bool:
        list_windows = prices[-3:]    <- hier möchte man nun anstelle der letzten drei Werte etwas dynamisches haben
        return prices[-1] < statistics.mean(list_windows)

    => Lösung mit closure
    def should_by_avg_clousure(window_size: int) -> StrategyFunction
        def should_by_avg(prices: list[int]) -> bool:
            list_windows = prices[-window_size:]    <- hier gepatcht
            return prices[-1] < statistics.mean(list_windows)
        return should_by_avg  <- hier wird dann eine Funktion wieder zurückgegeben

    Aufruf dann: 
    should_by_avg_closure(4)


