# hadoop_conf
라즈베리파이에서 Hadoop cluster 구성 시 사용한 설정 파일입니다.  
일반 설정은 상세 설정 부분을 다르게 하면 됩니다.  
  
## Hadoop Cluster 구성하기
0. 개발 환경
1. 네트워크 설정
2. Hadoop 설치
3. 중간 점검
4. 상세 설정
  
### 0. 개발 환경
* 라즈베리파이 3 x 3 대
* Ubuntu Mate
* 하둡 2.x 
* java 1.8
  
### 1. 네트워크 설정
#### 1-1. ssh 설정
* 하둡 분산 작업 시, 노드 간 자유롭고 원활한 통신이 필요하다.
* 그렇기 때문에 각각의 노드가 서로 비밀번호 없이 자유로운 통신을 할 수 있도록 한다.
* 설정 과정은 다음과 같다
  1. ssh 설치
  2. `ssh key-gen` 으로 rsa 방식의 공개키와 개인키 생성
  3. .ssh 디렉토리의 `id_rsa.pub`(공개키)를 `scp` 명령어를 통해 모든 노드 간 공유
  4. 공유된 id_rsa.pub를 `cat` 명령을 통해 `authorized_keys` 파일에 등록
  5. 비밀번호 입력 없는 ssh 접속 확인
* 만약 노드 A의 pub파일(공개키)를 노드 B에 등록하면 A는 B에 비밀번호 없이 접속할 수 있다.
* 참고 자료 https://opentutorials.org/module/432/3742
  
#### 1-2. 로컬 네임서버 지정
* 서로 통신에 있어 도메인 네임을 지정하여 개발자가 노드를 지정할 때 쉽게 할 수 있게 한다.
* 예) Slave 지정 등
* 로컬 네임서버는 DNS의 로컬 버전이라고 생각하면 된다.
* /etc/hosts를 통해 지정할 수 있다.
```
# /etc/hosts
...
172.30.1.16     hd_master_1
172.30.1.8      hd_slave_1
172.30.1.9      hd_slave_2
...
```
  
### 2. Hadoop 설치
#### 2-1. Hadoop 설치
* 클러스터 내의 설정은 모두 동일하니 마스터 노드에서 작업 후 다른 노드들에게 복사, 전송하는 방법이 효율적이다.
* wget을 통해 Hadoop binary 파일을 다운로드 받는다.
* tar.gz 파일을 압축 해제한다.
* 접근이 편하도록 디렉토리 이름을 hadoop으로 수정한다.
* 디렉토리를 `/usr/local` 로 옮긴다. (홈 디렉토리의 경우 NFS로 마운트되어 있는 경우가 많기 때문이다)
```
wget http://apache.mirror.cdnetworks.com/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz 
tar -zxf hadoop-2.7.7
mv hadoop-2.7.7 /usr/local/hadoop
```
* Hadoop 파일의 소유자(hadoop:hadoop)를 지정해준다.
```
sudo chown -R hadoop:hadoop hadoop
```
#### 2-2. Slave 설정
* `hadoop/etc/hadoop/slaves` 파일에 slave 노드들의 호스트네임을 명시해준다.
```
# hadoop/etc/hadoop/slaves
hd_slave_1
hd_slave_2
```

#### 2-3. hadoop-env.sh 설정
* `hadoop-env.sh` 파일은 하둡을 구동하는 스크립트에서 사용하는 환경변수를 명시하는 스크립트이다.
* `HADOOP_HOME`, `JAVA_HOME`, `HADOOP_HEAPSIZE` `HADOOP_LOG_DIR` 을 각각 지정
* 특히 `HADOOP_HEAPSIZE`의 경우 라즈베리파이 3은 전체 메모리가 1G이므로 128과 같이 작게 지정해줬다.
  
#### 2-3. hdfs-site.xml 설정
* `hdfs-site.xml` 파일은 HDFS에 관련된 설정 파일이다.

#### 2-4. yarn-site.xml 설정
* `yarn-site.xml` 파일은 YARN에 관련된 설정 파일이다.

#### 2-5. mapred-site.xml 설정
* `mapred-site.xml` 파일은 잡 히스토리 서버와 같은 맵리듀스 데몬과 관련된 설정 파일이다. 

#### 2-6. core-site.xml 설정
* `core-site.xml` 파일은 HDFS, YARN, 맵리듀스에서 공통적으로 사용되는 I/O 설정과 같은 하둡 코어를 위한 설정 파일이다.
