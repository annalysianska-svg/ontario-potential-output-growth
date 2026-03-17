# Ontario Potential Output Growth

## Overview
This project estimates Ontario’s potential output growth using a Cobb-Douglas production function and scenario analysis.

## Research question
What is the potential growth in GDP for Ontario, and how might it change under alternative immigration and trade disruption scenarios?

## Data

The historical model uses annual Ontario data from **1997 to 2023**, with dollar variables expressed in **2017 chained dollars** to account for inflation and depreciation. Below are the table sources of the data. However, the data section of the file also includes organized data tables that I cleaned in Excel.

### Main variables and sources

- **Real GDP (Y)**  
  Statistics Canada Table [**36-10-0222-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3610022201)  
  *Gross domestic product, expenditure-based, provincial and territorial, annual (x 1,000,000)*

- **Capital stock (K)**  
  Statistics Canada Table [**36-10-0096-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3610009601)  
  *Flows and stocks of fixed non-residential capital, by industry and type of asset, Canada, provinces and territories (x 1,000,000)*

- **Labour input (L)**  
  Statistics Canada Table [**36-10-0480-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3610048001)  
  *Labour productivity and related measures by business sector industry and by non-commercial activity consistent with the industry accounts*  
  I use hours worked for all jobs as the main labour proxy.The manufacturing hours profile over 2019-2022 (Covid-19) is used as an empirical template for a trade shock.

- **Multifactor productivity index (A for potential-output construction)**  
  Statistics Canada Table [**36-10-0208-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3610020801)  
  *Multifactor productivity, value-added, capital input and labour input in the aggregate business sector and major sub-sectors, by industry*  
  Total Factor Productivity has been obtained from the table to estimate the potential output. However, the historical regression does not use this measure, but rather considers TFP as a Solow’s residual, the error term.
    
- **Unemployment rate**  
  Statistics Canada Table [**14-10-0023-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=1410002301)
  *Labour force characteristics by industry, annual (x 1,000)*
  
- **Immigrant share of the labour force**    
  Statistics Canada Table [**17-10-0121-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=1710012101) and [**14-10-0022-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=1410002201)
  *Estimates of the number of non-permanent residents by type, quarterly* is used to capture temporary workers, specifically asylum seekers with work permits only, asylum seekers with both work and study permits, work permit holders, and temporary residents with both work and study permits. *Labour force characteristics by industry, monthly, unadjusted for seasonality* is used for the labour force measure at a monthly frequency, aligned with the quarterly immigrant-labour data to compute immigrants’ contribution to the labour force. Temporary workers are foreign nationals granted authorization to engage in paid work in Canada. As a result, it is reasonable to treat all work permit holders as part of the labour-force pool for the purposes of this paper, since holding (or applying for) a work permit signals an intention to participate in the labour market and actively seek employment. 

## Why I chose these data

I chose these series to match the production-function structure directly:

- **Y** is Ontario real GDP, the output to be explained.
- **K** is Ontario non-residential capital stock, which captures productive capital accumulation.
- **L** is total hours worked, rather than just employment counts, because hours better capture changes in labour utilization.
- **A** is treated in two different ways:
  - in the historical regressions, TFP is treated as the unexplained residual in Solow-style fashion
  - in the potential-output construction, I use the multifactor productivity index to build a smooth trend TFP path

This choice lets the historical model stay close to the growth-accounting logic while keeping the potential-output path anchored in an observed productivity series rather than leaving all long-run productivity entirely inside regression residuals.

## Methodology

## 1. Production function framework

The project is based on a Cobb-Douglas production function:

\[
Y_t = A_t K_t^{\alpha_K} L_t^{\alpha_L}
\]

Taking logs gives:

\[
\ln Y_t = \ln A_t + \alpha_K \ln K_t + \alpha_L \ln L_t
\]

Taking first differences gives the growth-rate form:

\[
\Delta \ln Y_t = g_0 + \alpha_K \Delta \ln K_t + \alpha_L \Delta \ln L_t + \varepsilon_t
\]

where the coefficients measure how capital and labour growth contribute to GDP growth.

## 2. Historical regressions: two models tried

### Model 1: Log-log OLS in levels

I first estimated a log-log regression in levels:

\[
\ln Y_t = c + \alpha_K \ln K_t + \alpha_L \ln L_t + u_t
\]

This model fits the historical GDP path closely and has a very high adjusted \(R^2\), but because the series share strong trends, that fit can be misleading. The model also showed substantial positive serial correlation in the residuals, so I did not use it for the simulation stage.

### Model 2: Log-first-differences model

I then estimated a first-difference growth model:

\[
\Delta \ln Y_t = g_0 + \alpha_K \Delta \ln K_t + \alpha_L \Delta \ln L_t + \varepsilon_t
\]

This specification is more useful for this project because:

