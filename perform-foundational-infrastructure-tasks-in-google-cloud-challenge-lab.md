# Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab

URL: <https://www.cloudskillsboost.google/focuses/10379>

Quest: <https://www.cloudskillsboost.google/quests/120>

## Challenge

You are just starting your junior cloud engineer role with Jooli inc. So far you have been helping teams create and manage Google Cloud resources.

You are now asked to help a newly formed development team with some of their initial work on a new project around storing and organizing photographs, called memories. You have been asked to assist the memories team with initial configuration for their application development environment; you receive the following request to complete the following tasks:

- Create a bucket for storing the photographs.
- Create a Pub/Sub topic that will be used by a Cloud Function you create.
- Create a Cloud Function.
- Remove the previous cloud engineer’s access from the memories project.

## Quick setup

Joolie Inc. requires you to create all the resourcess in `us-east1` region and `us-east1-b` zone, unless otherwise directed. Setup default zone and region.

```bash
gcloud config set compute/region us-east1
gcloud config set compute/zone us-east1-b
```

## Task 1

Create a bucket. `DEVSHELL_PROJECT_ID` variable contains current active project ID, so we can just use that instead of copy pasting project id each time.

```bash
gsutil mb -p $DEVSHELL_PROJECT_ID gs://$DEVSHELL_PROJECT_ID
```

## Task 2

Create a topic named `memories-topic`

```bash
gcloud pubsub topics create memories-topic
```

## Topic 3

- First we need to create a function. Create a new directory and change to it.

  ```bash
  mkdir -p thumbnail_function
  cd thumbnail_function
  ```

- Create files `index.js`, and `package.json`, and copy paste contents given in the lab. For this you can use `nano`, `vim`, `emacs` or can use embedded editor in google cloud by clicking on `Open Editor` button in terminal header
- In line 15 of `index.js` replace `REPLACE_WITH_YOUR_TOPIC ID` with `memories-topic`(our pubsub topic name)

- Deploy the function, using given parameters

  ```bash
  gcloud functions deploy thumbnail \
  --stage-bucket gs://$DEVSHELL_PROJECT_ID \
  --trigger-bucket gs://$DEVSHELL_PROJECT_ID \
  --runtime nodejs14
  ```

- To pass the assesement, you need to trigger the function by uploading an image to the cloud storage bucket

  ```bash
  wget https://storage.googleapis.com/cloud-training/gsp315/map.jpg
  gsutil cp map.jpg gs://$DEVSHELL_PROJECT_ID/
  ```

## Task 4

- Navigate to `Navigation Menu` -> `IAM`. Find the username which is given in connections panel as `username 2`.
- This user has `viewer` role assigned. Click edit and remove this viewer role, and save the changes.
