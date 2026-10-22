# Guía de estudio del Proyecto M2

Fernanda Dueñas, Santiago Villegas y Carlos Nieves · Equipo 1 · Presentación del 23 de octubre de 2026

## Qué hicimos y por qué

Pronosticamos las ventas semanales de 2026 W31 a W42 con las 290 observaciones disponibles, de 2021 W01 a 2026 W30. El modelo final es una descomposición STL anual robusta, ETS sin estacionalidad para la serie desestacionalizada (R eligió ETS(A,N,N)) y SNAIVE para el componente estacional. Trabajamos en pesos, sin transformar, y conservamos todas las observaciones.

Para comparar modelos reservamos 2026 W19 a W30 como prueba y entrenamos hasta W18. Son 12 semanas, igual que el horizonte que entregamos. Como un solo test es ruidoso, repetimos la comparación en la misma temporada que vamos a pronosticar: entrenando hasta W30 de 2023, 2024 y 2025 y evaluando W31 a W42 de cada año. Guerrero, la detección de outliers y la imputación se estimaron solo con el entrenamiento.

**Transformación.** Guerrero dio lambda de 0.531 con el entrenamiento (0.460 con toda la serie, solo como dato descriptivo). Con el mismo esquema de STL y ETS, el MAPE del test fue 11.47 % en pesos, 13.23 % con Box-Cox y 13.69 % con logaritmo. El logaritmo redujo el RMSE, pero el concurso se califica con MAPE, así que no transformamos.

**Outliers.** Aplicamos la regla IQR con umbral 3 al residuo de una STL anual robusta, como en la clase 3.1. Se marcaron seis semanas del entrenamiento: 2021 W43, W50 y W51; 2022 W50; 2025 W52 y 2026 W01. Las quitamos, rellenamos los huecos con `fill_gaps()` e imputamos con `ARIMA() |> interpolate()`. El MAPE pasó de 11.47 % a 11.50 %, así que conservamos las ventas originales. Usamos la descomposición anual completa y no `season(period = 1)`, porque con esta serie el residuo se llenaría de los picos de diciembre, que son estacionalidad.

**Modelos probados.** Benchmark (STL + caminata aleatoria con deriva + SNAIVE), STL + ETS, STL + ARIMA no estacional, SARIMA directa sobre las ventas, y las variantes de STL + ETS con logaritmo, Box-Cox y datos imputados. `report()` confirmó lo que estimó R en cada caso: ETS(A,N,N) en nuestro modelo, ARIMA(1,1,2) con deriva en STL + ARIMA, ARIMA(0,1,1)(0,1,1)[52] en la SARIMA y ETS(A,A,N) en las escalas transformadas.

**Evaluación.** En el test, STL + ETS tuvo 11.47 %, STL + ARIMA 11.84 %, la SARIMA 17.66 % y el benchmark 32.05 %. En la misma temporada de años anteriores, el promedio fue 14.15 % con ETS, 14.43 % con ARIMA y 15.63 % con el benchmark. Ningún modelo ganó siempre: en 2023 el mejor fue el benchmark (15.71 % contra 16.12 % de ETS), en 2024 ARIMA y en 2025 ETS.

**Por qué ETS y no ARIMA.** La diferencia de precisión es pequeña, así que también pesamos el nivel. ARIMA incluye una deriva y proyecta 18.1 % más que la misma temporada de 2025; ETS proyecta 13.8 %. En 2026 las ventas crecen 13.1 % en W01 a W30, pero solo 3.4 % en W19 a W30. Con un crecimiento que se frena, extrapolar una deriva aumenta el riesgo de sobrestimar. Además, ETS es más simple y respecto al benchmark solo cambia una pieza. Un nivel más prudente no garantiza un MAPE menor: es una decisión práctica apoyada en la temporada objetivo.

## Cómo funciona el modelo

### STL robusta y sus componentes

STL separa las ventas en tendencia, estacionalidad y residuo: `total = trend + season_year + remainder`. Con `robust = TRUE`, las semanas extremas pesan menos al estimar la tendencia y la estacionalidad. `season_adjust = trend + remainder` es la venta sin la estacionalidad.

Ejemplo: una semana vende 120 000 pesos, la tendencia es 80 000 y el efecto estacional 10 000. El residuo es 30 000 y `season_adjust` es 110 000. STL robusta evita que ese salto distorsione los componentes, pero el valor sigue presente en `season_adjust`. Por eso STL robusta no equivale a limpiar outliers.

