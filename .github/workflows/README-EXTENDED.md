# VAR-IO-Emissions: Monetary Policy Transmission and CO₂ Emissions in Spain

### *Transmisión de Política Monetaria del BCE y Emisiones de CO₂ en España*

> **Bachelor's Thesis (TFG) — Economics \| Universitat Autònoma de Barcelona**\
> Author: Cristian \| Last updated: 2026

------------------------------------------------------------------------

##English

### Overview

This project analyzes how **ECB monetary policy shocks** propagate through the Spanish economy and ultimately affect **sectoral CO₂ emissions**. The methodology chains three analytical blocks:

```         
ECB Monetary Shock (SSR)
        ↓
  Structural VAR (SVAR)
  → Impulse Response Functions (IRFs)
        ↓
  Input-Output Analysis (Leontief)
  → Sectoral production effects
        ↓
  Emission Intensity Factors
  → ΔCO₂eq by sector (ton)
```

The analysis covers Spain over the period **2002–2025** (monthly data, 285 observations) and examines four sectors under the CNAE classification: **Industry, Energy, Commerce, and Services**.

------------------------------------------------------------------------

### Methodology

#### 1. Structural VAR (SVAR)

Two separate VAR models are estimated to avoid multicollinearity between the aggregate ICN index and its sectoral components:

-   **Aggregate VAR**: `SSR → IPC → ICN_general`
-   **Sectoral VAR**: `SSR → IPC → ICN_industria → ICN_energia → ICN_comercio → ICN_servicios`

**Variable ordering (Cholesky):** The Shadow Short Rate (SSR) is ordered first — most exogenous — reflecting Spain's position as a *policy taker* within the Eurozone. IPC follows due to price stickiness; sectoral activity variables are most endogenous. This ordering follows Christiano, Eichenbaum & Evans (1999), adapted for a small open economy as in Peersman & Smets (2001).

**Monetary policy instrument:** The **Krippner Shadow Short Rate (SSR)** replaces the MRO rate to account for the Zero Lower Bound (ZLB) period (2014–2022), during which conventional interest rates lost informativeness (Krippner, 2015).

**Lag selection:** AIC criterion → 12 lags (monthly data).

**Exogenous variable:** A binary dummy controls for extreme-value periods (2009 financial crisis, COVID-19 shock, post-COVID recovery, 2022 energy volatility).

#### 2. Identification Strategies

| Method | Description | Reference |
|----|----|----|
| **Cholesky decomposition** | Primary identification via recursive ordering | Christiano et al. (1999) |
| **Wild Bootstrap** | Robust inference under ARCH effects and heteroskedasticity | Gonçalves & Kilian (2004) |
| **Sign Restrictions (Uhlig)** | Acceptance-rejection algorithm with random orthogonal rotations | Uhlig (2005) |
| **Sign Restrictions (Arias et al.)** | Bayesian uniform sampling via `bsvarSIGNs` | Arias, Rubio-Ramírez & Waggoner (2018) |

Sign restrictions imposed on the monetary shock: - SSR ≥ 0 at impact (h=0): contractionary shock raises the shadow rate - IPC ≤ 0 on average for h = 1,...,6: inflation falls

> **Note:** Wild bootstrap and sign restrictions are methodologically incompatible. Wild bootstrap is applied exclusively to Cholesky-identified models; Bayesian inference handles sign-restricted SVARs.

#### 3. Input-Output (Leontief) Analysis

Using Spain's **2022 Input-Output table** (64×64 sectors, aggregated to 4), the Leontief inverse matrix is computed:

$$L = (I - A)^{-1}$$

The IRF accumulated responses (in percentage points over 12 months) are converted to euros using:

$$\Delta Y_{\text{euros}} = \frac{\text{IRF accumulated (pp)}}{100} \times Y_{\text{base 2022}}$$

Total production effects (direct + indirect) are then obtained via:

$$\Delta x = L \cdot \Delta y$$

#### 4. Emissions Calculation

Sectoral emission intensity factors (kg CO₂eq / €) are derived from **Eurostat environmental accounts** and applied to the production changes:

$$\Delta \text{Emissions}_{\text{sector}} = \Delta x_{\text{sector}} \times \text{Intensity}_{\text{sector}}$$

------------------------------------------------------------------------

### Key Results

A contractionary monetary shock of **+1 pp in the SSR** (Uhlig sign restrictions, 12-month cumulative horizon) produces:

