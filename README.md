# GitOps with FluxCD

GitOps was born in 2017 when it was introduced by Weaveworks, a company specializing in Kubernetes and cloud-native technologies. 

GitOps is a software development methodology that leverages Git as the single source of truth for managing infrastructure and application deployments. It involves using Git repositories to store and version control all the desired state configurations for infrastructure and applications. With GitOps, any changes made to the desired state in the Git repository automatically trigger the necessary actions to bring the actual state of the system in line with the desired state.

GitOps is necessary because it provides several benefits over traditional DevOps approaches. Firstly, it brings the benefits of version control to infrastructure and application deployments, allowing for easy rollbacks, auditing, and collaboration. Secondly, it enables a declarative approach to managing infrastructure and application configurations, making it easier to define and maintain desired states. Thirdly, it promotes a self-service model where developers can make changes to the desired state in Git, triggering automated processes to apply those changes, reducing the need for manual intervention.

While DevOps focuses on collaboration and automation between development and operations teams, GitOps extends this concept by introducing Git as the central control plane for managing deployments. It brings the benefits of version control, declarative configuration management, and self-service to the DevOps workflow, making it more efficient and scalable.

![](/images/12-image01.png)

| GitOps | DevOps |
|--------|--------|
| GitOps is a software development methodology that focuses on using Git as the single source of truth for managing infrastructure and application deployments. | DevOps is a software development methodology that focuses on collaboration and communication between development and operations teams to automate and streamline the software delivery process. |
| GitOps uses Git repositories to store and manage infrastructure and application configuration files. | DevOps uses various tools and technologies to automate the software development lifecycle, including continuous integration, continuous delivery, and infrastructure as code. |
| GitOps follows a declarative approach, where the desired state of the infrastructure and applications is defined in Git repositories, and a GitOps tool ensures that the actual state matches the desired state. | DevOps follows an iterative and incremental approach, where developers and operations teams work together to continuously deliver software updates and improvements. |
| GitOps provides a clear audit trail of all changes made to the infrastructure and applications, as all changes are tracked in Git repositories. | DevOps focuses on improving collaboration and communication between teams, enabling faster and more frequent software releases. |
| GitOps allows for easy rollbacks and version control, as all changes are versioned in Git repositories. | DevOps aims to improve the overall software development process, including development, testing, deployment, and monitoring. |
| GitOps is well-suited for cloud-native and containerized environments, as it leverages Git's version control capabilities and integrates with container orchestration platforms like Kubernetes. | DevOps can be applied to any software development environment, regardless of the technology stack or infrastructure. |
| GitOps provides a centralized and standardized approach to managing infrastructure and application deployments, making it easier to maintain consistency and reliability. | DevOps promotes collaboration and cross-functional teams, enabling faster feedback loops and continuous improvement. |

## FluxCD

FluxCD is a popular open-source tool that supports GitOps principles and helps with continuous delivery for Kubernetes.

FluxCD provides a robust framework for implementing GitOps principles in Kubernetes environments. It automates the synchronization between the Git repository and the cluster, supports automated deployments, rollbacks, and rollouts, and treats the Git repository as the single source of truth for the cluster's configuration.

![](/images/12-image02.png)

herewith the command to install flux and connecting to GitHub with PAT

```txt
$ curl -s https://fluxcd.io/install.sh | sudo bash
    [INFO]  Downloading metadata https://api.github.com/repos/fluxcd/flux2/releases/latest
    [INFO]  Using 2.2.3 as release
    [INFO]  Downloading hash https://github.com/fluxcd/flux2/releases/download/v2.2.3/flux_2.2.3_checksums.txt
    [INFO]  Downloading binary https://github.com/fluxcd/flux2/releases/download/v2.2.3/flux_2.2.3_linux_amd64.tar.gz
    [INFO]  Verifying binary download
    [INFO]  Installing flux to /usr/local/bin/flux

$ flux --version
    flux version 2.2.3

$ flux bootstrap github
    Please enter your GitHub personal access token (PAT): 
    ► connecting to github.com
    ✗ failed to get Git repository "https://github.com//": provider error: multiple errors occurred: 
    - validation error for OrgRepositoryRef.Organization: field is required
    - validation error for OrgRepositoryRef.RepositoryName: field is required
```

