---
title: 'Topic modeling about recent military research trend'
date: 2019-09-28 00:00:00
featured_image: '/images/demo/mili.jpg'
excerpt: This page is a demo that shows everything you can do inside portfolio and blog posts. We've included everything you need to create engaging posts about your work, and show off your case studies in a beauti
---
![](/images/demo/mili.jpg)


### LDA기법을 활용한 국내 최근 군사보안 관련 연구 경향 분석
#### 국내 군사보안 관련 연구 주제 경향을 파악하기 위하여 진행한 프로젝트이다. KCI에 등록한 '군사보안' 키워드로 검색하여 나온 201개의 논문의 초록을 활용하여 토픽모델링을 진행하였다. 초록을 대상으로 한 이유는 초록에 핵심 키워드가 존재하기 때문에 연구 주제를 파악하는데 부족함이 없다고 판단했기 때문이다. 


### 라이브러리 설치

``` r
library(extrafont)
library(rJava)
library(NLP4kec)
library(NLP)
library(tm)
library(topicmodels)
library(LDAvis)
library(servr)
library(readr)
library(slam)
library(RcppMeCab)
library(Rmpfr)
library(tidyverse)
```

### 데이터 불러오기 및 작업환경 설정
#### 문서는 사전에 data.frame 형식으로 전처리하여 저장해두었다. 전처리 과정은 생략하도록 한다.

