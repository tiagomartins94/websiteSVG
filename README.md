Final Running Measure :=
VAR currentN = MAX('Axis'[Value])

// Último Change Consommé antes do currentN
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
            'Axis'[Value],
            DESC
        )
    )

// Current Consommé do ponto actual
VAR CurrentConsomme =
    CALCULATE(
        [CHANGE Consommé TJM réel (Value)],
        'Axis'[Value] = currentN
    )

// Running total ETP até currentN
VAR RunningETP =
    CALCULATE(
        SUMX(
            FILTER(
                ALL('Axis'),
                'Axis'[Value] < currentN
            ),
            [CHANGE ETP]
        )
    )

// Current ETP do ponto actual
VAR CurrentETP =
    CALCULATE(
        [CHANGE ETP],
        'Axis'[Value] = currentN
    )

// Resultado para este ponto do eixo
VAR ResultThisPoint =
    DIVIDE(
        IF(ISBLANK(CurrentConsomme), LastConsommeBefore, CurrentConsomme),
        RunningETP
    )
    / ((21+18)/2)
    * CurrentETP
    * 17.83

// Running total do resultado
RETURN
SUMX(
    FILTER(
        ALL('Axis'),
        'Axis'[Value] <= currentN
    ),
    VAR cn = 'Axis'[Value]
    VAR lc =
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
    VAR cc =
        CALCULATE(
            [CHANGE Consommé TJM réel (Value)],
            'Axis'[Value] = cn
        )
    VAR rETP =
        CALCULATE(
            SUMX(
                FILTER(ALL('Axis'),'Axis'[Value]<cn),
                [CHANGE ETP]
            )
        )
    VAR cETP =
        CALCULATE([CHANGE ETP],'Axis'[Value]=cn)
    RETURN
        DIVIDE(IF(ISBLANK(cc), lc, cc), rETP) / ((21+18)/2) * cETP * 17.83
)
