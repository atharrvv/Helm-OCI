##### Deploying Helm Charts Through FluxCD while storing on OCI Repository! DockerHub

### The idea was to Deploy helm chart through Flux with image update automation but as the core maintainer of flux stated that “highly inefficient and error prone”

### Source: https://github.com/fluxcd/flux2/discussions/4307#discussioncomment-7216060

### As he suggested the correct way:

- **Using an OCI-based registry :** https://helm.sh/docs/topics/registries/#helm-repositories-in-oci-based-registries
- **Helm-Controller for Helm Release** : https://fluxcd.io/flux/guides/helmreleases/#oci-repository

### But was facing issue while performing Flux official documentation, so i raised an issue on Git Hub and He acknowledge that and made corrections in document

### Source: https://github.com/fluxcd/website/pull/2208

### OCI **Open Container Initiative**

### Helm

- **We can store Helm chart on Registries as Artifact**

## Requriments - Installation

- Docker
    
    ```bash
    apt update
    apt install docker.io
    ```
    
- Kubectl
    
    Documentation: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
    
    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
    echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    chmod +x kubectl
    mkdir -p ~/.local/bin
    mv ./kubectl ~/.local/bin/kubectl
    # and then append (or prepend) ~/.local/bin to $PATH
    ```
    
- Flux CLI
    
    Documentation: https://fluxcd.io/flux/installation/#install-the-flux-cli
    
    ```bash
    curl -s https://fluxcd.io/install.sh | sudo bash
    . <(flux completion bash)
    ```
    
- Kind Cluster
    
    Documentation: https://kind.sigs.k8s.io/docs/user/quick-start/
    
    ```bash
    [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-amd64
    chmod +x ./kind
    mv ./kind /usr/local/bin/kind
    ```
    
- Helm
    
    ```bash
    curl -LO https://get.helm.sh/helm-v3.17.3-linux-amd64.tar.gz
    tar -C /tmp/ -zxvf helm-v3.17.3-linux-amd64.tar.gz
    rm helm-v3.17.3-linux-amd64.tar.gz
    mv /tmp/linux-amd64/helm /usr/local/bin/helm
    chmod +x /usr/local/bin/helm
    ```
    
- Jenkins
    
    ```bash
    apt-get update
    apt-get install default-jdk -y
    wget -O /usr/share/keyrings/jenkins-keyring.asc  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]"  https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
    apt-get update
    apt-get install jenkins docker.io docker-compose -y
    usermod -a -G docker jenkins
    systemctl restart jenkins
    systemctl restart docker
    ```
    

### Setting up GitHub Respository for CI

https://github.com/atharrvv/Helm-OCI.git

```bash
.
├── Dockerfile
├── Jenkinsfile
├── app.py
└── chart
    └── application
        ├── Chart.yaml
        ├── templates
        │   ├── _helpers.tpl
        │   ├── deployment.yaml
        │   └── service.yaml
        └── values.yaml