``` r
knitr::opts_chunk$set(error = TRUE)
setwd("~/Desktop/gsi/schoolproject/")
load("blank.RData")
head(blank,5)
```

    ##   doc
    ## 1   1
    ## 2   2
    ## 3   3
    ## 4   4
    ## 5   5

    ##text
    ## 1 사이버상에서 발생할 수 있는 범죄는 행위자가 누구이냐에 따라서 사인인 경우와 국가인 경우로 나눌 수 있고, 국가에 의한 사이버범죄는 정보전으로서 국가의 활동과 관련된다. 사인에 의한 사이버범죄는 행위지와 결과발생지가 각각 다름으로 인하여 형사관할권의 문제，죄형법정주의의 문제 및 형사사법공조의 문제가 있다. 반면에 정보전은 악성 컴퓨터프로그램으로 행한 다른 국가의 컴퓨터네트워크에 대한 공격이 UN헌장 제2조 4항에서 금지하고 있는 무력행사에 포함되는지 여부이다. 전시에 행해지는 정보전은 전쟁의 수단과 방법의 일 부분으로 볼 수 있으므로 평시에 전개되는 정보전으로 한정하면, 첫째, UN헌장 제2조 4항에서 금지하고 있는 무력행사의 의미가 기동력이 있는 물리적인 힘으로 표출되는 다양한 형태의 군사활동으로 볼 수 있다. 둘째, 다른 국가의 컴퓨터네트워크에 대한 공격도 4가지의 근거 및 5가지의 전제조건하에 무력의 행사로 볼 수 있다. 셋째，침략의 정의에 관한 UN총회의 결의에 따르면 최초의 무력의 행사는 침략 개시의 증거로 보고 있으므로 다른 국가의 컴퓨터네트워크에 대한 공격이 무력행사를 의미할 때 그것이 침략행위를 의미하는지 의문이 제기된다. 넷째，컴퓨터바이러스는 비살상무기로 분류되고，비살상무기로 침략행위가 개시된 사례가 없으므로 컴퓨터네트워크에 대한 공격 그 자체는 침략행위로 보기 어렵다. 다섯째, 다른 국가의 컴퓨터네트워크에 대한 공격이 5가지의 조건하에 무력의 행사로 본다고 하더라도 비례성의 원칙에 견지에서 보면 재래식무기로 자위조치도 할 수 없다. 여섯째，현재까지 사이버공격으로 발생한 문제는 인명살상이 아니라 물질적 손실로 한정되고 또 모든 국가가 국제분쟁의 평화적 해결의무를 부담하고 있으므로 평화적인 방법으로 해결할 수 있다. 일곱째，구체적으로 사이버공격과 관련된 범죄행위는 국제불법행위로서 국가책임과 개인에 대한 형사책임의 이론으로 처리가 가능하다. 현행 국제법의 패러다임은 주권과 영토의 개념이 전제가 된 국가를 기초로 한 구조로 되어 있다. 이러한 패러다임은 가상의 공간에서 또는 가상의 공간을 통하여 발생할 수 있는 국가의 범죄에 관련된 문제는 현행 국제법으로 규율할 수 없다. 따라서 인터넷을 통하여 형성된 사이버공간에 대한 새로운 패러다임의 형성이 필요하다.

    ## 2 현재 국방부에서는 국방정보체계간의 상호운용성을 보장하기 위해 아키텍처 산출물을 쉽고 일관성 있게 개발할 수 있는 국방아키텍처프레임워크를 개발하고 있다. 따라서, 개발된 아키텍처 산출물을 저장하여 재사용하고 국방 전반의 아키텍처 정보의 교환, 비교, 통합을 용이하게 하는 핵심아키텍처데이터모델(CADM)의 개발이 필요하다. CADM은 국방아키텍처프레임워크에서 도출된 데이터 요구사항을 통해 엔티티를 추출하여 관계를 정의하였으며 실사례를 통해 엔티티를 검증하였다. 실사례로는 군사정보통합처리체계의 아키텍처 산출물을 작성하여 그들의 아키텍처 데이터를 저장소에 저장한 후 다양한 쿼리를 통해 검증하였다. CADM을 통해 전군의 아키텍처에 대한 공통의 데이터 모델을 제공하여 국방정보체계에 대한 통합적인 정보자원관리와 상호운용 및 통합을 향상시킬 수 있다.

    ## 3 미래 전쟁양상이 첨단 정보전력체계 중심의 정보전이 될 것이나 이러한 정보전력체계 투자효과의 계량적인 평가방법이 부재한 실정이다. 본 연구에서는 미래 전쟁의 핵심인 정보전력체계의 투자비용에 대한 계량적인 효과평가 방법을 제시하였다. 이를 위해 우선 군의 정보화 및 정보전력체계 개념을 이해하고 기업 및 공공기관 정보화사업 효과평가방법과의 차이점을 비교분석한 후, 이를 기초로 국방정보화사업에 대한 계량적 효과평가방법 두 가지를 제안하였다. 첫째는 Schutzer의 C2효과분석모델을 이용한 방법이며, 둘째는 AHP기법을 이용한 방법이다. 이 두 방법에 의해 산출된 효과를 이용하여 비용대효과분석 결과도 제시하였다. 본 연구를 통한 기대효과는 점차 규모가 확대되고 있는 국방정보화사업 투자비용의 계량적 효파평가를 통하여 보다 효율적인 국방정보전력체계 획득에 기여할 수 있다는 것이다.

    ## 4 이 글은 정보실패의 개념, 유형, 그리고 그 의미를 규명하는데 목적을 두고 쓰여졌다. 이러한 목적을 위해 정보실패의 사례와 이론에 대한 기존 연구들을 검토해보고 이를 바탕으로 정보실패의 유형과 원인에 관한 인과관계를 이론적 분석틀로써 정립해 보았으며, 이를 부시 행정부 동안 발행한 2개의 사례들에 경험적으로 적용해보았다. 정보실패는 정책결정권자로 하여금 왜곡된 정책결정을 유도함으로써 국가의 생존과 번영에 치명적인 손실을 야기할 수 있는 심각한 문제로 인식된다. 정보실패는 크게 정보기관의 실책에서 비롯된 정보적 실패와 정보의 정치화, 정책결정권자의 오판 등 정보기관 외에 다른 외적 요소들에서 비롯된 ‘정보외적 실패’(non-intelligence failure)로 구분될 수 있겠다. 물론 정보실패의 상당 부분은 정보기관의 실책에서 비롯된다. 그런데, 외견상 정보실패로 보이지만 실상을 깊이 따져보면 정보기관의 잘못이라기보다는 ‘정보외적 요소들’(non- intelligence elements) 때문에 실패하게 되는 경우도 많다. 9.11 테러 사건은 분명히 정보기관의 실책에서 비롯된 ‘정보적 실패’로 규정될 수 있겠지만, 이라크 대량살상무기에 관한 정보판단의 왜곡은 정보공동체의 실책 이상으로 부시 대통령과 핵심 관료들이 정치적 목적을 위해 정보를 조작했을 가능성이 큰 것으로 평가되었다. 그러므로 정보실패의 많은 부분은 정책적인 대응이 미흡했거나 정보를 자신의 정치적 목적에 이용한 정책결정권자에게 있다고 보아야 할 것이다.

    ## 5 정보화 사회가 도래하고 기업 간의 경쟁이 치열해지면서 효율적인 정보기술 사용의 필요성이 증대되고 있다. 군에서도 정보화를 중요한 전력강화의 수단으로 간주하고 정보화 분야에 많은 투자가 이루어지고 있으며 정보화 평가의 중요성이 확대되고 있다. 이에 본 연구는 군장병의 정보화 수준을 진단․평가하여 정보화 수준과 성숙 수준을 제시하는 군장병 정보화 수준 평가시스템을 제시하고자 한다. 군장병 정보화 수준 평가시스템은 업무별･직급별로 요구되는 정보 능력을 도출하고 정보화 수준 평가를 시행하여 필요한 기술, 지식 등을 제시함으로써 다음 정보화 성숙단계로 발전하기 위해 요구되는 결정요인(KPI)을 제시한다. 주기적인 정보화 능력 평가를 통해 업무수행의 효율성을 향상시킬 수 있으며 정보화 수준 평가자료로 활용할 수 있다.