| Sector    | Δ Production (M€) | Δ Emissions (ton CO₂eq) |
|-----------|-------------------|-------------------------|
| Industry  | −13,608           | −1,321,028              |
| Energy    | −7,562            | −2,313,814              |
| Commerce  | −2,980            | −71,749                 |
| Services  | −12,203           | −812,680                |
| **Total** | **−36,352**       | **−4,519,271**          |

**FEVD results** (variance decomposition): the SSR shock explains approximately 13% of sectoral ICN variance at horizon h=12.

------------------------------------------------------------------------

### Diagnostics

| Test | Result |
|----|----|
| Ljung-Box (univariate, lag=16) | No autocorrelation in any equation (min p = 0.30) |
| Portmanteau (multivariate) | Cross-equation autocorrelation detected — trivial magnitude, handled by wild bootstrap |
| ARCH test (lag=8) | ARCH effects present → wild bootstrap justified |
| Jarque-Bera | Non-normality (common in macro, does not invalidate OLS) |
| Granger causality (SSR → real vars) | p \< 0.001 — SSR Granger-causes sectoral activity |

------------------------------------------------------------------------

### Data Sources

| Variable | Source | Frequency |
|----|----|----|
| ICN (sectoral business turnover indices) | INE (Instituto Nacional de Estadística) | Monthly |
| IPC (CPI) | INE | Monthly |
| Shadow Short Rate (SSR) | Krippner (RBNZ database) | Monthly |
| Input-Output table | INE — Spanish National Accounts (2022) | Annual |
| Emission intensities | Eurostat — Environmental accounts (NAMEA) | Annual |

------------------------------------------------------------------------

### Repository Structure

```         
├── analisis_var_emisiones.Rmd        # Main analysis (v1)
├── analisis_var_emisiones_v2.Rmd     # Main analysis (v2, current)
├── Datos_VAR_M.xlsx                  # Monthly VAR data (SSR, IPC, ICN sectors)
├── Tabla_IO_22.xlsx                  # Spain Input-Output table 2022
├── zcuentas_de_emisiones_de_gases.xlsx   # GHG emission accounts by sector
├── zintensidades_de_emisiones.xlsx       # Emission intensity factors
├── VAR_contexto_IA.pdf               # Methodological context document
└── README.md
```

------------------------------------------------------------------------

### Requirements

``` r
# R packages required
paquetes <- c(
  "tidyverse", "vars", "svars", "urca",
  "readxl", "tseries", "lubridate",
  "ggplot2", "gridExtra", "writexl",
  "knitr", "stargazer", "bsvarSIGNs"
)
```

R version ≥ 4.2.0 recommended. Install all packages with:

``` r
install.packages(paquetes)
```

------------------------------------------------------------------------

### References

-   Arias, J.E., Rubio-Ramírez, J.F. & Waggoner, D.F. (2018). Inference Based on Structural Vector Autoregressions Identified With Sign and Zero Restrictions. *Econometrica*, 86(2), 685–720.
-   Christiano, L.J., Eichenbaum, M. & Evans, C.L. (1999). Monetary policy shocks: What have we learned and to what end? *Handbook of Macroeconomics*, 1, 65–148.
-   Gonçalves, S. & Kilian, L. (2004). Bootstrapping autoregressions with conditional heteroskedasticity of unknown form. *Journal of Econometrics*, 123(1), 89–120.
-   Krippner, L. (2015). *Zero Lower Bound Term Structure Modeling: A Practitioner's Guide*. Palgrave Macmillan.
-   Lütkepohl, H. (2005). *New Introduction to Multiple Time Series Analysis*. Springer.
-   Miller, R.E. & Blair, P.D. (2009). *Input-Output Analysis: Foundations and Extensions*. Cambridge University Press.
-   Peersman, G. & Smets, F. (2001). The monetary transmission mechanism in the euro area. *ECB Working Paper*, No. 91.
-   Uhlig, H. (2005). What are the effects of monetary policy on output? Results from an agnostic identification procedure. *Journal of Monetary Economics*, 52(2), 381–419.

## 🇪🇸 Español

### Descripción del proyecto

Este proyecto analiza cómo los **shocks de política monetaria del BCE** se propagan a través de la economía española y afectan finalmente a las **emisiones sectoriales de CO₂**. La metodología encadena tres bloques analíticos:

