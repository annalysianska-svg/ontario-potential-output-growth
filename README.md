# Ontario Potential Output Growth

## Overview
This project builds a regression on historical economic data to estimate Ontario’s potential output over the year and its potential growth in 2025-2035 using a Cobb-Douglas production function. Additionally, there is scenario analysis on immigrant labour and tariff impacts, with Covid-19 as a stand-in for manufacturing hours reduction. I used Python for data analysis and visualisations, as well as Excel for basic data cleaning and preparation.

## Research question
What is the potential growth in GDP for Ontario, and how might it change under alternative immigration and trade disruption scenarios?

## Data

The historical model uses annual Ontario data from **1997 to 2023**, with dollar variables expressed in **2017 chained dollars** to account for inflation and depreciation. Below are the table sources of the data. However, the data section of the file also includes organized data tables that I cleaned in Excel.

- **Real GDP (Y)**  
  Statistics Canada Table [**36-10-0222-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3610022201)  
  *Gross domestic product, expenditure-based, provincial and territorial, annual (x 1,000,000)*

- **Capital stock (K)**  
  Statistics Canada Table [**36-10-0096-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3610009601)  
  *Flows and stocks of fixed non-residential capital, by industry and type of asset, Canada, provinces and territories (x 1,000,000)*

- **Labour input (L)**  
  Statistics Canada Table [**36-10-0480-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3610048001)  
  *Labour productivity and related measures by business sector, industry, and by non-commercial activity consistent with the industry accounts*  
  I use hours worked for all jobs as the main labour proxy. The manufacturing hours profile over 2019-2022 (Covid-19) is used as an empirical template for a trade shock.

- **Multifactor productivity index (A for potential-output construction)**  
  Statistics Canada Table [**36-10-0208-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3610020801)  
  *Multifactor productivity, value-added, capital input and labour input in the aggregate business sector and major sub-sectors, by industry*  
  Total Factor Productivity has been obtained from the table to estimate the potential output. However, the historical regression does not use this measure, but rather considers TFP as a Solow’s residual, the error term.
    
- **Unemployment rate**  
  Statistics Canada Table [**14-10-0023-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=1410002301)  
  *Labour force characteristics by industry, annual (x 1,000)*
  
- **Immigrant share of the labour force**     
  Statistics Canada Table [**17-10-0121-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=1710012101) and [**14-10-0022-01**](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=1410002201)  
  *Estimates of the number of non-permanent residents by type, quarterly* is used to capture temporary workers with work permits. *Labour force characteristics by industry, monthly, unadjusted for seasonality* is used for the labour force measure at a monthly frequency, aligned with the quarterly immigrant-labour data to compute immigrants’ contribution to the labour force. I believe it is reasonable to treat all work permit holders as part of the labour-force pool, since holding (or applying for) a work permit signals an intention to participate in the labour market and actively seek employment. 

## Methodology

## 1. Production function framework

The project is based on a Cobb-Douglas production function:

$$Y_t = A_t K_t^{\alpha} L_t^{(1-\alpha)}$$

Taking logs gives:

$$ln Y_t = \ln A_t + \alpha \ln K_t + \(1-\alpha) \ln L_t$$

Taking first differences gives the growth rate in percentage form (i.e. percentage change):

$$\\%\Delta \ln Y_t = \\% \Delta \ln A_t + \alpha \\% \Delta \ln K_t + (1-\alpha) \\% \Delta \ln L_t$$

where the coefficients measure how capital and labour growth contribute to GDP growth. A is productivity, which takes the form of residual in regressions.

## 2. Historical regressions: two models tried

### Model 1: Log-log OLS in levels

I first estimated a log-log regression in levels:

$$
\hat{Y}_{\text{bil}} = -321.393 + 0.8355K_{\text{bil}} + 0.0483L_{\text{mil}}
$$

where:

