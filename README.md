# Helm 101

- package manager for Kubernetes
- 將一個服務所需的 yaml 打包成 chart，透過賦值的方式，管理與設定這些 yaml
- 假設想要 deploy 一個 ELK stack 到 k8s，可能需要自己寫 deployment, service, ConfigMap, secret 等的 yaml, helm chart 會把這些都封裝好，使用時可以直接 deploy
- templating engine
- 一個 chart 可以被安裝多次到同一個 cluster，每一個都可以獨立地被管理及升級 (one chart can be installed multiple times into the same cluster. And each can be independently managed and upgraded.)

![流程圖](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/3f393187-d247-4bd9-880b-2752d9561bb4/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230308%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230308T152112Z&X-Amz-Expires=86400&X-Amz-Signature=057ad540d5a06131942c468c6406fad032caa0b570f1d4e31891a80c6958cfea&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

## How to use

首先要先安裝 [Helm cli](https://helm.sh/docs/intro/install/#from-script)

使用 Windows 的話，建議使用 package manager (chocolatey, scoop 等等)，會把環境變數的問題處理好，這邊我使用 scoop (Mac 可以使用 homebrew)

```bash
scoop install helm
```

再來把 repo 加進來，就可以安裝要使用的 chart

```bash
# Initialize a Helm Chart Repository
# local helm client add reference of this remote repo
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install my-nginx bitnami/nginx
```

接著到 http://localhost:80/ 就會看到 `Welcome to nginx!` 頁面

--

試著 upgrade

```bash
# 吃 custom-values.yaml
helm upgrade my-nginx bitnami/nginx --values=custom-values.yaml
```

網址要改成 http://localhost:8080/

因為 custom-values.yaml 裡面定義的 port 為 `8080`

試著確認版本，會看到 REVISION 版本變成 2

```bash
❯ helm ls
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
my-nginx        default         2               2023-03-09 00:16:53.5755009 +0800 CST   deployed        nginx-13.2.29   1.23.3
```

--

看歷史紀錄

```bash
❯ helm history my-nginx
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Thu Mar  9 00:30:58 2023        superseded      nginx-13.2.29   1.23.3          Install complete
2               Thu Mar  9 00:31:24 2023        deployed        nginx-13.2.29   1.23.3          Upgrade complete
```

--

rollback

```bash
# 回版本 1
helm rollback my-nginx 1
```

確認版本會發現變成 3 (雖然 rollback 回版本 1，實際版本還是會往上疊加)

```bash
❯ helm ls
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
my-nginx        default         3               2023-03-09 00:22:39.3289515 +0800 CST   deployed        nginx-13.2.29   1.23.3
```

會發現 http://localhost:8080/ 已經不能用，要改成 http://localhost:80/

## Command

```bash
# Make sure we get the latest list of charts
helm repo update

# local 有什麼 repo
helm repo ls

# List all released chart
helm list
helm ls

# install
helm install my-nginx bitnami/nginx # 吃 defaut 參數
helm install my-nginx-2 bitnami/nginx --values=custom-values.yaml # override values

# uninstall
helm uninstall release-name

# upgrade (version 會 +1)
helm upgrade [release-name] bitnami/nginx --values=custom-values.yaml # with values

# check history
helm history [release-name]

# rollback (version 會 +1)
helm rollback [release-name] [version]

# lint
helm lint

# download helm chart
helm fetch bitnami/nginx
```

## Chart files structure

執行 `helm create [your-chart-name]` 後會產生 default 檔案 (可以先把 mychart 資料夾刪掉再執行，以下用 mychart 代稱)

- 檔案結構

  ```bash
  mychart
  ├── Chart.yaml # 定義 metadata (apiVersion, name, description 等)
  ├── charts # chart dependencies
  ├── templates # actual template files
  │   ├── NOTES.txt # client 執行 helm install 時的提示訊息
  │   ├── _helpers.tpl
  │   ├── deployment.yaml # k8s deployment yaml
  │   ├── ingress.yaml # k8s ingress yaml
  │   ├── service.yaml # k8s service yaml
  │   ├── serviceaccount.yaml # k8s serviceaccount yaml
  │   └── tests
  │       └── test-connection.yaml
  └── values.yaml # template files 的值
  ```

- service.yaml

  `mychart.` 開頭的是從 `_helpers.tpl` 定義的

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: { { include "mychart.fullname" . } }
    labels: { { - include "mychart.labels" . | nindent 4 } }
  spec:
    type: { { .Values.service.type } }
    ports:
      - port: { { .Values.service.port } }
        targetPort: http
        protocol: TCP
        name: http
    selector: { { - include "mychart.selectorLabels" . | nindent 4 } }
  ```

- values.yaml (節錄其中一段)

  使用 `helm install`  或  `helm upgrade` 就會 render 到 `templates/service.yaml` 裡面的 `{{ .Values.service.type }}` 和 `{{ .Values.service.port }}`

  ```yaml
  service:
    type: ClusterIP
    port: 80
  ```

## How I learn Helm with ChatGPT

TLDR: 圖片大概就是一個被 ChatGPT 呼嚨的故事

我想要找把 `helm repo 刪掉的 command`，猜測是 `helm repo delete`

ChatGPT 一本正經地教我怎麼用

我試了一下發現不能用 XD

然後 ChatGPT 還是一本正經地講幹話 XDD

所以這一題我最後是 Google 來的

![](https://i.imgur.com/qg1wdEL.png)

![](https://i.imgur.com/gGdW8XH.png)

![](https://i.imgur.com/3JAIZLs.png)

## Reference

- [Kubernetes 基礎教學（三）Helm 介紹與建立 Chart](https://cwhu.medium.com/kubernetes-helm-chart-tutorial-fbdad62a8b61) (此篇使用舊版的 helm v2)

- [使用 Helm 管理 Kubernetes 应用 · Kubernetes 中文指南——云原生应用架构实战手册](https://jimmysong.io/kubernetes-handbook/practice/helm.html#使用helm管理kubernetes应用)
