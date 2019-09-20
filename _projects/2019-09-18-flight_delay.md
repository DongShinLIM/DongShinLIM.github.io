---
title: 'Predicting Flight Delays'
date: 2019-09-18 00:00:00
featured_image: '/images/demo/delaybg.jpg'
excerpt: This page is a demo that shows everything you can do inside portfolio and blog posts. We've included everything you need to create engaging posts about your work, and show off your case studies in a beauti
---

![](/images/demo/delay.jpg)

### 2019 빅콘테스트 항공기 지연 예측 모델 (R, Python, Randomforest)
#### 2017년 1월 1일부터 19년 6월 30일까지 국내의 모든 항공 운항데이터(약 100만건)를 기반으로 2019년 9월 16일부터 30일까지 예정되어 있는 항공기(약 1만6천건)의 지연 여부와 지연확률을 예측하는 것이 본 프로젝트의 최종 목표이다.

#### 1. 필요한 패키지를 설치한다

``` r
library(tidyverse)
library(randomForest)
library(MLmetrics)
library(caret)
library(dplyr)
library(ROCR)
library(pROC)
library(plyr)
```
#### 2. 제공된 Raw Data를 불러온다
해당 공모전에서 제공 된 파일은 모델링을 할 때 쓰이는 'AFSNT.csv'와 예측하여 제출해야 할 파일 'AFSNT_DLY.csv' 총 두개이다.

``` r
afsnt <- read.csv("AFSNT.csv", header = TRUE, fileEncoding = 'euc-kr')
afsnt_dly <- read.csv("AFSNT_DLY.csv", header = TRUE, fileEncoding = 'euc-kr')
afsnt_dly[3, 5] <- 'ARP1'
head(afsnt)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT      REG AOD IRR  STT
    ## 1   2017      1      1     일 ARP1 ARP3   A A1901 SEw3Nzc2   D   N 6:10
    ## 2   2017      1      1     일 ARP1 ARP3   A A1905 SEw4MjM2   D   N 6:15
    ## 3   2017      1      1     일 ARP1 ARP3   L L1751 SEw4MjM3   D   N 6:20
    ## 4   2017      1      1     일 ARP1 ARP3   F F1201 SEw4MjA3   D   N 6:25
    ## 5   2017      1      1     일 ARP3 ARP1   A A1900 SEw3NzAz   D   N 6:30
    ## 6   2017      1      1     일 ARP1 ARP3   H H1101 SEw4MDMx   D   N 6:30
    ##    ATT DLY DRR CNL CNR
    ## 1 6:18   N       N    
    ## 2 6:25   N       N    
    ## 3 6:30   N       N    
    ## 4 6:34   N       N    
    ## 5 6:37   N       N    
    ## 6 6:38   N       N

2017년 1월1일부터 2019년 6월30일까지 국내공항 간 비행 데이터이며 각 변수는 순서대로 
(년, 월, 일, 요일, 출발공항, 상대공항, 항공사, 편명, 등록기호, 출도착, 부정기, 예정시각, 실제시각, 지연여부, 지연코드, 결항여부, 결항코드) 를 의미한다.

``` r
head(afsnt_dly)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT AOD   STT DLY DLY_RATE
    ## 1   2019      9     16     월 ARP1 ARP3   L L1702   A  9:05  NA       NA
    ## 2   2019      9     16     월 ARP3 ARP1   L L1702   D  7:55  NA       NA
    ## 3   2019      9     16     월 ARP1 ARP3   L L1720   A 14:40  NA       NA
    ## 4   2019      9     16     월 ARP3 ARP1   L L1720   D 13:30  NA       NA
    ## 5   2019      9     16     월 ARP4 ARP3   L L1808   A 20:10  NA       NA
    ## 6   2019      9     16     월 ARP3 ARP4   L L1808   D 19:10  NA       NA
예측하여 제출해야 할 데이터는 2019년 9월 16일부터 30일까지 (년, 월, 일, 요일, 출발공항, 상대공항, 항공사, 편명, 출도착) 변수가 있으며 
최종적으로 '지연여부'와 '지연확률'을 예측하는 것이 이 프로젝트의 최종 목표이다.

### 3. 결항편 삭제 및 데이터 형태 변경
결항 데이터는 최종 예측 해야하는 지연여부에 영향을 끼치지 않았기 때문에 삭제하였으며, (년, 월, 일) 컬럼도 시기적으로 비행기 지연에 영향이 있다고 판단하여
범주형으로 바꾸었다.

