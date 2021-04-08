# subway-api-data-DW-Analysis-BI   
# Process   
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
![Screenshot_49](https://user-images.githubusercontent.com/66659846/114012069-d8895400-98a0-11eb-9f14-58e3c8e40390.png)   
# 환경 및 스펙   
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
# 서버 구성   