Please note that I use KinD (Kubernetes in Docker) for this purpose.<br>
Below is an example using KinD cluster

```txt
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.21.0/kind-linux-amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    97  100    97    0     0    400      0 --:--:-- --:--:-- --:--:--   400
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 6245k  100 6245k    0     0  6160k      0  0:00:01  0:00:01 --:--:-- 6160k

$ chmod +x ./kind
$ sudo mv ./kind /usr/local/bin/
$ kind --version
    kind version 0.21.0

$ sudo kind create cluster --config kindcluster.yaml
    Creating cluster "kind" ...
    ✓ Ensuring node image (kindest/node:v1.29.1) 🖼
    ✓ Preparing nodes 📦 📦 📦  
    ✓ Writing configuration 📜 
    ✓ Starting control-plane 🕹️ 
    ✓ Installing CNI 🔌 
    ✓ Installing StorageClass 💾 
    ✓ Joining worker nodes 🚜 
    Set kubectl context to "kind-kind"
    You can now use your cluster with:

    kubectl cluster-info --context kind-kind

    Thanks for using kind! 😊

$ sudo kubectl get nodes
    NAME                 STATUS   ROLES           AGE   VERSION
    kind-control-plane   Ready    control-plane   98s   v1.29.1
    kind-worker          Ready    <none>          63s   v1.29.1
    kind-worker2         Ready    <none>          60s   v1.29.1
```
But we will use K3S instead.
```txt
sudo apt-get update
sudo apt-get install -y apt-transport-https curl
curl -sfL https://get.k3s.io | sh -
sudo systemctl status k3s
sudo cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config
```
we can also test flux before bootstrapping
```txt
$ sudo flux check --pre --kubeconfig=kubeconfig.yaml
► checking prerequisites
✔ Kubernetes 1.28.6+k3s2 >=1.26.0-0
✔ prerequisites checks passed
```

Now, we can safely create a new github repo flux-infra as follow

```txt
$ flux bootstrap github --kubeconfig=kubeconfig.yaml --owner=FariusGitHub --repository=flux-infra --branch=main --path=./cluster/dev --personal --network-policy=false --components=source-controller,kustomize-controller --token-auth
Please enter your GitHub personal access token (PAT): 
► connecting to github.com
✔ repository "https://github.com/FariusGitHub/flux-infra" created
► cloning branch "main" from Git repository "https://github.com/FariusGitHub/flux-infra.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ committed component manifests to "main" ("d8702420ea3d12771456de6b276fbf1a1171637d")
► pushing component manifests to "https://github.com/FariusGitHub/flux-infra.git"
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ committed sync manifests to "main" ("5f999a39ab057d7a839a96a1b50512928730adb9")
► pushing sync manifests to "https://github.com/FariusGitHub/flux-infra.git"
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for GitRepository "flux-system/flux-system" to be reconciled
✔ GitRepository reconciled successfully
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ kustomize-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
```
Let's see FariusGitHub repo below. We found a newly repo was created.
Inside cluster/dev/flux-system we should three files like below
1. gotk-components.yaml
2. gotk-sync.yaml
3. kustomization.yaml <br>

![](/images/12-image03.png)

Let's review the new flux-system namespace resources

