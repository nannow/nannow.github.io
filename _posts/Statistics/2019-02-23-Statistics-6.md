---
layout: post
title: 우도, 큰 수의 법칙, 중심 극한 정리, 신뢰 구간
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

## <a id='lklhd'><font color="blue">우도 (Likelihood)</font></a>

**우도**에 대해 설명하고 싶지만 정말 직관적이고 이해하기 쉽게 쓰여진 글이 있어서 링크를 올리겠다.<br><br>
[http://rstudio-pubs-static.s3.amazonaws.com/204928_c2d6c62565b74a4987e935f756badfba.html](http://rstudio-pubs-static.s3.amazonaws.com/204928_c2d6c62565b74a4987e935f756badfba.html)
<br><br>

## <a id='lln'><font color="blue">큰 수의 법칙 (Law of Large Numbers)</font></a>

> 큰 수의 법칙 또는 대수의 법칙, 라플라스의 정리는 큰 모집단에서 무작위로 뽑은 표본의 평균이 전체 모집단의 평균과 가까울 가능성이 높다는 통계와 확률 분야의 기본 개념이다.

> *Source: [Wikipedia](https://ko.wikipedia.org/wiki/%ED%81%B0_%EC%88%98%EC%9D%98_%EB%B2%95%EC%B9%99)*

<br>

고혈압 환자의 비율을 확인하기 위한 실험을 하고 있다고 가정해보자. 그리고 데이터 set 의 비율이 실제 모집단 비율이라고도 가정해보겠다. 고혈압 결과에 대한 표본을 얻은 다음, 표본 사이즈가 커질 때 표본 비율과 모집단 비율의 차이가 0으로 수렴하는지 확인해보자.

<br>

```{r}
print(paste0("Ratio of people with Hypertension in population: ", round(mean(Stroke_Data$hypertension == 1),3)))
```

위의 코드의 결과는 0.088 이 나온다.

```{r}
n <- 500 # size of the samples

# covert hypertension column to string on (0,1 integers)
hypertension  <- as.numeric(levels(Stroke_Data$hypertension))[Stroke_Data$hypertension]

avg_value <- mean(hypertension)

# taking three random sample n-size each
x1 = sample(hypertension, size = n, replace = F)
x2 = sample(hypertension, size = n, replace = F)
x3 = sample(hypertension, size = n, replace = F)

# creating three vectors that will hold information about ratio for each size of sample (from 1 to n)
xbar1 = rep(0,length(x1))
xbar2 = rep(0,length(x2))
xbar3 = rep(0,length(x3))

for (i in 1:length(x1)) {
    xbar1[i] = mean(x1[1:i])
    xbar2[i] = mean(x2[1:i])
    xbar3[i] = mean(x3[1:i])
}

plot(1:n, xbar1-avg_value, type="l", col="red", lwd=1, ylim=c(-0.1,0.2),
     xlab="Number of subjects sampled",
     ylab="Distance to the mean")
lines(1:n, xbar2-avg_value, col="blue", lwd=1)
lines(1:n, xbar3-avg_value, col="orange", lwd=1)
lines(1:n, rep(0,n), lwd=3)

```
<p align="center">
<img width="329" alt="2019-02-21 8 41 22" src="https://user-images.githubusercontent.com/17719651/53285021-f10c8200-379e-11e9-9613-3c8d010b6a48.png">
</p>

표본에서 피험자의 수가 많을수록 표본의 평균이 모집단의 평균과 같아지는 것을 볼 수 있다. 이렇듯, 큰 수의 법칙은 n이 증가함에 따라 평균이 목표에 가깝다는 것을 입증 할 수 있다.
<br><br>

## <a id='clt'><font color="blue">중심 극한 정리 (Central Limit Theorem)</font></a>

> 표본의 크기가 커질수록 표본 평균의 분포는 모집단의 분포 모양과는 관계없이 정규 분포에 가까워진다. 이때 표본분포의 표본 평균은 모집단의 모 평균과 같고, 표본 표준 편차는 모집단의 모 표준 편차를 표본 크기의 제곱근으로 나눈 것과 같다.

> *Source: [나무위키](https://namu.wiki/w/%EC%A4%91%EC%8B%AC%20%EA%B7%B9%ED%95%9C%20%EC%A0%95%EB%A6%AC)*

<br>
뇌졸중을 가져본 적 없는 **BMI**의 분포를 살펴보자.<br>

```{r}
bmi <- Stroke_Data$bmi[Stroke_Data$stroke == 0]
hist(bmi, breaks=50, col="lightgreen")
print(paste0("Population mean: ", round(mean(bmi), 1)))
```

<p align="center">
<img width="286" alt="2019-02-23 7 38 50" src="https://user-images.githubusercontent.com/17719651/53285340-ad1b7c00-37a2-11e9-948f-60660f8a88f5.png">
</p>

```{r}
n = 30
sample = sample(bmi, size = n, replace = F)
xbar = rep(0,length(sample))

for (i in 1:length(sample)) {
    xbar[i] = mean(sample[1:i])
}

hist(xbar, breaks=50, col="orange", main = "Histogram of sample means (sample size = 30)")
print(paste0("Sample mean: ", round(mean(xbar), 1)))
```

<p align="center">
<img width="441" alt="2019-02-23 7 41 43" src="https://user-images.githubusercontent.com/17719651/53285385-13080380-37a3-11e9-98db-f25322b5dd24.png">
</p>

```{r}
n = 300
sample = sample(bmi, size = n, replace = F)
xbar = rep(0,length(sample))

for (i in 1:length(sample)) {
    xbar[i] = mean(sample[1:i])
}

hist(xbar, breaks=50, col="orange", main = "Histogram of sample means (sample size = 300)")
print(paste0("Sample mean: ", round(mean(xbar), 1)))
```
<p align="center">
<img width="448" alt="2019-02-23 7 45 32" src="https://user-images.githubusercontent.com/17719651/53285425-9c1f3a80-37a3-11e9-98c4-6dad6dda382e.png">
</p>

표본 평균의 분포가 표본 사이즈가 증가할수록 정규분포의 모양 (종모양) 으로 보인다는 것을 확인할 수 있다. 만약 사이즈를 늘리면 더 종 모양처럼 나올 것이다. 그리고 그러한 분포의 평균값은 모평균의 평균값과 거의 같다고 말할 수 있다. (물론, 모평균의 표준편차와 표본평균의 표준편차는 다르다.)
<br><br>

## <a id='ci'><font color="blue">신뢰 구간 (Confidence Intervals)</font></a>

> 통계학에서 신뢰 구간(信賴區間, 영어: confidence interval)은 모수가 어느 범위 안에 있는지를 확률적으로 보여주는 방법이다.

> *Source: [Wikipedia](https://ko.wikipedia.org/wiki/%EC%8B%A0%EB%A2%B0_%EA%B5%AC%EA%B0%84)*

$Z_{1-\frac{\alpha}{2}}$ 는 표준 정규 분포 ($\mu=0, \sigma = 1$) 의 $(1-\frac{\alpha}{2})$ 분위일 때, 임의의 구간 $X_n \pm Z_{1-\frac{\alpha}{2}}\frac{\sigma}{\sqrt{n}}$ 가 모평균 $\mu$ 를 포함할 확률은 대략 95% 이다. 이것을 95% 신뢰구간이라고 부른다.

```{r}
population <- rnorm(5000, 10, 5)
hist(population, breaks = 25, col="lightgreen")
print(paste0("Population mean: ", round(mean(population), 1)))
print(paste0("Population standard error: ", round(sd(population), 1)))
```
<p align="center">
<img width="383" alt="2019-02-25 3 33 41" src="https://user-images.githubusercontent.com/17719651/53303525-8b0f2000-38ae-11e9-84e8-351139882c5a.png">
</p>

```{r}
n=250
sample <- sample(population, size=n, replace = F)
print(paste0("Sample mean: ", round(mean(sample), 1)))
print(paste0("Sample standard error: ", round(sd(sample), 1)))
```

```{r}
# caclulation 95% CI
alpha = 0.05
quant <- 1 - alpha/2
Z_score <- qnorm(quant, lower.tail=TRUE)
lower <- mean(sample) - Z_score * sd(population) / sqrt(n)
upper <- mean(sample) + Z_score * sd(population) / sqrt(n)

hist(sample, breaks = 25, col="orange")
abline(v=lower, lwd = 3)
abline(v=upper, lwd = 3)
print(paste0("With 95% confidence we can say that population mean is in interval [", round(lower,1), "; ", round(upper,1), "]"))
```
<p align="center">
<img width="387" alt="2019-02-25 3 36 16" src="https://user-images.githubusercontent.com/17719651/53303526-8ba7b680-38ae-11e9-8674-0745d1a33664.png">
</p>

95% 확신을 가지고 9.2와 10.5 사이에 모평균이 포함한다고 말할 수 있다.

<br><br>
This post has been released under the Apache 2.0 open source license.
<br><br>