### 형태소 분석(Parsing)

``` r
# 초록별 형태소 이어 붙이기
doc <- levels(blank$doc)

# 최종 담을 데이터
word <- data.frame()

# 초록마다 형태소 분석한거 한줄로 만들어줌
  for(i in 1:length(doc)) {
    cat(i, '번째 진행중! \n')
    # 해당 초록 정보 불러오기
    info <- blank %>%
      dplyr::filter(doc == blank$doc[i])
    
    # 해당 초록에 대한 내용이 없다면 무시하고 진행 
    if(is_empty(is.na.data.frame(info))) next
    
    # 해당 초록 내용의 원하는 품사만 가져오기
    parse <- pos(info$text, format = 'data.frame') %>% filter(pos %in% c('NNP','NNG','VV','VA'))
    
    # UTF-8 변경이 필요한 경우
    if(is_empty(is.na.data.frame(parse))) {
      parse <- pos(iconv(info$text, to = 'UTF-8'), format = 'data.frame') %>% 
        filter(pos %in% c('NNP','NNG','VV','VA'))
    }
    
    tryCatch({
      # 내용 한줄로 이어 붙이기
      abstract <- ''
      for(j in 1:nrow(parse)) {
        abstract <- str_c(abstract, parse$token[j], ' ')
      }
    }, error = function(e) cat('-> 에러가 발생했습니다!\n'))
    
    # 뒤에 단어 완성 시켜주는 파싱
    # ex) 알 -> 알다, 잊 -> 잊다
    parse2 <- r_parser_r(contentVector = abstract, language = 'ko')
    
    # 데이터 합치기
    word <- rbind(word, data.frame(doc[j], parse2))
  }
```

    ## 1 번째 진행중! 
    ## 2 번째 진행중! 
    ## 3 번째 진행중! 
    ## 4 번째 진행중! 
    ##     .
    ##     .
    ##     .
    ## 198 번째 진행중! 
    ## 199 번째 진행중! 
    ## 200 번째 진행중! 
    ## 201 번째 진행중!

### 불용어 사전 제작
#### 불용어 사전은 사전에 '있다', '없다' 등과 같은 연구 경향 파악에 필요 없는 동사나, '정보'와 같은 모든 문서에 포괄적으로 분포하는 단어를 고려하여 제작하였다.

``` r
# 불용어 사전 (향후 제작)
stopWD <- read.csv(file = 'stopwords.csv', header = TRUE)
colnames(stopWD) <- "stop"
stopWD <- as.data.frame(stopWD)
stopWD$stop <- as.character(stopWD$stop)
```

### Document Term Matrix 만들기


