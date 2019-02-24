---
layout: post
title: 조건부 확률, 베이즈 정리, ROC and AUC
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
suppressMessages(library(randomForest))
suppressMessages(library(caret))
suppressMessages(library(cowplot))
suppressMessages(library(Metrics))
suppressMessages(library(AUC))

set.seed(123)
options(repr.plot.width=7, repr.plot.height=3)
```
<br>
## <a id='condprob'><font color="blue">조건부 확률</font></a>

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
<br>
## <a id='bayes'><font color="blue">베이즈 정리 (Bayes Rule)</font></a>

베이즈 정리는 사건과 관련된 상황에 대한 사전 지식을 기반으로 사건의 확률을 설명한다.

\begin{align}
P(B|A)=\frac{P(A|B)P(B)}{P(A|B)P(B)+P(A|B^C)P(B^C)}
\end{align}

**Example with diagnostic test** *(taken from the book **Methods in Biostatistics with R**)*

* $+$ 와 $-$ 는 진단 검사 결과가 양성인지 음성인지를 나타낸다.
* $ D $ 와 $ D ^ C $는 검사 대상자가 각각 질병에 걸렸거나 질병이 없는 사건을 나타낸다.
* **민감도 (sensitivity)** 는 질병이 있는 사람이 검사를 받았을 때 양성 판정을 받을 확률이다.
* **특이도 (specificity)** 는 질병이 없는 사람이 검사를 받았을 때 음성 판정을 받을 확률이다.
* **양성 예측도 (Positive predicted values)** 는 검사 결과가 양성일 때 실제로 환자가 질병을 가지고 있을 확률이다.
* **음성 예측도 (Negative predicted values)** 는 검사 결과가 음성일 때 실제로 환자가 질병을 가지고 있지 않을 확률이다.
* **우도비 (The likelihood ratio)** 는 양성 또는 음성 결과에 따라 질병의 확률이 얼마나 변하는지에 대한 정보를 제공한다.<br>
    * **양성 검정 우도비 (Diagnostic likelihood ratio of a postive test)**
    \begin{align}
    DLR_+ = \frac{P(+|D)}{P(+|D^C)} = \frac{sensitivity}{1-specificity}
    \end{align}
    * **음성 검정 우도비 (Diagnostic likelihood ratio of a negative test)**
    \begin{align}
    DLR_- = \frac{P(-|D)}{P(-|D^C)} = \frac{1-sensitivity}{specificity}
    \end{align}
* **유병률 (Prevalence)** 은 일정한 시점에서 전체 인원 중 질병을 가진 사람의 비율이다.
<img src="https://image.ibb.co/cqZ04T/Deepin_Screenshot_select_area_20180729205847.png">
<img src="https://image.ibb.co/g4eBB8/Deepin_Screenshot_select_area_20180729205900.png">

예제는 생략하겠다.
<br><br>

## <a id='roc'><font color="blue">ROC,AUC,Precision and Recall</font></a>

이 부분은 아무리 잘 설명하고 싶어도 명쾌하게 설명하기 힘들어서 동영상 링크를 올리겠다. 이해하기 쉽게 설명이 되어있다.<br><font color="blue">https://www.youtube.com/watch?v=nMAtFhamoRY</font>

위의 동영상에 근거해 설명을 하자면, 우리가 accuracy를 항상 신뢰할 수 없기 때문에 Precision 과 Recall 을 사용하고 그걸로도 부족하다 싶으면 ROC 와 AUC를 사용한다고 한다.<br><br>

This post has been released under the Apache 2.0 open source license.

<br><br>