```txt
$ kubectl get ns
    NAME              STATUS   AGE
    kube-system       Active   32h
    kube-public       Active   32h
    kube-node-lease   Active   32h
    default           Active   32h
    flux-system       Active   17m

$ kubectl get all -n flux-system
    NAME                                        READY   STATUS    RESTARTS   AGE
    pod/source-controller-597c57496d-f2ns6      1/1     Running   0          17m
    pod/kustomize-controller-59768b4b58-26zhv   1/1     Running   0          17m

    NAME                        TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/source-controller   ClusterIP   10.43.19.1   <none>        80/TCP    17m

    NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/kustomize-controller   1/1     1            1           17m
    deployment.apps/source-controller      1/1     1            1           17m

    NAME                                              DESIRED   CURRENT   READY   AGE
    replicaset.apps/source-controller-597c57496d      1         1         1       17m
    replicaset.apps/kustomize-controller-59768b4b58   1         1         1       17m

$ kubectl get crds -n flux-system
    NAME                                         CREATED AT
    buckets.source.toolkit.fluxcd.io             2024-02-18T07:00:28Z
    gitrepositories.source.toolkit.fluxcd.io     2024-02-18T07:00:28Z
    helmcharts.source.toolkit.fluxcd.io          2024-02-18T07:00:28Z
    helmrepositories.source.toolkit.fluxcd.io    2024-02-18T07:00:29Z
    kustomizations.kustomize.toolkit.fluxcd.io   2024-02-18T07:00:30Z
    ocirepositories.source.toolkit.fluxcd.io     2024-02-18T07:00:30Z

$ kubectl get gitrepository -n flux-system
    NAME          URL                                              AGE   READY   STATUS
    flux-system   https://github.com/FariusGitHub/flux-infra.git   8m    True    stored artifact for revision 'main@sha1:5f999a39ab057d7a839a96a1b50512928730adb9'
```
Take a look at spec section from flux-system gitrepository below we just created. <br>
The Timeout: 60s in the spec section of the GitRepository resource means that the synchronization process between the Git repository and the cluster will timeout after 60 seconds if it does not complete within that time frame. This setting helps to prevent the synchronization process from hanging indefinitely in case of any issues or delays.
```txt
$ kubectl describe gitrepository flux-system -n flux-system
    ...    
    Spec:
        Interval:  1m0s
        Ref:
            Branch:  main
        Secret Ref:
            Name:   flux-system
        Timeout:  60s
        URL:      https://github.com/FariusGitHub/flux-infra.git
    ...
```

Now, let's see what the app repo doing from step 1 to step 6 below.<br>
As the change was made, it would then trigger package changes, tell K8S, tell Kustomize, pick the changes and finally reconciled<br>

![](/images/12-image04.png)

## INSTAVOTE
We will use an example of instavote below as an app repo

```txt
$ git clone https://github.com/FariusGitHub/instavote
$ cd instavote
$ sudo apt install tree
$ tree
    .
    ├── LICENSE
    ├── MAINTAINERS
    ├── README.md
    ├── architecture.png
    ├── deploy
    │   ├── redis
    │   │   ├── redis-deployment.yaml
    │   │   └── redis-service.yaml
    │   └── vote
    │       ├── vote-deployment.yaml
    │       └── vote-service.yaml
    ├── e2e
    │   ├── docker-compose.test.yml
    │   └── tests
    │       ├── Dockerfile
    │       ├── render.js
    │       └── tests.sh
    ├── result
    │   ├── Dockerfile
    │   ├── Dockerfile-scratch
    │   ├── package.json
    │   ├── server.js
    │   ├── sonar-project.properties
    │   └── views
    │       ├── app.js
    │       ├── index.html
    │       ├── socket.io.js
    │       └── stylesheets
    │           └── style.css
    ├── vote
    │   ├── Dockerfile
    │   ├── Dockerfile-scratch
    │   ├── app.py
    │   ├── requirements.txt
    │   ├── sonar-project.properties
    │   ├── static
    │   │   └── stylesheets
    │   │       └── style.css
    │   ├── templates
    │   │   └── index.html
    │   └── tests
    │       └── test_frontend.py
    └── worker
        ├── pom.xml
        ├── sonar-project.properties
        ├── src
        │   ├── Worker
        │   │   ├── Program.cs
        │   │   └── Worker.csproj
        │   ├── main
        │   │   └── java
        │   │       └── worker
        │   │           └── Worker.java
        │   └── test
        │       └── java
        │           └── worker
        │               └── UnitWorker.java
        └── target
            └── classes
                └── worker
                    └── Worker.class

    25 directories, 36 files
```
we would make some changes here to trigger the CD
```txt
$ tree
.
├── redis
│   ├── redis-deployment.yaml
│   └── redis-service.yaml
└── vote
    ├── vote-deployment.yaml
    └── vote-service.yaml
```
A python webapp will lets you vote between two options.<br>
Therefore whatever inside the vote container pod need to come to outside world and service is the way to do that. 
![](/images/12-image05.png)

