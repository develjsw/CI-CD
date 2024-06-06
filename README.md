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
   - (미사용) 신규 유저 생성
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
   - (미사용) 유저 패스워드 추가
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
     - browser에서 jenkins 페이지(host명:8080) 접속
        - built-in node 오프라인 으로 표시되어 있는 경우 : 
        Jenkins 관리 → Nodes → Built-In Node 설정 → Disk Space Monitoring Thresholds의 Free Temp Space Threshold 값 적절하게 설정 변경
        - <h6>** 참고 : 'Disk space is below threshold of 1.00 GiB. Only 471.49 MiB out of 474.78 MiB left on /tmp.'의 문구를 통해 /tmp 디렉토리의 남은 용량인 471.49MiB 값이 설정된 임계값(Free Temp Space Threshold)인 1GiB(1024MiB)보다 낮아서 에러가 발생하고 빌드가 실행이 안되는 상태로 Free Temp Space Threshold 값을 1GiB → 200MiB로 변경해주어 해결 **</h6>
4. Cloud 환경 AWS EC2 기준 Docker Image 활용

<br>

**[ Github Personal Access Tokens 발급 ]**
   - github login → 프로필 Settings → Developer Settings → Personal access tokens → Tokens (classic) → Generate new token → Tokens (classic)
   - 설정 값 입력
      - Expiration : 90 days (원하는 기간으로 설정)
      - repo 전체, admin:repo_hook 전체, admin:org_hook 체크
   - Generate Token 버튼 클릭 <h6>** 참고 : 생성된 토큰은 별도로 보관하여 사용(재발급은 가능하나 잊어버리면 찾을 수 없음) **</h6>

<br>

**[ Docker Hub Personal Access Tokens 발급 ]**
   - docker hub login → Account settings → Security → New Access Token
   - 설정 값 입력
      - Access Token Description : docker hub image 접근/사용
	   - Access permissions : Read, Write, Delete
   - Generate 버튼 클릭 <h6>** 참고 : 생성된 토큰은 별도로 보관하여 사용(재발급은 가능하나 잊어버리면 찾을 수 없음) **</h6>
   
<br>

**[ AWS EC2 docker 사용자 그룹에 jenkins 사용자 추가 ]**
   - Execute shell에서 docker관련 명령어 사용을 위해 docker 그룹에 jenkins 사용자 추가
      ~~~
      # docker그룹에 jenkins 사용자 넣기
      $ sudo usermod -aG docker jenkins

      # docker그룹에 jenkins가 소속되었는지 확인
      $ groups jenkins
      ~~~

<br>

**[ Swap 영역 만들기 ]**
<h6>
   ** 참고 : AWS EC2 t2.micro의 RAM은 1GB 밖에 되지 않아 Jenkins 빌드/배포 job을 실행할 때 서버가 멈추는 현상이 발생하여 Swap 영역을 생성하여 임시 해소 <br>
   프리티어가 아닌 실제 운영 서비스에서는 높은 사양의 인스턴스 사용 권장 <br>
   만약 이미 빌드/배포를 시도하여 서버가 멈춘 상황에서는 인스턴스 재부팅 후 몇분 뒤 재시도
   **
</h6>

~~~
# 2GB 크기의 빈 파일을 /swapfile 이름으로 생성
$ sudo dd if=/dev/zero of=/swapfile bs=128M count=16

# /swapfile의 권한을 소유자만 읽고 쓸 수 있도록 설정
$ sudo chmod 600 /swapfile

# /swapfile을 스왑 공간으로 사용할 수 있도록 포맷
$ sudo mkswap /swapfile

# 시스템에서 /swapfile을 사용할 수 있도록 활성화
$ sudo swapon /swapfile

# 현재 활성화된 스왑 파티션이나 파일 정보를 표시
$ sudo swapon -s

# /etc/fstab 파일을 편집하여 스왑 파일 설정을 변경
$ sudo vi /etc/fstab
   # 파일 끝에 아래 내용 입력 후 저장
   /swapfile swap swap defaults 0 0

# 추가된 free 공간 확인
$ free -h
~~~

<br>

