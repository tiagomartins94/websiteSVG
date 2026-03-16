Final Running Projection :=
VAR currentN = MAX('Axis'[Value])

// Calcula o resultado para cada ponto do eixo
VAR ResultPerAxis =
    ADDCOLUMNS(
        FILTER(ALL('Axis'),'Axis'[Value] <= currentN),
        "@Result",
        VAR cn = 'Axis'[Value]
        VAR CurrentConsomme =
            CALCULATE([CHANGE Consommé TJM réel (Value)], 'Axis'[Value] = cn)
        VAR LastConsommeBefore =
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
        VAR RunningETP =
            CALCULATE(
                SUMX(
                    FILTER(ALL('Axis'), 'Axis'[Value] < cn),
                    [CHANGE ETP]
                )
            )
        VAR CurrentETP =
            CALCULATE([CHANGE ETP], 'Axis'[Value] = cn)
        RETURN
            DIVIDE(IF(ISBLANK(CurrentConsomme), LastConsommeBefore, CurrentConsomme), RunningETP)
            / ((21+18)/2)
            * CurrentETP
            * 17.83
    )

// Faz o running total do resultado
RETURN
SUMX(ResultPerAxis, [@Result])
