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
dn01: 하둡 클러스터의 노드 매니저, 데이터 노드, 스파크 클러스터의 워커 노드   
dn02: 하둡 클러스터의 노드 매니저, 데이터 노드, 스파크 클러스터의 워커 노드   
dn03: 하둡 클러스터의 노드 매니저, 데이터 노드, 스파크 클러스터의 워커 노드   
getdataserver: 아나콘다3 환경에서의 주피터 노트북 실행 및 데이터 수집, 배포 역할   
![Screenshot_52](https://user-images.githubusercontent.com/66659846/114016090-72530000-98a5-11eb-9172-1d450bc036e1.png)   
![Screenshot_53](https://user-images.githubusercontent.com/66659846/114016985-6a479000-98a6-11eb-90a5-4d8d1b32ef63.png)   
# RESULT   
- 2015.01.01부터 2020.12.31까지의 사용 일자, 지하철 호선, 역 이름, 승차 인원, 하차 인원, 등록 일자 키로 이루어진 데이터 1,256,645건   
- 데이터 적재와 유지, 분석 및 시각화의 과정   
## mobaXterm 원격 접속 환경에서 작업      
![Screenshot_54](https://user-images.githubusercontent.com/66659846/114018375-1fc71300-98a8-11eb-9d10-ab8a5cc3b26b.png)   
## getdataserver & jupyter notebook 수집 환경 구축   
![Screenshot_55](https://user-images.githubusercontent.com/66659846/114018644-6e74ad00-98a8-11eb-9d75-fbd87e90c897.png)
## 수집 코드   
