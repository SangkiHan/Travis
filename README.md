## .travis.xml
``` yml
---
language: java #언어
jdk:
  - openjdk11 #jdk버전
branches:
  only:
    - main #빌드 할 브랜치 설정
cache:
  directories:
    - $HOME/.m2/repository
    - $HOME/.gradle
script: ./gradlew clean build
before_deploy:
  - zip -r travis * #파일명 설정
  - mkdir -p deploy #deploy폴더 생성
  - mv travis.zip deploy/travis.zip #압축한 파일 이동
deploy:
  - provider: s3 #S3설정
    access_key_id: AKIAXKXLQ6GIVZKGAF52 #AWS 엑세스키
    secret_access_key: XgtLJuTGu66TGt52mUASLJzIkVvgfXXYCb68l36I #AWS 시크릿키
    bucket: springboot-travis-build #업로드 할 S3 버킷
    region: ap-northeast-2 #리전
    skip_cleanup: true
    acl: private
    local_dir: deploy
    wait-until-deployed: true
    on:
      all_branches: true #모든 브랜치에 권한 부여
  - provider: codedeploy #CodeDeploy설정
      access_key_id: $AWS_ACCESS_KEY #AWS 엑세스키
      secret_access_key: $AWS_SECRET_KEY #AWS 시크릿키
      bucket: springboot-travis-build #업로드 할 파일이 있는 S3 버킷
      key: travis.zip #업로드 할 파일
      bundle_type: zip #업로드 할 파일 형시 
      application: travis #CodeDeploy 어플리케이션 이름
      deployment_group: travis-group #CodeDeploy 그룹명
      region: ap-northeast-2 #리전
      wait-until-deployed: true
      on:
        all_branches: true
notifications:
  email:
    recipients:
      - tkdrl8908@naver.com #빌드 실패시 알림 보낼 메일
```

## appspec.yml
``` yml
---
version: 0.0 #버전
os: linux #OS
files:
  - source: /
    destination: /home/ec2-user/app/travis/build #zip파일을 저장할 경로
    overwrites: yes #덮어쓰기 허용

```