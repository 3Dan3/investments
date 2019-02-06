Untitled
================
DanielH
February 6, 2019

-   [bonds](#bonds)
-   [zero coupon bond pricing](#zero-coupon-bond-pricing)
-   [computing zcb ytm](#computing-zcb-ytm)
-   [coupon bond pricing](#coupon-bond-pricing)
-   [coupon bond ytm](#coupon-bond-ytm)
-   [accrued interest](#accrued-interest)
-   [the yield curve and future interest rates](#the-yield-curve-and-future-interest-rates)

``` r
library(tidyverse)
library(jrvFinance)
library(lubridate)
library(RQuantLib)
library(scales)
library(magrittr)
library(knitr)
```

bonds
-----

-   premium bonds: couppn rate &gt; current yield

-   discount bonds: coupon rate &lt; curent yield

-   par bonds: coupon rate = current yield

zero coupon bond pricing
------------------------

FV: *par value or value at maturity*

i: *interest rate*

n: *number of periods*

$$bond \\ price =  \\frac{FV}{(1 + i)^{n}}$$

##### example one

Here’s an example, assuming a zero-coupon bond that matures in five years, with a face value of $1,000 and a required yield of 6%. If we do the computations in R, we have:

``` r
bond.price(settle = today(),
           mature = today() + dyears(5) + 1,
           coupon = 0,
           yield = 0.06,
           comp.freq = 2) %>%
  map_dbl(~.x * 10 %>% round(2)) %>%
  round(2) %>% 
  set_names("price") %>%
  enframe(" ",  " ") %>% 
  kable()
```

|       |        |
|:------|-------:|
| price |  744.09|

computing zcb ytm
-----------------

FV: *par value or value at maturity*

i: *interest rate*

PV: *present value or current price*

$$yield = (\\frac{FV}{PV})^{\\frac{1}{n}} - 1$$

##### example one

Now we use the same example as before, but this time we compute the bond yield with the bond price given. Using R we have:

``` r
# ytm
bond.yield(settle = today(),
           mature = today() + dyears(5) + 1,
           coupon = 0,
           price=74.40939,
           comp.freq = 2) %>%
  round(6) %>%
  percent(accuracy = .01) %>%
  set_names("yield to maturity") %>%
  enframe(" ",  " ") %>% 
  kable()
```

|                   |       |
|:------------------|:------|
| yield to maturity | 6.00% |

##### example two

As an example, assume we’re considering a zero-coupon bond that matures in two years and has a future value of $1,000. We can buy it for $925. We can calculate its current yield using R:

``` r
# ytm
bond.yield(settle = today(),
           mature = today() + dyears(2) + 1,
           coupon = 0,
           price=92.5,
           comp.freq = 2) %>%
  round(6) %>%
  percent(accuracy = .01) %>%
  set_names("yield to maturity") %>%
  enframe(" ",  " ") %>% 
  kable()
```

|                   |       |
|:------------------|:------|
| yield to maturity | 3.94% |

As we can see we assumed yearly compounding, in other words

> we assume that a zcb pays coupon = 0 twice a year, unless told otherwise

coupon bond pricing
-------------------

$$bond \\ price = \\sum\_{t=1}^T \\frac{coupon}{(1 + i)^t} + \\frac{FV}{(1 + i)^T}$$

-   FV: *par value or value at maturity*

-   i: *interest rate*

-   n: *number of periods*

-   t: *present, current time*

-   T: *maturity*

##### example one

We’ll assume the bond matures in 10 years, has a par value of $1,000, a coupon rate of 10% and a required yield of 12%.

``` r
bond.price(settle = today(),
           mature = today() + dyears(10),
           coupon = 0.1,
           yield = 0.12) %>%
  map_dbl(~. * 10) %>%
  round(2) %>%
  set_names("price") %>%
  enframe(" ",  " ") %>% 
  kable()
```

|       |        |
|:------|-------:|
| price |  885.33|

##### example two

8% coupon, 30-year maturity bond with par value of $1,000 paying 60 semiannual coupon payments of $40 each. Suppose that the interest rate is 8% annually, or 4% per 6-month period.

``` r
bond.price(settle = today(),
           mature = today() + dyears(30) + 8,
           coupon = 0.08,
           yield = 0.08,
           comp.freq = 2) %>%
  map_dbl(~. * 10) %>%
  round(3) %>%
  set_names("price") %>%
  enframe(" ",  " ") %>% 
  kable()
```

|       |      |
|:------|-----:|
| price |  1000|

coupon bond ytm
---------------

Suppose an 8% coupon, 30-year bond is selling at $1,276.76. What average rate of return would be earned by an investor purchasing the bond at this price?

``` r
bond.yield(settle = today(),
           mature = today() + dyears(30) + 1,
           coupon = 0.08,
           price=127.676) %>%
  round(6) %>%
  percent(accuracy = .01) %>%
  set_names("yield to maturity or APR") %>%
  enframe(" ",  " ") %>% 
  kable()
```

|                          |       |
|:-------------------------|:------|
| yield to maturity or APR | 6.00% |

The ytm 6.00% is the annualized semiannual yield, the **annual percentage rate** (APR), also called **bond equivalent**.

However, the **effective annual rate** accounts for **compound interest**. If one earns 3% interest every 6 months, then after 1 year, each dollar invested grows with interest to (1 + 0.03)^2 = 1.0609. EAR: 6.09%

``` r
bond.yield(settle = today(),
           mature = today() + dyears(30) + 1,
           coupon = 0.08,
           price=127.676,
           comp.freq = 1,
           freq = 2) %>%
  round(6) %>%
  percent(accuracy = .01) %>%
  set_names("effective annual rate") %>%
  enframe(" ",  " ") %>% 
  kable()
```

|                       |       |
|:----------------------|:------|
| effective annual rate | 6.09% |

accrued interest
----------------

> The fraction of the coupon payment that the bond seller earns for holding the bond between payments

Since many bonds on the secondary market trade in between coupon payment dates, the bond seller has to be compensated for the portion of the coupon payment it earned for holding the bond since the last payment. This basically gives both the seller and buyer a pro-rated coupon payment for that period.

$$accrued\\ interest = coupon\\times\\frac{day\\ count}{total\\ days}$$

the yield curve and future interest rates
-----------------------------------------

> the yield curve is a plot of yield to maturity as a function of time to maturity

**Stripped Treasuries**

They are zero-coupon bonds created by selling each coupon or principal payment from a whole Treasury bond as a separate cash flow.

**law of one price**

#### forward rates formula

$$(f\_{n+1}) = \\frac{(1 + y\_{n+1})^{n+1}}{(1 + y\_{n})^n} - 1$$