```         
Shock monetario BCE (SSR)
        ↓
  VAR Estructural (SVAR)
  → Funciones de Impulso-Respuesta (IRF)
        ↓
  Análisis Input-Output (Leontief)
  → Efectos sobre producción sectorial
        ↓
  Factores de intensidad de emisiones
  → ΔCO₂eq por sector (toneladas)
```

El análisis cubre España durante el período **2002–2025** (datos mensuales, 285 observaciones) y examina cuatro sectores según la clasificación CNAE: **Industria, Energía, Comercio y Servicios**.

------------------------------------------------------------------------

### Metodología

#### 1. VAR Estructural (SVAR)

Se estiman dos modelos VAR separados para evitar multicolinealidad entre el ICN agregado y sus componentes sectoriales:

-   **VAR Agregado**: `SSR → IPC → ICN_general`
-   **VAR Sectorial**: `SSR → IPC → ICN_industria → ICN_energia → ICN_comercio → ICN_servicios`

**Orden Cholesky:** El Shadow Short Rate (SSR) se ordena primero — más exógeno — reflejando la posición de España como *tomadora de política* dentro de la Eurozona. El IPC sigue por la rigidez de precios; las variables de actividad sectorial son las más endógenas. Este orden sigue a Christiano, Eichenbaum & Evans (1999), adaptado para economía pequeña abierta según Peersman & Smets (2001).

**Instrumento de política monetaria:** El **Shadow Short Rate de Krippner (SSR)** sustituye al tipo MRO para capturar el período de Límite Inferior Cero (ZLB, 2014–2022), durante el cual los tipos convencionales perdieron capacidad informativa (Krippner, 2015).

**Selección de rezagos:** Criterio AIC → 12 rezagos (datos mensuales).

**Variable exógena:** Una dummy binaria controla los períodos con valores extremos (crisis financiera 2009, shock COVID-19, recuperación post-COVID, volatilidad energética 2022).

#### 2. Estrategias de identificación

| Método | Descripción | Referencia |
|----|----|----|
| **Descomposición de Cholesky** | Identificación principal por ordenación recursiva | Christiano et al. (1999) |
| **Wild Bootstrap** | Inferencia robusta ante efectos ARCH y heterocedasticidad | Gonçalves & Kilian (2004) |
| **Restricciones de signo (Uhlig)** | Algoritmo de aceptación-rechazo con rotaciones ortogonales aleatorias | Uhlig (2005) |
| **Restricciones de signo (Arias et al.)** | Muestreo bayesiano uniforme mediante `bsvarSIGNs` | Arias, Rubio-Ramírez & Waggoner (2018) |

Restricciones de signo impuestas sobre el shock monetario: - SSR ≥ 0 en impacto (h=0): el shock contractivo eleva el tipo sombra - IPC ≤ 0 en promedio para h = 1,...,6: la inflación se reduce

> **Nota metodológica:** Wild bootstrap y restricciones de signo son incompatibles. El wild bootstrap se aplica exclusivamente a modelos identificados por Cholesky; la inferencia bayesiana gestiona los SVAR con restricciones de signo.

#### 3. Análisis Input-Output (Leontief)

Utilizando la **tabla Input-Output española de 2022** (64×64 sectores, agregada a 4), se calcula la matriz inversa de Leontief:

$$L = (I - A)^{-1}$$

Las respuestas acumuladas de las IRF (en puntos porcentuales a 12 meses) se convierten a euros mediante:

$$\Delta Y_{\text{euros}} = \frac{\text{IRF acumulada (pp)}}{100} \times Y_{\text{base 2022}}$$

Los efectos totales sobre la producción (directos + indirectos) se obtienen como:

$$\Delta x = L \cdot \Delta y$$

#### 4. Cálculo de emisiones

Los factores de intensidad de emisiones sectoriales (kg CO₂eq / €) provienen de las **cuentas medioambientales de Eurostat** y se aplican a los cambios de producción:

$$\Delta \text{Emisiones}_{\text{sector}} = \Delta x_{\text{sector}} \times \text{Intensidad}_{\text{sector}}$$

------------------------------------------------------------------------

### Resultados principales

Un shock monetario contractivo de **+1 pp en el SSR** (restricciones de signo de Uhlig, horizonte acumulado 12 meses) produce:

| Sector    | Δ Producción (M€) | Δ Emisiones (ton CO₂eq) |
|-----------|-------------------|-------------------------|
| Industria | −13.608           | −1.321.028              |
| Energía   | −7.562            | −2.313.814              |
| Comercio  | −2.980            | −71.749                 |
| Servicios | −12.203           | −812.680                |
| **Total** | **−36.352**       | **−4.519.271**          |