``` r
afsnt <- afsnt %>% dplyr::filter(CNL == "N")
head(afsnt)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT      REG AOD IRR  STT
    ## 1   2017      1      1     일 ARP1 ARP3   A A1901 SEw3Nzc2   D   N 6:10
    ## 2   2017      1      1     일 ARP1 ARP3   A A1905 SEw4MjM2   D   N 6:15
    ## 3   2017      1      1     일 ARP1 ARP3   L L1751 SEw4MjM3   D   N 6:20
    ## 4   2017      1      1     일 ARP1 ARP3   F F1201 SEw4MjA3   D   N 6:25
    ## 5   2017      1      1     일 ARP3 ARP1   A A1900 SEw3NzAz   D   N 6:30
    ## 6   2017      1      1     일 ARP1 ARP3   H H1101 SEw4MDMx   D   N 6:30
    ##    ATT DLY DRR CNL CNR
    ## 1 6:18   N       N    
    ## 2 6:25   N       N    
    ## 3 6:30   N       N    
    ## 4 6:34   N       N    
    ## 5 6:37   N       N    
    ## 6 6:38   N       N

``` r
afsnt$SDT_YY <- as.factor(afsnt$SDT_YY)
afsnt$SDT_MM <- as.factor(afsnt$SDT_MM)
afsnt$SDT_DD <- as.factor(afsnt$SDT_DD)

head(afsnt)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT      REG AOD IRR  STT
    ## 1   2017      1      1     일 ARP1 ARP3   A A1901 SEw3Nzc2   D   N 6:10
    ## 2   2017      1      1     일 ARP1 ARP3   A A1905 SEw4MjM2   D   N 6:15
    ## 3   2017      1      1     일 ARP1 ARP3   L L1751 SEw4MjM3   D   N 6:20
    ## 4   2017      1      1     일 ARP1 ARP3   F F1201 SEw4MjA3   D   N 6:25
    ## 5   2017      1      1     일 ARP3 ARP1   A A1900 SEw3NzAz   D   N 6:30
    ## 6   2017      1      1     일 ARP1 ARP3   H H1101 SEw4MDMx   D   N 6:30
    ##    ATT DLY DRR CNL CNR
    ## 1 6:18   N       N    
    ## 2 6:25   N       N    
    ## 3 6:30   N       N    
    ## 4 6:34   N       N    
    ## 5 6:37   N       N    
    ## 6 6:38   N       N