```

- Includes:
    - Helm Chart
        
        ### Helm Chart
        
        ```bash
        application/
        ├── Chart.yaml
        ├── templates
        │   ├── _helpers.tpl
        │   ├── deployment.yaml
        │   └── service.yaml
        └── values.yaml
        ```
        
        **Chart.yaml**
        
        ```bash
        apiVersion: v2
        name: application
        description: A Helm chart for Kubernetes
        
        # A chart can be either an 'application' or a 'library' chart.
        #
        # Application charts are a collection of templates that can be packaged into versioned archives
        # to be deployed.
        #
        # Library charts provide useful utilities or functions for the chart developer. They're included as
        # a dependency of application charts to inject those utilities and functions into the rendering
        # pipeline. Library charts do not define any templates and therefore cannot be deployed.
        type: application
        
        # This is the chart version. This version number should be incremented each time you make changes
        # to the chart and its templates, including the app version.
        # Versions are expected to follow Semantic Versioning (https://semver.org/)
        version: 0.1.0
        
        # This is the version number of the application being deployed. This version number should be
        # incremented each time you make changes to the application. Versions are not expected to
        # follow Semantic Versioning. They should reflect the version the application is using.
        # It is recommended to use it with quotes.
        appVersion: "1.16.0"
        ```
        
        **deployment.yaml**
        
        ```bash
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: {{ .Values.name }}
          labels:
            app: {{ .Values.name }}
          namespace: {{ .Values.deployment.namespace }}
        spec:
          selector:
            matchLabels:
              app: {{ .Values.name }}
          replicas: {{ .Values.deployment.replicas | default 2 }}
          strategy:
            type: RollingUpdate
            rollingUpdate:
              maxSurge: 1
              maxUnavailable: 0
          template:
            metadata:
              labels:
                app: {{ .Values.name }}
            spec:
              containers:
              - name: {{ .Values.name }}
                image: {{ .Values.deployment.image }}:{{ .Values.deployment.tag }}
                ports:
                - containerPort: 5000
        ```
        
        **service.yaml**
        
        ```bash
        apiVersion: v1
        kind: Service
        metadata:
          namespace: {{ .Values.deployment.namespace }}
          name: {{ .Values.name }}
          labels:
            app: {{ .Values.name }}
        spec:
          type: {{ .Values.service.type }}
          selector:
            app: {{ .Values.name }}
          ports:
            - protocol: TCP
              name: http
              port: 80
              targetPort: 5000
        ```
        
        **values.yaml**
        
        ```bash
        name: "python-staging-application"
        
        # Deployment
        deployment:
          image: "docker.io/eatherv/python-application"
          tag: "0.0.9" 
          replicas: "3"
          namespace: "app"
        
        # Service
        service:
          type: "ClusterIP"
        ```
        
    - Source
        
        ```bash
        from flask import Flask
        app = Flask(__name__)
        
        @app.route("/")
        def hello():
            return "Hello World! v1.0.0.2"
        ```
        
    - Dockerfile
        
        ```bash
        FROM python:3.7.3-alpine3.9 as base
        
        RUN pip install Flask==2.0.3
        
        WORKDIR /app
        COPY app.py /app/
        ENV FLASK_APP=app.py
        CMD flask run -h 0.0.0 -p 5000
        ```
        
    - Jenkinsfile
        
        ```bash
        pipeline {
          agent any 
        
          environment {
            CHART_NAME = "application"
            CHART_VERSION = "4.0.0"
            OCI_REGISTRY = "oci://registry-1.docker.io/eatherv"
          }
        
          stages {
            stage ('docker buildd') {
              steps {
                script {
                  docker.build('eatherv/python-application:0.0.3', '.')
                }
              }
            }
            stage ('docker push') {
              steps {
                script {
                  docker.withRegistry('https://index.docker.io/v1/', 'docker') {
                      docker.image("eatherv/python-application:0.0.3").push()
                  }
                }
              }
            }
            stage ('helm package ') {
              steps {
                script {
                  sh "helm package ./chart/application --version ${CHART_VERSION}"
                }
              }
            }
            stage ("Helm Docker Login") {
              steps {
                script {
                  withCredentials([usernamePassword(
                            credentialsId: 'docker',  // Update with your Jenkins credential ID
                            usernameVariable: 'USER',
                            passwordVariable: 'TOKEN'
                        )]) {
                    sh """ helm registry login registry-1.docker.io -u ${USER} -p ${TOKEN} """
                    }
                  // sh "helm registry login registry-1.docker.io -u eatherv -p"
                }
              }
            }
             stage ('helm push') {
              steps {
                script {
                  sh "helm push ${CHART_NAME}-${CHART_VERSION}.tgz ${OCI_REGISTRY}"
                }
              }
            }
          }
        }
        
        ```
        

### Jenkins Setup: Credentials and GitHub Webhook

![Jenkins password](attachment:b92bf4b2-7f49-43ab-b71c-b65e1c023671:image.png)

Jenkins password

```bash
root@ip:/home/ubuntu# cat /var/lib/jenkins/secrets/initialAdminPassword
9ba12bb208974f2c8c01bf2ce4ff1e06

root@ip:/home/ubuntu# systemctl status jenkins
Warning: The unit file, source configuration file or drop-ins of jenkins.service changed on disk. Run 'systemctl daemon-reload' to reload units.
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/usr/lib/systemd/system/jenkins.service; enabled; preset: enabled)
    Drop-In: /etc/systemd/system/jenkins.service.d
             └─override.conf
     Active: active (running) since Sun 2025-05-04 09:03:38 UTC; 1min 27s ago
   Main PID: 4900 (java)
      Tasks: 37 (limit: 1129)
     Memory: 243.0M (peak: 258.9M)
        CPU: 13.443s
     CGroup: /system.slice/jenkins.service
             └─4900 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=9090