**Resultados FEVD** (descomposición de varianza): el shock SSR explica aproximadamente el 13% de la varianza del ICN sectorial en el horizonte h=12.

------------------------------------------------------------------------

### Diagnósticos del modelo

| Test | Resultado |
|----|----|
| Ljung-Box (univariado, lag=16) | Sin autocorrelación en ninguna ecuación (p mín. = 0,30) |
| Portmanteau (multivariante) | Autocorrelación cruzada detectada — magnitud trivial, gestionada por wild bootstrap |
| Test ARCH (lag=8) | Efectos ARCH presentes → wild bootstrap justificado |
| Jarque-Bera | No-normalidad (habitual en macro, no invalida OLS) |
| Causalidad de Granger (SSR → variables reales) | p \< 0,001 — SSR causa en sentido de Granger la actividad sectorial |

------------------------------------------------------------------------

### Fuentes de datos

| Variable | Fuente | Frecuencia |
|----|----|----|
| ICN (índices de cifra de negocios sectoriales) | INE | Mensual |
| IPC | INE | Mensual |
| Shadow Short Rate (SSR) | Krippner (base de datos RBNZ) | Mensual |
| Tabla Input-Output | INE — Contabilidad Nacional (2022) | Anual |
| Intensidades de emisiones | Eurostat — Cuentas medioambientales (NAMEA) | Anual |

------------------------------------------------------------------------

### Estructura del repositorio

```         
├── analisis_var_emisiones.Rmd        # Análisis principal (v1)
├── analisis_var_emisiones_v2.Rmd     # Análisis principal (v2, actual)
├── Datos_VAR_M.xlsx                  # Datos mensuales VAR (SSR, IPC, ICN sectoriales)
├── Tabla_IO_22.xlsx                  # Tabla Input-Output España 2022
├── zcuentas_de_emisiones_de_gases.xlsx   # Cuentas de emisiones GEI por sector
├── zintensidades_de_emisiones.xlsx       # Factores de intensidad de emisiones
├── VAR_contexto_IA.pdf               # Documento de contexto metodológico
└── README.md
```

------------------------------------------------------------------------

### Requisitos

``` r
# Paquetes R necesarios
paquetes <- c(
  "tidyverse", "vars", "svars", "urca",
  "readxl", "tseries", "lubridate",
  "ggplot2", "gridExtra", "writexl",
  "knitr", "stargazer", "bsvarSIGNs"
)
```

Se recomienda R versión ≥ 4.2.0. Instala todos los paquetes con:

``` r
install.packages(paquetes)
```

------------------------------------------------------------------------

### Referencias

-   Arias, J.E., Rubio-Ramírez, J.F. & Waggoner, D.F. (2018). Inference Based on Structural Vector Autoregressions Identified With Sign and Zero Restrictions. *Econometrica*, 86(2), 685–720.
-   Christiano, L.J., Eichenbaum, M. & Evans, C.L. (1999). Monetary policy shocks: What have we learned and to what end? *Handbook of Macroeconomics*, 1, 65–148.
-   Gonçalves, S. & Kilian, L. (2004). Bootstrapping autoregressions with conditional heteroskedasticity of unknown form. *Journal of Econometrics*, 123(1), 89–120.
-   Krippner, L. (2015). *Zero Lower Bound Term Structure Modeling: A Practitioner's Guide*. Palgrave Macmillan.
-   Lütkepohl, H. (2005). *New Introduction to Multiple Time Series Analysis*. Springer.
-   Miller, R.E. & Blair, P.D. (2009). *Input-Output Analysis: Foundations and Extensions*. Cambridge University Press.
-   Peersman, G. & Smets, F. (2001). The monetary transmission mechanism in the euro area. *ECB Working Paper*, No. 91.
-   Uhlig, H. (2005). What are the effects of monetary policy on output? Results from an agnostic identification procedure. *Journal of Monetary Economics*, 52(2), 381–419.

------------------------------------------------------------------------

*This project constitutes the Bachelor's Thesis (TFG) in Economics. The methodology was implemented autonomously in RStudio/RMarkdown.*\
*Este proyecto constituye el Trabajo de Fin de Grado (TFG) en Economía. La metodología fue implementada de forma autónoma en RStudio/RMarkdown.*