### 4. 'Hour', 'count' 변수 생성
시간 별 각 공항에 얼마나 많은 항공기가 예정되어 있는지를 알려주는 변수를 생성한다.
예정 시각 변수에서 시간을 추출하여 'Hour' 변수를 만들고 이를 이용하여 'count'변수를 만든다.
``` r
afsnt$STT <- as.character(afsnt$STT)
b <- sapply(afsnt[, "STT"],  function(x) {x %>% str_split(pattern = ':')  %>% `[[`(1) })
HOUR <- b[1, ] %>% as.numeric()

afsnt$HOUR <- HOUR
afsnt$HOUR <- as.factor(afsnt$HOUR)

afsnt_count <- afsnt %>%
 dplyr::group_by(SDT_YY, SDT_MM, SDT_DD, ARP, HOUR) %>%
 dplyr::summarise('count' = n())

afsnt <- left_join(afsnt, afsnt_count, by = c("SDT_YY", "SDT_MM", "SDT_DD", "ARP", "HOUR"))

head(afsnt)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT      REG AOD IRR  STT
    ## 1   2017      1      1     일 ARP1 ARP3   A A1901 SEw3Nzc2   D   N 6:10
    ## 2   2017      1      1     일 ARP1 ARP3   A A1905 SEw4MjM2   D   N 6:15
    ## 3   2017      1      1     일 ARP1 ARP3   L L1751 SEw4MjM3   D   N 6:20
    ## 4   2017      1      1     일 ARP1 ARP3   F F1201 SEw4MjA3   D   N 6:25
    ## 5   2017      1      1     일 ARP3 ARP1   A A1900 SEw3NzAz   D   N 6:30
    ## 6   2017      1      1     일 ARP1 ARP3   H H1101 SEw4MDMx   D   N 6:30
    ##    ATT DLY DRR CNL CNR HOUR count
    ## 1 6:18   N       N        6    10
    ## 2 6:25   N       N        6    10
    ## 3 6:30   N       N        6    10
    ## 4 6:34   N       N        6    10
    ## 5 6:37   N       N        6     2
    ## 6 6:38   N       N        6    10

### 5. 시간 차 변수 생성
예정시각과 실제시각의 차이를 구하기 위하여 각 변수를 시간 변수로 바꾼 뒤 시간 차 변수(timediff)를 생성한다.


``` r
afsnt$STT <- strptime(as.character(afsnt$STT), format = "%H:%M")
afsnt$ATT <- strptime(as.character(afsnt$ATT), format = "%H:%M")
head(afsnt)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT      REG AOD IRR
    ## 1   2017      1      1     일 ARP1 ARP3   A A1901 SEw3Nzc2   D   N
    ## 2   2017      1      1     일 ARP1 ARP3   A A1905 SEw4MjM2   D   N
    ## 3   2017      1      1     일 ARP1 ARP3   L L1751 SEw4MjM3   D   N
    ## 4   2017      1      1     일 ARP1 ARP3   F F1201 SEw4MjA3   D   N
    ## 5   2017      1      1     일 ARP3 ARP1   A A1900 SEw3NzAz   D   N
    ## 6   2017      1      1     일 ARP1 ARP3   H H1101 SEw4MDMx   D   N
    ##                   STT                 ATT DLY DRR CNL CNR HOUR count
    ## 1 2019-09-17 06:10:00 2019-09-17 06:18:00   N       N        6    10
    ## 2 2019-09-17 06:15:00 2019-09-17 06:25:00   N       N        6    10
    ## 3 2019-09-17 06:20:00 2019-09-17 06:30:00   N       N        6    10
    ## 4 2019-09-17 06:25:00 2019-09-17 06:34:00   N       N        6    10
    ## 5 2019-09-17 06:30:00 2019-09-17 06:37:00   N       N        6     2
    ## 6 2019-09-17 06:30:00 2019-09-17 06:38:00   N       N        6    10


``` r
afsnt$timediff <- difftime(afsnt$ATT, afsnt$STT, units = "mins")
head(afsnt)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT      REG AOD IRR
    ## 1   2017      1      1     일 ARP1 ARP3   A A1901 SEw3Nzc2   D   N
    ## 2   2017      1      1     일 ARP1 ARP3   A A1905 SEw4MjM2   D   N
    ## 3   2017      1      1     일 ARP1 ARP3   L L1751 SEw4MjM3   D   N
    ## 4   2017      1      1     일 ARP1 ARP3   F F1201 SEw4MjA3   D   N
    ## 5   2017      1      1     일 ARP3 ARP1   A A1900 SEw3NzAz   D   N
    ## 6   2017      1      1     일 ARP1 ARP3   H H1101 SEw4MDMx   D   N
    ##                   STT                 ATT DLY DRR CNL CNR HOUR count
    ## 1 2019-09-17 06:10:00 2019-09-17 06:18:00   N       N        6    10
    ## 2 2019-09-17 06:15:00 2019-09-17 06:25:00   N       N        6    10
    ## 3 2019-09-17 06:20:00 2019-09-17 06:30:00   N       N        6    10
    ## 4 2019-09-17 06:25:00 2019-09-17 06:34:00   N       N        6    10
    ## 5 2019-09-17 06:30:00 2019-09-17 06:37:00   N       N        6     2
    ## 6 2019-09-17 06:30:00 2019-09-17 06:38:00   N       N        6    10
    ##   timediff
    ## 1   8 mins
    ## 2  10 mins
    ## 3  10 mins
    ## 4   9 mins
    ## 5   7 mins
    ## 6   8 mins

시간 차 변수가 0보다 작은 것은 조기 출발한 항공편으로 0으로 바꾸어 준다.
``` r
afsnt <- afsnt[!is.na(afsnt$ATT), ]
afsnt[(afsnt$timediff) < 0, "timediff"] <- 0
afsnt$timediff <- as.numeric(afsnt$timediff)
head(afsnt)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT      REG AOD IRR
    ## 1   2017      1      1     일 ARP1 ARP3   A A1901 SEw3Nzc2   D   N
    ## 2   2017      1      1     일 ARP1 ARP3   A A1905 SEw4MjM2   D   N
    ## 3   2017      1      1     일 ARP1 ARP3   L L1751 SEw4MjM3   D   N
    ## 4   2017      1      1     일 ARP1 ARP3   F F1201 SEw4MjA3   D   N
    ## 5   2017      1      1     일 ARP3 ARP1   A A1900 SEw3NzAz   D   N
    ## 6   2017      1      1     일 ARP1 ARP3   H H1101 SEw4MDMx   D   N
    ##                   STT                 ATT DLY DRR CNL CNR HOUR count
    ## 1 2019-09-17 06:10:00 2019-09-17 06:18:00   N       N        6    10
    ## 2 2019-09-17 06:15:00 2019-09-17 06:25:00   N       N        6    10
    ## 3 2019-09-17 06:20:00 2019-09-17 06:30:00   N       N        6    10
    ## 4 2019-09-17 06:25:00 2019-09-17 06:34:00   N       N        6    10
    ## 5 2019-09-17 06:30:00 2019-09-17 06:37:00   N       N        6     2
    ## 6 2019-09-17 06:30:00 2019-09-17 06:38:00   N       N        6    10
    ##   timediff
    ## 1        8
    ## 2       10
    ## 3       10
    ## 4        9
    ## 5        7
    ## 6        8

### 6.지연코드를 활용한 변수 생성
지연코드의 비율은 'C02'(AC접속불량)이 가장 많다. 이를 이용하여 '편명별 C02' 변수를 생성한다.
``` r
afsnt$DRR.group <- ifelse(afsnt$DRR == "C02",
                          "C02",
                          "Non-C02")

afsnt$DRR.group <- as.factor(afsnt$DRR.group)

head(afsnt)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT      REG AOD IRR DLY DRR
    ## 1   2017      1      1     일 ARP1 ARP3   A A1901 SEw3Nzc2   D   N   N    
    ## 2   2017      1      1     일 ARP1 ARP3   A A1905 SEw4MjM2   D   N   N    
    ## 3   2017      1      1     일 ARP1 ARP3   L L1751 SEw4MjM3   D   N   N    
    ## 4   2017      1      1     일 ARP1 ARP3   F F1201 SEw4MjA3   D   N   N    
    ## 5   2017      1      1     일 ARP3 ARP1   A A1900 SEw3NzAz   D   N   N    
    ## 6   2017      1      1     일 ARP1 ARP3   H H1101 SEw4MDMx   D   N   N    
    ##   CNL CNR HOUR count timediff DRR.group
    ## 1   N        6    10        8   Non-C02
    ## 2   N        6    10       10   Non-C02
    ## 3   N        6    10       10   Non-C02
    ## 4   N        6    10        9   Non-C02
    ## 5   N        6     2        7   Non-C02
    ## 6   N        6  

``` r
a <- afsnt %>% 
  dplyr::group_by(FLT) %>% 
  dplyr::summarise(n = n())
b <- afsnt %>% 
  dplyr::filter(DRR.group == "C02") %>% dplyr::group_by(FLT) %>% dplyr::summarise(n = n())
c <- as.data.frame(left_join(a, b, by = "FLT"))

c[is.na(c)] <- 0

colnames(c) <- c("FLT", "x", "y")
c$x <- as.numeric(c$x)
c$FLT_C02_ratio <-  (c$y/c$x)*100
colnames(c) <- c("FLT", "x", "y", "FLT_C02_ratio")
c <- c %>% select(-c(x,y))


head(c)
```

    ##     FLT FLT_C02_ratio
    ## 1 A1001      2.398524
    ## 2 A1002     36.229508
    ## 3 A1003     11.299435
    ## 4 A1004     12.109375
    ## 5 A1005     11.574074
    ## 6 A1006      7.421875

### 7.편명별 부정기 비율, 평균 지연시간, 지연율 변수를 추가한다.

``` r
f <- afsnt %>% 
  dplyr::group_by(FLT) %>% 
  dplyr::summarise(mean(timediff, na.rm = TRUE))

colnames(f) <- c("FLT", "FLT_meantimediff")
f$FLT_meantimediff <- as.numeric(f$FLT_meantimediff)
head(f)
```

    ## # A tibble: 6 x 2
    ##   FLT   FLT_meantimediff
    ##   <fct>            <dbl>
    ## 1 A1001             10.3
    ## 2 A1002             27.9
    ## 3 A1003             18.3
    ## 4 A1004             18.6
    ## 5 A1005             21.3
    ## 6 A1006             18.4


``` r
f <- afsnt %>% 
  dplyr::group_by(FLT) %>% 
  dplyr::summarise(mean(timediff, na.rm = TRUE))

colnames(f) <- c("FLT", "FLT_meantimediff")
f$FLT_meantimediff <- as.numeric(f$FLT_meantimediff)
head(f)
```

    ## # A tibble: 6 x 2
    ##   FLT   FLT_meantimediff
    ##   <fct>            <dbl>
    ## 1 A1001             10.3
    ## 2 A1002             27.9
    ## 3 A1003             18.3
    ## 4 A1004             18.6
    ## 5 A1005             21.3
    ## 6 A1006             18.4


``` r
a <- afsnt %>% 
  dplyr:::group_by(FLT) %>% 
  dplyr:::summarise(n = n())
b <- afsnt %>% dplyr:::filter(DLY == "Y") %>%
  dplyr:::group_by(FLT) %>% 
  dplyr:::summarise(n = n())
g <- as.data.frame(left_join(a, b, by = "FLT"))

g[is.na(g)] <- 0

colnames(g) <- c("FLT", "x", "y")
g$x <- as.numeric(g$x)
g$FLT_DLY_ratio <-  (g$y/g$x)*100
g <- g %>%  select(-c(x,y))
colnames(g) <- c("FLT","FLT_DLY_ratio")


m <- left_join(c, i, by = "FLT")
m <- left_join(m, f, by = "FLT")
m <- left_join(m, g, by = "FLT")
afsnt <- left_join(afsnt, m, by = "FLT")

head(afsnt)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT      REG AOD IRR DLY DRR
    ## 1   2017      1      1     일 ARP1 ARP3   A A1901 SEw3Nzc2   D   N   N    
    ## 2   2017      1      1     일 ARP1 ARP3   A A1905 SEw4MjM2   D   N   N    
    ## 3   2017      1      1     일 ARP1 ARP3   L L1751 SEw4MjM3   D   N   N    
    ## 4   2017      1      1     일 ARP1 ARP3   F F1201 SEw4MjA3   D   N   N    
    ## 5   2017      1      1     일 ARP3 ARP1   A A1900 SEw3NzAz   D   N   N    
    ## 6   2017      1      1     일 ARP1 ARP3   H H1101 SEw4MDMx   D   N   N    
    ##   CNL CNR HOUR count timediff DRR.group FLT_C02_ratio FLT_IRR_ratio
    ## 1   N        6    10        8   Non-C02     0.8324084     0.0000000
    ## 2   N        6    10       10   Non-C02     0.7168459     0.0000000
    ## 3   N        6    10       10   Non-C02     0.0000000     0.0000000
    ## 4   N        6    10        9   Non-C02     0.6365741     0.0000000
    ## 5   N        6     2        7   Non-C02     0.2257336     0.3386005
    ## 6   N        6    10        8   Non-C02     0.4540295     0.0000000
    ##   FLT_meantimediff FLT_DLY_ratio
    ## 1         8.551054      2.164262
    ## 2         9.765233      2.508961
    ## 3         8.232143      2.380952
    ## 4         9.984954      3.298611
    ## 5         6.276524      1.467269
    ## 6        10.640182      3.064699

### 8. 위에 생성한 변수를 예측 데이터인 afsnt_dly 데이터 프레임에도 생성한다.
위에 새로 만든 변수는 편명을 기준으로 하여 left_join 하여 붙여준다.
``` r
afsnt_dly <- left_join(afsnt_dly, m, by = "FLT")
```

    ## Warning: Column `FLT` joining factors with different levels, coercing to
    ## character vector

``` r
afsnt_dly$FLT <- as.factor(afsnt_dly$FLT)

head(afsnt_dly)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT AOD   STT DLY DLY_RATE
    ## 1   2019      9     16     월 ARP1 ARP3   L L1702   A  9:05  NA       NA
    ## 2   2019      9     16     월 ARP3 ARP1   L L1702   D  7:55  NA       NA
    ## 3   2019      9     16     월 ARP1 ARP3   L L1720   A 14:40  NA       NA
    ## 4   2019      9     16     월 ARP3 ARP1   L L1720   D 13:30  NA       NA
    ## 5   2019      9     16     월 ARP4 ARP3   L L1808   A 20:10  NA       NA
    ## 6   2019      9     16     월 ARP3 ARP4   L L1808   D 19:10  NA       NA
    ##   FLT_C02_ratio FLT_IRR_ratio FLT_meantimediff FLT_DLY_ratio
    ## 1      5.091312     1.8262313         12.28832      5.755396
    ## 2      5.091312     1.8262313         12.28832      5.755396
    ## 3     22.232472     0.1845018         20.32472     22.785978
    ## 4     22.232472     0.1845018         20.32472     22.785978
    ## 5     22.436604     0.2205072         21.50331     20.507166
    ## 6     22.436604     0.2205072         21.50331     20.507166

### 9. 예측 셋의 편명 별 비율 변수 결측치를 채운다
``` r
FLO_J <- afsnt %>% 
  dplyr::filter(FLO == "J") %>% 
  select(20:23) %>% 
  summarise(mean_FLT_CO2_ratio = median(FLT_C02_ratio),
            mean_FLT_IRR_ratio = median(FLT_IRR_ratio),
            mean_FLT_meantimediff = median(FLT_meantimediff),
            mean_FLT_DLY_ratio = median(FLT_DLY_ratio))

IRR_y <- afsnt %>% 
  dplyr::filter(IRR == "Y") 

IRR_yy <- IRR_y %>% 
  select(20:23) %>% 
  summarise(mean_FLT_CO2_ratio = median(FLT_C02_ratio),
            mean_FLT_IRR_ratio = 0,
            mean_FLT_meantimediff = median(FLT_meantimediff),
            mean_FLT_DLY_ratio = median(FLT_DLY_ratio))

afsnt_dly[afsnt_dly$FLO == "J" & afsnt_dly$FLT_C02_ratio %>% is.na(), c(13:16)] <- FLO_J

afsnt_dly[afsnt_dly$FLO == "M", c(13:16)] <- IRR_yy
```
### 10. afsnt와 마찬가지로 'Hour', 'count' 변수를 생성한다.
``` r
afsnt_dly$STT <- as.character(afsnt_dly$STT)
b <- sapply(afsnt_dly[, "STT"],  function(x) {x %>% str_split(pattern = ':')  %>% `[[`(1) })
HOUR_nd <- b[1, ] %>% as.numeric()

afsnt_dly$HOUR <- HOUR_nd
afsnt_dly$HOUR <- as.factor(afsnt_dly$HOUR)

afsnt_dly_count <- afsnt_dly %>%
 dplyr::group_by(SDT_YY, SDT_MM, SDT_DD, ARP, HOUR) %>%
 dplyr::summarise('count' = n())

afsnt_dly <- left_join(afsnt_dly, afsnt_dly_count, by = c("SDT_YY", "SDT_MM", "SDT_DD", "ARP", "HOUR"))

head(afsnt_dly)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP  ODP FLO   FLT AOD   STT DLY DLY_RATE
    ## 1   2019      9     16     월 ARP1 ARP3   L L1702   A  9:05  NA       NA
    ## 2   2019      9     16     월 ARP3 ARP1   L L1702   D  7:55  NA       NA
    ## 3   2019      9     16     월 ARP1 ARP3   L L1720   A 14:40  NA       NA
    ## 4   2019      9     16     월 ARP3 ARP1   L L1720   D 13:30  NA       NA
    ## 5   2019      9     16     월 ARP4 ARP3   L L1808   A 20:10  NA       NA
    ## 6   2019      9     16     월 ARP3 ARP4   L L1808   D 19:10  NA       NA
    ##   FLT_C02_ratio FLT_IRR_ratio FLT_meantimediff FLT_DLY_ratio HOUR count
    ## 1      5.091312     1.8262313         12.28832      5.755396    9    25
    ## 2      5.091312     1.8262313         12.28832      5.755396    7    24
    ## 3     22.232472     0.1845018         20.32472     22.785978   14    26
    ## 4     22.232472     0.1845018         20.32472     22.785978   13    30
    ## 5     22.436604     0.2205072         21.50331     20.507166   20     3
    ## 6     22.436604     0.2205072         21.50331     20.507166   19    28

### 11. Train set, Test set 샘플링
grid-search 결과 Train set 내 미지연과 지연 비율을 73:27로 맞추는 것이 AUROC가 가장 높게 나왔다.
다만 Test set은 예측해야 하는 기존 데이터 셋과 마찬가지로 87:13을 유지한다.

``` r
afsnt <- afsnt[ , -c(6,8,9,11,13,14,15,18,19)]

afsnt_dly$SDT_YY <- as.factor(afsnt_dly$SDT_YY)
afsnt_dly$SDT_MM <- as.factor(afsnt_dly$SDT_MM)
afsnt_dly$SDT_DD <- as.factor(afsnt_dly$SDT_DD)
afsnt_dly$HOUR <- as.factor(afsnt_dly$HOUR)

set.seed(seed = 123)

# 목적변수 비율 맞춰 샘플링 하기 
afsnt_Y <- afsnt[(afsnt$DLY == "Y"), ]
afsnt_N <- afsnt[(afsnt$DLY == "N"), ]

# 트레인 테스트 셋 샘플링하기 위해 다음과 같이 처리합니다. 
index <- sample(x = 1:2, 
                size = nrow(x = afsnt_N), 
                prob = c(0.7, 0.3), 
                replace = TRUE) 

trainSet_N <- afsnt_N[index == 2,]

index <- sample(x = 1:2, 
                size = nrow(x = afsnt_Y), 
                prob = c(0.8, 0.2), 
                replace = TRUE) 

trainSet_Y <- afsnt_Y[index == 1,]
trainSet <- rbind(trainSet_Y, trainSet_N)

# 테스트 데이터 셋은 실제 미지연과 지연 데이터 비율인 87:13에 맞춰 적합
testSet_N <- afsnt_N[index == 1,]
testSet_N <- sample_n(testSet_N, 160000)

testSet_Y <- afsnt_Y[index == 2, ]


testSet <- rbind(testSet_Y, testSet_N)

# 훈련용, 시험용 데이터셋의 목표변수 비중을 확인합니다.  
trainSet$DLY %>% table() %>% prop.table()
```

    ## .
    ##         N         Y 
    ## 0.7305399 0.2694601

``` r
testSet$DLY %>% table() %>% prop.table()
```

    ## .
    ##         N         Y 
    ## 0.8704925 0.1295075

### 12. 랜덤포레스트 모델 성능 확인
임의로 ntree = 50, mtry = 5로 모델 적합을 실시하여 개략적인 OOB와 미지연과 지연 예측 오류정도를 확인한다.

``` r
fitRFC <- randomForest(x = trainSet[, -8], 
                       y = trainSet[, 8], 
                       xtest = testSet[, -8], 
                       ytest = testSet[, 8], 
                       ntree = 50, 
                       mtry = 5, 
                       importance = TRUE, 
                       do.trace = 50, 
                       keep.forest = TRUE)
```

    ## ntree      OOB      1      2|    Test      1      2
    ##    50:  22.05% 11.91% 49.52%|  13.02%  7.59% 49.54%

``` r
# 모형 적합 결과를 확인합니다. 
print(x = fitRFC)
```

    ## 
    ## Call:
    ##  randomForest(x = trainSet[, -8], y = trainSet[, 8], xtest = testSet[,      -8], ytest = testSet[, 8], ntree = 50, mtry = 5, importance = TRUE,      do.trace = 50, keep.forest = TRUE) 
    ##                Type of random forest: classification
    ##                      Number of trees: 50
    ## No. of variables tried at each split: 5
    ## 
    ##         OOB estimate of  error rate: 22.05%
    ## Confusion matrix:
    ##        N     Y class.error
    ## N 227195 30717   0.1190988
    ## Y  47113 48018   0.4952434
    ##                 Test set error rate: 13.02%
    ## Confusion matrix:
    ##        N     Y class.error
    ## N 147859 12141  0.07588125
    ## Y  11793 12011  0.49542094
 
 ### 13. 혼동행렬, F1점수 RUROC(레이블 & 추정확률)로 모델의 성능을 확인
``` r
# testSet의 추정확률과 추정값(레이블)을 trProb, trPred에 할당합니다. 
trProb <- fitRFC$test$votes[, 2]
trPred <- fitRFC$test$predicted

# 시험셋의 실제값을 trReal에 할당합니다. 
trReal <- testSet$DLY


# 혼동행렬을 출력합니다. 
confusionMatrix(data = trPred, reference = trReal, positive = 'Y')
```

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction      N      Y
    ##          N 147859  11793
    ##          Y  12141  12011
    ##                                           
    ##                Accuracy : 0.8698          
    ##                  95% CI : (0.8682, 0.8713)
    ##     No Information Rate : 0.8705          
    ##     P-Value [Acc > NIR] : 0.8177          
    ##                                           
    ##                   Kappa : 0.426           
    ##  Mcnemar's Test P-Value : 0.0249          
    ##                                           
    ##             Sensitivity : 0.50458         
    ##             Specificity : 0.92412         
    ##          Pos Pred Value : 0.49731         
    ##          Neg Pred Value : 0.92613         
    ##              Prevalence : 0.12951         
    ##          Detection Rate : 0.06535         
    ##    Detection Prevalence : 0.13140         
    ##       Balanced Accuracy : 0.71435         
    ##                                           
    ##        'Positive' Class : Y               
    ## 

``` r
# F1 점수를 확인합니다.
F1Score(y_pred = trPred, y_true = trReal, positive = 'Y')
```

    ## [1] 0.5009175

``` r
# 추정레이블로 ROC 그래프를 그리고 AUROC를 확인합니다. 
getROC(real = trReal, pred = trPred)
```
![](/images/demo/lable.png)

``` r
# 추정확률로 ROC 그래프를 그리고 AUROC를 확인합니다. 
getROC(real = trReal, pred = trProb)
```
![](/images/demo/prb.png)

### 14. afsnt_dly 예측 train set 적합
afsnt 전체 데이터 미지연과 지연 비율을 73:27로 맞춘 뒤, 그리드 서칭 결과 최적의 파라미터로 평가 된 ntree = 500, mtry = 5를 할당하여 모델 적합
afsnt_dly 데이터에는 존재하지만 train set 인 afsnt 데이터는 존재하는 편명 때문에 "FLO" 변수는 제거한다.
``` r
#위와 마찬가지로 미지연과 지연의 비율을 73:27로 맞춰 학습셋 적합
afsnt_Y <- afsnt[(afsnt$DLY == "Y"), ]
afsnt_N <- afsnt[(afsnt$DLY == "N"), ]

# 존재하는 지연 데이터 개수에 맞는 미지연 데이터(32150행) 추출 후 바인딩
trainSet_N <- sample_n(afsnt_N, 321560)
trainSet <- rbind(afsnt_Y, trainSet_N)
afsnt_dly <- afsnt_dly[c(colnames(trainSet))]

# 랜덤 포레스트 분류모형을 적합합니다. (그리드 서칭 결과 최적의 ntree, mtry 결정)
# 다른 범주가 존재하는 'FLO' 변수 제거
fitRFC <- randomForest(x = trainSet[, -c(6,8)], 
                       y = trainSet[, 8], 
                       xtest = afsnt_dly[, -c(6,8)], 
                       ntree = 500, 
                       mtry = 5, 
                       importance = TRUE, 
                       do.trace = 50, 
                       keep.forest = TRUE)
```

    ## ntree      OOB      1      2
    ##    50:  21.91% 11.63% 49.70%
    ##   100:  21.08% 10.66% 49.24%
    ##   150:  20.79% 10.34% 49.05%
    ##   200:  20.66% 10.17% 49.02%
    ##   250:  20.58% 10.07% 49.00%
    ##   300:  20.53% 10.03% 48.91%
    ##   350:  20.47%  9.97% 48.85%
    ##   400:  20.42%  9.92% 48.80%
    ##   450:  20.42%  9.91% 48.85%
    ##   500:  20.40%  9.89% 48.82%

#### 15. 최종 예측 결과(DLY, DLY_RATE) 확인 후 데이터 프레임 결합
``` r
trProb <- fitRFC$test$votes[, 2]
trPred <- fitRFC$test$predicted

final <- cbind(trPred, trProb) %>% as.data.frame()
colnames(final) <- c("DLY", "DLY_RATE")
final$DLY <- as.factor(final$DLY)
final$DLY = revalue(final$DLY, replace=c("1"="N", "2"="Y"))

# 기존 afsnt_dly에 예측한 DLY, DLY_RATE 컬럼 추가
afsnt_dly_final <- cbind(afsnt_dly, final)

head(afsnt_dly_final)
```

    ##   SDT_YY SDT_MM SDT_DD SDT_DY  ARP FLO AOD DLY HOUR count FLT_C02_ratio
    ## 1   2019      9     16     월 ARP1   L   A  NA    9    25      5.091312
    ## 2   2019      9     16     월 ARP3   L   D  NA    7    24      5.091312
    ## 3   2019      9     16     월 ARP1   L   A  NA   14    26     22.232472
    ## 4   2019      9     16     월 ARP3   L   D  NA   13    30     22.232472
    ## 5   2019      9     16     월 ARP4   L   A  NA   20     3     22.436604
    ## 6   2019      9     16     월 ARP3   L   D  NA   19    28     22.436604
    ##   FLT_IRR_ratio FLT_meantimediff FLT_DLY_ratio DLY DLY_RATE
    ## 1     1.8262313         12.28832      5.755396   N    0.062
    ## 2     1.8262313         12.28832      5.755396   N    0.272
    ## 3     0.1845018         20.32472     22.785978   N    0.346
    ## 4     0.1845018         20.32472     22.785978   Y    0.842
    ## 5     0.2205072         21.50331     20.507166   N    0.346
    ## 6     0.2205072         21.50331     20.507166   Y    0.560

### The End

