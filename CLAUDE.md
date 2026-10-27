# SERIES DE TIEMPO

Curso del ITESO, otoño 2026 (O2026_MAF3074I), con el profesor Pablo Benavides Herrera. Alumno: Carlos Enrique Nieves Ochoa.
El trabajo se hace en documentos Quarto (`.qmd`) con R, `tidyverse` y `fpp3` (el profesor usa Positron), y se
entrega el `.html` renderizado con `embed-resources: true`.

Las carpetas `contrahuella/`, `output/` y `tmp/` son un proyecto personal (un juego) que no forma parte de la
materia: no las revises ni las tomes como referencia.

## Fuente principal: el sitio del profesor

Las clases se dan a partir del sitio del curso, que tiene el material de todo el semestre:
https://pbenavidesh.github.io/narsil/docs/modules/

Al hacer una actividad, guíate por el código de las clases vistas en ese sitio. Cada clase se lee en
`https://pbenavidesh.github.io/narsil/docs/modules/<ruta>` con las rutas de "Clases vistas". Los archivos `.qmd`
de este repositorio son complementarios.

## Regla principal: solo código visto en clase

Esta regla está por encima de escribir código más corto, eficiente o elegante.

- Usa únicamente librerías, funciones, argumentos y procedimientos que aparezcan en alguno de estos lugares:
  1. Las clases vistas del sitio del profesor (lista en "Clases vistas").
  2. Los archivos de clase del repositorio: `Primer documento de Quarto.qmd` y `Descomposición.qmd`.
  3. Las instrucciones, el enunciado o el código inicial de la actividad nueva.
- No uses nada que solo aparezca en clases del sitio que todavía no se ven.
- `pronostico_importaciones_autos.qmd` y `Ejercicio_ETS.qmd` se hicieron con ayuda de una IA: cuentan como
  tareas hechas, pero no como fuente de métodos. De ellas solo vale lo que también aparece en las clases vistas.
- No uses atajos ni versiones más sofisticadas de lo visto, aunque den el mismo resultado.
- `theme_narsil()` es un tema propio del profesor: no lo uses salvo que la actividad lo defina.
- Si la actividad pide algo que no aparece en lo visto, avisa qué falta y pregunta antes de continuar.
- Antes de entregar, revisa cada función que usaste y confirma en qué clase aparece.

La lista detallada de lo visto, con su estructura y su clase de referencia, vive en `.github/`, la misma que usa
Copilot para recomendar código en los exámenes a partir de un comentario:

@.github/copilot-instructions.md
@.github/instructions/r-clase.instructions.md

## Cómo hacer una actividad

1. Revisa si avanzó el curso (sección siguiente).
2. Lee las instrucciones completas de la actividad (Canvas o el enunciado que dé el alumno) y el código inicial, si lo hay.
3. Lee en el sitio la clase del tema y revisa su estructura en `.github/instructions/r-clase.instructions.md`.
4. Completa solo lo que se pide, en un `.qmd` en la raíz del repositorio, con los datos junto al documento.
5. Comprueba que el documento se renderice sin errores (`quarto render <archivo>.qmd`).
6. Al terminar, propone actualizar la sección "Avance".

## Revisar el avance al empezar cada sesión

El sitio tiene todas las clases desde el inicio, así que el avance no se ve en el sitio. Se marca así:

- La clase más avanzada que ya se vio marca hasta dónde va el curso; esa clase y todas las anteriores cuentan
  como vistas. El orden es el del sitio: `1.1` < `1.2` < `1.3` < `1.4` < `2.1` < … < `2.4` < `3.1` < … < `4.4`.
- Una clase pasa a estar vista cuando ocurre cualquiera de estas cosas: el alumno lo dice; un commit nuevo
  trabaja su tema; o en Canvas (curso 58150) hay una actividad o un examen de ese tema o de un módulo posterior.

Al empezar una actividad de esta materia:

