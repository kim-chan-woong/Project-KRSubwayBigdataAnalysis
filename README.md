# subway-api-data-DW-Analysis-BI   
## PROCESS      
1. 원천 데이터 수집 및 HDFS 적재   
- 서울 열린데이터 광장의 지하철 이용 관련 데이터 API 키 발급 및 호출   
- Anaconda3를 통해 python 3.8.5 환경을 구축하고 jupyter notebook 포트 개방 후 수집 코드 수행   
- Nifi flow를 통해 HDFS 적재 및 원본 데이터 유지, Hadoop 고가용성(HA:High Availability) Cluster로 구축
      
2. HDFS에 적재된 원본 데이터를 Hive를 통해 테이블화, 쿼리문 접근   
- 데이터가 HDFS에 적재되면 Nifi가 이를 감지해 Hive create table문을 실행   
- 데이터가 적재된 경로가 지정된 Hive 외부(External) 테이블이 생성되어 자동으로 데이터를 테이블화  
   
3. Spark Cluster 구축 및 Hive 연동 후 Zeppelin 작업   
- spark cluster는 구축되어있는 hadoop cluster의 yarn위에서 구동된다.   
- hive2 thrift server를 통해 spark에서 hive 테이블을 읽고, 쓸 수 있도록 한다.   
- zeppelin과 연동하여 spark(scala)코드를 수행, sparkSQL을 통해 테이블 분석, 시각화   
![Screenshot_50](https://user-images.githubusercontent.com/66659846/114016084-70893c80-98a5-11eb-91e3-5ec4246347e9.png)   
## 환경 및 스펙   
Virtual Box - 6.1   
MobaXterm - 8.6   
CentOS - 7   
JDK - 1.8   
Python - 3.8.5   
Scala - 2.12.10   
Hadoop - 3.1.0   
Zookeeper - 3.4.10   
Spark - 3.1.0   
Zepplein - 0.9.0   
## 서버 구성   
nn01: 하둡 클러스터의 액티브 네임노드, 저널노드, 주키퍼, 하이브 마스터, 나이파이 마스터 역할   
rm01: 하둡 클러스터의 스탠바이 네임노드, 저널노드, 리소스 매니저, 주키퍼, 스파크 클러스터의 마스터, 제플린 마스터 역할   
jn01: 하둡 클러스터의 저널노드, 주키퍼   
dn01: 하둡 클러스터의 노드 매니저, 데이터 노드, 스파크 클러스터의 워커 노드 역할   
dn02: 하둡 클러스터의 노드 매니저, 데이터 노드, 스파크 클러스터의 워커 노드 역할   
dn03: 하둡 클러스터의 노드 매니저, 데이터 노드, 스파크 클러스터의 워커 노드 역할   
getdataserver: 아나콘다3 환경에서의 주피터 노트북 실행 및 데이터 수집, 원본데이터 nn01서버에 배포 역할   
![Screenshot_52](https://user-images.githubusercontent.com/66659846/114016090-72530000-98a5-11eb-9172-1d450bc036e1.png)   
![Screenshot_53](https://user-images.githubusercontent.com/66659846/114016985-6a479000-98a6-11eb-90a5-4d8d1b32ef63.png)   
# 결과1:      
- 2015.01.01부터 2020.12.31까지의 각 일별 사용 일자, 지하철 호선, 역 이름, 승차 인원, 하차 인원, 등록 일자 키로 이루어진 6년간의 데이터 1,256,645건   
- 데이터 적재와 유지, 분석 및 시각화의 과정   
- 각 소프트웨어 및 프레임워크의 연동 과정, 데이터 수집 코드는 별도 표기
## mobaXterm 원격 접속 환경에서 작업      
![Screenshot_54](https://user-images.githubusercontent.com/66659846/114018375-1fc71300-98a8-11eb-9d10-ab8a5cc3b26b.png)   
## getdataserver & jupyter notebook 수집 환경 구축(getdataserver:8887)   
![Screenshot_55](https://user-images.githubusercontent.com/66659846/114018644-6e74ad00-98a8-11eb-9d75-fbd87e90c897.png)
## 수집 코드 실행, nn01 서버에 배포   
![Screenshot_92](https://user-images.githubusercontent.com/66659846/114032914-c1a22c00-98b7-11eb-8669-a9fab4ed2d8b.png)   
![Screenshot_59](https://user-images.githubusercontent.com/66659846/114021258-78e47600-98ab-11eb-802d-59b451925259.png)   
## Nifi flow(nn01:18080)   
1. GetFile: getdataserver에서 nn01서버의 지정된 경로에 데이터가 배포되면, 그 데이터를 Nifi 상으로 가져오고, 로컬의 데이터는 삭제   
2. PutHDFS: 수집된 원본 데이터(.csv)파일을 지정된 HDFS 경로에 put   
3. ReplaceText: HDFS에 파일이 들어오면 Hive 외부 테이블을 생성하고 새롭게 들어온 파일의 경로를 location하여 csv파일을 테이블화 하도록 HiveQL문을 작성   
4. PutHiveQL: 작성한 HiveQL문 실행   
5. LogAttribute: 각 프로세스별 로그 확인   
6. Queued: 프로세스별 성공 시 데이터 이동 및 변환을 확인하고, 필요 시 파일 형태로 바로 추출하기 위함
![Screenshot_58](https://user-images.githubusercontent.com/66659846/114021269-7b46d000-98ab-11eb-88b6-a629102b4690.png)   
## HDFS 적재 결과(nn01:9870)   
- 폴더 구조:hdfs://user/subway_data/YYYY/subway_YYYY.csv
![Screenshot_62](https://user-images.githubusercontent.com/66659846/114023087-6834ff80-98ad-11eb-9ee2-8d791a7c0acf.png)   
![Screenshot_64](https://user-images.githubusercontent.com/66659846/114023402-c9f56980-98ad-11eb-9ab5-6c3f26be5451.png)   
## HIVE 테이블화 결과  
- HDFS 적재됨과 동시에 Nifi를 활용, Hive 외부 테이블 생성 및 적재 확인
![Screenshot_73](https://user-images.githubusercontent.com/66659846/114026601-62d9b400-98b1-11eb-945f-d905fbf00291.png)    
- Hiveserver2 Web UI를 통해 정상적으로 쿼리문이 적용됨을 확인   
![Screenshot_114](https://user-images.githubusercontent.com/66659846/114119392-83435600-9925-11eb-8780-d1f67198027e.png)   
## Zeppelin 환경에서 Spark-Hive 연결 및 쿼리 분석   
- zeppelin 사용자 계정, 패스워드, 포트 설정 및 spark와 연동 설정 후 spark가 hive metastore의 thrift server를 통해 원격 서버의 hive에 연결     
- 기존 하둡 클러스터의 yarn 위에서 동작하기 위해 파일 설정 다수 필요   
- zeppelin: rm01:8899
- spark master UI: rm01:8080
- spark context UI: rm01:7077   
1. HiveContext 객체 생성,HiveQL 수행, test DB(subway데이터 저장 데이터 베이스) 접근     
2. 전체 통계 및 분석을 위해 각 년도별 테이블을 통합한 테이블 생성, Hive 내에서 생성 확인      
(* OpenCSVSerde를 사용해 원본 csv 파일을 hive테이블화 하는 과정에서 모든 컬럼이 string으로 되어있으므로 승, 하차 인원의 데이터의 형을 int로 변환)   
![Screenshot_77](https://user-images.githubusercontent.com/66659846/114027093-fd39f780-98b1-11eb-8b95-6ac9324eceec.png)   
![Screenshot_81](https://user-images.githubusercontent.com/66659846/114028494-7ede5500-98b3-11eb-9090-78aaf9d5250f.png)   
![Screenshot_82](https://user-images.githubusercontent.com/66659846/114028746-c7960e00-98b3-11eb-8afb-1f2c03b3644c.png)   
![Screenshot_83](https://user-images.githubusercontent.com/66659846/114028753-c8c73b00-98b3-11eb-925c-84f1e97474ea.png)   
## spark를 활용하여 HiveQL문 실행 및 zeppelin 시각화    
### 결과2: 연도별 지하철 역 개수의 변화  
- 지하철 역의 개수는 꾸준히 늘고 있다.   
- 2015년 12월 31일 (549개 역), 2020년 12월 31일 (597개 역)   
![Screenshot_96](https://user-images.githubusercontent.com/66659846/114037794-42fbbd80-98bc-11eb-8fd0-e323ef8ca041.png)   
![Screenshot_97](https://user-images.githubusercontent.com/66659846/114037802-43945400-98bc-11eb-981b-46c8f62162f8.png)   
![Screenshot_98](https://user-images.githubusercontent.com/66659846/114037805-442cea80-98bc-11eb-922a-06fc7b308ca9.png)   
### 결과3: 연도별 승, 하차 인원 합계   
- 전체적으로 승차 인원과 하차 인원의 차이는 크게 나지 않았다.   
- 2015년 ~ 2019년까지는 큰 변동이 없었으나 2020년 갑작스레 줄어든 것으로 보아 코로나의 영향이 끼쳤음을 추측할 수 있다.   
- 가장 승, 하차 인원이 많은 해는 2019년(승차 기록: 2,716,706,229건, 하차 기록: 2,705,861,190건)이다.   
- 가장 승, 하차 인원이 적은 해는 2020년(승차 기록: 1,968,367,337건, 하차 기록: 1,961,954,817건)이다.   
![Screenshot_99](https://user-images.githubusercontent.com/66659846/114038135-9241ee00-98bc-11eb-8e74-3bc8b71c0479.png)   
![Screenshot_100](https://user-images.githubusercontent.com/66659846/114038138-93731b00-98bc-11eb-8434-21c59a620155.png)   
![Screenshot_101](https://user-images.githubusercontent.com/66659846/114038144-93731b00-98bc-11eb-866c-e32e00decd46.png)   
### 결과4: 6년간 데이터 통합 호선별 승, 하차 인원 합계 TOP 10   
- 가장 많은 승, 하차 기록은 중심가 이자, 여러 회사가 밀집되어 있고, 유동 인구가 많은 2호선이었다.   
- 2호선 승차 기록: 3,192,988,506건, 하차 기록:3,234,195,267건   
- 2위를 기록한 7호선(승차 기록: 1,503,784,559건, 하차 기록: 1,746,216,635건)과 약 2배 가량 차이를 나타냈다.   
![Screenshot_102](https://user-images.githubusercontent.com/66659846/114038455-d9c87a00-98bc-11eb-8eb0-c459e0696027.png)   
![Screenshot_103](https://user-images.githubusercontent.com/66659846/114038460-da611080-98bc-11eb-8881-05f5df1d0777.png)   
![Screenshot_104](https://user-images.githubusercontent.com/66659846/114038464-daf9a700-98bc-11eb-993d-cae663aa43a1.png)   
### 결과5: 6년간 데이터 통합 역 명별 승, 하차 인원 합계 TOP 10   
- 가장 많은 승, 하차 기록을 보유한 역은 2호선 강남역이었다.(승차 기록: 209,302,630건, 하차 기록: 211,921,531건)   
- 유동 인구가 많은 2호선 고속터미널역(승차 기록: 197,393,452건, 하차 기록: 200,409,450건)과   
  1호선 서울역(승차 기록: 184,630,600건, 하차 기록: 185,731,775건)이 뒤를 이었다.   
![Screenshot_105](https://user-images.githubusercontent.com/66659846/114038809-2613ba00-98bd-11eb-8c04-82eb0f035730.png)   
![Screenshot_106](https://user-images.githubusercontent.com/66659846/114038800-23b16000-98bd-11eb-8fe4-e8b0beaa9844.png)   
