<h6>** 공유 목적으로 작성된 내용으로 누구나 따라서 진행할 수 있도록 상세히 기재 **</h6>
<h3>Jenkins를 활용한 build/deploy 자동화</h3>

<br>

**[ AWS EC2 인스턴스 생성 ]**
   - AWS console login → 서비스 → EC2 → 인스턴스 시작
   - 설정 값 입력
      - Amazon Linux AMI 사용
      - t2.micro 선택
      - Key Pair 생성
      - 보안그룹 생성
      - SSH 트래픽 허용 - 위치 무관
      - 스토리지 30 GB - SSD gp3

<br>

**[ AWS EC2 방화벽 설정 (Jenkins 접근 용도) ]**
   - AWS console login → 서비스 → EC2 → 인스턴스(실행 중) → 인스턴스 ID 선택 → 보안 → 보안 그룹 선택 → 인바운드 규칙 편집 → 규칙 추가, 사용자 지정 TCP, 포트 범위 8080, 내 IP → 규칙 저장 

<br>

**[ AWS EC2 초기 설정 ]**
   - 초기 로그인
      - 방법1. Xshell(SSH Client Tool) → AWS에서 제공해주는 계정 (Amazon Linux AMI의 경우 - ec2-user)으로 Key Pair와 함께 로그인
      - 방법2. AWS console login → 서비스 → EC2 → 인스턴스(실행 중) → 해당 인스턴스 ID 클릭 → 연결 버튼 클릭 → 연결 버튼 클릭
   - 신규 유저 생성
      ~~~
      # 유저 생성
      $ sudo adduser <생성할 유저명> -u <UID>
         ex) sudo adduser test
         ex) sudo adduser test -u 1111

      # root 권한 추가
      $ sudo usermod -aG wheel <UID>
         ex) sudo usermod -aG wheel test
      
      # 유저 생성/권한 정상동작 확인
      $ id <생성한 유저명>
         ex) $ id test
      $ cat /etc/passwd | grep <생성한 유저명>
         ex) $ cat /etc/passwd | grep test
      $ cat /etc/group | grep <생성한 유저명>
         ex) $ cat /etc/group | grep test
      ~~~
   - 유저 패스워드 추가
      ~~~
      $ sudo passwd <생성한 유저명>
         ex) sudo passwd test
      ~~~
   - 패스워드 로그인 설정 (Public Key가 아닌 Password로도 SSH 로그인 가능하도록 설정 변경)
      ~~~
      $ sudo vi /etc/ssh/sshd_config
      $ :/Password
      $ i
         > PasswordAuthentication no 부분을 yes로 변경
         > esc
      $ :wq

      # sshd 설정 파일 변경사항 적용을 위한 service 재시작
      $ sudo systemctl restart sshd
      ~~~

<br>

**[ Docker 설치 및 설정 ]**
1. Local 환경
   - 작성중...

