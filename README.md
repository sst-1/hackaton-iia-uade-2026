T04 · ¿Está en fuera de juego?
Integrantes: Lautaro Cordoba· **IA declarada:**  Claude (la de la ficha). En este trabajo también se usó Claude, ver bitácora. **Pista:** Reglas **Publicación en LinkedIn:** 


1. Formulación y PEAS
La pregunta. Para cada caso (una jugada revisada), ¿el atacante está en posición adelantada? SÍ significa: el atacante está en fuera de juego.

PEAS
Su sistema
Medida de rendimiento
Anular solo los fueras de juego reales, evitando sobre todo anular goles válidos. Se mide con la matriz de confusión, priorizando precisión y especificidad.
Entorno
Una jugada revisada por el sistema semiautomático de fuera de juego, con seguimiento por cámaras y el árbitro del VAR.
Actuadores
Señala SÍ (posición adelantada) o NO. La decisión final la toma el árbitro del VAR con esa señal.
Sensores
Adelanto medido (cm, positivo = adelantado), error del seguimiento (cm) y cámaras con visión clara (de 12).


Objetivo y métrica. El objetivo es no anular goles válidos: el reglamento pide favorecer al atacante en la duda, sin dejar pasar fueras de juego claros. La métrica principal es la precisión y la especificidad, que castigan los falsos positivos. La exactitud se reporta, pero no es lo mismo que el objetivo porque trata igual los dos errores, y siempre se compara contra una línea base.
2. El sistema
Cómo decide. Sistema de reglas si-entonces:

Si el adelanto medido es 0 o negativo, entonces NO.
Si el adelanto es positivo pero no supera el error del seguimiento, entonces NO (hay duda y se favorece al atacante).
Si el adelanto supera el error del seguimiento, entonces SÍ.

Resumen: SÍ si y solo si adelanto > error.

Variables que usa y por qué.

Usa el adelanto medido (señal principal) y el error del seguimiento (indica cuánto confiar en esa medición).
Descarta las cámaras con visión clara. En entrenamiento no cambian ninguna decisión: los casos con pocas cámaras ya tienen adelanto negativo y dan NO igual. Un umbral de cámaras elegido mirando la prueba sería ajustar a la prueba.
Las tres variables se conocen en el momento de decidir y el resultado real no se usa como entrada.

Cómo se corre en vivo. Con el caso nuevo: calcular adelanto menos error. Si da mayor que 0, responder SÍ; si da 0 o menos, responder NO. Se puede hacer a mano o en una planilla con la fórmula =SI(adelanto>error;"SÍ";"NO").

Dos casos de ejemplo.

Caso inventado
Valores (adelanto, error, cámaras)
Salida del sistema
A
25 cm, 6 cm, 11
SÍ (25 > 6)
B
5 cm, 10 cm, 12
NO (5 no supera 10, se favorece al atacante)

3. Medición
Entrenamiento (casos 1–14)



Predice SÍ
Predice NO
Real SÍ
VP = 5
FN = 0
Real NO
FP = 0
VN = 9


Prueba (casos 15–20)



Predice SÍ
Predice NO
Real SÍ
VP = 2
FN = 1
Real NO
FP = 1
VN = 2


Métrica
Entrenamiento
Prueba
Exactitud
100% (14/14)
66,7% (4/6)
Precisión
100% (5/5)
66,7% (2/3)
Sensibilidad
100% (5/5)
66,7% (2/3)
Especificidad
100% (9/9)
66,7% (2/3)


Nota sobre la prueba. Las cuatro métricas de prueba dan 66,7% porque la matriz es simétrica (VP = VN = 2 y FP = FN = 1) y la prueba tiene 3 casos SÍ y 3 NO: todas se reducen a 2/3. No es un error de cálculo. Las matrices cierran con la ficha: 2 + 1 + 1 + 2 = 6 casos, con 3 reales SÍ y 3 reales NO.

