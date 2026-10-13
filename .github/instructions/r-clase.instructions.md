---
applyTo: "**/*.qmd,**/*.R"
---

# Estructuras de R vistas en clase

Estas reglas se aplican a los documentos Quarto y R. La lista reúne lo visto en las clases 1.1 a 2.4 del sitio del profesor (https://pbenavidesh.github.io/narsil/docs/modules/) y en los archivos de clase del repositorio. Crece conforme se ven clases nuevas. Si algo no está aquí ni en la actividad actual, no debe proponerse.

## Librerías permitidas

- `tidyverse` y `fpp3` (que carga `tsibble`, `tsibbledata`, `feasts` y `fable`), en el primer bloque del documento.
- `tidyquant` con `tq_get` para descargar series (clases 2.2 a 2.4).
- `readxl::read_excel` para archivos de Excel y `stats::filter` cuando se necesita el `filter` de `stats`.
- Conjuntos de datos de clase: `tourism`, `aus_production`, `pelt`, `global_economy`, `prices`, `aus_livestock`, `us_change`, `pedestrian`, `aus_retail`.

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

## Respuesta a comentarios

Cuando un bloque termine con un comentario como `# convertir a tsibble`, `# lambda`, `# descomposición STL`, `# train/test`, `# ajustar modelos`, `# residuales`, `# Ljung-Box`, `# accuracy`, `# diferenciar`, `# ACF y PACF`, `# ARIMA` o `# pronóstico final`, completa únicamente ese paso con las estructuras anteriores. No adelantes pasos posteriores.