- $\hat{Y}_{\text{bil}}$ = predicted Ontario real GDP, in billions of dollars  
- $K_{\text{bil}}\$ = capital stock, in billions of dollars  
- $L_{\text{mil}}\$ = labour input, in millions of hours  
- $\beta_0 = -321.393$ is the intercept and a residual A in this case 

> An adjusted R-squared is 0.994, which is extremely high but not unusual for regressions using historical GDP series with strong common trends. The estimated output elasticities of capital and labour are both below one, implying decreasing returns to scale in the Cobb-Douglas production function, while total factor productivity A is the intercept and is therefore treated as a constant factor. However, the Durbin-Watson statistic is 0.616 (see the output), well below the benchmark value of 2, indicating substantial positive serial correlation in the residuals.

Meaning, the tight fit of the regression may be explained by common trends among the series rather than relationships, so an alternative model is regressed.

### Model 2: Log-first-differences (i.e. percentage change)

I then estimated a first-difference growth model, linking annual percentage changes in real output to percentage changes in capital and labour:

$$
\\%\Delta \widehat{Y}_t = 0.1486 + 0.8764 \\%\Delta K_t + 0.4857 \\%\Delta L_t
$$

where:

- $\\%\Delta \widehat{Y}_t$ = predicted percentage change in output
- $\\%\Delta K_t$ = percentage change in capital
- $\\%\Delta L_t$ = percentage change in labour
- $\beta_0 = 0.1486$ is the intercept and a residual A

> For example, holding labour growth constant, a 1% increase in capital growth is associated with approximately a 0.876% increase in GDP growth.
> The Durbin-Watson statistic for this model is 1.228, which represents a substantial improvement relative to the log-log regression, although the adjusted R-squared falls to 0.707, which remains statistically and economically meaningful. The model captures the main pattern of accelerations and slowdowns in GDP growth, even if it does not track individual observations as tightly as the levels regression. Moreover, the fit improves toward the end of the sample.

For these reasons, the log-first-differences model is used to obtain the capital and labour coefficients that are applied in the growth-rate simulations and then mapped back into the level series.

## 3. Potential output construction

A baseline estimate of Ontario’s potential output is constructed and used to assess how far the economy has operated above or below capacity over time and how future shocks may affect that path. 

### Potential Capital K

Potential capital K is constructed by taking the actual Ontario non-residential capital stock in the base year 1997 and then growing it forward at the average log growth rate of capital observed over the full sample period.

$$K_t^{pot} = K_0 \cdot e^{\bar g_K (t-t_0)}\$$

So K follows a smooth exponential trend rather than the year-to-year fluctuations in the data.

### Potential Labour L

Potential labour is built in two steps:

First, actual hours worked are converted into a full-employment hours series using the lowest unemployment rate observed in the sample, **5.5% in 2019**. 

$$L^{FE}_t = L_t \cdot \frac{1-u^*}{1-u_t}\$$

where:

- $u^* = 0.055\$ i.e. 5.5%
- $u_t\$ is the actual unemployment rate in a given year

Then that full-employment series is smoothed using its average log growth rate from the base year:

$$L_t^{pot} = L^{FE}_0 \cdot e^{\bar g_{L^{FE}} (t-t_0)}$$

>**Why 5.5%?** My reasoning was that if natural unemployment is always present in the economy (frictional, structural, and seasonal), then the lowest unemployment rate observed in the historical data should include that stable natural unemployment, while minimizing cyclical unemployment. Thus, giving the "best case scenario" while keeping it realistic to the economy's natural state.

### Potential TFP, i.e. A

TFP is not taken from regression residuals. Instead, I use the multifactor productivity index (from Statistics Canada) to estimate an average log productivity growth rate and then construct a smooth trend:

$$A_t^{pot} = A_0 \cdot e^{\bar g_A (t-t_0)}$$

The approach is similar to that of potential capital. However, the initial level of A in the base year is calculated as a residual of Y, that is, left after deducting K and L.

$$
\ln A_0^{pot} = \ln Y_0 - 0.8764 \ln K_0^{pot} - 0.4857 \ln L_0^{pot}
$$

### Potential output formula