May 04 09:03:34 ip-172-31-85-222 jenkins[4900]: Jenkins initial setup is required. An admin user has been created and a password generated.
May 04 09:03:34 ip-172-31-85-222 jenkins[4900]: Please use the following password to proceed to installation:
May 04 09:03:34 ip-172-31-85-222 jenkins[4900]: 9ba12bb208974f2c8c01bf2ce4ff1e06
May 04 09:03:34 ip-172-31-85-222 jenkins[4900]: This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword
May 04 09:03:34 ip-172-31-85-222 jenkins[4900]: *************************************************************
May 04 09:03:34 ip-172-31-85-222 jenkins[4900]: *************************************************************
May 04 09:03:34 ip-172-31-85-222 jenkins[4900]: *************************************************************
May 04 09:03:38 ip-172-31-85-222 jenkins[4900]: 2025-05-04 09:03:38.541+0000 [id=30]        INFO        jenkins.InitReactorRunner$1#onAttained: Completed i>
May 04 09:03:38 ip-172-31-85-222 jenkins[4900]: 2025-05-04 09:03:38.587+0000 [id=24]        INFO        hudson.lifecycle.Lifecycle#onReady: Jenkins is full>
May 04 09:03:38 ip-172-31-85-222 systemd[1]: Started jenkins.service - Jenkins Continuous Integration Server.
```

![Install suggested plugins](attachment:75b288d6-16b9-4cbe-978a-5208ab89f60b:image.png)

Install suggested plugins

![Create New User](attachment:8d35b0ec-7fcd-43b4-b182-873556437a37:image.png)

Create New User

![Our jenkins URL](attachment:c0eaa721-3749-4681-bdc1-591db9cda7e5:image.png)

Our jenkins URL

![Dashboard>Manage Jenkins>Plugins](attachment:93d95373-e2f4-44e0-bf03-dba2c59d99fd:image.png)

Dashboard>Manage Jenkins>Plugins

![Install GitHub and Docker plugins](attachment:8fd42b83-4948-45ba-ac02-bdaf4b9ccfdc:image.png)

Install GitHub and Docker plugins

### Credentials

![Dashboard>Manage Jenkins>Credentials](attachment:de7bded5-09c0-4955-9bfe-c87bec11232b:image.png)

Dashboard>Manage Jenkins>Credentials

![image.png](attachment:671cdfc5-5f52-4e1b-a8d3-5f7d7efbda8d:image.png)

![Add GitHub Username, Personal Access Token and ID for GitHub](attachment:e2b96d3d-bf7f-4252-9b29-4b1934ca7311:image.png)

Add GitHub Username, Personal Access Token and ID for GitHub

**Same for Docker aswell** 

### GitHub WebHook

![image.png](attachment:fb686d76-f06e-4d48-a7bb-867a58889fd8:image.png)

![Add Jenkins URL http://<JENKINS_URL>/github-webhook/](attachment:654c9078-bea6-4b99-b51a-bdc0cd8bf650:image.png)

Add Jenkins URL http://<JENKINS_URL>/github-webhook/

### FluxCD Setup

- Export GitHub Credentials or add to .bashrc

```bash
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>
```

- Cluster check

```bash
flux check --pre
► checking prerequisites
✔ Kubernetes 1.32.2 >=1.30.0-0
✔ prerequisites checks passed
```

- Flux Bootstrapping

```bash
  flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=flux-fleet-infra \
  --branch=main \
  --path=./clusters/my-cluster \
  --components-extra=image-reflector-controller,image-automation-controller \
  --personal
