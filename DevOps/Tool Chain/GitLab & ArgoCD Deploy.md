# GitLab & ArgoCD = K8s Deploy

GitLab과 ArgoCD를 이용해 Kubernetes 배포를 설정하는 방법에 대해 다룹니다.

## 사전 작업

- K8s Cluster에 GitLab , ArgoCD pod 설치
- GitLab Group 생성 > subgroups > project 생성 > project 내부에 deploy.yaml 작성 **(예정)**
- GitHub 혹은 Harbor와 같은 image registry에 프로젝트 이미지가 올라가 있어야 한다.
- !!! 여기서는 GitHub를 사용한다.

## GitLab Deploy.yaml 작성

- 사전 작업 확인

![image](https://user-images.githubusercontent.com/96723249/223963837-cbdaf3b4-6d57-49b6-bfb4-f0dc5ea6eaf2.png)

- deploy.yaml 작성
    
    ```yaml
    # Deployment
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: msa1    
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: msa1
      template:
        metadata:
          labels:
            app: msa1
        spec:
          containers:
          - name: msa1
            image: yj0326/msa:msa1 # GitHub project 주소
            ports:
            - containerPort: 80
    
    ---
    # Service
    apiVersion: v1
    kind: Service
    metadata:
      name: msa1-service  
    spec:
      selector:
        app: msa1
      ports:
        - protocol: TCP
          port: 80
          targetPort: 8080
          nodePort: 31111
      type: NodePort
    ```
    

## ArgoCD Repository 등록 (Https 방법)

- ArgoCD 좌측 메뉴 Settings > Repository
    - + CONNECT REPO 클릭
        - VIA HTTPS 선택
        
        ![image](https://user-images.githubusercontent.com/96723249/223963931-11d07282-619c-4574-8cd7-7be8f27482aa.png)

        
- Repository 연결 확인
    
   ![image](https://user-images.githubusercontent.com/96723249/223963970-9406e2be-6754-4d30-827f-85deee012334.png)
    

## Application 등록

- 좌측의 Applications 클릭
    - + NEW APP 클릭
    
    ![image](https://user-images.githubusercontent.com/96723249/223964023-7d735329-2ecd-415c-831b-c7cb2ce64f16.png)
    
    - Name 지정
    - Project Name : default
    - Sync Policy : manual : 수동으로 Sync // Auto : 자동으로 3분마다 Sync
    
    ![image](https://user-images.githubusercontent.com/96723249/223964259-b2381f9a-63ed-4a04-8aae-146c876e5248.png)

    
    - Git URL 입력
    - HEAD / Branches
    - Path에는 Git URL의 하위 프로젝트 폴더 (sync하여 배포 할 해당 deploy.yaml이 있는 Path)
    
    ![image](https://user-images.githubusercontent.com/96723249/223964294-3b996fd2-ca0a-4d1d-bdd1-402e9e1903d9.png)
    
    - Cluster URL 선택
    - Namesapce 지정
        
        ** GitLab deploy.yaml에 namespace가 지정되어 있을 경우, yaml의 Namespace가 우선함
        

## ArgoCD 및 K8s 해당 서버에서 배포 확인

- ArgoCD 에서 확인 및 Sync

![image](https://user-images.githubusercontent.com/96723249/223964355-31861c1a-cbce-486a-8b69-2a31b426f584.png)

- K8s 에서 확인
    
    ![image](https://user-images.githubusercontent.com/96723249/223964387-7e41eb80-512d-4a8c-885a-4216c57c7b8e.png)
    
    - 해당 서버의 Port : 31111 에 접속하여 프로젝트 배포 확인.