Finally, potential output in Cobb-Douglas form is calculated as:

$$
Y_t^{pot} = A_t^{pot}\left(K_t^{pot}\right)^{0.8764}\left(L_t^{pot}\right)^{0.4857}
$$

and the output gap is the standard percentage deviation formula:

$$\text{Gap}_t = 100 \times \frac{Y_t - Y_t^{pot}}{Y_t^{pot}}$$

## 4. Scenario analysis
## Baseline projection and scenario simulations

This section projects Ontario’s potential output from **2025 to 2035** using the estimated Cobb-Douglas production function and then simulates how immigration restrictions and tariff-related trade disruptions may alter that path.

### Estimated production function

The historical elasticities are estimated from the log-first-difference regression:

$$
\Delta \ln Y_t = g_0 + \alpha_K \Delta \ln K_t + \alpha_L \Delta \ln L_t
$$

with estimated coefficients:

- $\alpha_K = 0.8764$
- $\alpha_L = 0.4857$

These coefficients are then used to project output under the baseline and under the alternative scenarios.

## 1. Baseline projection

The baseline projection extends the latest observed values of productivity, capital, and labour forward from **2023 to 2035** using their **average pre-2020 log growth rates**, specifically over **2010 to 2019**. In the code, these average growth rates are calculated as:

$$
g_A = \overline{\Delta \ln A_t}, \qquad
g_K = \overline{\Delta \ln K_t}, \qquad
g_L = \overline{\Delta \ln L_t}
\quad \text{for } t \in [2010,2019]
$$

Then, starting from the last observed year, each factor is projected forward recursively:

$$
A_t^{base} = A_{t-1}^{base} e^{g_A}
$$

$$
K_t^{base} = K_{t-1}^{base} e^{g_K}
$$

$$
L_t^{base} = L_{t-1}^{base} e^{g_L}
$$

Baseline output is then calculated using the Cobb-Douglas function:

$$
Y_t^{base} = A_t^{base}\left(K_t^{base}\right)^{0.8764}\left(L_t^{base}\right)^{0.4857}
$$

### Baseline result

Under this projection, the average annual potential GDP growth rate over **2025 to 2035** is:

- **Baseline: 2.85% per year**

This serves as the reference path against which the immigration and tariff simulations are compared.


## 2. Immigration scenarios

The immigration scenarios begin with the baseline and then adjust only the labour input. The starting point is the share of Ontario’s labour force associated with selected temporary-resident categories in **2023Q4**:

$$
\text{imm\_share\_base}=\frac{L^{imm}_{2023}}{LF_{2023}}
$$

The policy target is based on reducing the temporary-resident ratio from **6.5%** to **5.0%** over three years. The proportional target factor is:

$$
\frac{5.0}{6.5}=0.7692
$$

To spread this reduction evenly in proportional terms over three years, the code calculates the annual log reduction rate as:

$$
g_{\text{share}}=\frac{\ln(5.0/6.5)}{3}\approx -0.0874
$$

which implies an annual multiplier of:

$$
m=e^{g_{\text{share}}}\approx 0.9163
$$

So during the adjustment phase, the temporary-resident share is multiplied by approximately **0.9163 each year**, meaning it falls by about **8.37% relative to the previous year**.

### IRCC-style scenario

In the IRCC-style scenario, this annual multiplier is applied from **2025 to 2027**, and then the share is held constant at the new lower level from **2028 through 2035**.

The share multiplier is:

$$
\text{share\_multiplier}_{ircc,t}=
\begin{cases}
1, & t \leq 2024 \\
m^{\,t-2024}, & 2025 \leq t \leq 2027 \\
m^3 = \frac{5.0}{6.5}=0.7692, & 2028 \leq t \leq 2035
\end{cases}
$$

This means the reduction stops after 2027. In simple terms, if the ratio starts at **6.5**, then it moves approximately like this:

