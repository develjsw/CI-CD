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
4. Cloud 환경 AWS EC2 기준 Docker Image 활용