### ETS(A,N,N)

`ETS(season_adjust ~ season("N"))` impide que ETS agregue otra estacionalidad y deja que R elija el error y la tendencia. `report()` confirmó ETS(A,N,N): error aditivo, sin tendencia y sin estacionalidad, es decir, suavizamiento exponencial simple.

El nivel se actualiza como nivel nuevo = alpha * dato nuevo + (1 - alpha) * nivel anterior, con alpha cercano a 0.138. Si el nivel anterior fuera 80 000 y el dato desestacionalizado 100 000, el nivel nuevo sería 82 760. El pronóstico de `season_adjust` es plano; las ventas pronosticadas cambian de semana a semana porque se suma la estacionalidad.

### SNAIVE dentro de `decomposition_model()`

Si no se especifica un modelo para `season_year`, `decomposition_model()` usa SNAIVE, que repite el último ciclo del componente estacional (no las ventas completas del año pasado). Si ETS proyecta 82 760 de nivel y el efecto estacional de una semana es 5 000, el pronóstico es 87 760; con un efecto de -3 000, sería 79 760.

Un año tiene unas 52.18 semanas y SNAIVE redondea el periodo a 52; por eso aparece el aviso de R en cada ajuste. Repetir 52 posiciones no siempre equivale a la misma semana ISO, y los años con semana 53 y Semana Santa desplazan las comparaciones.

### Lambda de Guerrero

Guerrero estima el lambda de Box-Cox que mejor estabiliza la dispersión entre bloques de la serie. Lambda igual a 0 es el logaritmo y lambda igual a 1 deja la escala casi igual. Estabilizar la varianza no es lo mismo que pronosticar mejor; por eso decidimos con el MAPE fuera de muestra.

### Regla IQR sobre el residuo

IQR = Q3 - Q1. Marcamos residuos menores que Q1 - 3 * IQR o mayores que Q3 + 3 * IQR. Si Q1 fuera -4 000 y Q3 5 000, los límites serían -31 000 y 32 000. Se aplica al residuo y no a las ventas, porque con tendencia las ventas recientes siempre parecerían altas.

### MAPE y por qué un solo test es ruidoso

MAPE = 100 * promedio(|real - pronóstico| / real). Pronosticar 90 000 cuando se venden 100 000 es 10 % de error; el mismo error con ventas de 50 000 es 20 %. Las semanas bajas pesan más. Con solo 12 semanas, un evento aislado puede cambiar el orden de los modelos, y por eso revisamos también la misma temporada de otros años.

## Residuales y límites

Ljung-Box con 24 rezagos dio p = 0.0098: al 5 % rechazamos la ausencia de autocorrelación, así que los residuales no son ruido blanco. Lo usamos como diagnóstico exploratorio junto con la ACF. La media de las innovaciones es positiva (cerca de 1 586 pesos), una ligera subestimación dentro de muestra, y la dispersión aumenta en 2025 y 2026. Las primeras 52 semanas no tienen innovación porque SNAIVE necesita un año previo; de ahí los avisos de filas omitidas en la gráfica.

El modelo no incluye Semana Santa, cierres ni eventos. Si la desaceleración se acentúa, podríamos quedar por arriba de las ventas reales y el benchmark podría resultar mejor en las semanas del concurso.

## Errores que debemos evitar al explicar el trabajo

- Decir que estimamos lambda o limpiamos con toda la serie: todo se hizo solo con el entrenamiento.
- Afirmar que STL robusta elimina los outliers: solo reduce su peso en tendencia y estacionalidad.
- Llamar Holt a ETS(A,N,N): no tiene tendencia, y eso lo confirmó `report()`.
- Decir que SNAIVE repite las ventas de 2025: repite el componente estacional.
- Decir que los residuales son ruido blanco.
- Presentar a ETS como ganador de todos los cortes: en 2023 ganó el benchmark y en 2024 ARIMA.
- Prometer que superaremos al benchmark en las semanas reales.

## Guion de cinco minutos

| Diapositiva | Tiempo | Acumulado |
|---|---:|---:|
| Ventas semanales y esquema de evaluación | 25 s | 0:25 |
| Transformación: la escala original pronostica mejor | 30 s | 0:55 |
| Outliers: los identificamos, pero no los sustituimos | 35 s | 1:30 |
| Modelos probados | 30 s | 2:00 |
| Verificación con `report()` | 20 s | 2:20 |
| Modelo final | 30 s | 2:50 |
| Prueba 2026 W19–W30 | 30 s | 3:20 |
| La misma temporada en años anteriores | 45 s | 4:05 |
| Diagnóstico de residuales | 25 s | 4:30 |
| Pronóstico entregado | 30 s | 5:00 |
| Validación y guardado | No se presenta | — |

