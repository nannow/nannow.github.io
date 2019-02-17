---
layout: post
title: 실험, 확률, 확률 변수
comments: false
use_math: true
categories: [Statistics]
tags: [Statistics]
---


![biostatisticswordle](https://user-images.githubusercontent.com/17719651/52768499-07b12b80-3071-11e9-8b9b-1c451f8108e1.jpg)

<br>
## <a id='setup'><font color="blue">Setting up the environment</font></a>
<br>
<font color="red">이 포스트에서는 예제를 실습하기 위해 R을 사용한다.</font>
<br>
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

## <a id='exp'><font color="blue">Experiments</font></a>

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
<br><br>
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
## <a id='prob'><font color="blue">Probability</font></a>

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
## <a id='rv'><font color="blue">확률 변수 (random variable)</font></a>

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
<br>


<br><br>
