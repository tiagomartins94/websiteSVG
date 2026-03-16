Final Projection :=
VAR currentN = MAX('Axis'[Value])

VAR LastConsommeBefore =
    CALCULATE(
        [CHANGE Consommé TJM réel (Value)],
        TOPN(
            1,
            FILTER(
                ALL('Axis'),
                'Axis'[Value] < currentN &&
                NOT ISBLANK([CHANGE Consommé TJM réel (Value)])
            ),
            'Axis'[Value], DESC
        )
    )

VAR CurrentConsomme =
    CALCULATE(
        [CHANGE Consommé TJM réel (Value)],
        'Axis'[Value] = currentN
    )

VAR RunningETP =
    CALCULATE(
        SUMX(
            FILTER(ALL('Axis'), 'Axis'[Value] < currentN),
            [CHANGE ETP]
        )
    )

VAR SafeRunningETP = IF(RunningETP = 0, 1, RunningETP)

VAR CurrentETP =
    CALCULATE(
        [CHANGE ETP],
        'Axis'[Value] = currentN
    )

VAR ResultThisPoint =
    DIVIDE(
        IF(ISBLANK(CurrentConsomme), LastConsommeBefore, CurrentConsomme),
        SafeRunningETP
    )
    / ((21+18)/2)
    * CurrentETP
    * 17.83

RETURN
SUMX(
    FILTER(
        ALL('Axis'),
        'Axis'[Value] <= currentN
    ),
    VAR cn = 'Axis'[Value]
    VAR CurrentConsommeAux =
        CALCULATE([CHANGE Consommé TJM réel (Value)], 'Axis'[Value] = cn)
    VAR LastConsommeAux =
        CALCULATE(
            [CHANGE Consommé TJM réel (Value)],
            TOPN(
                1,
                FILTER(
                    ALL('Axis'),
                    'Axis'[Value] < cn &&
                    NOT ISBLANK([CHANGE Consommé TJM réel (Value)])
                ),
                'Axis'[Value], DESC
            )
        )
    VAR RunningETPAux =
        CALCULATE(
            SUMX(
                FILTER(ALL('Axis'), 'Axis'[Value] < cn),
                [CHANGE ETP]
            )
        )
    VAR SafeRunningETPAux = IF(RunningETPAux = 0, 1, RunningETPAux)
    VAR CurrentETPAux =
        CALCULATE([CHANGE ETP], 'Axis'[Value] = cn)
    RETURN
        DIVIDE(
            IF(ISBLANK(CurrentConsommeAux), LastConsommeAux, CurrentConsommeAux),
            SafeRunningETPAux
        )
        / ((21+18)/2)
        * CurrentETPAux
        * 17.83
)
