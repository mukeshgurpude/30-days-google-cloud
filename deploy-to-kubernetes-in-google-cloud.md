# Deploy to Kubernetes in Google Cloud: Challenge Lab

## Setup

```bash
gcloud config set compute/region us-east1
gcloud config set compute/zone us-east1-b
```

## Task 1: Create a Docker image and store the Dockerfile

Build the image

```bash
source <(gsutil cat gs://cloud-training/gsp318/marking/setup_marking.sh)
gcloud source repos clone valkyrie-app

cat > Dockerfile << EOF
FROM golang:1.10
WORKDIR /go/src/app
COPY source .
RUN go install -v
ENTRYPOINT ["app","-single=true","-port=8080"]
EOF

docker build -t valkyrie-app:v0.0.1 .
```

## Task 2: Test the created Docker image

```bash
docker run -p 8080:8080 --name test-container valkyrie-app:v0.0.1 &
fg
```

press <kbd>ctrl</kbd> + <kbd>c</kbd> to kill the container

## Task 3: Push the Docker image in the Container Repository

Rename the image to required format (`gcr.io/<project-id>/<image-name>:v<tag>`), and push to `gcr`

```bash
docker build -t gcr.io/$DEVSHELL_PROJECT_ID/valkyrie-app:v0.0.1 .
docker push gcr.io/$DEVSHELL_PROJECT_ID/valkyrie-app:v0.0.1
```

## Task 4: Create and expose a deployment in Kubernetes

```bash
gcloud container clusters get-credentials valkyrie-dev --zone us-east1-d
cd k8s
```

Replace < image here> in `deployments.yaml` with `gcr.io/<project-id>/valkyrie-app:v0.0.1`

```bash
for file in $(ls *.yaml); do kubectl apply -f $file; done
```

## Task 5: Update the deployment with a new version of valkyrie-app

```bash
kubectl scale deployments valkyrie-dev --replicas=3
```

Add current changes and merge upstream changes

```bash
git commit -m 'add conf'
git config credential.helper gcloud.sh
git merge origin/kurt-dev
# Build and push image with new tag
docker build -t gcr.io/$DEVSHELL_PROJECT_ID/valkyrie-app:v0.0.2 .
docker push gcr.io/qwiklabs-gcp-03-66e5e0a42927/valkyrie-app:v0.0.2
```

Change `v0.0.1` to `v0.0.2` at two places, by editing the deployment

```bash
kubectl edit deployment valkyrie-dev
```

## Task 6: Create a pipeline in Jenkins to deploy your app

```bash
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```

```bash
cd ~/marking/valkyrie-app
sed -i s/YOUR_PROJECT/$DEVSHELL_PROJECT_ID/g Jenkinsfile
```

Follow `Connect to Jenkins` in <https://www.cloudskillsboost.google/focuses/1104?parent=catalog> to configure the builds on jenkins

```bash
sed -i s/green/orange/g source/html.go
git add .
git commit -m 'setup Jenkins'
git push origin master
```

Manually trigger the build in jenkins...
Lab will be complete once the build is triggered