Those deployment and service yaml files were initially created  like below through dry runs, where vote folder we could do below. 

```txt
$ kubectl create deployment vote --image=schooldevops/vote:v1 --replicas 2 --dry-run=client -o yaml > vote-deployment.yaml

$ cat vote-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    creationTimestamp: null
    labels:
        app: vote
    name: vote
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: vote
    strategy: {}
    template:
        metadata:
        creationTimestamp: null
        labels:
            app: vote
        spec:
        containers:
        - image: schooldevops/vote:v1
            name: vote
            resources: {}
    status: {}

$ kubectl create service nodeport vote --tcp=80 --node-port=30000 --dry-run=client -o yaml > vote-service.yaml

$ cat vote-service.yaml
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: null
    labels:
        app: vote
    name: vote
    spec:
    ports:
    - name: "80"
        nodePort: 30000
        port: 80
        protocol: TCP
        targetPort: 80
    selector:
        app: vote
    type: NodePort
    status:
    loadBalancer: {}
```
and similarly inside redis folder we could do below

```txt
$ kubectl create deployment redis --image=redis:alpine --dry-run=client -o yaml > redis-deployment.yaml

$ cat redis-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    creationTimestamp: null
    labels:
        app: redis
    name: redis
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: redis
    strategy: {}
    template:
        metadata:
        creationTimestamp: null
        labels:
            app: redis
        spec:
        containers:
        - image: redis:alpine
            name: redis
            resources: {}
    status: {}

$ kubectl create service clusterip redis --tcp=6379 --dry-run=client -o yaml > redis-service.yaml
$ cat redis-service.yaml
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: null
    labels:
        app: redis
    name: redis
    spec:
    ports:
    - name: "6379"
        port: 6379
        protocol: TCP
        targetPort: 6379
    selector:
        app: redis
    type: ClusterIP
    status:
    loadBalancer: {}
```
Next, we will create a new namespace called instavote

```txt
$ kubectl get ns
    NAME              STATUS   AGE
    kube-system       Active   39h
    kube-public       Active   39h
    kube-node-lease   Active   39h
    default           Active   39h
    flux-system       Active   7h53m

$ kubectl create ns instavote
    namespace/instavote created

$ kubectl get ns
    NAME              STATUS   AGE
    kube-system       Active   39h
    kube-public       Active   39h
    kube-node-lease   Active   39h
    default           Active   39h
    flux-system       Active   7h54m
    instavote         Active   2s
```
Imagine Source Controller below as a nosy neighbour. The arrow between App Repo and Source Controller is supposed to be the other way around.
Ideally like the arrow line between K8S Events Queue and Kustomize Controller. 

![](/images/12-image06.png)

Let's see