**Ventas semanales.** «Tenemos 290 semanas con tendencia y un patrón anual fuerte. Elegimos el modelo con un test de las últimas 12 semanas y, como un solo test es ruidoso, lo revisamos también en la misma temporada de 2023, 2024 y 2025.»

**Transformación.** «Guerrero sugiere lambda de 0.53, pero al probar Box-Cox y logaritmo con el mismo modelo, el MAPE sube unos dos puntos. Conservamos los pesos.»

**Outliers.** «Con la regla IQR sobre el residuo de una STL robusta marcamos seis semanas. Imputarlas con ARIMA no mejora el MAPE, así que conservamos los datos originales.»

**Modelos probados.** «Todos usan la misma STL del benchmark; cambia el modelo de la serie desestacionalizada. Probamos ETS, ARIMA, una SARIMA directa y las variantes con transformación y limpieza.»

**Verificación.** «Con `report()` confirmamos lo que estimó R: ETS(A,N,N), ARIMA(1,1,2) con deriva y SARIMA(0,1,1)(0,1,1)[52].»

**Modelo final.** «STL robusta, ETS(A,N,N) con alpha de 0.14 para el nivel y SNAIVE para la estacionalidad. Frente al benchmark solo cambiamos la deriva por ETS.»

**Prueba.** «En el test, ETS tiene 11.47 % contra 32.05 % del benchmark. ARIMA queda muy cerca.»

**Misma temporada.** «En W31 a W42 de años anteriores ninguno gana siempre, pero ETS tiene el mejor promedio. Además, ARIMA proyecta 18 % de crecimiento y ETS 14 %, cuando las ventas recientes solo crecen 3 %. Por eso elegimos ETS.»

**Residuales.** «Queda autocorrelación, p de 0.0098, así que no son ruido blanco. Semana Santa, la semana 53 y los eventos son limitaciones del modelo.»

**Pronóstico.** «Reestimamos con las 290 semanas: media de unos 86 300 pesos por semana, cerca de 14 % más que la misma temporada de 2025. Las bandas son los intervalos de predicción de 80 y 95 %.»

**Validación.** No se presenta. Queda como última diapositiva para que se vea que la validación pasa y que `equipo_1.RData` se genera desde el propio documento. Si el profesor pregunta, ahí está la evidencia.

## Cinco preguntas posibles del profesor

1. Si Guerrero da un lambda distinto de 1, ¿por qué no transformaron?
2. ¿Por qué conservaron las semanas marcadas como outliers?
3. ¿Cómo varía el pronóstico si ETS(A,N,N) no tiene tendencia ni estacionalidad?
4. Si ARIMA fue tan bueno como ETS, ¿por qué eligieron ETS?
5. ¿Qué implica el resultado de Ljung-Box?

## Respuestas

1. Guerrero busca estabilizar la varianza, no minimizar el MAPE futuro. Con lambda estimado solo en el entrenamiento, Box-Cox y logaritmo dieron 13.23 % y 13.69 % de MAPE, contra 11.47 % en pesos.
2. Porque imputarlas no mejoró el pronóstico (11.50 % contra 11.47 %) y no sabemos si fueron eventos reales o errores. Sustituir ventas reales sin esa información es arriesgado. La STL robusta ya reduce su influencia.
3. ETS(A,N,N) solo pronostica el nivel de `season_adjust`, que es plano. A ese nivel se le suma el componente estacional que repite SNAIVE, y eso hace que cada semana tenga un valor distinto.
4. La precisión fue parecida: ETS ganó en el test y en el promedio de la temporada objetivo, ARIMA ganó en 2024. Elegimos ETS porque es más simple y porque ARIMA proyecta 18 % de crecimiento con una deriva, mientras las ventas recientes crecen 3 %. ETS proyecta un nivel más prudente, aunque eso no garantiza menor MAPE.
5. Con p = 0.0098 rechazamos la ausencia de autocorrelación al 5 %: el modelo deja estructura sin capturar. No invalida el pronóstico, pero no podemos llamar ruido blanco a los residuales. Parte de esa estructura puede venir de festivos móviles y eventos que el modelo no incluye.