- Ejecuta `git log --name-only --oneline <commit de "Última revisión">..HEAD` y revisa si la actividad o un
  commit nuevo corresponde a una clase posterior a la última vista.
- Si avanzó, lee en el sitio el código de las clases nuevas y actualiza:
  - `.github/instructions/r-clase.instructions.md`: lo nuevo en "Librerías permitidas" y una sección con la
    estructura de cada clase nueva, tal como aparece en el sitio.
  - `.github/copilot-instructions.md`: la clase en "Fuentes de referencia por tema" y quita de "No usar" lo que
    ya apareció.
  - Este archivo: "Clases vistas", "Avance" y "Última revisión".
- Después dile al alumno, antes de empezar, qué detectaste y qué quedó permitido.
- Si no hay cambios, no digas nada y sigue con la actividad.

Última revisión: commit `fecc52c` (27 oct 2026). El alumno solicitó el ejercicio de la clase 4.1 el 27 oct
2026. Conforme a la regla de avance, las clases hasta 4.1 cuentan como vistas; esto no registra sus
ejercicios como terminados.

## Clases vistas

Clase más avanzada vista: `4.1` (módulos 1, 2 y 3 completos y la primera clase del módulo 4).

- Módulo 1, métodos de pronóstico basados en descomposición:
  - `1.1` R, tsibble y series de tiempo: `module_1/01_time_series/r_time_series.html`
  - `1.2` Descomposición: `module_1/02_ts_dcmp/ts_dcmp.html`
  - `1.3` Fundamentos de pronóstico: `module_1/03_fcst/forecasting.html`
  - `1.4` Diagnóstico de modelos y pronóstico avanzado: `module_1/04_model_diagnostics/model_diagnostics.html`
- Módulo 2, filtros ETS y ARIMA:
  - `2.1` Suavizamiento exponencial: `module_2/01_ets/ets.html`
  - `2.2` Estacionariedad y diferenciación: `module_2/02_stationarity/stationarity.html`
  - `2.3` Modelos ARIMA: `module_2/03_arima/arima.html`
  - `2.4` Pronóstico modular, ETS y ARIMA combinados: `module_2/04_modular_fcst/modular_forecasting.html`

- Módulo 3:
  - `3.1` Problemas prácticos: outliers, valores faltantes y transformaciones con límites: `module_3/01_practical_issues/practical_issues.html`
  - `3.2` Regresión de series de tiempo: `module_3/02_regression/regression.html`
  - `3.3` Regresión dinámica y armónica: `module_3/03_dynamic/dynamic_regression.html`
  - `3.4` Prophet: `module_3/04_prophet/prophet.html`

- Módulo 4:
  - `4.1` Estacionalidad compleja: `module_4/01_complex_seasonality/complex_seasonality.html`

Siguiente clase que se volverá vista: `4.2` Robustez y combinación de pronósticos.

Archivos de clase del repositorio: `Primer documento de Quarto.qmd` (clase 1.1) y `Descomposición.qmd` (clase 1.2).
Tareas del alumno (con ayuda de IA): `pronostico_importaciones_autos.qmd`, `Ejercicio_ETS.qmd`.

## Avance

| Entrega | Estado |
| :- | :- |
| Primer documento de Quarto | Terminado (21 ago 2026) |
| Descomposición (medicinas) | Terminada (1 sep 2026) |
| Pronosticar las importaciones de autos | Terminado (13 sep 2026) |
| Proyecto M1: modelo baseline | Entregado (18 sep 2026, en equipo) |
| Ejercicio ETS | Terminado (23 sep 2026) |
| Exámenes de los módulos 1 y 2 | Hechos |
| Actividad SARIMA | Sin fecha en Canvas |
| Proyecto M2: pronóstico de ventas semanales | En curso (entrega 23 oct 2026, equipo 1) |
| Examen del módulo 3 | 9 nov 2026 |
| Proyecto M4 | 3 dic 2026 |