```txt
$ cat kubeconfig.yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tL...FLS0tLS0K
    server: https://127.0.0.1:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    client-certificate-data: LS0tLS1...0tCg==
    client-key-data: LS0t...LS0tLQo=

$ flux get sources git --kubeconfig=kubeconfig.yaml
    ✗ no GitRepository objects found in "flux-system" namespace

$ flux create source git instavote --url https://github.com/FariusGitHub/instavote.git --branch main --interval 30s --kubeconfig=kubeconfig.yaml
    ✚ generating GitRepository source
    ► applying GitRepository source
    ✔ GitRepository source created
    ◎ waiting for GitRepository source reconciliation
    ✔ GitRepository source reconciliation completed
    ✔ fetched revision: main@sha1:6510065228d5a56bc54dfc7d8e18d6fadb3d3b88

$ flux get sources git --kubeconfig=kubeconfig.yaml
    NAME       	REVISION          	SUSPENDED	READY	MESSAGE                                           
    flux-system	main@sha1:34fc5af6	False    	True 	stored artifact for revision 'main@sha1:34fc5af6'	
    instavote  	main@sha1:65100652	False    	True 	stored artifact for revision 'main@sha1:65100652'	
```
A new source controller was created. Next, we will also create a new Kustomization controller vote-dev as follow.<br>
![](/images/12-image07.png)

```txt
$ flux get kustomizations --kubeconfig=kubeconfig.yaml
    NAME       	REVISION          	SUSPENDED	READY	MESSAGE                              
    flux-system	main@sha1:5f999a39	False    	True 	Applied revision: main@sha1:5f999a39	

$ flux create kustomization vote-dev --source=instavote --path="./deploy/vote" --interval=1m --target-namespace=instavote --kubeconfig=kubeconfig.yaml
    ✚ generating Kustomization
    ► applying Kustomization
    ✔ Kustomization created
    ◎ waiting for Kustomization reconciliation
    ✔ Kustomization vote-dev is ready
    ✔ applied revision main@sha1:6510065228d5a56bc54dfc7d8e18d6fadb3d3b88

$ flux get kustomizations --kubeconfig=kubeconfig.yaml
NAME       	REVISION          	SUSPENDED	READY	MESSAGE                              
flux-system	main@sha1:5f999a39	False    	True 	Applied revision: main@sha1:5f999a39	
vote-dev   	main@sha1:65100652	False    	True 	Applied revision: main@sha1:65100652	
```

Please note, we did not do any kubectl create deploy. But below pods happened by its own as we created a bootstrapping, source and kustomization. The first time we create a source, fluxCD will do a complimentary pods below even we do not make any changes.

```txt
$ kubectl get pods -n instavote
    NAME                    READY   STATUS    RESTARTS   AGE
    vote-649dc6575c-5tvxz   1/1     Running   0          2m46s
    vote-649dc6575c-jlgjk   1/1     Running   0          2m46s
    vote-649dc6575c-c26nh   1/1     Running   0          2m46s
```
As soon as we change something like the number of replicas in GitHub repo, we can see the consequence below subsequently.
![](/images/12-image08.png)

```txt
$ kubectl get pods -n instavote
NAME                    READY   STATUS    RESTARTS   AGE
vote-649dc6575c-stspp   1/1     Running   0          14m
vote-649dc6575c-9vf44   1/1     Running   0          14m
```

# SUMMARY
Flux is a tool that automates the deployment of applications and infrastructure changes in a Kubernetes cluster. Flux source is a component of Flux that monitors a Git repository for changes to the configuration files of the applications running in the cluster. Flux kustomization is a way to define customizations to the deployment process, such as specifying which images to use or which environment variables to set.

Continuous Deployment with Flux involves the following steps:

1. Developers push changes to the Git repository containing the configuration files for the applications running in the Kubernetes cluster.

2. Flux source monitors the Git repository for changes and detects when a new commit has been pushed.

3. Flux kustomization applies the changes specified in the configuration files to the Kubernetes cluster. This can include updating the image used by a deployment, scaling up or down the number of replicas, or making other configuration changes.

4. The changes are automatically deployed to the cluster, ensuring that the applications are always running the latest version of the code.

By using Flux source and Flux kustomization, teams can automate the deployment process and ensure that changes are deployed quickly and consistently. This helps to reduce the risk of errors and allows teams to focus on developing new features and improving their applications.