```

- Output

```bash
► connecting to github.com
✔ repository "https://github.com/atharrvv/flux-dockerhub" created
► cloning branch "main" from Git repository "https://github.com/atharrvv/flux-dockerhub.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ committed component manifests to "main" ("25cb217489f3a34ce1eff03a8d85e237c766c5c6")
► pushing component manifests to "https://github.com/atharrvv/flux-dockerhub.git"
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
✔ public key: ecdsa-sha2-nistp384 AAAAE2VjZHNhLXNoYTItbmlzdHAzODQAAAAIbmlzdHAzODQAAABhBK28EBWTk+ZXS0aZRMmesDeXBJIxhcRDkkRqMNx2SbvltfiH3j2o/3n7HGsUDe4sxPDujTIlQS+dzNDF1/8oVdtCVLFNO9izIPvhxxaB1TtGHyxdC+d8CqoBkcYRatMyRA==
✔ configured deploy key "flux-system-main-flux-system-./clusters/my-cluster" for "https://github.com/atharrvv/flux-dockerhub"
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ committed sync manifests to "main" ("26d7d4895b1a68827158751850cc385953d51e79")
► pushing sync manifests to "https://github.com/atharrvv/flux-dockerhub.git"
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for GitRepository "flux-system/flux-system" to be reconciled
✔ GitRepository reconciled successfully
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ image-automation-controller: deployment ready
✔ image-reflector-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
```

### OCI Repository

- Now we will store our Helm Charts to OCI Repository i.e DockerHub
- It will monitor the repository and will look for a change
- sources: https://helm.sh/docs/topics/registries/ https://fluxcd.io/flux/components/source/ocirepositories/

```bash
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: python
  namespace: flux-system
spec:
  interval: 1m
  url: oci://registry-1.docker.io/eatherv/application
  ref:
    semver: ">=0.1.0"
```

### Helm Release

- Helm Release will install the Helm Chart on Cluster

```bash
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: python
  namespace: flux-system
spec:
  interval: 1m
  releaseName: python
  chartRef:
    kind: OCIRepository
    name: python
```

```bash
root@atharv:/home/ubuntu/dhanush# ls
helmrelease.yaml  ocirepository.yaml

root@atharv:/home/ubuntu/dhanush# kubectl apply -f ocirepository.yaml ; kubectl apply -f helmrelease.yaml
ocirepository.source.toolkit.fluxcd.io/python created
helmrelease.helm.toolkit.fluxcd.io/python created
```

```bash
root@atharv:/home/ubuntu/dhanush# kubectl describe ocirepository -n flux-system
Name:         python
Namespace:    flux-system
Labels:       <none>
Annotations:  <none>
API Version:  source.toolkit.fluxcd.io/v1beta2
Kind:         OCIRepository
Metadata:
  Creation Timestamp:  2025-05-04T09:49:53Z
  Finalizers:
    finalizers.fluxcd.io
  Generation:        1
  Resource Version:  304814
  UID:               6e5e5f49-8fde-4c45-ad99-32d0f04a977f
Spec:
  Interval:  1m
  Provider:  generic
  Ref:
    Semver:  >=0.1.0
  Timeout:   60s
  URL:       oci://registry-1.docker.io/eatherv/application
Status:
  Artifact:
    Digest:            sha256:0f7825e84984dc4219ec97dffbda1b3f1c9a6f7b19f5be4639a93bb6293d944d
    Last Update Time:  2025-05-04T09:49:54Z
    Metadata:
      org.opencontainers.image.created:      2025-05-03T11:12:05Z
      org.opencontainers.image.description:  A Helm chart for Kubernetes
      org.opencontainers.image.title:        application
      org.opencontainers.image.version:      4.0.0
    Path:                                    ocirepository/flux-system/python/sha256:81386b42d601e424ed5a3db1ca994655a70b5e8be5568a43a9c45aceeb88fc02.tar.gz
    Revision:                                4.0.0@sha256:81386b42d601e424ed5a3db1ca994655a70b5e8be5568a43a9c45aceeb88fc02
    Size:                                    1511
    URL:                                     http://source-controller.flux-system.svc.cluster.local./ocirepository/flux-system/python/sha256:81386b42d601e424ed5a3db1ca994655a70b5e8be5568a43a9c45aceeb88fc02.tar.gz
  Conditions:
    Last Transition Time:  2025-05-04T09:49:54Z
    Message:               stored artifact for digest '4.0.0@sha256:81386b42d601e424ed5a3db1ca994655a70b5e8be5568a43a9c45aceeb88fc02'
    Observed Generation:   1
    Reason:                Succeeded
    Status:                True
    Type:                  Ready
    Last Transition Time:  2025-05-04T09:49:54Z
    Message:               stored artifact for digest '4.0.0@sha256:81386b42d601e424ed5a3db1ca994655a70b5e8be5568a43a9c45aceeb88fc02'
    Observed Generation:   1
    Reason:                Succeeded
    Status:                True
    Type:                  ArtifactInStorage
  Observed Generation:     1
  URL:                     http://source-controller.flux-system.svc.cluster.local./ocirepository/flux-system/python/latest.tar.gz