- 2024: 6.5
- 2025: $6.5 \times 0.9163 \approx 5.96$
- 2026: $5.96 \times 0.9163 \approx 5.46$
- 2027: $5.46 \times 0.9163 \approx 5.00$
- 2028 to 2035: remains at **5.00**

This share adjustment is then converted into a labour multiplier:

$$
\text{labour\_multiplier}_{ircc,t}
=
1-\text{imm\_share\_base}\left(1-\text{share\_multiplier}_{ircc,t}\right)
$$

and applied to baseline labour:

$$
L_t^{ircc}=L_t^{base}\cdot \text{labour\_multiplier}_{ircc,t}
$$

Output under the IRCC-style scenario is then:

$$
Y_t^{ircc}=A_t^{base}\left(K_t^{base}\right)^{0.8764}\left(L_t^{ircc}\right)^{0.4857}
$$

Under this scenario, the average annual potential GDP growth rate over **2025 to 2035** is:

- **IRCC-style scenario: 2.78% per year**

### Progressive scenario

In the progressive scenario, the **same annual multiplier** continues to be applied every year from **2025 all the way to 2035**. Unlike the IRCC-style case, the reduction does not stop after 2027.

The share multiplier is therefore:

$$
\text{share\_multiplier}_{prog,t}=
\begin{cases}
1, & t \leq 2024 \\
m^{\,t-2024}, & 2025 \leq t \leq 2035
\end{cases}
$$

So the ratio keeps shrinking throughout the full horizon. Starting from **6.5** in 2024, it moves approximately like this:

- 2024: 6.50
- 2025: $6.50 \times 0.9163 \approx 5.96$
- 2026: $5.96 \times 0.9163 \approx 5.46$
- 2027: $5.46 \times 0.9163 \approx 5.00$
- 2028: $5.00 \times 0.9163 \approx 4.58$
- 2029: $4.58 \times 0.9163 \approx 4.20$
- 2030: $4.20 \times 0.9163 \approx 3.85$
- 2031: $3.85 \times 0.9163 \approx 3.53$
- 2032: $3.53 \times 0.9163 \approx 3.24$
- 2033: $3.24 \times 0.9163 \approx 2.97$
- 2034: $2.97 \times 0.9163 \approx 2.72$
- 2035: $2.72 \times 0.9163 \approx 2.49$

Thus, by **2035**, the implied temporary-resident ratio falls to roughly **2.49**, compared with **5.00** in the IRCC-style scenario.

This is again translated into a labour multiplier:

$$
\text{labour\_multiplier}_{prog,t}
=
1-\text{imm\_share\_base}\left(1-\text{share\_multiplier}_{prog,t}\right)
$$

and applied to labour:

$$
L_t^{prog}=L_t^{base}\cdot \text{labour\_multiplier}_{prog,t}
$$

Output under the progressive scenario is then:

$$
Y_t^{prog}=A_t^{base}\left(K_t^{base}\right)^{0.8764}\left(L_t^{prog}\right)^{0.4857}
$$

Under this scenario, the average annual potential GDP growth rate over **2025 to 2035** is:

- **Progressive scenario: 2.59% per year**

### Immigration scenario gaps relative to baseline

The percentage deviation from the baseline is calculated as a standard percentage deviation:

$$
\text{gap}_{ircc,t}=100\times\frac{Y_t^{ircc}-Y_t^{base}}{Y_t^{base}}
$$

$$
\text{gap}_{prog,t}=100\times\frac{Y_t^{prog}-Y_t^{base}}{Y_t^{base}}
$$

### Interpretation

The key difference between the two immigration scenarios is that the **IRCC-style scenario stops tightening after 2027**, while the **progressive scenario keeps applying the same annual reduction multiplier through 2035**. As a result, the IRCC-style scenario creates a one-time downward shift in labour supply growth, whereas the progressive scenario creates a continuously compounding drag on labour input and therefore a larger long-run slowdown in projected output growth.

## 3. Tariff scenarios

The tariff scenarios also begin from the baseline, but instead of changing immigration-related labour supply, they impose a trade-disruption shock using the **Covid-19 collapse and recovery of manufacturing hours** as an empirical stand-in for a major tariff shock.