``` r
# 말뭉치 만들기
corpus <- word$parse2 %>% VectorSource() %>% VCorpus()

# DTM 만들기
dtm <- DocumentTermMatrix(x = corpus,
                          control = list(removeNumbers = TRUE,
                                         removePunctuation = TRUE,
                                         stripWhitespace = TRUE,
                                         stopwords = stopWD$stop,
                                         wordLengths = c(2, Inf)))

# 단어(=컬럼명) 양옆의 공백을 제거합니다.
colnames(x = dtm) <- trimws(x = colnames(x = dtm), which = 'both')

# 단어의 수가 많으므로 term-frequency가 희박한(sparse) 컬럼을 제거하는 방식으로 
# dtm의 차원을 줄입니다. 
# sparse의 인자가 작을수록 dtm의 열 개수가 크게 줄어듭니다. 
dtm <- removeSparseTerms(x = dtm, sparse = as.numeric(x = 0.97))

# 차원을 확인합니다.
dim(x = dtm)
```

#### 총 201개의 문서에 대하여 389개의 핵심 단어가 추출 된 것을 알 수 있다.
    ## [1] 201 389

``` r
inspect(dtm)
```

    ## <<DocumentTermMatrix (documents: 201, terms: 389)>>
    ## Non-/sparse entries: 6577/71612
    ## Sparsity           : 92%
    ## Maximal term length: 5
    ## Weighting          : term frequency (tf)
    ## Sample             :
    ##      Terms
    ## Docs  국가 기관 기술 분석 사이버 시스템 안보 연구 전쟁 체계
    ##   106    7    0    0    0     17      0    7    1    1    0
    ##   119    6    2    0    0     13      0   10    0    0    1
    ##   121    0    0    2    0      0     19    0    3    0    0
    ##   124    3    0    0    4      0      1    0    9    2    0
    ##   136    9    0    1    0      5      0    1    0    0    0
    ##   164    0    0    0    5      0      0    0    3    0    0
    ##   170    8    1    0    1     21      0   17    2    0    4
    ##   171   13    1    0    3      9      0   12    0    0    0
    ##   178    4    5    0    1     15      0   13    0    0    1
    ##   183    6    5    0    1      0      0    1    1    0    0

### LDA기법 활용한 토픽 추출 (7개 토픽, 토픽별 10개의 대표 단어)

``` r
# 인자값
control_LDA_Gibbs = list(# TopicModelcontrol 
seed          = 100, # 같은 결과 값이 나오게하는 임의의 난수 생성
verbose       = 0,   # 값이 양의 정수이면 진행 마다 진행률이 나타나고 0이면 나타나지 않는다
save          = 0,   # 값이 양의 정수이면 진행 마다 모델이 저장되고 0이면 저장되지 않는다
prefix        = tempfile(), # 중간결과를 저장할 장소
nstart        = 1,   # 반복이 시작되는 숫자
best          = TRUE, # TRUE이면 사후최대 가능도를 반환
keep          = 0,   # 값이 양의 정수이면 진행 마다 로그-가능도가 매 반복마다 저장된다
estimate.beta = TRUE, # 주제가 정해져있으면 FALSE, 주제가 정해져있지 않으면 TRUE
    
# LDAcontrol
alpha  = 0.1, # 초기 알파 값
delta  = 0.1, # 초기 델타 값
iter   = 5000, # Gibbs iteration 반복 횟수
thin   = 2000, # 중간에 생략 된 Gibbs 반복 횟수, 기본적으로 iter와 같습니다.
burnin = 0) # 처음에 생략 된 Gibbs 반복 횟수
  
# Gibbs방식을 이용한 LDA, k = topic 숫자(임의로 10개 토픽으로 고정)                      
lda_tm = LDA(dtm, 7, method = "Gibbs", control = control_LDA_Gibbs)
  
# 토픽별 핵심단어 10개 저장하기
term_topic = terms(lda_tm, 10)
print(term_topic)
```

    ##       Topic 1 Topic 2  Topic 3 Topic 4 Topic 5  Topic 6    Topic 7 
    ##  [1,] "전쟁"  "국가"   "문제"  "활동"  "연구"   "전투"     "체계"  
    ##  [2,] "미국"  "사이버" "국제"  "안보"  "분석"   "네트워크" "연구"  
    ##  [3,] "한국"  "안보"   "활용"  "기관"  "조직"   "시스템"   "개발"  
    ##  [4,] "정책"  "위협"   "비밀"  "테러"  "시스템" "전술"     "시스템"
    ##  [5,] "실패"  "개인"   "원칙"  "방첩"  "평가"   "체계"     "기술"  
    ##  [6,] "정치"  "공간"   "포함"  "필요"  "영향"   "데이터"   "제시"  
    ##  [7,] "북한"  "미국"   "절차"  "변화"  "결과"   "운용"     "모델"  
    ##  [8,] "일본"  "범죄"   "문서"  "공유"  "활용"   "기반"     "효율"  
    ##  [9,] "기관"  "공격"   "적용"  "위협"  "수준"   "환경"     "적용"  
    ## [10,] "작전"  "규정"   "보장"  "대응"  "기술"   "통신"     "분야"