Events:
  Type    Reason            Age   From               Message
  ----    ------            ----  ----               -------
  Normal  NewArtifact       59s   source-controller  stored artifact with revision '4.0.0@sha256:81386b42d601e424ed5a3db1ca994655a70b5e8be5568a43a9c45aceeb88fc02' from 'oci://registry-1.docker.io/eatherv/application'
  Normal  ArtifactUpToDate  1s    source-controller  artifact up-to-date with remote revision: '4.0.0@sha256:81386b42d601e424ed5a3db1ca994655a70b5e8be5568a43a9c45aceeb88fc02'
```

```bash
root@atharv:/home/ubuntu/dhanush# kubectl describe helmrelease -n flux-system
Name:         python
Namespace:    flux-system
Labels:       <none>
Annotations:  <none>
API Version:  helm.toolkit.fluxcd.io/v2
Kind:         HelmRelease
Metadata:
  Creation Timestamp:  2025-05-04T09:49:53Z
  Finalizers:
    finalizers.fluxcd.io
  Generation:        1
  Resource Version:  305047
  UID:               03957d60-baa3-4f2b-9d39-d484b9ab177e
Spec:
  Chart Ref:
    Kind:        OCIRepository
    Name:        python
  Interval:      1m
  Release Name:  python
Status:
  Conditions:
    Last Transition Time:  2025-05-04T09:49:56Z
    Message:               Helm install succeeded for release flux-system/python.v1 with chart application@4.0.0+81386b42d601
    Observed Generation:   1
    Reason:                InstallSucceeded
    Status:                True
    Type:                  Ready
    Last Transition Time:  2025-05-04T09:49:56Z
    Message:               Helm install succeeded for release flux-system/python.v1 with chart application@4.0.0+81386b42d601
    Observed Generation:   1
    Reason:                InstallSucceeded
    Status:                True
    Type:                  Released
  History:
    App Version:                   1.16.0
    Chart Name:                    application
    Chart Version:                 4.0.0+81386b42d601
    Config Digest:                 sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
    Digest:                        sha256:d6fd964baaaa9da1bfa2f12d927fe8d37843f21d4343f998d486f169dbd4ba59
    First Deployed:                2025-05-04T09:49:54Z
    Last Deployed:                 2025-05-04T09:49:54Z
    Name:                          python
    Namespace:                     flux-system
    Oci Digest:                    sha256:81386b42d601e424ed5a3db1ca994655a70b5e8be5568a43a9c45aceeb88fc02
    Status:                        deployed
    Version:                       1
  Last Attempted Config Digest:    sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
  Last Attempted Generation:       1
  Last Attempted Release Action:   install
  Last Attempted Revision:         4.0.0+81386b42d601
  Last Attempted Revision Digest:  sha256:81386b42d601e424ed5a3db1ca994655a70b5e8be5568a43a9c45aceeb88fc02
  Observed Generation:             1
  Storage Namespace:               flux-system
Events:
  Type    Reason            Age   From             Message
  ----    ------            ----  ----             -------
  Normal  InstallSucceeded  99s   helm-controller  Helm install succeeded for release flux-system/python.v1 with chart application@4.0.0+81386b42d601
```

```bash
root@atharv:/home/ubuntu/dhanush# kubectl get pods -n app
NAME                                          READY   STATUS    RESTARTS   AGE
python-staging-application-7b56bd8f56-5nrwk   1/1     Running   0          2m5s
python-staging-application-7b56bd8f56-fxp7c   1/1     Running   0          2m5s
python-staging-application-7b56bd8f56-xh684   1/1     Running   0          2m5s
```

### Pipeline flow - Stages

![Jenkins.drawio.svg](attachment:151d7847-3728-4846-8f11-b5e12a2d41df:Jenkins.drawio.svg)

## Push Helm Chart with New Version, It should Pull !!!!

# Thank You!
