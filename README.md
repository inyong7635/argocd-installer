# Guide for Users
### ArgoCD를 이용한 resource 배포
1. prerequsites
    - argocd([Install guide link](https://github.com/tmax-cloud/install-argocd))
---
## Master cluster
1. cluster configuration
    - cluster에 필요한 정보(registry domain, hyperclou domain등)를 셋팅한다.
    - 수정해야 하는 파일은 아래와 같다.
        ```
        $ application/helm/shared-values.yaml
        $ application/helm/master-values.yaml
        ```
    - git에 push한다.
---
2. application 변수 셋팅
    - application/app_of_apps/master-applications.yaml file을 수정한다.
    - 설치 환경에서 바라볼 git repo url, branch를 설정
        ```
        spec:
          ...
          source:
            ...
            repoURL: {{ git_repo_url }}
            targetRevision: {{ target_branch_or_release }}
    - gitlab의 경우 git repo url 마지막에 .git을 추가해주어야함  
    ex) repoURL: https://gitlab.com/root/argocd-installer.git
---
3. application 등록
    - 설치 환경에 application을 등록
        ```
        $ kubectl -n argocd apply -f application/app_of_apps/master-applications.yaml
        ```
---
4. resource 배포(application sync)
    - application sync 순서는 [docs/INSTALL_ORDER.md](INSTALL_ORDER.md)를 참조
    - 순서에 맞춰서 모듈을 sync
    - sync 방식은 아래와 같음
    1) argocd server에 접속후 로그인
        - argocd server 주소는 다음과 같이 알 수 있음
            ```
            $ kubectl get svc -n argocd argocd-server
            ```
            ![img](../figure/1_main.png)
    
    2) application sync
        - sync하고자 하는 application을 찾아서 sync 버튼을 클릭  
        ![img](../figure/2_app.png)

        - option 및 sync target을 설정하고 synchronize 버튼 클릭
        ![img](../figure/3_sync.png)

    3) sync status 확인
        - sync status(health check, sync check)를 확인
        ![img](../figure/4_synced.png)

        - app card를 누르면 리소스별 status 체크 가능
        ![img](../figure/5_details.png)
---
## Single cluster
1. application file 생성
    - 아래 명령어를 통해 application file을 생성해준다.
    ```
    $ cp application/app_of_apps/single-applications.yaml application/app_of_apps/{{ cluster namespace }}-{{ cluster name }}-applications.yaml
    ```
    ex) "cluster"라는 이름의 클러스터가 default namespace에 있을 경우,  
    ```
    $ cp application/app_of_apps/single-applications.yaml application/app_of_apps/default-cluster-applications.yaml
    ```
---
2. application 변수 셋팅
    - 1번에서 생성한 파일을 수정한다.
    - 변경해야 하는 값은 파일안의 주석을 참조한다.
---
3. application 등록
    - "마스터클러스터 환경"에 application을 등록
        ```
        $ kubectl -n argocd apply -f application/app_of_apps/{{ cluster namespace }}-{{ cluster name }}-applications.yaml
        ```
---
4. resource 배포(application sync)
    - 마스터와 동일