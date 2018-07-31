## 072918
### 하나의 값 조작하기
5. 1. 코드값을 레이블로 변경하기 (5-1)
    - CASE문이 중요.
5. 2. URL에서 요소 추출하기 (5-2)
    - 유저가 어떤 경로를 통해 우리 서비스에 접근했나 분석 필요
    - 이럴 때 정규표현식 사용
    - *substring(referrer from 'https?://([^/]*)') AS referrer_host* #R의 substr, paste0 등등..
    - 미들웨어에 따라 작성법이 다를 것 (HIVE, SparkSQL: parse_url)
5. 3. 문자열을 배열로 분해하기 (5-2)
    - url 값을 /를 통해 좀더 세부적으로 분해 가능
    - 규봉: A/B test 중 B안의 버튼1, 버튼2...
    - Postgres: *split_part* / HIVE, SparkSQL: *split(parse_url()*, 인덱스가 0부터 시작
5. 4. 날짜와 타임스탬프 다루기 (5-2)
    - *CURRENT_DATE, CURRENT_TIMESTAMP* (HIVE, SparkSQL은 () 추가)
    - 임의로 지정한 날짜와 시각으로 '날짜 자료형'과 '타임스탬프 자료형' 만들 때는 CAST
    - **규봉: (SELECT CAST('2018-07-03 13:21:33' AS timestamp) AS stamp) AS t  여기서 왜 임의의 AS로 한번 더 묶을까?**
    - 가장 기초 *substring: substring(stamp, 1, 4)* # text 형태의 stamp col에서 첫번째 문자열부터 네번째 문자열까지 추출
    - *EXTRACT*는 자료형이 date일 때 YEAR, MONTH 등 함수 바로 적용 가능
5. 5. 결측치를 디폴트 값으로 대치 (5-5)
    - null과 문자열 결합 -> null, null과 숫자 연산 -> null
    - *COALESCE* fn으로 null value를 0으로 imputation

### 여러개의 값에 대한 조작
- 단순하게 숫자 비교 이외에도 '개인별' or '비율' 등의 지표를 사용하면 다양한 관점 구사 가능
6. 1. 문자열 연결 (6-1)
    - *CONCAT*: 문자열 간 연결
6. 2. 여러개의 값 비교하기
    - *CASE* 구문으로 전분기와의 증감 형태가 어떤지 알 수 있다.
    - 규봉: CASE 구문 내 WHEN들과 ELSE 간 콤마 없음
    - *SIGN* fn은 부호 (+1, 0, -1)
    - MIN / MAX의 역할을 least / greatest
    - **규봉: R이나 Python처럼 최대인 행 혹은 분기를 불러오려면?**
    - [**규봉: 열별로 특정 칼럼들의 조합으로 비율 혹은 평균 만들기**](https://stackoverflow.com/questions/7367750/average-of-multiple-columns)
    >SELECT <br/>
            year<br/>
            , AVG(q1 + q2 + q3) AS average<br/>
        FROM<br/>
            quarterly_sales<br/>
            ; --\#동작 안하는 전형적인 모습!
6. 3. 2개의 값 비율 개산하기 (6-3)
    - CTR(Click Through Rate): '클릭 / 노출 수' 비율
    - HIVE, SparkSQL: 정수로 나눌 때 자동적으로 실수 변환
    - PostgreSQL: *CAST( - AS double precision)* 변환 필요
    - 클릭 수 혹은 노출 수가 0인 경우에 대비해, 0으로 나누는 것을 피할 필요가 있다.<br/>
    *CASE* 이용해 ERROR:  division by zero 대비 or *NULLIF(노출수, 0)* 이용
6. 4. 두 값의 거리 계산하기 (6-4))
    - 절대값, 제곱평균 제곱근(RMS) 계산 시, *abs, power( -, 2), sqrt* 활용
    - 유클리드 거리 계산도 같은 활용함수를 갖는다. *sqrt(power(x1 - x2, 2) + power(y1 - y2, 2))*
6. 5. 날짜 / 시간 계산하기 (6-5)
    - 유져의 나이 계산, 가입 기간, 연령대 분포 파악 등: 두 날짜 데이터의 차이, 시간 데이터 기준 n 시간 후 산출
    - ex) *register_stamp::timestamp - '30 minutes'::interval AS minus_30_min<br/>
    (register_stamp::date - '1 mon'::interval)::date AS minus_1_month*
    - ::timestamp는 초 단위까지의 시각 / ::date는 년-월-일까지
    - 가입일과 오늘 날짜의 차이 계산으로 이용일수 계산
    > SELECT<br/>
    user_id<br/>
    , CURRENT_DATE AS today<br/>
    , register_stamp::date AS register_date<br/>
    , register_stamp::timestamp AS regi_timestamp<br/>
    , today - register_date AS diff_days<br/>
    FROM<br/>
    mst_users_with_birthday<br/>
    ;
    - >Hive, SparkSQL의 경우 날짜/시각을 계산하기 위한 기본 함수가 제공되지 않으므로<br/>
    unixtime으로 변환 후 초 단위로 계산을 한 뒤 다시 타임스탬프로 변환한다.
    ex) CAST(register_stamp AS timestamp) AS timestamp) AS register_stamp<br/>
    , from_unixtime(unix_timestamp(register_stamp) + (60 * 60)) AS plus_1_hour <br/>
    -- 문자열을 날짜로 변환할 때는 to_date 함수 활용(규봉: as.date() in R) <br/>
    , to_date(register_stamp) AS register_date <br/>
    -- 일과 월을 계산할 때는 date_add or add_months fn 활용 <br/>
    , add_months(to_date(register_stamp), -1) AS minus_1_month <br/>
    -- 날짜 간 차이 계산은 date_diff()<br/>
    
    
    
    
    
    
    
    