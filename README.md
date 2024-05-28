<h6>** 공유 목적으로 작성된 내용으로 누구나 따라서 진행할 수 있도록 상세히 기재 **</h6>
<h3>Jenkins를 활용한 build/deploy 자동화</h3>
  
**[ Docker 설치 및 설정 ]**
1. Local 환경
   - 작성중...

2. Cloud 환경 AWS EC2 기준
   - 작성중...

**[ Jenkins 설치 및 설정 ]**
1. Local 환경
   - JDK 설치(JRE 포함)
   - 작성중...
  
2. Local 환경 Docker Image 활용
   - Docker Hub Registry에서 Jenkins Image 찾기
   ~~~
   $ docker search jenkins
   ~~~
   - Docker Image Local로 가져오기
   ~~~
   # 'jenkins'이름의 official image는 deprecated 되었으므로 두번째로 많이 다운받은 'jenkins/jenkins' 이미지를 받음
   # 참고 : https://hub.docker.com/_/jenkins/
   $ docker pull jenkins/jenkins:lts-jdk17 # 원하는 태그 사용
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
   $ docker run -p 8080:8080 --restart=on-failure -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk17
   ~~~
   - Jenkins 설치 및 계정 생성
     - browser에서 jenkins 페이지(localhost:8080) 접속
     - jenkins 페이지에서 보여지는 초기 비밀번호 경로로 이동
     ~~~
     $ docker exec -it <container_id> bash
     $ cat /var/jenkins_home/secrets/initialAdminPassword
     ~~~
     - jenkins 페이지에 초기 비밀번호 입력 후
     - Install suggested plugins
     - 계정 생성
     - URL 설정(변경없이 진행)

3. Cloud 환경 AWS EC2 기준 (참고 - https://green-joo.tistory.com/12)
    - 방화벽 설정 (EC2 서버 내부에 설치된 Jenkins 접근 허용)
      - AWS 콘솔 로그인 → AWS EC2 인스턴스 → 보안 → 보안 그룹 → 인바운드 규칙 → 인바운드 규칙 편집 → 사용자 지정 TCP, 8080, 내 IP 추가
    - JDK 설치(JRE 포함)
      ~~~
      $ yum update -y
      $ yum install java-11-amazon-corretto
      ~~~
    - 저장소 다운로드 (공식 문서 : https://www.jenkins.io/doc/book/installing/linux/)
      ~~~
      $ wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
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
        $ vi /etc/yum.repos.d/jenkins.repo
        # gpgcheck=1 → gpgcheck=0으로 변경 후 설치 명령어 다시 실행
        ~~~
    - 서버 부팅 시 Jenkins 자동 재시작
      ~~~
      $ systemctl enable jenkins
      ~~~
    - Jenkins 서비스 실행
      ~~~
      $ systemctl start jenkins
      ~~~
    - Jenkins 서비스 상태 확인
      ~~~
      # active로 되어 있으면 정상
      $ systemctl status jenkins
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
     $ cat /var/lib/jenkins/secrets/initialAdminPassword
     ~~~
     - jenkins 페이지에 초기 비밀번호 입력 후
     - Install suggested plugins
     - 계정 생성
     - URL 설정(변경없이 진행)
      
4. Cloud 환경 AWS EC2 기준 Docker Image 활용
