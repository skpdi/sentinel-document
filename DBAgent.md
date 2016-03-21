# DB Agent

## 입수 파일 생성 가이드
### 레코드 선별 기준
* 입수 유형 (extract_type)에 따라 추출 쿼리 등을 유의해 사용해주십시오.
* full
    * 테이블 모든 레코드 추출
* delta (변동분)
    * 주기 사이에  생성, 수정된 레코드만 추출
    * 시각 기준: part_date인 경우, 생성 or 수정 시각이 D-1 00:00 ~ D-1 24:00 YYYYMMDD가 D-1인 것)
* incremental (증분)
    * 주기 사이에 생성된 레코드만 추출
    * 시각 기준: part_date인 경우, 생성 시각이 D-1 00:00 ~ D-1 24:00 (YYYYMMDD가 D-1인 것)

### 파일 명
* 기준일자별로 파일을 별도 생성합니다.
* ${DATA_ID_NAME}_YYYYMMDD.tsv

### 파일 아웃풋 포멧
* tsv 포멧
    * 필드 구분자 \t
    * 라인 (레코드) 구분자 \n
    * 문자열 타입 필드는 \t, \n 을 " " (공백)으로 치환할 것

### 필드 문자열 변환 가이드
* 암호화된 필드
    * 분석용으로 사용된다면 복호화해서 파일 생성할 것
* 날짜
    * 포멧1) yyyyMMddHHmmssSSS
    * 포멧2) ISO Date Format (ISO 8601)
* null-string
    * empty string으로 "" 생성할 것
* null-nonstring
    * emptry string으로 "" 생성할 것

### LogAgent 사용 시
* 첫번째 필드로 기준일자 (D-1)를 센티넬에 정의하고 yyyyMMdd 형태로 생성할 것
