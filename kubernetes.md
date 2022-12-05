### 安裝類型
+ minikube
  + 微型k8s
  + 來自k8s-SIG(Special Interest Group)
  + 基於VM，1.7.0開始可透過參數```--driver=docker```在docker上執行
  + 單節點
+ kind
  + kubernetes in docker
  + 來自k8s-SIG(Special Interest Group)
  + 支援多節點（需要額外設定）
  + 需要docker和Golang
+ kubeadm
  + 提供群集的完整功能

#### kind安裝
參考：https://kind.sigs.k8s.io/
1. Golang安裝
   > https://go.dev/doc/install
+ 下載binary file
    ```bash
    wget -P /tmp https://go.dev/dl/go1.19.3.linux-amd64.tar.gz
    ```
+ 解壓縮
    ```bash
    tar -C /usr/local -xzf /tmp/go1.19.3.linux-amd64.tar.gz
    ```
+ 新增軟連接 or 將/usr/local/go/bin/加到PATH
    ```bash
    sudo ln -sf /usr/local/go/bin/* /usr/local/bin/
    ```

2. 安裝kind
   > https://kind.sigs.k8s.io/
+ 安裝
    ```bash
    # for go 1.17+
    go install sigs.k8s.io/kind@v0.17.0
    
    # else
    GO111MODULE="on" go get sigs.k8s.io/kind@v0.17.0
    ```
+ 將GOPATH加到PATH裡，讓系統找的到安裝的kind
    ```bash
    echo "PATH=$PATH:$(go env GOPATH)/bin" | tee -a ~/.bashrc
    source ~/.bashrc
    ```
+ 創建群集
  > 更詳細 : https://kind.sigs.k8s.io/docs/user/quick-start
    ```bash
    kind create cluster
    
    # 若要刪除
    kind delete cluster
    ```
+ 創建群集(with config)
  > 參考：https://kind.sigs.k8s.io/docs/user/configuration/

  需要先準備一份資源描述文件，命名為「kind_config.yml」
  ```yml
  kind: Cluster
  apiVersion: kind.x-k8s.io/v1alpha4
  name: learning
  nodes:
          - role: control-plane
          - role: control-plane
          - role: control-plane
          - role: worker
          - role: worker
          - role: worker
          - role: worker
          - role: worker
          - role: worker
  ```
  這份文件設定的群集很簡單，只有修改群集名稱以及節點設定，這裡我設定了3個control-plane、6個worker。
  ```bash
  # 使用設定檔建立群集
  kind create cluster --config=kind_config.yml
  
  # 查看狀態
  kind get nodes
  or
  kubectl get nodes

  # 刪除群集，這裡需要指定群集名稱，也就是之前設定檔定義的「learning」
  kind delete cluster --name=learning
  ```

#### 啟用dashboard
> https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

+ 開啟功能
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
    ```
+ 創建管理帳戶
  > **IMPORTANT:** 這種簡單的創見方法將會賦予管理者權限，這會有風險，僅提供教育用途
  > 參考 : https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

    1. 在kubernetes-dashboard之下創建一個服務帳戶(Service Account)
        + 新增文件user.yaml
        ```yml
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: admin-user
          namespace: kubernetes-dashboard
        ```
        + 應用該設定
        ```bash
        # apply和create主要差在對已存在資源的處理，create會報錯，apply會更新
        kubectl apply -f user.yaml

        # 若要清除帳戶
        kubectl -n kubernetes-dashboard delete serviceaccount admin-user
        ```
    2. 給予該帳戶「角色」，也就是賦予權限
        + 新增文件roleBind.yaml
        ```yml
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRoleBinding
        metadata:
          name: admin-user
        roleRef:
          apiGroup: rbac.authorization.k8s.io
          kind: ClusterRole
          name: cluster-admin
        subjects:
        - kind: ServiceAccount
          name: admin-user
          namespace: kubernetes-dashboard
        ```
        + 應用該設定
        ```bash
        kubectl apply -f roleBind.yaml
        
        # 若要清除綁定
        kubectl -n kubernetes-dashboard delete clusterrolebinding admin-user
        ```
    3. 產生token (Bearer Token)
    ```bash
    kubectl -n kubernetes-dashboard create token admin-user
    ```
+ 開啟kubernetes的LoadBalance的代理功能
  > 由於k8s的網路預設是只對內部開放，外部需要訪問的話需要使用代理功能
    ```bash
    # 後綴的&表示該命令將在背景執行，也就是變成daemon
    kubectl proxy &
    ```

+ 接著就可以訪問http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy

+ 輸入前面步驟產生的token即可

#### 部署應用
> 思想上會和部屬在docker上有所不同，在docker上部署對等的是「創建pod」，k8s提供了deployment的資源，是部署的抽象類，行為上是以pod來實現，但提供更高階的功能，可以定義備份數、版本紀錄、滾動升級。
```bash
kubectl create deployment DEPLOYMENT_NAME --image=OCI_IMAGE:TAG

# ex
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```
創建後可以透過 kubectl get 來查詢
```bash
kubectl get deployments
# NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
# kubernetes-bootcamp   1/1     1            1           4m52s

kubectl get pods
# NAME                                   READY   STATUS    RESTARTS   AGE
# kubernetes-bootcamp-75c5d958ff-6zjl2   1/1     Running   0          39s

# 由於kubectl是透過訪問api server實現的，在掛上代理時也能適用REST API訪問
curl http://localhost:8001/api/v1/namespaces/default/pods/
```

