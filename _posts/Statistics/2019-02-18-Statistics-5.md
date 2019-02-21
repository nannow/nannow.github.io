---
layout: post
title: EDA, 검정 결과 분석 with Stroke Data Set
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

## <a id='stroke'><font color="blue">Healthcare Dataset Stroke Data</font></a>

다음 링크 [Healthcare Dataset Stroke Data](https://www.kaggle.com/asaumya/healthcare-dataset-stroke-data) 에서 데이터 셋을 가져왔다. 이 데이터셋은 뇌졸중에 대한 정보가 담겨있다.

```{r}
Stroke_Data <- read.csv("../input/train_2v.csv")
sample_n(Stroke_Data, 5)
```

```{r}
glimpse(Stroke_Data)
```

```{r}
#some data cleaning
Stroke_Data <- Stroke_Data %>% filter(gender != "Other" & !(is.na(bmi))) %>%
  select(-id)

#convert some variables to a factor
Stroke_Data$hypertension <- as.factor(Stroke_Data$hypertension)
Stroke_Data$heart_disease <- as.factor(Stroke_Data$heart_disease)
Stroke_Data$stroke <- as.factor(Stroke_Data$stroke)
```

<br><br>
### <a id='eda'>EDA (Exploratory Data Analysis)</a>

> 통계에서 **EDA (Exploratory Data Analysis)** 는 주로 시각적 방법으로 주요 특징을 요약하기 위해 데이터 세트를 분석하는 접근 방식이다.

<br>

**예제를 살펴보자**

```{r}
#사람들은 결혼하면 체중이 늘어나느 경향이 있다는 것이 사실일까?
#(이번 경우에는 bmi에 대해)

Stroke_Data %>%
    select(ever_married, bmi) %>%
    group_by(ever_married) %>%
    summarise(mean = round(mean(bmi), 2),
              median = median(bmi),
              variance = round(var(bmi), 2),
              std = round(sd(bmi), 2))
```
코드 실행 결과는 다음과 같다.<br><br>
<p align="center">
<img width="486" alt="2019-02-18 7 08 20" src="https://user-images.githubusercontent.com/17719651/52943499-a52d9780-33b0-11e9-9323-c5ae880d8305.png">
</p>

결혼한 사람이 결혼하지 않은 사람에 비해 표준 편차가 작고 평균값은 더 크다는 것을 확인할 수 있다.
<br><br>
**어떻게 숫자들을 해석할 수 있을까?**
<br><br>
평균적으로, 모든 BMI는 **7.1** 만큼 평균값에서 퍼져있다. 평균값이 중앙값보다 크므로 히스토그램이 오른쪽으로 비뚤어 진다고 말할 수 있다.
<br>

**시각화 해보자**
<br>
```{r}
ggplot(Stroke_Data, aes(x = bmi, fill=ever_married)) +
    geom_histogram(bins = 40, alpha=.5, position="identity") +
    geom_vline(aes(xintercept=mean(Stroke_Data$bmi[Stroke_Data$ever_married == 'Yes'], na.rm = T)),
               colour = "blue", size=0.5) +
    geom_vline(aes(xintercept=mean(Stroke_Data$bmi[Stroke_Data$ever_married == 'No'], na.rm = T)),
               colour = "red", size=0.5) +
    geom_vline(aes(xintercept=median(Stroke_Data$bmi[Stroke_Data$ever_married == 'Yes'], na.rm = T)),
               colour = "blue", linetype="dashed", size=0.5) +
    geom_vline(aes(xintercept=median(Stroke_Data$bmi[Stroke_Data$ever_married == 'No'], na.rm = T)),
               colour = "red", linetype="dashed", size=0.5)+
    theme_bw()
```
<p align="center">
<img width="436" alt="2019-02-18 7 29 37" src="https://user-images.githubusercontent.com/17719651/52944776-91376500-33b3-11e9-9b6a-cd7140f7cb88.png">
</p>
```{r}
ggplot(Stroke_Data, aes(x = ever_married, y = bmi)) +
    geom_boxplot(aes(fill=ever_married)) +
    stat_summary(fun.y=mean, colour="darkred", geom="point", size=2)+
    theme_bw()
```
<p align="center">
<img width="450" alt="2019-02-18 7 30 42" src="https://user-images.githubusercontent.com/17719651/52944828-b5934180-33b3-11e9-9f5b-0b35960fbcd7.png">
</p>
```{r}
summary(Stroke_Data)
```
<br>
<p align="center">
<img width="470" alt="2019-02-18 7 48 41" src="https://user-images.githubusercontent.com/17719651/52946313-196b3980-33b7-11e9-825f-8c6e502f65b2.png">
</p>
<br><br>

```{r}
distribution_plot <- function(y){

    box_plt <- ggplot(Stroke_Data, aes(x=stroke, y = eval(parse(text = y)))) +
                geom_boxplot(aes(fill=stroke)) +
                stat_summary(fun.y=mean, colour="darkred", geom="point", size=2) +
                labs(x = "Stroke outcome",
                    y = y) +
                theme_bw()

    hstgm <- ggplot(Stroke_Data, aes(x = eval(parse(text = y)), fill=stroke)) +
                geom_histogram(bins = 40, alpha=.5, position="identity") +
                geom_vline(aes(xintercept=mean(Stroke_Data[[y]][Stroke_Data$stroke == 1], na.rm = T)),
                   colour = "blue", size=0.5) +
                geom_vline(aes(xintercept=mean(Stroke_Data[[y]][Stroke_Data$stroke == 0], na.rm = T)),
                   colour = "red", size=0.5) +
                geom_vline(aes(xintercept=median(Stroke_Data[[y]][Stroke_Data$stroke == 1], na.rm = T)),
                   colour = "blue", linetype="dashed", size=0.5) +
                geom_vline(aes(xintercept=median(Stroke_Data[[y]][Stroke_Data$stroke == 0], na.rm = T)),
                   colour = "red", linetype="dashed", size=0.5) +
                labs(x = y) +
                theme_bw()

    return(plot_grid(hstgm, box_plt))
}
```
<br><br>


```{r}
distribution_plot("age")
```

<p align="center">
<img width="553" alt="2019-02-20 8 43 49" src="https://user-images.githubusercontent.com/17719651/53089979-62ef8c00-3551-11e9-8cc1-f75fd1e5bd73.png">
</p>

```{r}
distribution_plot("avg_glucose_level")
```

```{r}
distribution_plot("bmi")
```
<br><br>
위의 변수 세가지에 대해 각각 box plot과 히스토그램을 확인할 수 있다.
<br>
<br>

```{r}
distribution_ratio  <- function(x){

    plt <- Stroke_Data %>%
            select(stroke, x, gender) %>%
            group_by(stroke, var = eval(parse(text = x))) %>%
            summarise(count = length(gender)) %>%
            group_by(stroke) %>%
            mutate(ratio = round(count*100/sum(count), 1)) %>%
            ggplot(aes(y = ratio, x = stroke, fill = var)) +
            geom_bar(stat="identity") +
            labs(title=paste0("Ratio of stroke outcome by ", x), fill = x) +
            theme_bw()
    return(plt)
}
```
```{r}
distribution_ratio("hypertension")
```

<p align="center">
<img width="453" alt="2019-02-20 8 44 54" src="https://user-images.githubusercontent.com/17719651/53089980-63882280-3551-11e9-9444-19160487a71a.png">
</p>

어떤 유의미한 값을 부여하고 싶다면 다음과 같은 연산을 할 수도 있을 것이다.<br>
```{r}
H <- as.numeric(levels(Stroke_Data$hypertension))[Stroke_Data$hypertension]
S <- as.numeric(levels(Stroke_Data$stroke))[Stroke_Data$stroke]

p_SP_and_HP  <- mean(H&S)
p_SP_or_HP <- mean(H|S)
p_SP_and_HN <- mean(H&!S)

print(paste0("Probability of suffering from hypertension and having a stroke is: ", round(p_SP_and_HP, 3)))
print(paste0("Probability of suffering from hypertension or having a stroke is: ", round(p_SP_or_HP, 3)))
print(paste0("Probability of having a stroke without having a hypertension: ", round(p_SP_and_HN, 3)))
```

```{r}
distribution_ratio("heart_disease")
```

```{r}
distribution_ratio("ever_married")
```

**위의 코드를 실행했을 때 결혼 한 적이있는 사람들에게 뇌졸중이 더 자주 나타나는 것이 이상하지 않은가?**

## <a id='outcome'><font color="blue">Test Outcome Analysis</font></a>

지금부터 **랜덤 포레스트(Random Forest)** 를 사용하여 뇌졸중 결과를 예측해보자. 그전에 랜덤 포레스트에 대해 간단하게 설명해보겠다. 의사결정트리를 공부하고 랜덤 포레스트를 이해하는 것이 좋을것같지만 의사결정트리에 대해는 생략하겠다. <br><br>

머신러닝 회귀분석에서 변수로 취급하는 특징 (feature) 들은 어떤 분석을 할 때 실제로 유용한 특징일 수도 있고 분석에 유용하지 않은 특징일 수도 있다. 유용한 특징은 남기고 유용하지 않은 특징은 제거하거나 분석에 반영되는 계수값을 감소시키는 것이 좋을 것이다. 혹은 서로 다른 두 특징이 회귀분석 예측에 중첩되는 성질을 가지고 있을 수도 있는데 이것을 **상관성** 이 존재한다고 말한다. 의사결정트리에서 이러한 상관성을 줄이는 방법으로 **랜덤 포레스트** 를 사용한다.<br><br>

의사결정트리에는 사실 랜덤 포레스트보다 **베깅(Bagging)** 이라는 방법을 더 먼저 사용했다. 랜덤 포레스트가 강력한 이유는 의사결정트리와 베깅보다 분산을 훨씬 많이 줄일 수 있어서 **과적합(overfitting)** 을 막을 수 있기 때문이다. (분산을 줄이는 이유이다) 베깅은 모집단에서 K번만큼 샘플링하여 K개의 각 훈련 set에 대해 모델을 만들고 모델에 평균을 취하는 것이다. 이런식으로 샘플링하는 과정을 **부트스트랩 (Bootstrap)** 이라고 부른다. 그렇게 만들어진 의사결정트리 모델은 샘플링을 하지 않은 단일의 의사결정트리에 비해 낮은 분산을 갖을 수 있다. <br><br>

언뜻 보면 베깅은 훌륭한 성능을 낼 수 있을 것 같다. 그렇지만 의사결정트리 노드분할과정에서 쓰는 알고리즘에 의해 (설명 생략) 더 가중치가 높은 특징이 의사결정트리의 상단노드에 위치하게 된다. 이러한 과정 자체가 상관성이 생겼다는 증거이다. 상관성이 존재한 채로 모델링을 하는것보다 상관성이 배제된 상태로 모델링을 하는 것이 과적합을 방지하여 검정set에 대해 모델의 예측성능을 향상시킬 수 있을 것 같다. 그래서 랜덤 포레스트를 사용한다. 글로 이해가 가지 않는다면 다음 링크의 그림과 설명을 참조하면 도움이 될 것이다. <br>
[http://blog.naver.com/PostView.nhn?blogId=sw4r&logNo=221032295777&parentCategoryNo=&categoryNo=86&viewDate=&isShowPopularPosts=true&from=search](http://blog.naver.com/PostView.nhn?blogId=sw4r&logNo=221032295777&parentCategoryNo=&categoryNo=86&viewDate=&isShowPopularPosts=true&from=search)


```{r}
frml <- (stroke~gender+age+hypertension+heart_disease+ever_married+work_type+Residence_type+avg_glucose_level+bmi)

model_rf <- randomForest(formula = frml, data = Stroke_Data)

prediction_rf <- predict(object = model_rf,
                         newdata = select(Stroke_Data, -stroke),
                         type = "class")

confusionMatrix(data = prediction_rf, reference = Stroke_Data$stroke)
```
<p align="center">
<img width="255" alt="2019-02-21 5 42 36" src="https://user-images.githubusercontent.com/17719651/53155462-7574ce80-3600-11e9-8588-8f352559b6ed.png">
</p>


* 피험자가 뇌졸중을 앓고 있을 때 검사 결과가 양성일 확률 (sensitivity) $P(T+\|D+) = 1.0$
* 피험자가 뇌졸중을 앓지 않고 있을 때 검사결과가 음성일 확률 (specificity) $P(T-\|D-) = 0.36$
* 검사 결과가 양성일 때 환자가 질병을 앓고 있을 확률 (Positive predicted values) $P(D+\|T+)=0.99$.
* 검사 결과가 음성일 때 환자가 질병을 앓고 있지 않을 확률 (Negative predicted values) $P(D-\|T-)=1.0$
* Diagnostic likelihood ratio of a postive test
\begin{align}
DLR_+ = \frac{sensitivity}{1-specificity} = \frac{1}{1-0.36} = 1.56
\end{align}
* Diagnostic likelihood ratio of a negative test
\begin{align}
DLR_- = \frac{1-sensitivity}{specificity} = \frac{1-1}{0.36} = 0
\end{align}
<br>

```{r}
print(paste0("AUC = ", round(Metrics::auc(actual = Stroke_Data$stroke, predicted = prediction_rf), 4)))
```
<br>
```{r}
plot(roc(prediction_rf, Stroke_Data$stroke), main = "ROC", cex.axis=0.6, cex.main=0.8, cex.lab=0.8)
```

<br><br>