Entrenamiento contra prueba. Hay una caída de 33 puntos. Con solo 14 casos de entrenamiento y una regla de dos variables, el ajuste perfecto es esperable y no prueba que el sistema sea bueno. La prueba tiene solo 6 casos, así que un error mueve la métrica 17 puntos y no se puede distinguir bien entre sobreajuste y mala suerte. Los dos errores de prueba no los explica la regla: el caso 18 (FN) tiene adelanto negativo (−18 cm) pero era SÍ, y el caso 19 (FP) tiene 18 cm de adelanto contra 8 cm de error pero era NO. La evidencia de que el sistema generaliza es débil por el tamaño de los datos. No se ajustó la regla para corregir esos casos.
4. El error más caro
El falso positivo es peor: anular un gol válido cambia el resultado, y el reglamento pide favorecer al atacante en la duda. El falso negativo (convalidar un gol en fuera de juego) también es grave, pero la ficha pone el peso en no anular un gol válido.

Para cometer menos falsos positivos cambiamos la regla de adelanto > 0 a adelanto > error. En entrenamiento eso bajó los falsos positivos de 1 a 0 (el caso 4: 1 cm de adelanto con 7 cm de error). En prueba sigue habiendo un falso positivo (caso 19). También evaluamos usar las cámaras: un umbral habría arreglado el caso 19 pero rompía el caso 20 (SÍ real con 7 cámaras), cambiando un falso positivo por un falso negativo. Lo descartamos porque en entrenamiento no hay base para elegir ese umbral.
5. ¿Comparado contra qué?
Línea base trivial.

Regla
Exactitud entrenamiento
Exactitud prueba
Siempre NO
64,3%
50%
Siempre SÍ
35,7%
50%
adelanto > 0
92,9%
66,7%
adelanto > error (nuestra)
100%
66,7%


Nuestro sistema supera las dos líneas base en ambos conjuntos, pero en prueba la ventaja es chica (66,7% contra 50%) y empata con adelanto > 0. La ventaja de adelanto > error se ve solo en entrenamiento y se sostiene por el criterio del reglamento (favorecer al atacante en la duda).

Checklist (Clase 6)
Respuesta
¿Qué datos?
20 casos inventados de la ficha T04: 14 de entrenamiento y 6 de prueba, sin cambiar la división.
¿Qué métrica?
Precisión y especificidad como principales (castigan el falso positivo), más exactitud y sensibilidad.
¿Comparado contra qué?
Siempre NO, siempre SÍ y la regla adelanto > 0.
¿Quién lo midió?
El grupo, con cálculo en código sobre T04.csv, asistido por IA. Falta la medición del evaluador.

6. Riesgo: el camino completo
Datos → modelo → decisión → consecuencia.

Datos: solo 20 casos inventados, con 6 de prueba. El error del seguimiento reportado podría estar subestimado en la realidad.
Modelo: una regla que supone que el error reportado es confiable y no usa las cámaras.
Decisión: señalar SÍ cuando el adelanto supera el error.
Consecuencia: un falso positivo anula un gol válido y cambia el resultado del partido; un falso negativo convalida un gol en fuera de juego.

Riesgo concreto: el caso 19 muestra un adelanto bastante mayor que el error y aun así era NO, así que el sistema puede anular un gol válido incluso con un margen cómodo. Por eso la decisión final debe quedar en el árbitro del VAR y no en el sistema.
7. Bitácora
Uso de la IA

#
Qué le pedimos
Qué aceptamos
Qué rechazamos y por qué
1
Explicar la problemática T04 y proponer cómo resolverla con los datos
La pista Reglas y la regla adelanto > error, que corrige el falso positivo del caso 4
—
2
Medir la regla en entrenamiento y prueba y compararla con las líneas base
Las matrices y métricas calculadas con código sobre T04.csv
Presentar el 100% de entrenamiento como prueba de que el sistema es bueno: lo corregimos y priorizamos la prueba
3
Revisar si la regla usaba las cámaras
El análisis de que las cámaras no cambian decisiones en entrenamiento; la IA corrigió un error suyo (confundió las cámaras del caso 15 con las del 19)
Elegir un umbral de cámaras mirando la prueba: sería ajustar a los datos de prueba


Corridas contra el evaluador

Corrida: 1
Nota 9,5/10 
Agujero que encontró:El caso 19 supera el error por 10cm y aun asi era no. ¿Que tan confiable es el error del seguimiento que estas usando como umbral?
Qué cambiamos
1
