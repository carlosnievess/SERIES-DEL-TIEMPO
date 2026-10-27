---
applyTo: "**/*.qmd,**/*.R"
---

# Estructuras de R vistas en clase

Estas reglas se aplican a los documentos Quarto y R. La lista reúne lo visto en las clases 1.1 a 4.1 del sitio del profesor (https://pbenavidesh.github.io/narsil/docs/modules/) y en los archivos de clase del repositorio. El avance hasta 4.1 corresponde a la actividad solicitada el 27 oct 2026; no implica que los ejercicios estén terminados. Crece conforme se ven clases nuevas. Si algo no está aquí ni en la actividad actual, no debe proponerse.

## Librerías permitidas

- `tidyverse` y `fpp3` (que carga `tsibble`, `tsibbledata`, `feasts` y `fable`), en el primer bloque del documento.
- `tidyquant` con `tq_get` para descargar series (clases 2.2 a 2.4).
- `readxl::read_excel` para archivos de Excel y `stats::filter` cuando se necesita el `filter` de `stats`.
- `plotly` para `ggplotly` (clases 3.2 a 4.1), `car` (cargada en 3.2), `GGally::ggpairs` (3.2) y `fable.prophet` para `prophet` (3.4). Cargar solo lo que necesite la actividad.
- Conjuntos de datos de clase: `tourism`, `aus_production`, `pelt`, `global_economy`, `prices`, `aus_livestock`, `us_change`, `pedestrian`, `aus_retail`, `boston_marathon`, `vic_elec`, `insurance` y `bank_calls`.

## Documento Quarto

```yaml
---
title: "..."
format:
  html:
    embed-resources: true
---
```

Las librerías van en el primer bloque (`library(tidyverse)` y `library(fpp3)`). Un bloque que no debe ejecutarse lleva `#| eval: false`.

## Clase 1.1: tsibbles y gráficas

```r
medicinas_tsb <- medicinas |> 
    as_tsibble(index = fecha) |> 
    mutate(fecha = yearmonth(fecha))
```

- Con clave: `as_tsibble(index = month, key = store)`.
- Explorar: `index_var()`, `key_vars()`, `key_data()`, `distinct(tourism, State)`, `has_gaps()` y `fill_gaps()`.
- Resumen por grupo: `filter(...) |> as_tibble() |> group_by(Purpose) |> summarise(mean_trips = mean(Trips))`.
- Gráficas: `autoplot(Gas)`, `autoplot(Gas) + geom_point()`, `gg_season(Gas)`, `gg_subseries(Gas)` y `gg_tsdisplay(Gas, plot_type = "season")`.
- Facetas: `autoplot(Trips) + facet_wrap(vars(Purpose), scales = "free_y") + theme(legend.position = "none")`.
- Exportar: `mutate(Quarter = as.Date(Quarter)) |> write_csv("./datos/archivo.csv")`. Importar: `read_csv("archivo.csv")`.

## Clase 1.2: transformaciones y descomposición

```r
lambda <- serie |> 
  features(y, features = guerrero) |> 
  pull(lambda_guerrero)
```

- Transformaciones dentro del modelo o la gráfica: `log(y)` y `box_cox(y, lambda)`.
- Descomposición clásica: `model(clasica = classical_decomposition(Cost, type = "additive")) |> components() |> autoplot()`, también con `log(Cost)` o `type = "multiplicative"`.
- STL: `model(stl = STL(Cost ~ season(window = "periodic"), robust = TRUE)) |> components() |> autoplot()`. Con ventana de tendencia: `STL(y ~ trend(window = 7) + season(window = "periodic"), robust = TRUE)`.

## Clase 1.3: flujo de pronóstico y métodos de referencia

```r
h <- 24

serie_train <- serie |> 
  slice_head(n = nrow(serie) - h)

serie_test <- serie |> 
  slice_tail(n = h)

serie_fit <- serie_train |> 
  model(
    mean   = MEAN(y),
    naive  = NAIVE(y),
    snaive = SNAIVE(y),
    drift  = RW(y ~ drift())
  )

serie_fc <- serie_fit |> 
  forecast(h = nrow(serie_test))

serie_fc |> 
  accuracy(serie) |> 
  select(.model, RMSE, MAE, MAPE, MASE, RMSSE) |> 
  arrange(RMSSE)
```

- Partición por fecha: `filter_index(. ~ "2019 Dec")` y `filter_index("2020 Jan" ~ .)`, o `filter(year(date) <= 2019)`.
- Residuales: `serie_fit |> select(snaive) |> gg_tsresiduals()`.
- Ljung-Box: `serie_fit |> augment() |> filter(.model == "snaive") |> features(.innov, ljung_box, lag = 10, dof = 0)`.
- Reajuste con toda la serie y pronóstico final: `serie |> model(snaive = SNAIVE(y)) |> forecast(h = 24)`, graficado con `autoplot(serie) + labs(title = ..., y = ..., x = NULL)`.
- Horizonte como texto: `forecast(h = "2 years")`.

## Clase 1.4: intervalos, transformaciones y `decomposition_model`

- Intervalos: `autoplot(serie, level = 95)` y `hilo(level = 95) |> select(fecha, .mean, \`95%\`)`.
- Desviación de los residuales por modelo: `augment() |> as_tibble() |> group_by(.model) |> summarise(sigma_hat = sd(.innov, na.rm = TRUE))`.
- Modelos con Box-Cox: `SNAIVE(box_cox(y, lambda))` y `RW(box_cox(y, lambda) ~ drift())`. El pronóstico regresa a la escala original con ajuste por sesgo; la mediana se agrega con `mutate(.median = median(y))`.
- Intervalos bootstrap: `forecast(h = 30, bootstrap = TRUE, times = 500, seed = 123)`.
- Comparar varios pronósticos: `bind_rows(fc_1 |> accuracy(serie), fc_2 |> accuracy(serie))`.
- STL dentro del modelo:

```r
stl_spec <- decomposition_model(
  STL(box_cox(y, lambda) ~ trend() + season(), robust = TRUE),
  SNAIVE(season_year),
  RW(season_adjust ~ drift())
)

serie_fit <- serie_train |> 
  model(stl_bc = stl_spec)
```

## Clase 2.1: suavizamiento exponencial (ETS)

- SES: `ETS(y ~ error("A") + trend("N") + season("N"))`.
- Holt: `ETS(y ~ error("A") + trend("A") + season("N"))`; amortiguado: `trend("Ad", phi = 0.9)`.
- Holt-Winters aditivo: `ETS(y ~ error("A") + trend("A") + season("A"))`; multiplicativo: `ETS(y ~ error("M") + trend("A") + season("M"))`; amortiguado: `ETS(y ~ error("M") + trend("Ad") + season("M"))`.
- Selección automática: `ETS(y)` o restringida, `ETS(y ~ trend(c("A", "Ad")))`.
- STL + ETS: `decomposition_model(STL(...), ETS(season_adjust ~ error("M") + trend("A") + season("N")), SNAIVE(season_year))`.
- Parámetros estimados: `report()` y `tidy()` del modelo seleccionado con `select(SES)`.
- Prueba sobre el pronóstico: `autoplot(...) + autolayer(serie_test, y, colour = "#C0392B", linetype = "dashed")`.
- Datos diarios a totales: `index_by(Date) |> summarise(Count = sum(Count))`.

## Clase 2.2: estacionariedad y diferenciación

- Diferencias: `difference(y)`, `difference(y, differences = 2)` y estacional `difference(log(y), lag = 12)`.
- KPSS: `serie |> features(y, unitroot_kpss)`.
- Número de diferencias: `features(box_cox(y, lambda), unitroot_nsdiffs)` y `features(difference(box_cox(y, lambda), 12), unitroot_ndiffs)`.
- ACF y PACF juntas:

```r
serie |> 
  gg_tsdisplay(
    difference(box_cox(y, lambda), 12) |> difference(1),
    plot_type = "partial",
    lag_max = 48
  )
```

## Clase 2.3: ARIMA y SARIMA

```r
serie_fit <- serie |> 
  model(
    sarima_011_011 = ARIMA(box_cox(y, lambda) ~ pdq(0, 1, 1) + PDQ(0, 1, 1)),
    sarima_auto    = ARIMA(box_cox(y, lambda),
                           stepwise      = FALSE,
                           approximation = FALSE)
  )

glance(serie_fit) |> 
  arrange(AICc) |> 
  select(.model, AIC, AICc, BIC)
```

- No estacional: `ARIMA(y ~ pdq(1, 0, 0) + PDQ(0, 0, 0))`; búsqueda acotada: `pdq(1:3, 0, 0:2)`.
- Resumen: `serie_fit |> select(sarima_auto) |> report()`.
- Residuales: `gg_tsresiduals(lag_max = 48)` y Ljung-Box con `dof` igual al número de coeficientes AR y MA.
- STL + ARIMA: `decomposition_model(STL(box_cox(y, lambda) ~ season(window = "periodic"), robust = TRUE), ARIMA(season_adjust ~ PDQ(0, 0, 0)))`.

## Clase 2.4: pronóstico modular

Las especificaciones se guardan en objetos y se ajustan juntas:

```r
stl_ets_spec <- decomposition_model(
  STL(log(y) ~ trend(window = NULL) + season(window = "periodic"), robust = TRUE),
  ETS(season_adjust ~ season("N"))
)

serie_fit <- serie_train |> 
  model(
    stl_ets = stl_ets_spec,
    ets     = ETS(log(y)),
    sarima  = ARIMA(log(y), stepwise = FALSE, approximation = FALSE)
  )

serie_fit |> 
  forecast(h = nrow(serie_test)) |> 
  accuracy(serie) |> 
  select(.model, RMSE, MAE, MAPE, MASE) |> 
  arrange(RMSE)
```

- Otras combinaciones: STL + ARIMA (`ARIMA(season_adjust ~ PDQ(0, 0, 0))`), STL + ETS para la tendencia y `ARIMA(season_year)` para la estacionalidad, y STL + ARIMA con `ETS(season_year ~ trend("N"))`.
- Descarga de series con `tq_get()`, fecha con `yearmonth()` y la columna renombrada a `y`.

## Clase 3.1: problemas prácticos (outliers, faltantes y límites)

Outliers con el residuo de una STL que solo quita la tendencia y la regla IQR:

```r
serie_dcmp <- serie |>
  model(
    stl = STL(y ~ season(period = 1), robust = TRUE)
  ) |>
  components()

c_threshold <- 1.5

serie_outliers <- serie_dcmp |>
  filter(
    remainder < quantile(remainder, 0.25) - c_threshold * IQR(remainder) |
    remainder > quantile(remainder, 0.75) + c_threshold * IQR(remainder)
  )
```

- `c_threshold = 3` marca solo los casos extremos. Si se sospechan outliers dentro del patrón estacional, se usa una descomposición estacional completa.
- Quitar outliers y dejar el hueco explícito: `serie |> anti_join(serie_outliers, by = "fecha") |> fill_gaps()`.
- Imputar con ARIMA (ETS no admite `NA`): `serie_miss |> model(ARIMA(y)) |> interpolate(serie_miss)`.
- Comparar original, faltante e imputado: `rename()` y `left_join(..., by = "fecha")`, luego `filter(fecha %in% serie_outliers$fecha)`.
- Pronósticos positivos con `log(y)` dentro del modelo; límites `[a, b]` con un logit escalado creado con `new_transformation(scaled_logit, inv_scaled_logit)`.
- En sus gráficas aparece `geom_hline(yintercept = 0, colour = ..., linetype = "dashed")`.

## Clase 3.2: regresión de series de tiempo

Fuente: `module_3/02_regression/regression.html` (código fuente `regression.qmd`).

```r
us_change_fit <- us_change |>
  model(
    simple = TSLM(Consumption ~ Income),
    multiple = TSLM(Consumption ~ Income + Production + Savings + Unemployment)
  )

us_change_fit |>
  glance() |>
  select(.model, adj_r_squared, AIC, AICc, BIC) |>
  arrange(AICc)
```

- Explorar predictores: `as_tibble() |> select(-Quarter) |> GGally::ggpairs()`.
- Diagnóstico de `TSLM`: `gg_tsresiduals()` y `augment() |> features(.resid, ljung_box, lag = 10, dof = ...)`; contar los coeficientes con `tidy() |> nrow()`.
- Selección de predictores: ajustar explícitamente los modelos candidatos con `TSLM` y comparar AICc; las búsquedas hacia adelante y atrás de la clase se escriben por etapas, sin una función automática adicional.
- Pronóstico con predictores conocidos: `forecast(new_data = serie_test)`. Para pronosticar los predictores, usar `pivot_longer`, `model(arima = ARIMA(value))`, `forecast`, `as_tsibble`, `select` y `pivot_wider(names_from = variable, values_from = .mean)`.
- Tendencia y estacionalidad: `TSLM(Beer ~ trend() + season())`. Indicadores manuales con `row_number()`, `quarter()`, `if_else()` y `yearquarter()`; intervenciones como un pulso o un cambio de nivel se incorporan a la fórmula.
- Formas funcionales: `TSLM(Minutes ~ trend())`, `TSLM(log(Minutes) ~ trend())` y `TSLM(Minutes ~ trend(knots = c(1940, 1980)))`.
- Gráficas de ajustes con `fitted()`, `geom_line(data = fitted(...), aes(y = .fitted))` y `autolayer(forecast(...), alpha = ..., level = ...)`. El código de la clase también usa `ggplot`, `aes`, `geom_point`, `geom_abline` y `facet_wrap`.
- Conversión a valores numéricos con `as.numeric()`: en la clase 3.2 se usa `mutate(Minutes = as.numeric(Time) / 60)`; la clase 2.2 también convierte valores de ACF con esta función.

## Clase 3.3: regresión dinámica y armónica

Fuente: `module_3/03_dynamic/dynamic_regression.html` (código fuente `dynamic_regression.qmd`).

```r
us_change_fit_dyn <- us_change |>
  model(
    tslm = TSLM(Consumption ~ Income + Production + Savings + Unemployment),
    dynamic = ARIMA(Consumption ~ Income + Production + Savings + Unemployment)
  )

us_change_fit_dyn |>
  select(dynamic) |>
  report()
```

- Forzar diferenciación en la regresión: `ARIMA(y ~ x1 + x2 + pdq(d = 1))`.
- Distinguir errores de regresión e innovaciones con `residuals(modelo, type = "regression")` y `type = "innovation"`. Para Ljung-Box, usar `.innov` y contar coeficientes AR/MA con `coefficients() |> filter(str_detect(term, "^ar|^ma")) |> nrow()`.
- Crear fechas futuras con `new_data(serie, 8)`. Construir escenarios mediante `mutate` o pronosticar predictores con `pivot_longer` + `model(MEAN(value))` + `forecast` + `pivot_wider`, y pasarlos a `forecast(new_data = ...)`.
- No linealidad en predictores: `ARIMA(Demand ~ Temperature + I(Temperature^2) + (Day_Type == "Weekday"))`. El calendario se construye con `wday`, `case_when` y `%in%`; los escenarios de la clase usan `c`, `seq` y `rep`.
- Rezagos de predictores: `ARIMA(Quotes ~ pdq(d = 0) + TVadverts + lag(TVadverts))`; comparar candidatos con más rezagos mediante AICc.
- Regresión armónica: `ARIMA(log(Turnover) ~ fourier(K = 3) + PDQ(0, 0, 0))`. Ajustar varios valores de `K` como modelos explícitos y comparar `glance() |> select(.model, AICc, BIC) |> arrange(AICc)`.

## Clase 3.4: Prophet

Fuente: `module_3/04_prophet/prophet.html` (código fuente `prophet.qmd`).

```r
library(fable.prophet)

lax_fit <- lax_train |>
  model(
    Prophet = prophet(passengers ~
      growth("linear") + season("year", type = "multiplicative")),
    Prophet_auto = prophet(passengers),
    ARIMA = ARIMA(passengers),
    ETS = ETS(passengers),
    Harmonic = ARIMA(passengers ~ fourier(K = 3) + PDQ(0, 0, 0))
  )
```

- Preparación del ejemplo LAX: `read.csv`, `lubridate::mdy_hms`, `group_by(month = yearmonth(datetime), type = ...)`, `summarise(..., .groups = "drop")` y `as_tsibble(index = month, key = type)`.
- Configurar tendencia con `growth("linear", n_changepoints = 25, changepoint_range = 0.8, changepoint_prior_scale = 0.05)`.
- Configurar estacionalidad con `season("year", period = 365.25, order = 10, type = "additive")`; la clase también muestra periodos semanales y diarios y el tipo `"multiplicative"`.
- Componentes: `select(type, Prophet) |> components() |> autoplot()`. Pronósticos por dos años con `forecast(h = "2 years")` y comparación por grupo con `accuracy`, `select` y `arrange(type, MAPE)`.
- Gráficas por grupo con `facet_wrap(~ type, ncol = 1, scales = "free_y")`; comparación de componentes con `lubridate::month(..., label = TRUE)` e `interaction(type, lubridate::year(month))`. Las tablas de comparación del código fuente usan `knitr::kable()`.

## Clase 4.1: estacionalidad compleja

Fuente: `module_4/01_complex_seasonality/complex_seasonality.html` (código fuente `complex_seasonality.qmd`).

- Agregar datos intradiarios: `index_by(Hour = floor_date(Time, "hour")) |> summarise(Demand = mean(Demand), Temperature = mean(Temperature), Holiday = any(Holiday))`.
- Calendario: `mutate(WorkDay = !Holiday & wday(Hour) %in% 2:6)`.
- Gráficas estacionales: `gg_season(Demand, period = "day", alpha = 0.4)` y `period = "week"`. Partición con fechas y horas en `filter_index`.

```r
stl_spec <- STL(Demand ~
  season(period = 24) + season(period = 168),
  robust = TRUE)

stl_fit <- vic_elec_train |>
  model(
    STL_ETS = decomposition_model(
      stl_spec,
      ETS(season_adjust ~ season("N"))
    ),
    STL_ARIMA = decomposition_model(
      stl_spec,
      ARIMA(season_adjust ~ PDQ(0, 0, 0))
    )
  )
```

- Fourier con varios periodos: `ARIMA(Demand ~ fourier(period = 24, K = 10) + fourier(period = 168, K = 6) + PDQ(0, 0, 0), stepwise = TRUE, approximation = TRUE)`.
- Covariables en la misma fórmula: `Temperature + I(pmax(Temperature - 18, 0)) + WorkDay`; construir sus valores futuros con `new_data` y `mutate`, y usar `forecast(new_data = ...)`.
- La búsqueda opcional de `K` usa `expand_grid`, `map2` con `\(k1, k2)`, `map_dbl` con `\(m) glance(m)$AICc`, `select` y `arrange(aicc)`. No hace falta añadirla cuando el ejercicio solo pide los valores iniciales.
- Comparación: `bind_rows(stl_fc, fourier_fc) |> accuracy(serie_completa) |> select(.model, RMSE, MAE, MAPE, MASE) |> arrange(RMSE)`. Diagnóstico con `gg_tsresiduals(lag_max = ...)`.
- Ejercicio `bank_calls`: explicitar cierres con `fill_gaps(Calls = 0)` antes de agregar a medias horas mediante `index_by` y `summarise(Calls = sum(Calls))`; reservar las últimas dos semanas. A cinco minutos, los periodos son 288 y 2016; a medias horas, 48 y 336. Para Fourier, comenzar con `K = 6` diario y `K = 3` semanal, con `stepwise = TRUE, approximation = TRUE`.
- El ejercicio pide revisar ACF por al menos siete días, residuos alrededor de los rezagos 48 y 336, exactitud y pronósticos sobre el test. El indicador `IsOpen` o filtrar cierres aparecen como alternativas para discutir si los pronósticos nocturnos no son cero, no como pasos obligatorios.

## Respuesta a comentarios

Cuando un bloque termine con un comentario como `# convertir a tsibble`, `# lambda`, `# descomposición STL`, `# train/test`, `# ajustar modelos`, `# residuales`, `# Ljung-Box`, `# accuracy`, `# diferenciar`, `# ACF y PACF`, `# ARIMA` o `# pronóstico final`, completa únicamente ese paso con las estructuras anteriores. No adelantes pasos posteriores.
