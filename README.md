# CI_CD_test
- CI/CD 테스트


# CI 절차
- github 사이트 이동 -> 현 프로젝트로 이동
- Settings > Security > Secrests and variables > Actions
- New repository secrets 버튼 클릭
    - git에 노출되면 안 되는 정보들 등록
    - 환경 변수 등록 (노출되면 안 되는 KEY 정보, DB정보 등)
    - AWS 계정 관리하는 IAM에서 계정별로 access ID/KEY 발급받아서 등록 -> 외부에서 AWS access 가능
    - action에 의해 .env 등 설정 파일로 이관
        - 즉, 소스코드 상에는 없다.
    - 등록 (필요한 만큼, 수시로)
        - 이름 & 값
    
    - gitaction에서 작동할 내용 기술
        - deploy.yml
        - 위치
            - 프로젝트루트/.github/workflows/deploy.yml
            - gitaction을 작동시키는 이벤트(명령, 트리커)는 서브 브런치일 경우 아래처럼 추가 가능
            ``` pull_request: branches: [dev] ```
            - jobs > steps > name => 검증할 내용들을 추가할 수 있다.
                - paaking .json 추가
                    - "build": "babel routes -d build"
                        - 개발된 소스를 표준으로 변환
                        - $ npm install --save--dev babel-cli
                        - $ npm run build 

# CD 절차
- STEP 1
    - 빌드한 소스코드, 리소스, 기타 설정 파일 => 압축(.zip)
    
    - STEP 2
        - AWS 접속
            - IAM 계정상 access key, access ID가 gitaction 시크릿 변수에 등록되어 있어야 함
            - 계정 생성 => Key/ID 발급 => 등록
            - 노출, 분실 금지 !
    
    - STEP 3
        - 압축한 리소스 => s3에 업로드
    
    - STEP 4
        - AWS codeDeploy 서비스 절차에 따라 배포 시작
    
    - AWS 세팅
        - S3 > 버킷리스트 만들기 클릭
            - pusan2024-deploy-bucket
        - EC2에 역할 부여
            - IAM > 액세스 관리 > 역할 > 역할 생성
                - AWS 서비스
                - 사용 사례 > EC2
                - 권한 추가
                    - AmazonS3FullAccess 체크
                    - AWSCodeDeployFullAccess 체크
                - 역할 이름 : pusan_deploy
            - EC2에 위에서 만든 역할을 보안에 적용
                - 인스턴스 선택
                - 작업 > 보안 > IAM 역할 수정 > pusan_deploy 선택
        - CodeDeploy 역할 부여
            - IAM > 액세스 관리 > 역할 > 역할 생성
                - AWS 서비스
                - 사용 사례 > CodeDeploy
                - 역할 이름 : pusan-code-deploy
            - CodeDeploy > 애플리케이션 > 애플리케이션 생성
                - 애플리케이션 이름 : pusan-code-deploy
                - 컴퓨팅 플랫폼 : EC2/온프레미스
            - 배포 그룹 생성
                - 배포 그룹 이름 : dev
                - 서비스 역할 : pusan-code-deploy
                - 환경 구성 : Amazon EC2 인스턴스
                - 키 : Name
                - 값 : 인스턴스 지정 (aws-cloud9-node-...)
                - 로드 밸런서 체크 해제
                - 배포 그룹 생성
        - IAM 사용자 추가
            - 목적 : accessKey, accessID 발급
            - IAM > 액세스 관리 > 사용자 > 사용자 생성
                - 이름 : user-cicd
                - 권한 옵션 : 직접 정책 연결
                - 권한 정책
                    - AmazonS3FullAccess 체크
                    - AWSCodeDeployFullAccess 체크
            - 해당 유저로 진입
                - 액세스키 만들기 클릭
                - 1단계 : 기타 선택
                - 2단계 : access-cicd
                - 3단계
                    - 액세스 키 => AK...
                    - 비밀 액세스 키 => z2...
                - 위 2개 값을 가지고 github에 환경변수 등록

# CD - EC2에서 수행할 작업
- ``$ sudo apt install awscli``
- ``$ sudo aws configure``
    - ID, 시크릿키, region, 형식 입력
- CodeDeploy agent
    - ``$ wget https://aws-codedeploy-ap-northeast-2.s3.amazonaws.com/latest/install``
    - ``$ chmod +x ./install``
    - ``$ sudo apt-get install ruby``
    - ``$ sudo ./install auto``
    - ``$ sudo service codedeploy-agent status``
        - active (running) : 에이전트 작동 중

- EC2 서버가 재가동 됐을 때 자동으로 에이전트가 가동되도록 설정 (옵션)
    - ``$ sudo nano /etc/init.d/codedeploy-startup.sh``
    - 편집
    ```
        #!bin/sh
        sudo service codedeploy-agent restart
    ```
    - 저장 후 nano 종료 (ctrl + x, y, 엔터)
    - ``$ sudo chmod +x /etc/init.d/codedeploy-startup.sh``


# CD-소스파일
- appsec.yml 파일 생성 (루트)
    - codeDeploy가 배포하는 애플리케이션에 대한 사양 정리