#### 7개의 토픽들은 그 특성을 담고 있는 키워드를 통해 설명될 수 있다. 토픽 ①(보안 위협)은 국가의 안보를 위협하는 요소들을 규정하고 분석하는 연구 분야이다. 실제 공간과 사이버 공간을 망라하여 보안을 위협하는 공격ㆍ범죄 등을 아우르는 분야다. 토픽 ②(위협 대응)는 토픽 ①에서 언급한 보안 위협이나 보안환경의 변화에 대응하기 위한 주요기관의 활동을 다룬 분야다. 대테러ㆍ방첩 등의 범주도 이에 속한다. 토픽 ③(법규ㆍ제도)은 보안을 다룬 법규나 제도에 대한 연구분야다. 보안과 관련된 원칙이나 절차를 다루고 있으며, 비밀 등 문서보안과 관련된 범주 역시 취급 절차를 규정하고 있다는 점에서 여기에 포함된다. 토픽 ④(국가별 비교)는 주요 국가들의 사례나 정책을 비교하거나 분석하는 과정을 통해 개선방향을 모색하는 연구분야로, 키워드에 언급된 한국ㆍ북한ㆍ미국ㆍ일본 등의 정책이나 전쟁ㆍ작전 사례가 주로 다뤄지고 있는 것으로 보인다. 토픽 ⑤(기술ㆍ개발)는 보안 체계ㆍ시스템을 효율적으로 발전시키기 위해 새로운 기술을 개발하거나, 그 기술을 적용하여 새로운 모델을 구상하는 연구다. 토픽 ⑥(체계 운용)은 토픽 ⑤를 통해 개발된 체계ㆍ시스템을 실제 운용하는 활동에 대한 연구범주다. 데이터ㆍ통신ㆍ네트워크 등의 수단을 기반으로 하고 있으며, 특히 군에서 폭넓게 활용되는 전투ㆍ전술과 관련된 프로그램에 대한 연구들이 포함되어있다. 토픽 ⑦(분석ㆍ평가)은 보안과 관련된 조직ㆍ시스템ㆍ기술이 제대로 활용되고 있는지, 그 영향력은 어느 정도인지를 분석하고 평가하는 카테고리다. 토픽 ⑤ㆍ⑥ㆍ⑦은 보안 체계ㆍ시스템이 개발 → 운용 → 평가되는 일련의 과정이 고스란히 토픽 분류에 반영되어있다고 판단할 수 있는 부분이다.

#### 토픽 모델링 시각화

``` r
theme_set(theme_gray(base_family="AppleGothic"))
phi = posterior(lda_tm)$term %>% as.matrix

theta = posterior(lda_tm)$topics %>% as.matrix

# vocab는 전체 단어 리스트 입니다.
vocab = colnames(phi)

# 각 단어별 빈도수를 구합니다.
new_dtm = as.matrix(dtm)
freq_matrix = data.frame(word = colnames(new_dtm),
                         Freq = colSums(new_dtm))

# bar plot으로 빈도수 확인하기
theme_set(theme_gray(base_family="AppleGothic"))
freq_matrix %>% 
  arrange(desc(Freq)) %>% 
  head(10) %>% 
  ggplot(mapping = aes(x = reorder(word, -Freq), y = Freq, fill = word)) +
  geom_bar(stat = "identity")
```

#### 가장 빈도수가 높은 10개의 단어를 볼 수 있다.
![](/images/demo/lda.png)


``` r
## LDA 시각화 ##
q_lda <- LDA(dtm, k=7, seed=1234)

q_topics <- tidy(lda_tm, matrix='beta')
q_top_terms <- q_topics %>%
  group_by(topic) %>%
  top_n(10, beta) %>%
  ungroup() %>%
  arrange(topic, -beta)
  q_top_terms %>%
  mutate(term=reorder(term, beta)) %>%
  ggplot(aes(term, beta, fill=factor(topic))) +
  geom_col(show.legend=FALSE) +
  facet_wrap(~ topic, scales='free') +
  coord_flip() +
  theme(axis.text.y=element_text(family='Apple SD Gothic Neo'))
```
#### 각 토픽별 주요 단어를 볼 수 있다.
![](/images/demo/Rplot.jpeg)

