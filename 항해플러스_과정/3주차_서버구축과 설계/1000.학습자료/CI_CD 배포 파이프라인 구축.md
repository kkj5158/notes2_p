 ⛵ **학습목표 Github Actions와 Docker를 활용하여 웹 어플리케이션의 CI/CD 파이프라인을 구축합니다.**



- CI/CD 기초학습 ****
    
    - **CI**
        
        - **Continuous Integration - 지속적인 통합 ( develop - test - build )**
        
    ![[Pasted image 20250101065806.png]]
        
    - **CD**
        
        - **Continuous Delivery - 지속적인 전달 ( ~ Staging )**
        - **Continuous Deployment - 지속적인 배포 ( ~ Production )**
        
![[Pasted image 20250101065814.png]]
        
    
     ℹ️ [CI/CD Pipeline](https://www.redhat.com/ko/topics/devops/what-cicd-pipeline)
    
    
    
- Docker 의 Container 관련 기술을 학습합니다.
    
- [Github Actions](https://docs.github.com/en/actions)의 구성요소에 대해 학습합니다.
    
- semantic versioning 기반의 태그/릴리즈 관리 전략을 수립해 봅니다.
    
![[Pasted image 20250101065841.png]]
    

 🚩 **What to do: 이번 챕터에 해야 할 것. 이것만 집중하세요!**



### STEP 01 서버 환경 분리

- 서버의 Phase 에 대한 설정을 추가해보세요.
    
    - `dev`, `prod` 환경으로 구성합니다.
    - 저는 현업에서 이렇게 하고 있어요 ! by `alen.heo`
        - `dev` 개발 환경
        - `alpha` 알파 릴리즈 환경 ( QA test )
        - `staging` 사내 릴리즈 환경 ( 인하우스 test )
        - `prod` 운영 릴리즈 환경
    
![[Pasted image 20250101065907.png]]
    
- 위 서버 스택과 배포 전략 등을 문서화하고 프로젝트의 전반적인 flow 를 이해할 수 있도록 합니다.
    

### STEP 02 빌드 환경 구축

- `docker` 혹은 `docker-compose` 등 컨테이너 기술을 활용해 이미지를 빌드, 배포할 수 있도록 합니다.
    
    - 컨테이너 기술을 이용하면 어떤 이점을 가져갈 수 있을지 고민해봅니다.
    - 아래 두 가지 방법을 시도할 수 있어요.
        - `docker` + RDS 조합을 통해 db 와 애플리케이션의 배포 lifecycle 을 분리
        - `docker-compose` 를 이용해 db 까지 함께 하나의 lifecycle 로 배포
    
     ℹ️ [Docker Docs](https://www.docker.com/) [Docker Hub](https://hub.docker.com/) [Docker Compose 쉽게 알아보기](https://scarlett-dev.gitbook.io/all/docker/untitled) [docker 입문편 - 따라하기 class](https://www.44bits.io/ko/post/easy-deploy-with-docker)
    
    
    
- `Github Actions` 를 이용해 Push, PR 등에 대해 Trigger 되는 CI/CD 환경을 구축합니다.
    
     ℹ️ [Github Actions Docs](https://docs.github.com/en/actions) [Github Actions 소개와 핵심 개념](https://www.daleseo.com/github-actions-basics/)
    
    
    
    - Lint 체크, Coverage 리포트, Test 체크 등을 추가하면 더 좋습니다.
    
    ```jsx
    저는 보통 CI Step을 아래와 같이 구성해요. by alen.heo 
    1. checkout : Git 으로부터 소스코드를 가져오는 Step
    2. build : 현재 env 에 따라 애플리케이션 빌드
    3. test : 테스트 코드 실행
    4. lint : 린트 실행
    5. report : 테스트 커버리지 리포트, 린트 리포트 등 CI 보고서 아카이빙 및 통계 전송  
    ```
    
    - phase 별로 서로 다른 파이프라인을 생성해보세요.
        - `연관 키워드` : active_profiles , NODE_ENV

### STEP 03 배포 환경 구축

- AWS 의 ECR, ECS 서비스를 이용할 거예요 !
    
     ℹ️ [AWS ECR Docs](https://aws.amazon.com/ko/ecr/) [AWS ECS fargate Docs](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
    
    
    
- 계정의 경우 필요한 권한만을 허용한 AWS IAM 계정을 만들어 이용할 수 있도록 합니다.
    
    - 가능한 한 root 계정이 아닌, 일부 권한을 가진 IAM을 이용하도록 합니다. 해킹이나 키 유출시에 과도한 비용청구를 막을 수 있습니다.
    
    [IAM 살펴보기 기초편 | DevelopersIO](https://dev.classmethod.jp/articles/what-is-aws-iam-kr/)
    
- 보안이 민감한 정보의 관리는 Github Actions의 secret 을 활용합니다.
    
    - 비밀번호 등 민감정보는 코드로 관리할 시에 보안 측면에서 위험합니다.
    
    [Github Actions 와 secret ?](https://www.notion.so/Github-Actions-secret-1692dc3ef51481f98ed3ff92d4617c29?pvs=21)
    
- ECR 에 CI 를 통해 성공적으로 빌드된 이미지를 업로드합니다.
    
- ECS Fargate 를 이용해 ECR 의 이미지를 기반으로 CD 환경을 구축할 수 있도록 합니다.
    

 ℹ️ [AWS fargate를 이용하여 AWS ECS에 서비스 배포하기 Docs](https://docs.aws.amazon.com/ko_kr/prescriptive-guidance/latest/patterns/deploy-java-microservices-on-amazon-ecs-using-amazon-ecr-and-aws-fargate.html) [AWS ECS 실습하기](http://ecs.catsdogs.kr.s3-website.ap-northeast-2.amazonaws.com/ko/1.-intro/)



- **주의 !** 위 AWS 설정 등에서 vpc / network / security 등을 고려해야 합니다.

 🔥 **어렵다면 아래 튜토리얼을 보고 따라해봅시다.**

[Github Actions + ECR/ECS CICD Pipeline Tutorial](https://www.notion.so/Github-Actions-ECR-ECS-CICD-Pipeline-Tutorial-1692dc3ef514817eac93d3f2038f0a84?pvs=21)



### FAQ

- Q. 저는 이런거 사용하고 싶은데요 !
    
    - Q : Jenkins, k8s 등 최신 트렌드를 익히고 싶은데 안되나요 ?
        
    - A : 됩니다. 다만, 메인 가이드로는 진행하지 않습니다.
        
    - Q : 요즘 현업에서는 저런 툴들이 핫하지 않나요 ?
        
    - A : 짧은 기간 내에 CI, CD 를 직접 구성해보는 형태로 진행하기에는 적합하지 않다고 판단했습니다. 최신 기술의 단순한 셋업을 진행해보는 것 보다, 빠르게 파이프라인을 구축하고 개선점을 찾아 진행하는 과정이 좀 더 많은 인사이트를 높일 수 있습니다. 이에, 제공되는 가이드를 먼저 수행해보고 추가로 진행을 원한다면 개인적으로 수행 후 코치님들에게 질문을 통해 답변을 얻는 과정으로 진행되었으면 좋겠습니다.
        
        by `alen.heo`
        
- Q. Docker 를 쓰는 이유 ?
    
    - Q : Docker 와 같은 Container 기술을 쓰는 이유가 뭔가요 ?
    - A : 불과 몇 년전만 해도, 심지어 근래 적지 않은 회사들은 매번 소스코드를 세팅한 서버에 접속해 당겨와 배포를 진행하는 등의 작업을 수행했습니다. 이 때의 가장 큰 문제점은 배포하는 개발자의 실수에 의한 휴먼에러가 섞여 들어간다던지, 항상 같은 환경 / 의존성을 유지하지 못한다던지 하는 문제 등입니다. 이에 코드가 항상 우리가 원하는 의존성/환경 내에서 동작하도록 유지하고 캡슐화 시키는 데에 가장 큰 의의가 있습니다. 더불어 OS 와 시스템에 미치는 영향이 적기 때문에 속도가 빠르고 특정 운영체제 혹은 플랫폼에 의존적이지 않은 균일한 배포를 진행할 수 있다는 장점이 있습니다.