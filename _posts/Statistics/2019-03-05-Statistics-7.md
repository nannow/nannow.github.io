---
layout: post
title: t-검정, f-검정, 데이터 Resampling
comments: false
use_math: true
categories: [Statistics]
tags: [Statistics]
---


![biostatisticswordle](https://user-images.githubusercontent.com/17719651/52768499-07b12b80-3071-11e9-8b9b-1c451f8108e1.jpg)
## <a id='setup'><font color="blue">Setting up the environment</font></a>
<br>
<font color="red">이 포스트에서는 예제를 실습하기 위해 R을 사용한다.</font>
<br>
```{r}
suppressMessages(library(tidyverse))
suppressMessages(library(repr))
suppressMessages(library(cowplot))
suppressMessages(library(bootstrap))
suppressMessages(library(binom))
suppressMessages(library(infer))
set.seed(123)
options(repr.plot.width=7, repr.plot.height=3)

Stroke_Data <- read.csv("../input/train_2v.csv")

# some data cleaning
Stroke_Data <- Stroke_Data %>% filter(gender != "Other" & !(is.na(bmi))) %>%
  select(-id)

# convert some variables to a factor
Stroke_Data$hypertension <- as.factor(Stroke_Data$hypertension)
Stroke_Data$heart_disease <- as.factor(Stroke_Data$heart_disease)
Stroke_Data$stroke <- as.factor(Stroke_Data$stroke)

sample_n(Stroke_Data, 5)
```
<br>



## <a id='tnf'><font color="blue"><i>t</i> 검정 and F 검정</font></a>

t 분포는 보통 작은 표본 (n<30) 일 때 정규 분포 대신 사용된다. 표본 크기가 클수록 t 분포는 정규 분포와 유사하다. 정규 분포 신뢰 구간에서 모평균을 추정할 때 Z 통계량을 사용하여 95% 신뢰 구간을 구했었다. 여기에서 전제조건은 표준 편차가 주어져 있다는 것이었다. 하지만, 모평균 값을 알지 못하기 때문에 모평균을 추정하는 것인데 어떻게 모표준편차를 안다고 가정할 수 있을까? 이러한 질문에 대답을 하기위해 t분포가 등장하였다. 모표준편차 $\sigma$ 를 알지 못해서 사용하지 못하기 때문에 $\sigma$ 대신 표본 표준 편차 $s$를 사용한다.

**t-분포를 이용한 모평균 추정 신뢰 구간**<br>
\begin{align}
\bar{X} \pm t_{n-1,1-\frac{\alpha}{2}}\frac{s}{\sqrt{n}}
\end{align}

$(n-1)$ - 자유도, $n$ - 표본 사이즈<br>

t 검정에는 3 가지 방법이 있다. 그 중에서 Two sample t 검정에 대해 살펴보겠다. Two sample t 검정은 두 집단의 평균을 비교하여 얼마나 차이가 있는지 알려준다.<br>

우리가 iid (independent and identically distributed) 한 확률 변수를 다음과 같이 가지고 있다고 가정해보자.

$X_1,...,X_n \sim N(\mu_x, \sigma^2)$;

$Y_1,...,Y_n \sim N(\mu_y, \sigma^2)$.

$100(1-\alpha$)% CI for the difference between means of this two sets is:

두 집합의 평균 차이에 대한 $100(1-\alpha$)% 신뢰 구간

\begin{gather}
\bar{X} - \bar{Y} \pm t_{n_X+n_Y-2, 1-\frac{\alpha}{2}} S_{n, pooled}(\frac{1}{n_X} + \frac{1}{n_Y})^\frac{1}{2} \\
S^2_{n, pooled} = \frac{(n_X-1)\sigma^2 + (n_Y-1)\sigma^2}{n_X + n_Y - 2}
\end{gather}

$S^2_{n, pooled}$ - pooled variance estimator, 그룹 분산의 mixture 이다. 더 큰 표본 크기를 갖는 곳에 더 큰 가중치를 둔다.

<br>

만약 $\sigma^2_X \neq \sigma^2_Y$ 이라면 :

\begin{gather}
\bar{X} - \bar{Y} \pm t_{n_X+n_Y-2, 1-\frac{\alpha}{2}} (\frac{S^2_X}{n_X} + \frac{S^2_Y}{n_Y})^\frac{1}{2} \\
\end{gather}

과 같은 신뢰 구간을 설정하면 된다.

Let's assume that we want to check whether the difference in age for people who had stroke and who didn't is significant. To do that we can run a **t-test** to find out. First, lets visually inspect the distribution of **Age** by **Stroke** outcome.






<br><br>
This post has been released under the Apache 2.0 open source license.
<br><br>