### LDAvis를 활용한 시각화

#### LDAvis 는 두 가지 정보를 출력합니다. 첫째는 HTML 의 왼쪽에 출력되는 topic 의 2 차원 embedding vector 입니다. 비슷한 위치에 존재하는 토픽들은 서로 비슷한 문맥을 지니고 있습니다. 그리고 둘째는 오른쪽에 출력되는 각 토픽의 키워드 입니다. 이들의 원리를 살펴봅니다.

### 2-D visualization of topic vectors

#### 토픽은 단어 개수의 차원을 지니고 있습니다. 이를 2 차원으로 압축하기 위해서는 차원 축소 방법이 이용되어야 합니다. LDAvis 는 Principal Component Analysis (PCA) 를 이용하여 n\_terms 차원의 벡터들을 2 차원으로 압축합니다.

``` r
## ldavis 사용해보기
G <- 5000
alpha <- 0.02

#fit <- LDA(dtm, k=7, method='Gibbs', control=list(iter=G, alpha=alpha))
lda_tm = LDA(dtm, 7, method = "Gibbs", control = control_LDA_Gibbs)

phi <- posterior(lda_tm)$terms %>% as.matrix
theta <- posterior(lda_tm)$topics %>% as.matrix
vocab <- colnames(phi)
doc_length <- c()
for(i in 1:length(corpus)) {
  temp <- paste(corpus[[i]]$content, collapse=" ")
  doc_length <- c(doc_length, stri_count(temp, regex='\\S+'))
}
temp_frequency <- as.matrix(dtm)
freq_matrix <- data.frame(ST=colnames(temp_frequency),
                          Freq=colSums(temp_frequency))
rm(temp_frequency)

json_lda <- createJSON(phi=phi,
                       theta=theta,
                       vocab=vocab,
                       doc.length=doc_length,
                       term.frequency=freq_matrix$Freq)
serVis(json_lda, out.dir='vis', open.browser=TRUE)
```
#### 아래 시각화를 통해 각 토픽의 연관성과 출현율(prevalence)을 파악할 수 있다. 출현율이란 특정 토픽에서 사용된 용어가 전체 토픽에서는 얼마나 전반적으로 사용되고 있는가를 나타내는 정도다. 각 토픽은 원 형태로 그려지며, 출현율의 값이 클수록 원의 면적이 더욱 넓어진다. 일례로, ‘기술ㆍ개발’의 원이 ‘위협 대응’의 원보다 큰 것으로 미루어보아, ‘기술ㆍ개발’과 관련된 연구가 ‘위협 대응’에 관한 연구보다 출현빈도가 높다는 것을 알 수 있다. 또한, 토픽과 토픽 사이의 거리는 토픽간의 연관성을 나타낸다. 지근거리에 있을수록 밀접한 관계에 있는 토픽들이며, 멀리 떨어져 있는 토픽들은 상호 독립성이 강하다고 할 수 있다.

![](/images/demo/1.png)

![](/images/demo/2.png)

![](/images/demo/3.png)

![](/images/demo/4.png)

![](/images/demo/5.png)

![](/images/demo/6.png)

![](/images/demo/7.png)

#### 본 연구에서 추출된 7개의 토픽들이 그 성격의 유사성에 따라 크게 3개 군집을 이루고 있는 모습을 볼 수 있다. 보안 위협이 무엇인지 규정하고, 그 위협에 대응하고, 그 위협 및 대응이 국가별로 어떠한지 분석하는 토픽 ①(보안 위협)ㆍ②(위협 대응)ㆍ④(국가별 비교)가 서로 근접거리에 위치하고 있으며, 보안 관련 체계ㆍ시스템을 개발하고 운용하고 평가하는 일련의 과정인 토픽 ⑤(기술ㆍ개발)ㆍ⑥(체계 운용)ㆍ⑦(분석ㆍ평가) 또한 서로 가까운 거리를 유지하고 있다.

