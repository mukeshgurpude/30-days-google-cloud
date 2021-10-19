# Deploy and Manage Cloud Environments with Google Cloud: Challenge Lab

> Do not wait for the lab to provision! You can complete tasks 1 and 2 before the lab provisioning has ended, just ensure the jumphost exists.

## Task 1: Create the production environment

```bash
gcloud compute instances list
gcloud compute ssh kraken-jumphost
```

```bash
cd /work/dm
gcloud deployment-manager deployments create --config prod-network.yaml
```

```bash
gcloud container clusters create kraken-prod \
--network=kraken-prod-vpc \
--subnetwork=kraken-prod-subnet \
--num-nodes 2 \
--zone us-east1-b
```

```bash
gcloud container clusters get-credentials kraken-prod \
--zone us-east1-b
```

```bash
cd /work/k8s
for f in $(ls *.yaml); do kubectl create -f $f; done
logout
```

## Task 2: Setup the Admin instance

- Navigate to `compute Engine > instances`, and create new instance `kraken-admin` with given network interfaces.
- Create an alerting policy in `Monitoring -> Alerting`

## Task 3: Verify the Spinnaker deployment

```bash
gcloud container clusters get-credentials spinnaker-tutorial
```

```bash
export DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" \
    -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &
```

```bash
gcloud source repos clone sample-app
cd sample-app
touch new.txt
git add .
git config user.name '<name>'
git config user.email '<email>'
git commit -m 'trigger deployment'
git tag v1.0.1
git push --tags
```

Navigate to `cloud-build/history`, and wait untill latest build finishes..
