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
                ```
                    pull_request:
                        branches: [dev]
                ```
                - jobs > steps > name => 검증할 내용들을 추가할 수 있다.
                    - paaking .json 추가
                        - "build": "babel routes -d build"
                            - 개발된 소스를 표준으로 변환
                            - $ npm install --save--dev babel-cli
                            - $ npm run build 
