# Instrucciones del repositorio SERIES-DEL-TIEMPO

Repositorio del alumno para la materia Series de Tiempo (ITESO). El trabajo se hace en documentos Quarto (`.qmd`) con R, `tidyverse` y `fpp3`. Las clases se dan a partir del sitio del profesor (https://pbenavidesh.github.io/narsil/docs/modules/), que tiene todo el semestre; ya se vieron las clases 1.1 a 3.1. El objetivo es ayudar con actividades y exámenes usando únicamente lo que ya se vio en clase. Las carpetas `contrahuella/`, `output/` y `tmp/` no son de la materia y no deben usarse como referencia.

## Regla principal

- Usa únicamente librerías, funciones y argumentos que aparezcan en `.github/instructions/r-clase.instructions.md` o en las instrucciones y el código inicial de la actividad actual.
- No uses nada de las clases 3.2 en adelante del sitio mientras no se vean.
- Las tareas `pronostico_importaciones_autos.qmd` y `Ejercicio_ETS.qmd` no son fuente de métodos: no copies estructuras suyas que no estén en las clases vistas.
- No propongas atajos ni versiones más sofisticadas de lo visto aunque den el mismo resultado.
- Si no hay antecedente en lo visto, no inventes una técnica: explica brevemente que todavía no se ha visto.
- No agregues análisis, gráficas ni validaciones que no se pidieron.

## Completar a partir de un comentario

Uso principal durante los exámenes. Cuando un bloque de código tenga un comentario que describe un paso, como `# convertir a tsibble`, `# lambda de Guerrero`, `# train/test`, `# ajustar los modelos`, `# diagnóstico de residuales`, `# accuracy` o `# pronóstico final`:

- Completa únicamente ese paso, con la estructura de la clase del tema.
- Adapta los nombres de la serie, las columnas y el horizonte al problema actual, sin copiar valores de otro ejercicio.
- No adelantes pasos posteriores ni generes la solución completa si el comentario pide una sola parte.

## Forma de escribir el código

- Nivel estudiante, bloques cortos con un propósito claro y texto breve en español entre bloques.
- Encadena los pasos con el pipe nativo `|>` (nunca `%>%`), un paso por línea.
- Nombres como en clase: `<serie>_train`, `<serie>_test`, `<serie>_fit`, `<serie>_fc`, `<serie>_accu`, `<serie>_final_fit`, `<serie>_final_fc`, `h`, `lambda`, y modelos con nombre dentro de `model()` (`snaive = SNAIVE(y)`).
- Gráficas con `autoplot`, `gg_season`, `gg_subseries`, `gg_tsdisplay` y `gg_tsresiduals`, con `labs(title = ..., y = ..., x = NULL)`.
- En las interpretaciones, explica primero el resultado (residuales, p-value de Ljung-Box, MASE o RMSSE, AICc) y después qué significa para el pronóstico de la serie.

## Fuentes de referencia por tema

### Módulo 1: pronóstico basado en descomposición

- tsibbles, índice y claves, `has_gaps`, `fill_gaps`, gráficas de tiempo, estacionales y de subseries: clase 1.1 (`module_1/01_time_series/r_time_series.html`) y `Primer documento de Quarto.qmd`.
- Transformaciones (log, Box-Cox con Guerrero), ajustes, descomposición clásica y STL: clase 1.2 (`module_1/02_ts_dcmp/ts_dcmp.html`) y `Descomposición.qmd`.
- Flujo de pronóstico (train/test, `model`, `forecast`, `accuracy`, reajuste), métodos de referencia `MEAN`, `NAIVE`, `SNAIVE` y `RW` con drift, residuales y Ljung-Box: clase 1.3 (`module_1/03_fcst/forecasting.html`).
- Intervalos de predicción, `hilo`, pronóstico con Box-Cox, ajuste por sesgo, intervalos bootstrap, `decomposition_model` con STL: clase 1.4 (`module_1/04_model_diagnostics/model_diagnostics.html`).

### Módulo 2: filtros ETS y ARIMA

- SES, Holt, tendencia amortiguada, Holt-Winters aditivo y multiplicativo, ETS automático, STL + ETS: clase 2.1 (`module_2/01_ets/ets.html`).
- Diferenciación regular y estacional, KPSS, `unitroot_ndiffs`, `unitroot_nsdiffs`, ACF y PACF: clase 2.2 (`module_2/02_stationarity/stationarity.html`).
- ARIMA y SARIMA manuales y automáticos, `glance` con AICc, `report`, diagnóstico de residuales, STL + ARIMA: clase 2.3 (`module_2/03_arima/arima.html`).
- Combinaciones modulares STL + ETS + ARIMA y su comparación con `accuracy`: clase 2.4 (`module_2/04_modular_fcst/modular_forecasting.html`).

### Módulo 3: problemas prácticos

- Outliers con el residuo de STL (`season(period = 1)`, `robust = TRUE`) y la regla IQR, quitarlos con `anti_join` + `fill_gaps`, imputar con `ARIMA` + `interpolate`, `log` para pronósticos positivos y logit escalado con `new_transformation`: clase 3.1 (`module_3/01_practical_issues/practical_issues.html`).

## No usar

- Lo de las clases 3.2 en adelante mientras no se vea: `TSLM`, `trend()` y `season()` como regresores de `TSLM`, `fourier`, regresión dinámica con variables exógenas, `prophet`, `cross_validate` o `stretch_tsibble`, `bootstrapped` o `generate` para *bagging*, `reconcile`, `aggregate_key`, `combination_model`.
- Lo que solo aparece en las tareas hechas con IA: `ggplot` construido desde cero con `geom_segment`, `geom_vline` o `scale_*`, `theme_set`, `theme_update`, `pivot_wider`, `transmute`, `knitr::kable`, `scales::label_number`, `stopifnot`.
- `%>%`, `theme_narsil()` y funciones de paquetes que no se cargan en clase.

Esta lista cambia conforme se ven clases nuevas: cuando una clase vista use algo de aquí, pasa a estar permitido.

- Antes de terminar una respuesta o una edición, comprueba que cada función propuesta aparezca en la lista autorizada y que la solución responda solo a lo solicitado.