- it reduces the problems created by trending level series
- it improves the autocorrelation issue relative to the log-log model
- its coefficients are directly interpretable for simulations of changes in labour and capital growth

This is the model used for the remainder of the project.

## Estimated coefficients used in the potential-output model

From the log-first-difference regression:

- \(\alpha_K = 0.8764\)
- \(\alpha_L = 0.4857\)
- intercept \(g_0 = 0.0015\)

These coefficients are then used to construct and simulate potential output.

## Potential output construction

Potential output is built from smooth “best-case” factor paths rather than actual year-to-year series.

### Potential capital

Potential capital is constructed by taking the actual capital stock in the base year **1997** and growing it forward using the **average log growth rate of capital** over the sample:

\[
K_t^{pot} = K_0 \cdot e^{\bar g_K (t-t_0)}
\]

### Potential labour

Potential labour is built in two steps.

First, actual hours worked are converted into a **full-employment hours** series using the lowest unemployment rate observed in the sample, **5.5% in 2019**:

\[
L^{FE}_t = L_t \cdot \frac{1-u^*}{1-u_t}
\]

where:

- \(u^* = 0.055\)
- \(u_t\) is the actual unemployment rate in year \(t\)

Then that full-employment series is smoothed using its average log growth rate from the base year:

\[
L_t^{pot} = L^{FE}_0 \cdot e^{\bar g_{L^{FE}} (t-t_0)}
\]

This gives a labour trend consistent with operating at the sample’s best unemployment outcome rather than with cyclical slack.

### Potential TFP

For the potential-output path, TFP is not taken from regression residuals. Instead, I use the observed multifactor productivity index to estimate an average log productivity growth rate and then construct a smooth trend:

\[
A_t^{pot} = A_0 \cdot e^{\bar g_A (t-t_0)}
\]

The base-year level of potential TFP is backed out residually from the production function.

### Potential output formula

Finally, potential output is calculated as:

\[
Y_t^{pot} = A_t^{pot} \left(K_t^{pot}\right)^{\alpha_K} \left(L_t^{pot}\right)^{\alpha_L}
\]

and the output gap is:

\[
\text{Gap}_t = 100 \times \frac{Y_t - Y_t^{pot}}{Y_t^{pot}}
\]

## Scenarios

## Baseline projection, 2025–2035

The baseline extends the historical average growth trends of capital, full-employment labour, and TFP forward into the next decade.

Average annual projected potential GDP growth under the baseline:

- **2.85% per year**

## Immigration scenarios

To model immigration-related labour-force effects, I calculate the share of Ontario’s labour force accounted for by temporary residents using quarterly non-permanent resident categories aligned with labour-force data.

### Scenario 1: IRCC-style adjustment

The temporary-resident share is reduced from **6.5% to 5%** over three years, then held constant.

Average annual projected potential GDP growth:

- **2.78% per year**

### Scenario 2: Progressive tightening

The temporary-resident share continues to decline proportionally every year beyond 2024.

Average annual projected potential GDP growth:

- **2.59% per year**

These immigration scenarios mainly flatten the slope of the potential-output path rather than causing a sharp one-time collapse.

## Tariff scenarios

To approximate tariff-related trade disruption, I use the Covid-era decline and recovery of **manufacturing hours** as an empirical stand-in shock.

Manufacturing hours relative to 2019:

- **2020 / 2019 = 0.848**
- **2021 / 2019 = 0.908**
- **2022 / 2019 = 0.963**

Manufacturing’s share of total hours in 2019:

- **10.44%**

### Scenario 1: Short-term tariff shock

Covid-sized manufacturing losses are applied cumulatively from **2025 to 2028**, then partially reversed using the observed recovery pattern.

Estimated GDP effect:

- trough of about **2.5% below baseline** in **2028**
- gap narrows to about **1.0% below baseline** by **2035**

### Scenario 2: Long-term tariff shock

Covid-sized shocks are applied repeatedly from **2025 onward**, representing a persistent trade-cost burden without sufficient adjustment.

Estimated GDP effect:

- GDP reaches about **4.35% below baseline** by **2035**

## Selected results

- Historical average actual GDP growth: **2.36% per year**
- Historical average potential GDP growth: **2.45% per year**
- Baseline potential GDP growth, 2025–2035: **2.85% per year**
- Immigration restrictions reduce potential growth modestly but persistently
- Tariff-style shocks create sharper level losses, especially if they persist

## Code samples

### 1. ADF stationarity tests

```python
import pandas as pd
from statsmodels.tsa.stattools import adfuller

df = df_input.copy()
df = df.set_index('Year')
df = df[['Y', 'K', 'L']].astype(float)

for c in ['Y', 'K', 'L']:
    series = df[c].dropna()
    res = adfuller(series, regression='ct', autolag='AIC')
    print(c, res[0], res[1])