**[ Git 설치 및 설정 + 빌드/배포 디렉토리 생성 ]**
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
    - git 자격증명 저장
      ~~~
      # git 자격증명 설정을 해둠으로써 git 명령어 사용 시 '사용자 이름 + 비밀번호(Personal Access Token)' 입력하지 않도록 설정
      # 사용자마다 git config 설정이 되므로 주의 #
      $ sudo -u 설정할계정 git config --global credential.helper store
         ex) $ sudo -u jenkins git config --global credential.helper store
            > 사용자명 입력
            > 비밀번호(Personal Access Token) 입력
      ~~~
    - 사용자 변경(root)
      ~~~
      $ sudo su
      ~~~
    - git branch별 환경 구성
      ~~~
      $ sudo mkdir -p /data/프로젝트명/브랜치명
         ex) $ mkdir -p /data/test-api/develop
         ex) $ mkdir -p /data/test-api/master
         ex) $ mkdir -p /data/test-api/main
      ~~~
    - 특정 branch 기준 clone
      ~~~
      $ cd /data/test-api/develop
      
      # git clone 레포지토리URL -b 브랜치명 .
      $ git clone https://github.com/사용자명/test-api -b develop .
      ~~~
    - 폴더 권한 jenkins로 변경 (Jenkins Execute Shell에서 명령어 실행할 수 있도록)
      ~~~
      $ cd /data

      # jenkins 설치시에 설정파일을 바탕으로 jenkins 사용자와 그룹이 생성되어 있음
      $ chown -R jenkins:jenkins /data/프로젝트명
         ex) $ chown -R jenkins:jenkins /data/test-api
      ~~~
    - jenkins 사용자로 git 명령어 동작 테스트
      ~~~
      $ cd /data/프로젝트명/브랜치명
          ex) $ cd /data/test-api/develop
      $ sudo -u jenkins git fetch
      $ sudo -u jenkins git pull
      ~~~

<br>

**[ Jenkins 빌드/배포 Pipeline 설정 ]**
- Jenkins에 Credential 추가
   - Jenkins 관리자 페이지 로그인 → Jenkins 관리 → Credentials (관리) (버전에 따라 약간의 명칭 다름) → Stores from parent - Domains (global) → Add Credential
   - 설정 값 입력
      - Github Personal Access Token
         - Kind : Username with password
         - Scope : Global (Jenkins, nodes, items, all child items, etc)
         - Username : Github UserName ex) develjsw
         - Password : Github Personal Access Token ex) ghp_토큰
         - ID : 선택사항, 입력하지 않으면 Jenkins가 자동으로 고유 식별자 생성
         - Description : 선택사항이나 입력 권장 (Username이 같은 다른 credential과 구분하기 위해)
     - Docker Hub Personal Access Token
        - Kind : Username with password
        - Scope : Global (Jenkins, nodes, items, all child items, etc)
        - Username : Docker Hub UserName ex) develjsw
        - Password : Docker Hub Personal Access Token ex) dckr_pat_토큰
        - ID : 선택사항, 입력하지 않으면 Jenkins가 자동으로 고유 식별자 생성
        - Description : 선택사항이나 입력 권장 (Username이 같은 다른 credential과 구분하기 위해)
- Build Job 생성
   - 새로운 Item 생성 → Freestyle project → 이름 입력 후 설정 페이지로 이동 ex) test-api-build
   - 설정 값 입력
      - 소스 코드 관리
         - Git - Repositories - Repository URL : github repository 입력<br>
            ex) https://github.com/사용자명/test-api
         - Git - Repositories - Credentials : 위에서 생성 해둔 Github 자격증명 선택<br>
            ex) develjsw/****** (gibhub 자격증명용)
            <h6>** 참고 : github private repository인 경우에만 자격증명 선택, 아닌경우 None 선택 **</h6>
         - Git - Branches to build - Branch Specifier (blank for 'any') : 빌드 할 기준 브랜치명 입력<br>
            ex) */develop
         - Git - Repository browser : (자동)
      - 빌드 유발
         - Poll SCM - Schedule : 정기적으로 소스 코드 관리(SCM) 저장소를 검사하여 변경 사항이 있는지 확인하여 변경 사항이 발견되면 빌드를 트리거하는 기능<br>
           ex) * * * * *
      - 빌드 환경
         - Use secret text(s) or file(s) → Add - Username and password (separated)
            - 설정 값 입력
               - Username Variable : username으로 설정할 변수명<br>
                  ex) DOCKER_HUB_USERNAME
               - Password Variable : password로 설정할 변수명<br>
                  ex) DOCKER_HUB_ACCESS_TOKEN
               - Credentials : 생성해둔 Docker Hub PAT<br>
                  - 형식 : Username/*****(Credential에 Description로 설정한 값)
                  - 예시 : develjsw/*****(Docker-Hub-PAT)
      - Build Steps
         - Execute shell (Linux OS 환경)
           ~~~
           ## git fetch & pull ##
           cd /data/test-api/develop
           git fetch
           git pull
           
           ## docker image build ##
           # docker build -t <docker hub registry>:<tag> -f <도커 파일 위치> .
           docker build -t develjsw/nest-api-registry:${BUILD_NUMBER} -f dockerfile/Dockerfile-local .

           ## docker image tag ##
           # docker image tag <docker hub registry>:<tag> <docker hub registry>:latest
           docker image tag develjsw/nest-api-registry:${BUILD_NUMBER} develjsw/nest-api-registry:latest

           ## docker hub login ##
           docker login -u ${DOCKER_HUB_USERNAME} -p ${DOCKER_HUB_ACCESS_TOKEN} docker.io

           ## image upload to docker hub registry ##
           docker push develjsw/nest-api-registry:${BUILD_NUMBER}
           docker push develjsw/nest-api-registry:latest
           ~~~
- Deploy Job 생성
   - 작성중... 
