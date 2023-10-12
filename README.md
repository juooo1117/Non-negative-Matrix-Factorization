# NMF Model for Recommendation system
   NMF(Non-negative Matrix Factorization) 모델을 활용한 맞춤형 취미 강의 추천 서비스




## ✏️ 프로그램 소개
  - 오늘의 일기를 작성하면, 요즘 나에게 꼭 맞는 취미를 추천해준다?
  - 사용자가 오늘 나의 하루를 일기로 적으면, 그 글을 분석해서 각각의 사용자에게 적합한 취미를 추천해 주는 시스템


#### [System Flow]

1. 사용자는 오늘의 일기를 작성 (네이버 블로그 #오늘의 일기)
2. Class 101 사이트의 카테고리별 취미 소개글을 분석 (Class 101 #취미 카테고리)
3. 사용자가 작성한 일기를 불러와서 Text Analysis
4. 사용자 일기와 Class 101 사이트 카테고리별로 취미를 매칭
5. 사용자가 작성한 일기를 토대로 가장 연관성 있는 취미를 추천 ( → 일기를 작성한 사용자들에게 맞춤형 class101 광고 추천)




## 📝 데이터 선정 및 수집, 전처리

#### 1. 데이터 크롤링(총 27616개) 
   - 검색옵션을 프로그램이 자동으로 설정하여 크롭 웹드라이버를 이용해서 url을 수집하는 방식으로 진행

    from bs4 import BeautifulSoup      # html 데이터 전처리
    from selenium import webdriver     # 웹 브라우저 자동화
        
   - 수집한 데이터는 url, 숫자, 영어, #, 특수문자제거, 정규화 과정을 통해 전처리 수행   
   - Class 101(4884개): 학습데이터 keyword 크롤링 → 창업, 외국어, 언어, 디자인, 음료, 운동, 금융, 영상, 데이터, 음악, 사진, 개발, 요리, 재테크, 부업, 베이킹, 글쓰기, 직무교육, 공예, 드로잉, 성공 마인드, 아동교육.   (총 22개)
   - 네이버블로그(1360개): 네이버블로그 ‘오늘일기’를 키워드로 크롤링 후 본문에 해당하는 열 정보만 리스트(contents)로 저장
   - 블라인드(21372개)


#### [참고] Keyword Category
<p align="center">
  <img src="https://github.com/juooo1117/NMF_for_Recommendation_system/assets/95035134/b7d8d8f6-6ca3-48ba-a3d3-8304a60f50ed">
</p>
    
#### 3. 데이터 전처리(형태소 분석)
   - 형태소 분석기 중 (Kkma, Komoran, Hannanum, Mecab, Okt)  Okt로 선택 및 설치
   - morphs는 형태소 단위로 구문을 분석(형태소 단위로 토큰화)
   - 토큰화가 완료되면 리스트 형태로 명사만 추출 (Okt.nouns)




## 🏆 모델링
  ###   Clustering with NMF Model 적용


#### [Modeling Flow]
<p align="center">
  <img src="https://github.com/juooo1117/NMF_for_Recommendation_system/assets/95035134/395175d1-85e0-4b27-a22e-eb9132e94a9c">
</p>


#### 1. TF-IDF
   - TfidfVectorizer 에 적용하기 위해서 모든 전처리가 완료된 데이터를 문장으로 만듦 (명사만 추출되어 문장화 된 형태)
   - sklearn의 TfidfVectorizer 사용 → 2개 미만의 문서 또는 90%이상의 문서에서 발생하는 단어는 제외
   - CountVectorizer 를 사용하지 않고 TfidfVectorizer 를 활용 → 단어를 개수 그대로 카운트하지 않고 모든 문서에 공통적으로 들어가있는 단어의 경우 문서를 구별하는 능력이 떨어진다고 봐서 가중치를 축소하고자 했기 때문


#### 2. NMF Model 생성
   - TfidfVectorizer 적용된 matrix 생성
   - sklearn이용해서 NMF model 생성: 최종적으로 모아질 클러스터의 개수는 22개(n_components=22)로 설정 → 분류할 클러스터의 개수가 많아질수록 성능은 떨어지지만, 원래 데이터의 카테고리 개수가 22개이므로 22로 설정
   - TfidfVectorizer 적용으로 사라지지 않은 데이터 중에서 카테고리별로 가장 많이 등장한 상위 단어(term) 확인
   - 출력된 각각의 cluster별 빈번하게 발생한 단어를 확인 → 각 cluster에 연관성 높은 카테고리 이름(공예, 성공 마인드, 운동, 부업…) 22개를 부여
   - 분류 확인 테스트: 알맞은 cluster로 잘 분류되는지 확인하기 위해서, 임의로 단어를 데이터 모델에 입력

      |DATA|CLUSTER|
   |----|-------|
   |[다이어트, 메뉴, 샐러드, 파스타, 레시피]|8번째 카테고리(요리)|
   |[오늘, 그림, 색깔, 핑크, 패턴, 스티커]|0번째 카테고리(공예)|
   |[삼성전자, 은행, 돈, 주식, 실망, 이직, 자격증, 술]|7번째 카테고리(재테크)|
   |...|...|...|    


#### 3. 신규 데이터 모델에 적용
   - 신규데이터로 Test → 최종적으로 완성된 모델에 적용하기 위해서 전처리 과정을 거친 #오늘일기 데이터를 test data(list 형태)로 활용
   - NMF모델에 의해서 묶여진 클러스터 내용을 보고 매칭한 cluster별 category 이름이 존재(ex. 음악, 외국어, 디자인..)
   - test data(#오늘일기)를 완성된 모델에 넣으면, 일기에 쓰인 단어를 분석하여 가장 비슷한 수준의 cluster를 찾음
   - 해당 cluster에 매칭된 category 이름(ex. 베이킹, 요리, 재테크…)을 출력
   - 모델에서 출력된 결과 값(category)을 키워드로 클래스101 검색
   - ‘chromedriver’로 접속해서 자동으로 해당 키워드 검색 후, 결과 페이지 중에서 랜덤으로 3개의 강의를 가져옴 **(강의 이름, 강의 링크)**


#### 4. Confusion matrix 매칭도 확인
   - 클러스터가 비슷한 것끼리 잘 묶여져 있는지 확인하기 위해서 confusion matrix 수행
   - 예측한 카테고리값과 실제 카테고리 값(정답)을 가지고 계산



## 🏆 결과분석

#### 1. 결론
   - 사용자는 오늘의 일기를 작성하면, 작성한 텍스트를 분석하여 작성한 글과 가장 비슷하고 어울리는 취미를 ‘클래스 101’ 사이트에서 추천 받게 됨
   - 클래스101 사이트(취미 추천) 뿐만 아니라 다양한 범위에서 적용 가능한 추천 시스템
   - 글을 작성한 사용자에 대해서 실시간으로 맞춤형 광고를 제공할 수 있음


#### 2. 발전가능성
   - 다양한 분야의 개인 맞춤형 추천 시스템으로 활용이 가능함
   - SNS, 커뮤니티 상에 작성된 사용자의 게시글 내용을 분석 → 개인 맞춤형 추천 시스템으로 활용 (사용자 맞춤 광고 서비스)


