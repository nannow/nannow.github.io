---
layout: post
title: 생물통계학 in R - (1)
comments: false
use_math: true
categories: [Statistics]
tags: [Statistics]
---


![biostatisticswordle](https://user-images.githubusercontent.com/17719651/52768499-07b12b80-3071-11e9-8b9b-1c451f8108e1.jpg)

- <a href='#intro'>Project Introduction</a>
- <a href='#setup'>Setting up the Environment</a>
- <a href='#probthnk'>Probabilistic Thinking</a>
  - <a href='#exp'>Experiments</a>
  - <a href='#prob'>Probability</a>
  - <a href='#rv'>Random Variables</a>
  - <a href='#chardist'>Main Characteristics of Distribution</a>
  - <a href='#condprob'>Conditional Probability</a>
  - <a href='#bayes'>Bayes Rule</a>
  - <a href='#roc'>ROC and AUC</a>
- <a href='#stroke'>Healthcare Dataset Stroke Data</a>
  - <a href='#eda'>EDA (Exploratory Data Analysis)</a>
  - <a href='#outcome'>Test Outcome Analysis</a>

<br><br>

## <a id='intro'><font color="blue">Project Introduction</font></a>
<br>
**생물통계학 이란?**

>*생물통계학(biometrics statistics, 生物統計學)은 생물학 중심에서 사용되는 통계적 방법과 이론을 다루는 통계학의 한 영역으로 생물학적 현상과 관련된 통계를 취급하며 생물측정을 포함된 학문이다.

<br>
## <a id='setup'><font color="blue">Setting up the environment</font></a>

```{r}
suppressMessages(library(tidyverse))
suppressMessages(library(repr))
suppressMessages(library(randomForest))
suppressMessages(library(caret))
suppressMessages(library(cowplot))
suppressMessages(library(Metrics))
suppressMessages(library(AUC))

set.seed(123)
options(repr.plot.width=7, repr.plot.height=3)
```

<br>
## <a id='probthnk'><font color="blue">Probabilistic Thinking</font></a>

### <a id='exp'>Experiments</a>

* **표본 공간** $\Omega$ : 실험의 가능한 결과물 집합
* **사건** $E$ : 표본공간 $\Omega$의 부분집합
* **단순 사건** $\omega$ : 실험에서 하나의 특정 결과
* $\emptyset$ : 공집합
* $\omega \in E$ : $\omega$가 발생했을 때 $E$가 발생
* $\omega \notin E$ : $\omega$가 발생했을 때 $E$가 발생하지 않음
* $E \subset F$ : $E$ 가 $F$의 부분집합
* $E \cap F$ : $E$ 와 $F$ 동시에 발생하는 사건 **R function: `E & F`**
* $E \cup F$ : $E$ 또는 $F$ 가 발생하는 사건 **R function: `E | F`**
* $E^C$ or $\bar{E}$ : $E$ 여사건 (사건 $E$ 발생하지 않음) **R function: `!E`**
* $E$ \ $F=E \cap F^C$ : $E$ 는 발생하고 $F$ 는 발생하지 않는 사건

**R을 이용하여 동전던지기 예제를 풀어보자.**

지금부터 평범한 동전을 가지고 있고 앞면이 나올 확률이 0.5인 이항분포를 따른다고 가정하자.

이항분포 정의 :
>A binomial experiment is a statistical experiment that has the following properties: The experiment consists of n repeated trials. Each trial can result in just two possible outcomes. We call one of these outcomes a success and the other, a failure. The probability of success, denoted by P, is the same on every trial.

```{r}
#동전 10개에 대한 2개의 표본공간을 만들고 각각을 한번씩 던진다. 사건 E = 1 은 앞면이 나올 확률 0.5를 가지고 있다.

Exp_A  <- rbinom(10, 1, .5)
Exp_B  <- rbinom(10, 1, .5)

print(paste0("표본공간 A: ", list(Exp_A)))
print(paste0("표본공간 B: ", list(Exp_B)))
cat("\n")
print(paste0("표본 A에 대한 E의 여사건 (뒷면): ", list(!Exp_A)))
print(paste0("표본 B에 대한 E의 여사건 (뒷면): ", list(!Exp_B)))
cat("\n")
print(paste0("표본 A와 B에서 전부 앞면이 나오는 경우: ",
list(Exp_A & Exp_B)))
print(paste0("표본 A 또는 B에서 앞면이 나오는 경우: ",
list(Exp_A | Exp_B)))
cat("\n")
print(paste0("표본 A에서 앞면이 나오고 표본 B에서 뒷면이 나오는 경우: ",
list(Exp_A & !Exp_B)))
```

<br>
### <a id='prob'>Probability</a>

**Probability** 은 사건이 발생할 가능성이다.
```{r}
print(paste0("A 와 B 모두 앞면이 나올 확률 : ",
round(mean(Exp_A & Exp_B), 3)))
print(paste0("A 또는 B 가 앞면이 나올 확률 : ",
round(mean(Exp_A | Exp_B), 3)))
print(paste0("A 가 앞면이 나오고 B가 뒷면이 나올 확률 : ",
round(mean(Exp_A & !(Exp_B)), 3)))
```
<br>
### <a id='rv'>확률 변수 (random variable)</a>

**확률 변수** 는 무작위 실험의 결과를 나타내는 수치이다. 여기서는 *이산적* 이거나 *연속적* 확률 변수에 대해 다룰 것이다.
* **이산 확률 변수 (discrete random variable)** 는 확률 변수 X가 취할 수 있는 모든 값을 x1, x2, x3, ... 처럼 셀 수 있을 때 X를 이산 확률 변수라고 한다.
* **연속 확률 변수 (continuous random variable)** 는 적절한 구간 내의 모든 값을 취하는 확률 변수이다.
<br><br>
#### 확률 질량 함수 (probability mass function, pmf)

> **확률 질량 함수 (pmf)** 는 이산 확률 변수 X가 취할 수 있는 값 x1, x2, x3, ... 의 각각에 대해서 확률 P(X=x1), P(X=x2), P(X=x3), ... 을 대응시켜주는 관계이다.

동전던지기의 pmf : $p(x)=\theta^x(1 - \theta)^{1-x}$,

$\theta$ 은 1 (앞면) 이 나올 확률이고, $x$ 는 1(앞면) 또는 0 (뒷면) 을 입력으로 받는다.

동전던지기의 pmf를 다시써보면 다음과 같다. $p(x)=(\frac{1}{2})^x(\frac{1}{2})^{1-x}$.

유효한 pmf를 갖기 위해서, 함수 $p(x)$ 는 반드시 다음을 만족해야 한다.

* $p(k) \geqslant 0$ for all $k \in K$, where $K$ is the set of all possible outcomes for $X$;
* $\sum_{k\in{K}} p(k) = 1$.
<br><br>
#### 확률 밀도 함수 (probability density function, pdf)

> **Probability Density Function** is a function of a **continuous** random variable, whose integral across an interval gives the probability that the value of the variable lies within the same interval.

$f(x)=\frac{x^4}{3}$, for $x \in (0;2)$ 라는 분포가 있다고 가정하자.

유효한 pdf를 갖기 위해서, 함수 $f(x)$ 는 반드시 다음을 만족해야 한다.

* $f(x) \geqslant 0$ for all $x$;
* $\int\limits_{-\infty}^\infty f(x) \,dx = 1$.
<br><br>
#### 독립 사건

$P(E \cap F)=P(E)P(F)$ 를 만족하는 $E$ 와 $F$ 는 서로 독립이다.
<br><br><br>
### <a id='chardist'>분포의 주요 특성</a>

#### 기댓값

**평균** (or expected value) 은 어떤 확률 과정을 무한히 반복했을 때, 얻을 수 있는 값의 평균으로서 기대할 수 있는 값. 보다 엄밀하게 정의하면 기댓값은 확률 과정에서 얻을 수 있는 모든 값의 가중 평균이다.

* **이산** 확률 변수의 PMF $p(x)$ 가 있을 때 기댓값 $E[X]=\sum_x xp(x)$ 이다.
* **연속** 확률 변수의 PDF $f(x)$ 가 있을 때  $E[X]=\int\limits_{-\infty}^{\infty} xf(x)\,dx$.

모평균과 표본평균의 차이점은 아래 설명에서 확인해보자.<br>

population mean : 모평균 <br>
sample mean : 표본 평균

You could hear terms like *population mean* and *sample mean*. What's the difference? For example, you want to check duration of time from first exposure to HIV infection to AIDS diagnosis, so your *population* is going to be *every* human who had exposure to HIV. In real life it is never possible to gather information about population, but what you could is gather information about some people (*sample*) and then make assumptions about population based on information about the sample.
<br><br>
![target-population](https://user-images.githubusercontent.com/17719651/52778397-494dd080-3089-11e9-9fb2-c9081de4e1c3.jpg)
<br><br>
* **표본 평균** 다음과 같은 식으로 정의된다. $\bar{X}=\frac{1}{n}\sum_{1}^{n}X_i$
<br><br>
#### 분산 (variance)

**분산** 은 어떤 확률변수가 기댓값으로부터 얼마나 떨어진 곳에 분포하는지를 가늠하는 숫자이다.

* $X$ 가 **이산 확률 변수** 이고 pmf 가 $p(x) = P(X = x)$ 를 따른다면 :  
\begin{align}
Var(X)=\sum_{x}(x-\mu)^2p(x)
\end{align}

$\mu$ 는 평균이다.
* $X$ 가 **연속 확률 변수** 이고 pdf 가 $f(x)$ 를 따른다면 :
\begin{align}
Var(X)=\int\limits_{-\infty}^\infty (x-\mu)^2f(x) \,dx
\end{align}
* 분산 $\sigma^2$ 을 따르는 확률 변수들 $X_i$, $i\in[1;n]$ 의 **표본 분산** 은 다음으로 정의된다 :
\begin{align}
s^2=\frac{\sum_{i=1}^{n}(X_i-\bar{X})^2}{n-1}
\end{align}
n이 아니라 n-1로 나눈다는 것을 주의하자. 값에 큰 영향을 주지는 않지만 통계적인 관습이라고 생각하면 된다.
<br><br>
#### 표준 편차 (standard deviation)

**표준 편차** 은 분산의 제곱근으로 정의되고 평균으로부터 데이터 포인트의 변화에 대해서 알려준다. 표준 편차가 작을수록 평균값에서 변량들의 거리가 가깝다. 표준 편차는 데이터 분포와 같은 단위를 가지고 있기 때문에 분산보다 해석하기가 용이하다.
\begin{align}
sd(X)=\sqrt{Var(X)}
\end{align}

**표본 표준 편차:** $s=\sqrt{s^2}$
<br><br>
#### 중앙값 (median)

**중앙값** 은 **정렬**했을 때 가장 중앙에 위치하는 값을 의미한다.

For example we have dataset {1, 4, 5, 6, 1, 1, 5, 7, 120}. First we have to sort it ad then find the middle value {1, 1, 1, 4, **5**, 5, 6, 7, 120}. So the median is **5**.

왜 평균보다 중앙값을 사용하는 것이 더 나을때가 있을까? 평균값은 *이상값* (outlier) 에 취약하다. 예를들면 데이터셋 {1, 4, 5, 6, 1, 1, 5, 7, 120} 의 평균은 **16.67** 이다. 대다수 데이터들의 값은 평균값보다 많이 작음에도 불구하고 120짜리 데이터 때문에 평균값이 크게 증가하였다. 이런 경우에는 평균값보다 중앙값을 사용하는 것이 데이터 해석에 더 용이할 것이다.
<br><br>
#### 공분산 및 상관 (Covariance and Correlation)

**공분산** 은 2개의 확률 변수의 상관정도를 나타내는 값이다. 확률 변수 X의 증감에 따른 확률 변수 Y의 증감의 경향에 대한 측도이다.
\begin{align}
Cov(X_1,X_2)=E(X_1 X_2) - E(X_1)E(X_2)
\end{align}
**표본 공분산** (estimator of the true covariance):
\begin{align}
\hat{Cov}(X_1, X_2) = \frac{1}{n}\sum_{i}^{n}(X_{1i} - \bar{X_1})(X_{2i} - \bar{X_2})
\end{align}

공분산은 해석하기가 어렵다. 예를 들면, $Cov(X,Y)=200$ 은 두 변수 X와 Y 사이의 관계가 있다는 것을 말해주지만 어느정도의 강한 관계가 있는지는 말해주지 않는다. 이럴 경우에 **상관(Correlation)** 이 효율적이다.

상관의 정의를 보자.
\begin{align}
Cor(X_1, X_2) = \frac{\hat{Cov}(X_1, X_2)}{\sqrt{Var(X_1)Var(X_2)}} = \frac{\hat{Cov}(X_1, X_2)}{sd(X_1)sd(X_2)}
\end{align}

\begin{align}
-1 \leqslant Cor(X_1, X_2) \leqslant 1
\end{align}
<br>
* $Cor(X_1, X_2)=0$ - $X_1$ 과 $X_2$ 는 상관 관계가 없다.
* $Cor(X_1, X_2)=1$ - $X_1$ 과 $X_2$ 사이에 강한 양의 상관 관계가 있다. $X_1$ 이 감소할 때 $X_2$ 도 **감소** 한다.
* $Cor(X_1, X_2)=-1$ - $X_1$ and $X_2$ 사이에 강한 음의 상관 관계가 있다. $X_1$ 이 증가할 때 $X_2$ 는 **감소** 한다.
<br>

***
<br>
**예제를 살펴보자.**

**The duration of time from first exposure to HIV infection to AIDS diagnosis is called the *incubation period*. The incubation periods of a random sample of 14 HIV infected individuals is given (in years).**

* 표본 평균, 표본 중앙값, 표본 표준 편차를 계산해보자.

```{r}
inc_period <- c(12.0, 10.5, 5.2, 9.5, 6.3, 13.1, 13.5, 12.5, 10.7, 7.2, 14.9, 6.5, 8.1, 7.9)

print(paste0("Sample mean: ", round(mean(inc_period), 3)))
print(paste0("Sample median: ", median(inc_period)))
print(paste0("Sample standard deviation: ", round(sd(inc_period), 3)))
```

결과값을 뭐라고 해석할 수 있을까?<br>

표본의 평균 잠복기는 10 년이고 평균 기간은 그 평균으로부터 평균 3년차이가 난다.

***
**Serum cholesterol (mmol/L) measured on a sample of 86 stroke patients (data of Markus et al., 1995). Calculate the distribution characteristics and plot the distribution.**

```{r}
cholesterol <- c(3.7,4.8,5.4,5.6,6.1,6.4,7.0,7.6,8.7,3.8,4.9,5.4,5.6,6.1,6.5,7.0,7.6,8.9,3.8,4.9,5.5,5.7,6.1,
                 6.5,7.1,7.6,9.3,4.4,4.9,5.5,5.7,6.2,6.6,7.1,7.7,9.5,4.5,5.0,5.5,5.7,6.3,6.7,7.2,7.8,10.2,4.5,
                 5.1,5.6,5.8,6.3,6.7,7.3,7.8,10.4,4.5,5.1,5.6,5.8,6.4,6.8,7.4,7.8,4.7,5.2,5.6,5.9,6.4,6.8,7.4,
                 8.2,4.7,5.3,5.6,6.0,6.4,7.0,7.5,8.3,4.8,5.3,5.6,6.1,6.4,7.0,7.5,8.6)

print(paste0("Sample mean: ", round(mean(cholesterol), 3)))
print(paste0("Sample median: ", median(cholesterol)))
print(paste0("Sample variance: ", round(var(cholesterol), 3)))
print(paste0("Sample standard deviation: ", round(sd(cholesterol), 3)))
```
<br>
표본에서 평균 세럼 콜레스테롤 레벨은 6.3 이고 레벨들은 평균으로부터 평균 1.4년 차이가 난다.<br><br>


**이제 시각화를 해보자**

**Histogram vs Boxplot**


* **Histogram** : 데이터 그룹의 빈도를 보여주며 데이터 셋의 가장 빈번한 값이 어디인지를 알 수 있다. 데이터가 어느쪽으로 기울어져 있는지도 확인할 수 있다.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/1/1d/Example_histogram.png/220px-Example_histogram.png)

* **Boxplot** : 데이터의 .25, .5, .75 분위수, 최소값 및 최대 값을 보여 주며, 특이점 검출에 편리하다.

![](https://pro.arcgis.com/en/pro-app/help/analysis/geoprocessing/charts/GUID-0E2C3730-C535-40CD-8152-80D794A996A7-web.png)

```{r}
df <- data.frame(cholesterol)
ggplot(df, aes(x = cholesterol)) +
    geom_histogram(bins = 20, fill="green", alpha=0.3) +
    geom_vline(aes(xintercept=mean(cholesterol)), ## straight line for the mean
               colour = "red", size=0.5) +
    geom_vline(aes(xintercept=median(cholesterol)), ## dashed line for the median
               colour = "red", linetype="dashed", size=0.5)
```
![a](https://user-images.githubusercontent.com/17719651/52897934-703c0c00-321b-11e9-879f-01533f47ee94.png)
```{r}
ggplot(df, aes(x = "", y = cholesterol)) + geom_boxplot(fill = "green", alpha = 0.3) +
    stat_summary(fun.y=mean, colour="darkred", geom="point", size=2) #red dot for the mean
```
![b](https://user-images.githubusercontent.com/17719651/52897935-703c0c00-321b-11e9-9352-a0b017a73343.png)
***
**The data below show the consumption of alcohol (X, liters per year per person, 14 years or older) and
the death rate from cirrhosis, a liver disease (Y, death per 100,000 population) in 15 countries (each
country is an observation unit).**


* **관계를 알아보기 위해 다이어그램을 그려라. 만약 두 변수 사이에 어떠한 관계가 존재한다면, 어떠한 계산없이 관계 결과를 도출할 수 있는가?**
* **상관 계수와 공분산을 계산하라.**

```{r}
X <- c(24.7, 15.2, 12.3, 10.9, 10.8, 9.9, 8.3, 7.2, 6.6, 5.8, 5.7, 5.6, 4.2, 3.9, 3.1)
Y <- c(46.1, 23.6, 23.7, 7, 12.3, 14.2, 7.4, 3.0, 7.2, 10.6, 3.7, 3.4, 4.3, 3.6, 5.4)

df <- data.frame(X, Y)

## Good way to visualize the connection is to draw a scatter plot
ggplot(df, aes(x=X, y=Y)) +
    geom_point() +
    geom_smooth(method=lm, se=FALSE) +
    labs(x = "Alcohol consumption",
         y = "Death Rate from Cirrhosis",
         title = "Relation between two variables")
```

![c](https://user-images.githubusercontent.com/17719651/52897936-70d4a280-321b-11e9-9200-f646c9f380ea.png)

그래프를 보면 강한 양의 상관 관계가 있는것 처럼 보인다. 알코올 소비가 증가하면 간경화로 인한 사망률 또한 증가한다고 볼 수 있다. 우리의 결론이 옳은지 상관 계수를 계산하여 확인해보자.<br>

```{r}
print(paste0("Covariance between Alcohol consumption and Death Rate from Cirrhosis: ", round(cov(X, Y), 2)))
print(paste0("Correlation between Alcohol consumption and Death Rate from Cirrhosis: ", round(cor(X, Y), 2)))
```

위의 코드를 실행하면 상관값이 0.94가 나오는 것을 확인할 수 있고 따라서 알코올 소비와 간경화로 인한 사망률이 어떤 관계를 가지고 있는지 확인할 수 있다.<br><br>

### <a id='condprob'>조건부 확률</a>

사건 B가 발생했을 때, 사건 A가 발생할 확률을 구한다고 가정해보자. (단, $P(B)>0$)

\begin{align}
P(A|B)=\frac{P(A \cap B)}{P(B)}
\end{align}
만약 $A$ 와 $B$ 가 독립이면 :
\begin{align}
P(A|B)=P(A)
\end{align}

**예제:**

**0세와 30세 사이에 어떤 특정 질병에 걸릴 확률은 몇일까? 그 나이대에 포함될 확률은 0.4 이며, 전체 인구 중 이 병에 걸릴 확률은 0.1 이라는 가정이 있다.**


```{r}
p_A <- .4
p_B <- .1
p_A_given_B <- p_A * p_B / p_B

print(paste0("Probability of getting a certain illness is: ", p_A_given_B))
```
위의 값은 0.4가 나오는데 이로부터 A와 B가 서로 독립임을 확인할 수 있다.<br>

**외래 진료를 받는 사람들에게 각각 0.5, 0.3 및 0.2의 확률로 발생하는 3 가지 질병 A, B, C 가 있다. 이 질병에서 흉통을 가질 확률은 각각 0.8, 0.4 및 0.2 이다. 외래 진료소에서 환자에게 흉통이 발생할 확률은 얼마인가? 흉통이 없는 사람에게서 질병 C가 발생할 확률은 얼마인가?**

```{r}
p_A <- .5
p_B <- .3
p_C <- .2
p_cp_A <- .8
p_cp_B <- .4
p_cp_C <- .2

p_chest_pain <- p_A*p_cp_A + p_B*p_cp_B + p_C*p_cp_C
p_C_given_no_cp <- p_C*(1-p_cp_C)/(1-p_chest_pain)

print(paste0("Probability that people at the outpatient clinic experience chest pain: ", p_chest_pain))
print(paste0("Probability of disease C in a person who has no chest pain: ", round(p_C_given_no_cp,3)))
```
<br><br>
### <a id='bayes'>베이즈 정리 (Bayes Rule)</a>

베이즈 정리는 사건과 관련된 상황에 대한 사전 지식을 기반으로 사건의 확률을 설명한다.

\begin{align}
P(B|A)=\frac{P(A|B)P(B)}{P(A|B)P(B)+P(A|B^C)P(B^C)}
\end{align}

**Example with diagnostic test** *(taken from the book **Methods in Biostatistics with R**)*

* Let $+$ and $-$ be the events that the result of a diagnostic test is positive or negative respectively.

* Let $D$ and $D^C$ be the event that the subject of the test has or does not have a disease.

* The **sensitivity** is the probability that the test is positive given that the subject has the disease $P(+|D)$.

* The **specificity** is the probability that the test is negative given that the subject doesn't have the disease $P(-|D^C)$.

* **Positive predicted values** is the probability that the patient has the disease given that the test results are positive $P(D|+)$.

* **Negative predicted values** is the probability that the patient doesn't have the disease given that the test results are negative $P(D^C|-)$.

* **The likelihood ratio** gives an information of how much the odds of disease change based on a positive or a negative result.

    * **Diagnostic likelihood ratio of a postive test**
    \begin{align}
    DLR_+ = \frac{P(+|D)}{P(+|D^C)} = \frac{sensitivity}{1-specificity}
    \end{align}

    * **Diagnostic likelihood ratio of a negative test**
    \begin{align}
    DLR_- = \frac{P(-|D)}{P(-|D^C)} = \frac{1-sensitivity}{specificity}
    \end{align}

* **Prevalence** is a proportion of participants with the disease at a particular point of time.

* **Incidence** is a likelihood of developing a disease among people free of disease who are at risk of developing disease.

<img src="https://image.ibb.co/cqZ04T/Deepin_Screenshot_select_area_20180729205847.png">
<img src="https://image.ibb.co/g4eBB8/Deepin_Screenshot_select_area_20180729205900.png">


<br><br>