2. Cloud 환경 AWS EC2 기준
   <h6>** 참고 : 2023년 이전에는 amazon-linux-extras 설치 후 amazon-linux-extras를 통해 docker를 설치 했었으나, 2023년부터 해당 패키지는 없어졌고 yum을 통해 docker 바로 설치 가능해짐 (공식 문서 - https://docs.aws.amazon.com/ko_kr/serverless-application-model/latest/developerguide/install-docker.html) **</h6>

   1. Amazon Linux IAM 2022년 이하 버전에서 docker 설치
      ~~~
      # yum 패키지 업데이트
      $ sudo yum update -y
   
      # amazon-linux-extras 패키지 설치
      $ sudo yum install -y amazon-linux-extras
   
      # docker 설치
      $ sudo amazon-linux-extras install docker -y
   
      # docker 상태/버전 확인
      $ sudo systemctl status docker
      $ docker --version
   
      # docker 데몬 시작, 부팅 시 자동 재시작
      $ sudo systemctl start docker
      $ sudo systemctl enable docker
      ~~~
   3. Amazon Linux IAM 2023년 이상 버전에서 docker 설치
      ~~~
      # yum 패키지 업데이트
      $ sudo yum update -y
   
      # docker 설치
      $ sudo yum install -y docker
   
      # docker 설치/상태/버전 확인
      $ rpm -qa | grep docker
      $ sudo systemctl status docker
      $ docker --version
   
      # docker 데몬 시작, 부팅 시 자동 재시작
      $ sudo systemctl start docker
      $ sudo systemctl enable docker
      ~~~
<br>

**[ Jenkins 설치 및 설정 ]**
1. Local 환경
   - JDK 설치(JRE 포함)
   - 작성중...
  
2. Local 환경 Docker Image 활용
   - Docker Hub Registry에서 Jenkins Image 찾기
     ~~~
     $ sudo docker search jenkins
     ~~~
   - Docker Image Local로 가져오기
     ~~~
     # 'jenkins'이름의 official image는 deprecated 되었으므로 두번째로 많이 다운받은 'jenkins/jenkins' 이미지를 받음
     # 참고 : https://hub.docker.com/_/jenkins/
     $ sudo docker pull jenkins/jenkins:lts-jdk17 # 원하는 태그 사용
     ~~~
   - Jenkins 기본 Port(8080) 사용 중인지 확인
     ~~~
     # 모든 운영체제 사용 가능 명령어
         $ netstat -a
         ex) netstat -ano -p tcp | findstr ":8080 "
  
     # Windows OS 전용 명령어
         $ Get-NetTCPConnection
         ex) Get-NetTCPConnection | Where-Object { $_.LocalPort -eq 8080 } | Select-Object OwningProcess, LocalAddress, LocalPort
     ~~~
   - Jenkins Image 실행 (참고 - https://github.com/jenkinsci/docker/blob/master/README.md)
     ~~~
     $ sudo docker run -p 8080:8080 --restart=on-failure -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk17
     ~~~
   - Jenkins 설치 및 계정 생성
     - Browser에서 Jenkins 페이지(localhost:8080) 접속
     - Jenkins 페이지에서 보여지는 초기 비밀번호 경로로 이동
       ~~~
       $ sudo docker exec -it <container_id> bash
       $ sudo cat /var/jenkins_home/secrets/initialAdminPassword
       ~~~
     - jenkins 페이지에 초기 비밀번호 입력 후
     - Install suggested plugins 선택
     - 계정 생성
     - URL 설정(변경없이 진행)

3. Cloud 환경 AWS EC2 기준 (참고 - https://green-joo.tistory.com/12)
    - 방화벽 설정 (EC2 서버 내부에 설치된 Jenkins 접근 허용)
      - AWS 콘솔 로그인 → AWS EC2 인스턴스 → 보안 → 보안 그룹 → 인바운드 규칙 → 인바운드 규칙 편집 → 사용자 지정 TCP, 8080, 내 IP 추가
    - JDK 설치(JRE 포함)
      ~~~
      $ sudo yum update -y
      $ sudo yum install java-11-amazon-corretto -y
      ~~~
    - 저장소 다운로드 (공식 문서 : https://www.jenkins.io/doc/book/installing/linux/)
      ~~~
      $ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
      ~~~
    - Jenkins 개발사 전자서명 공개키 Import
      ~~~
      # 2023년 이전
      ## $ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key ##
      
      # 2023년 이후
      $ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
      ~~~
        - 만약 전자서명 공개키를 2023년 이후 key로 import하지 않으면 'Error: GPG check FAILED'에러가 발생하며 아래와 같이 임시방편으로 해결은 가능하나 추천하지 않음
           ~~~
           $ sudo vi /etc/yum.repos.d/jenkins.repo
           # gpgcheck=1 → gpgcheck=0으로 변경 후 설치 명령어 다시 실행
           ~~~
    - Jenkins 설치
      ~~~
      $ sudo yum install jenkins -y
      ~~~    
    - 서버 부팅 시 Jenkins 자동 재시작
      ~~~
      $ sudo systemctl enable jenkins
      ~~~
    - Jenkins 서비스 실행
      ~~~
      $ sudo systemctl start jenkins
      ~~~
    - Jenkins 서비스 상태 확인
      ~~~
      # active로 되어 있으면 정상
      $ sudo systemctl status jenkins
      $ :q
      ~~~
    - Jenkins 설정 파일 위치/내용 확인(작업 디렉토리 경로, 사용자 계정 권한, 기본 포트 등)
      ~~~
      # 운영체제 버전에 따라 설정 파일이 다름
      
      # 구버전
      $ cat /etc/sysconfig/jenkins
      
      # 신버전
      $ cat /usr/lib/systemd/system/jenkins.service
      ~~~
    - Jenkins 설치 및 계정 생성
     - browser에서 jenkins 페이지(host명:8080) 접속
     - jenkins 페이지에서 보여지는 초기 비밀번호 경로로 이동
       ~~~
       $ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
       ~~~
     - jenkins 페이지에 초기 비밀번호 입력 후
     - Install suggested plugins 선택
     - 계정 생성
     - URL 설정(변경없이 진행)
      
4. Cloud 환경 AWS EC2 기준 Docker Image 활용

<br>

**[ Git 설치 및 설정 + 배포 디렉토리 생성 ]**
1. Local 환경
    - 작성중...
2. Cloud 환경 AWS EC2 기준
    - git 설치
      ~~~
      $ sudo yum install git -y
      ~~~
    - git 버전 확인(=설치 확인)
      ~~~
      $ git --version
      ~~~
    - 권한 설정(credential.helper 설정)
      ~~~
      # 한번 권한 설정을 해두어 다음 번에 입력을 요구하지 않도록 함
      ## 사용자 마다 config 설정이 되므로 AWS EC2 초기 설정에서 생성한 계정으로 로그인 후 아래 명령어 실행 ##
      $ su 사용할계정
         ex) $ su test
            > 패스워드 입력
      $ git config --global credential.helper store
      ~~~
    - git branch별 환경 구성
      ~~~
      $ sudo su
      $ mkdir -p /data/프로젝트명/브랜치명
         ex) $ mkdir -p /data/test-api/develop
         ex) $ mkdir -p /data/test-api/master
         ex) $ mkdir -p /data/test-api/main
      ~~~
    - 특정 branch clone
      ~~~
      $ cd /data/test-api/develop
      
      # git clone 레포지토리URL -b 브랜치명 .
      $ git clone https://github.com/사용자명/nest-test -b develop .
         > 사용자명 입력
         > token값 입력(Personal access tokens)
      ~~~
    - 폴더 권한 변경
      ~~~
      $ cd /data
      $ chown -R jenkins:jenkins /data/프로젝트명
          # jenkins 설치시에 디폴트로 jenkins 사용자가 생성되고 그룹이 형성됨 - $ cat /etc/passwd 명령어와 $ cat /etc/group 명령어를 통해 확인 가능
          # jenkins 설정파일을 바탕으로 생성된 것임을 확인할 수 있음
          ex) $ chown -R jenkins:jenkins /data/test-api
      ~~~
    - Jenkins 사용자로 git 명령어 동작 테스트
      ~~~
      $ cd /data/프로젝트명/브랜치명
          ex) $ cd /data/test-api/develop
      $ sudo -u jenkins git fetch
      ~~~
    - 추가 내용 작성중...

<br>

**[ Jenkins 빌드/배포 Pipeline 설정 ]**
- 작성중...
