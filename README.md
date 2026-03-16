CHANGE Projection TJM réel :=
VAR CurrentMonth =
    MAX('Axis'[Value])

VAR LastActualMonth =
    CALCULATE(
        MAX('Axis'[Value]),
        FILTER(
            ALL('Axis'),
            NOT(ISBLANK([CHANGE Consommé TJM réel (Value)]))
        )
    )

VAR ActualValue =
    [CHANGE Consommé TJM réel (Value)]

VAR LastActualConsomme =
    CALCULATE(
        [CHANGE Consommé TJM réel (Value)],
        'Axis'[Value] = LastActualMonth
    )

VAR RunningETPLastActual =
    CALCULATE(
        [CHANGE ETP (Running Total)],
        'Axis'[Value] = LastActualMonth
    )

VAR Ratio =
    DIVIDE(LastActualConsomme, RunningETPLastActual)

VAR RunningETPCurrent =
    [CHANGE ETP (Running Total)]

VAR Forecast =
    Ratio * RunningETPCurrent

RETURN
IF(
    CurrentMonth <= LastActualMonth,
    ActualValue,
    Forecast
)