The logic is that pandemic-era border closures and logistical disruptions effectively raised trade costs and restricted the movement of goods, especially in manufacturing. This makes the Covid manufacturing-hours profile a useful benchmark for simulating how renewed tariffs might affect Ontario’s production structure.

From the data:

- manufacturing hours in **2020** fall to **84.8%** of their **2019** level
- manufacturing hours in **2021** recover to **90.8%** of their **2019** level
- manufacturing hours in **2022** recover to **96.3%** of their **2019** level

So the key observed shock and recovery factors are:

$$
\frac{H_{2020}^{mfg}}{H_{2019}^{mfg}}=0.848
$$

$$
\frac{H_{2021}^{mfg}}{H_{2019}^{mfg}}=0.908
$$

$$
\frac{H_{2022}^{mfg}}{H_{2019}^{mfg}}=0.963
$$

This implies an average annual recovery rate of about:

- **6.6% per year** between 2020 and 2022

Manufacturing accounts for approximately:

- **10.4% of total hours in 2019**

These observed manufacturing shocks are then mapped into the baseline labour path to simulate how a severe tariff shock would feed through to Ontario output.

### 3.1 Short-term tariff scenario

In the **short-term tariff scenario**, Covid-sized manufacturing losses are applied **cumulatively from 2025 to 2028**, interpreted in the paper as the period of a **tariff-intensive U.S. administration ending in 2028 with the presidential elections**.

After 2028, the shock is partially reversed using the observed Covid-era recovery rate. In other words:

- 2025 to 2028: manufacturing losses accumulate
- after 2028: manufacturing hours recover gradually

This creates a temporary but meaningful drag on output.

#### Short-term tariff result

Under this scenario:

- Ontario GDP reaches a trough of about **2.5% below baseline in 2028**
- the gap then narrows to roughly **1.0% below baseline by 2035**

So this scenario represents a **temporary but significant tariff shock**, followed by gradual adjustment and recovery.

### 3.2 Long-term tariff scenario

In the **long-term tariff scenario**, the Covid-sized manufacturing shock is applied **every year from 2025 onward**, so the manufacturing factor continues to decline instead of recovering.

This means the labour effect keeps worsening rather than stabilizing. By the end of the projection horizon:

- the aggregate labour factor is about **9% below baseline by 2035**
- GDP is around **4.3% below baseline by 2035**

This is a deliberately severe reference case. It does not assume that firms successfully diversify suppliers or redirect trade flows in response to the shock.

### Tariff scenario gaps relative to baseline

The percentage gap for the tariff scenarios is likewise interpreted as a standard percentage deviation from the no-tariff baseline:

$$
\text{gap}_{tariff,t}=100\times\frac{Y_t^{tariff}-Y_t^{base}}{Y_t^{base}}
$$

---

## 4. Summary of results

### Average annual growth rates, 2025 to 2035

- **Baseline:** 2.85%
- **IRCC-style scenario:** 2.78%
- **Progressive scenario:** 2.59%

### Tariff scenario results

- **Short-term tariff scenario:** GDP trough about **2.5% below baseline in 2028**, narrowing to about **1.0% below baseline by 2035**
- **Long-term tariff scenario:** GDP about **4.3% below baseline by 2035**, with aggregate labour input about **9% below baseline**

### Interpretation

The immigration scenarios mainly flatten the slope of Ontario’s potential-output path by reducing labour-force growth. The IRCC-style case produces a modest slowdown because the temporary-resident share declines only once and then stabilizes. The progressive case produces a larger slowdown because the same proportional reduction continues every year.

The tariff scenarios, by contrast, generate sharper level effects. Because they are concentrated in manufacturing and other trade-exposed sectors, they push output below the no-tariff baseline more abruptly. The short-term tariff scenario allows for recovery after the 2028 U.S. presidential election cycle, while the long-term scenario assumes that trade barriers remain in place and continue to drag on output throughout the full horizon.
