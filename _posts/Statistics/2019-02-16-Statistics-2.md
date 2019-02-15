---
layout: post
title: 생물통계학 in R - (1)
comments: false
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
- <a href='#nextpart'>What to Expect Next</a>

## <a id='intro'>Project Introduction</a>



**생물통계학 이란?**

>*생물통계학(biometrics statistics, 生物統計學)은 생물학 중심에서 사용되는 통계적 방법과 이론을 다루는 통계학의 한 영역으로 생물학적 현상과 관련된 통계를 취급하며 생물측정을 포함된 학문이다.

> *Source: [Wikipedia](https://ko.wikipedia.org/wiki/%EC%83%9D%EB%AC%BC%ED%86%B5%EA%B3%84%ED%95%99)*

## <a id='setup'>Setting up the environment</a>

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


## <a id='probthnk'>Probabilistic Thinking</a>


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

### <a id='rv'>확률 변수</a> 

**확률 변수** 는 무작위 실험의 결과를 나타내는 수치이다. 여기서는 *이산적* 이거나 *연속적* 확률 변수에 대해 다룰 것이다.

* **이산 확률 변수** 는 확률 변수 X가 취할 수 있는 모든 값을 x1, x2, x3, ... 처럼 셀 수 있을 때 X를 이산 확률 변수라고 한다.

* **연속 확률 변수** 는 적절한 구간 내의 모든 값을 취하는 확률 변수이다.

#### 확률 질량 함수

> A **확률 질량 함수 (PMF)** 는 이산 확률 변수 X가 취할 수 있는 값 x1, x2, x3, ... 의 각각에 대해서 확률 P(X=x1), P(X=x2), P(X=x3), ... 을 대응시켜주는 관계이다.

동전던지기의 PMF : $p(x)=\theta^x(1 - \theta)^{1-x}$,

$\theta$ 은 1 (앞면) 이 나올 확률이고, $x$ 는 1(앞면) 또는 0 (뒷면) 을 입력으로 받는다. 

동전던지기의 PMF를 다시써보면 다음과 같다. $p(x)=(\frac{1}{2})^x(\frac{1}{2})^{1-x}$.

유효한 PMF를 갖기 위해서, 함수 $p(x)$ 는 반드시 다음을 만족해야 한다.

* $p(k) \geqslant 0$ for all $k \in K$, where $K$ is the set of all possible outcomes for $X$;

* $\sum_{k\in{K}} p(k) = 1$.

#### 확률 밀도 함수

> **Probability Density Function** is a function of a **continuous** random variable, whose integral across an interval gives the probability that the value of the variable lies within the same interval.

$f(x)=\frac{x^4}{3}$, for $x \in (0;2)$ 라는 분포가 있다고 가정하자. 

유효한 PDF를 갖기 위해서, 함수 $f(x)$ 는 반드시 다음을 만족해야 한다.

* $f(x) \geqslant 0$ for all $x$;

* $\int\limits_{-\infty}^\infty f(x) \,dx = 1$.

#### 독립 사건

$P(E \cap F)=P(E)P(F)$ 를 만족하는 $E$ 와 $F$ 는 서로 독립이다.

### <a id='chardist'>분포의 주요 특성</a>

#### 기댓값

**평균** (or expected value) 은 어떤 확률 과정을 무한히 반복했을 때, 얻을 수 있는 값의 평균으로서 기대할 수 있는 값. 보다 엄밀하게 정의하면 기댓값은 확률 과정에서 얻을 수 있는 모든 값의 가중 평균이다.

* **이산** 확률 변수의 PMF $p(x)$ 가 있을 때 기댓값 $E[X]=\sum_x xp(x)$ 이다.

* **연속** 확률 변수의 PDF $f(x)$ 가 있을 때  $E[X]=\int\limits_{-\infty}^{\infty} xf(x)\,dx$.

모평균과 표본평균의 차이점은 아래 설명에서 확인해보자.<br>

population mean : 모평균 <br>
sample mean : 표본 평균

You could hear terms like *population mean* and *sample mean*. What's the difference? For example, you want to check duration of time from first exposure to HIV infection to AIDS diagnosis, so your *population* is going to be *every* human who had exposure to HIV. In real life it is never possible to gather information about population, but what you could is gather information about some people (*sample*) and then make assumptions about population based on information about the sample. 

![target-population](https://user-images.githubusercontent.com/17719651/52778397-494dd080-3089-11e9-9fb2-c9081de4e1c3.jpg)

* **표본 평균** 다음과 같은 식으로 정의된다. $\bar{X}=\frac{1}{n}\sum_{1}^{n}X_i$

#### 분산

**분산** 은 어떤 확률변수가 기댓값으로부터 얼마나 떨어진 곳에 분포하는지를 가늠하는 숫자이다.

* If $X$ is a **discrete** random variable with PMF $p(x) = P(X = x)$, then:  
\begin{align}
Var(X)=\sum_{x}(x-\mu)^2p(x)
\end{align}

where $\mu$ is the mean.

* If $X$ is a **continuous** random variable with PDF $f(x)$, then: 
\begin{align}
Var(X)=\int\limits_{-\infty}^\infty (x-\mu)^2f(x) \,dx
\end{align}

* If $X_i$, $i\in[1;n]$ are random variables with variance $\sigma^2$ the **sample variance** is defined as:
\begin{align}
s^2=\frac{\sum_{i=1}^{n}(X_i-\bar{X})^2}{n-1}
\end{align}

#### Standard deviation

**Standard deviation** is defined as the square root of the variance. It also tells about variation of data points from the mean. The key defference is that standard deviation is in the same units as the data in disctribution which makes it easy to interpret. 
\begin{align}
sd(X)=\sqrt{Var(X)}
\end{align}

**Sample Standard deviation:** $s=\sqrt{s^2}$

#### Median    

**Median** tells where the middle of **sorted** dataset is (.5 quantile). 

For example we have dataset {1, 4, 5, 6, 1, 1, 5, 7, 120}. First we have to sort it ad then find the middle value {1, 1, 1, 4, **5**, 5, 6, 7, 120}. So the median is **5**.

Why sometimes it is better to use median instead of mean? Average value is very sensitive to *outliers* which can give a wrong interpretation of variables. For the dataset {1, 4, 5, 6, 1, 1, 5, 7, 120} the mean = **16.67** because of the value of 120. In this case we would rather use median to estimate average value of dataset.

#### Covariance and Correlation

**Covariance** is a measure of how much two random variables vary together.
\begin{align}
Cov(X_1,X_2)=E(X_1 X_2) - E(X_1)E(X_2)
\end{align}
Sample covariance (estimator of the true covariance): 
\begin{align}
\hat{Cov}(X_1, X_2) = \frac{1}{n}\sum_{i}^{n}(X_{1i} - \bar{X_1})(X_{2i} - \bar{X_2})
\end{align}

However, the covariance is hard to interpret. For example, the $Cov(X,Y)=200$ tells us that there is a relationship (correlation) between two variables $X$ and $Y$, but doesn't tell exactly how strong it is. That when it is usefull to use **correlation**

Correlation is defined as follows:
\begin{align}
Cor(X_1, X_2) = \frac{\hat{Cov}(X_1, X_2)}{\sqrt{Var(X_1)Var(X_2)}} = \frac{\hat{Cov}(X_1, X_2)}{sd(X_1)sd(X_2)}
\end{align}

\begin{align}
-1 \leqslant Cor(X_1, X_2) \leqslant 1
\end{align}

* If $Cor(X_1, X_2)=0$ - there is no relationship between variables $X_1$ and $X_2$.

* If $Cor(X_1, X_2)=1$ - there is a strong positive connection between $X_1$ and $X_2$. As $X_1$ increases $X_2$ also **increases**.

* If $Cor(X_1, X_2)=-1$ - there is a strong negative connection between $X_1$ and $X_2$. As $X_1$ increases $X_2$ **decreases**.



<br>
<br><br>


<br><br>


