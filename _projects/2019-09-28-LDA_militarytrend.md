---
title: 'Topic modeling about recent military research trend'
date: 2019-09-28 00:00:00
featured_image: '/images/demo/mili.jpg'
excerpt: This page is a demo that shows everything you can do inside portfolio and blog posts. We've included everything you need to create engaging posts about your work, and show off your case studies in a beauti
---

### 라이브러리 설치
```{r}
library(extrafont)
library(tidyverse)
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
```


### 데이터 불러오기 및 작업환경 설정
```{r}
knitr::opts_chunk$set(error = TRUE)
setwd("~/Desktop/gsi/schoolproject/")
load("blank.RData")
head(blank,5)
```

### 형태소 분석(Parsing)
```{r}
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
### 불용어 사전 제작
```{r}
# 불용어 사전 (향후 제작)
stopWD <- read.csv(file = 'stopwords.csv', header = TRUE)
colnames(stopWD) <- "stop"
stopWD <- as.data.frame(stopWD)
stopWD$stop <- as.character(stopWD$stop)

```


### Document Term Matrix 만들기
```{r}
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
inspect(dtm)
```


### 불용어 사전 만들기 위한 토픽 추출
```{r}
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

#### 토픽 모델링 시각화
```{r echo=TRUE}
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

#### 7개의 토픽들은 그 특성을 담고 있는 키워드를 통해 설명될 수 있다. 토픽 ①(보안 위협)은 국가의 안보를 위협하는 요소들을 규정하고 분석하는 연구 분야이다. 실제 공간과 사이버 공간을 망라하여 보안을 위협하는 공격ㆍ범죄 등을 아우르는 분야다. 토픽 ②(위협 대응)는 토픽 ①에서 언급한 보안 위협이나 보안환경의 변화에 대응하기 위한 주요기관의 활동을 다룬 분야다. 대테러ㆍ방첩 등의 범주도 이에 속한다. 토픽 ③(법규ㆍ제도)은 보안을 다룬 법규나 제도에 대한 연구분야다. 보안과 관련된 원칙이나 절차를 다루고 있으며, 비밀 등 문서보안과 관련된 범주 역시 취급 절차를 규정하고 있다는 점에서 여기에 포함된다. 토픽 ④(국가별 비교)는 주요 국가들의 사례나 정책을 비교하거나 분석하는 과정을 통해 개선방향을 모색하는 연구분야로, 키워드에 언급된 한국ㆍ북한ㆍ미국ㆍ일본 등의 정책이나 전쟁ㆍ작전 사례가 주로 다뤄지고 있는 것으로 보인다. 토픽 ⑤(기술ㆍ개발)는 보안 체계ㆍ시스템을 효율적으로 발전시키기 위해 새로운 기술을 개발하거나, 그 기술을 적용하여 새로운 모델을 구상하는 연구다. 토픽 ⑥(체계 운용)은 토픽 ⑤를 통해 개발된 체계ㆍ시스템을 실제 운용하는 활동에 대한 연구범주다. 데이터ㆍ통신ㆍ네트워크 등의 수단을 기반으로 하고 있으며, 특히 군에서 폭넓게 활용되는 전투ㆍ전술과 관련된 프로그램에 대한 연구들이 포함되어있다. 토픽 ⑦(분석ㆍ평가)은 보안과 관련된 조직ㆍ시스템ㆍ기술이 제대로 활용되고 있는지, 그 영향력은 어느 정도인지를 분석하고 평가하는 카테고리다. 토픽 ⑤ㆍ⑥ㆍ⑦은 보안 체계ㆍ시스템이 개발 → 운용 → 평가되는 일련의 과정이 고스란히 토픽 분류에 반영되어있다고 판단할 수 있는 부분이다.

```{r echo=TRUE}
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

### LDAvis를 활용한 시각화
#### LDAvis 는 두 가지 정보를 출력합니다. 첫째는 HTML 의 왼쪽에 출력되는 topic 의 2 차원 embedding vector 입니다. 비슷한 위치에 존재하는 토픽들은 서로 비슷한 문맥을 지니고 있습니다. 그리고 둘째는 오른쪽에 출력되는 각 토픽의 키워드 입니다. 이들의 원리를 살펴봅니다.

### 2-D visualization of topic vectors 
####토픽은 단어 개수의 차원을 지니고 있습니다. 이를 2 차원으로 압축하기 위해서는 차원 축소 방법이 이용되어야 합니다. LDAvis 는 Principal Component Analysis (PCA) 를 이용하여 n_terms 차원의 벡터들을 2 차원으로 압축합니다. 
```{r}
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

#### 위 시각화를 통해 각 토픽의 연관성과 출현율(prevalence)을 파악할 수 있다. 출현율이란 특정 토픽에서 사용된 용어가 전체 토픽에서는 얼마나 전반적으로 사용되고 있는가를 나타내는 정도다. 각 토픽은 원 형태로 그려지며, 출현율의 값이 클수록 원의 면적이 더욱 넓어진다. 일례로, ‘기술ㆍ개발’의 원이 ‘위협 대응’의 원보다 큰 것으로 미루어보아, ‘기술ㆍ개발’과 관련된 연구가 ‘위협 대응’에 관한 연구보다 출현빈도가 높다는 것을 알 수 있다. 또한, 토픽과 토픽 사이의 거리는 토픽간의 연관성을 나타낸다. 지근거리에 있을수록 밀접한 관계에 있는 토픽들이며, 멀리 떨어져 있는 토픽들은 상호 독립성이 강하다고 할 수 있다.

#### 본 연구에서 추출된 7개의 토픽들이 그 성격의 유사성에 따라 크게 3개 군집을 이루고 있는 모습을 볼 수 있다. 보안 위협이 무엇인지 규정하고, 그 위협에 대응하고, 그 위협 및 대응이 국가별로 어떠한지 분석하는 토픽 ①(보안 위협)ㆍ②(위협 대응)ㆍ④(국가별 비교)가 서로 근접거리에 위치하고 있으며, 보안 관련 체계ㆍ시스템을 개발하고 운용하고 평가하는 일련의 과정인 토픽 ⑤(기술ㆍ개발)ㆍ⑥(체계 운용)ㆍ⑦(분석ㆍ평가) 또한 서로 가까운 거리를 유지하고 있다.


