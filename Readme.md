# Simple Model Serving with Cloud Run

This post talks about how to get started with deploying models on Google Cloud Run, along with the pros and cons of using this system for inference.

## Pros
- No infrastructure (in the case of fully managed)
- Dashboards / logs / revision history (in the case of fully managed or on customer’s Anthos cluster)
- Cost that scales linearly with usage
## Cons
- Limits (that cannot be changed) and Quotas (which can be requested to change) to put constraints on what kind of models that can be run
- Cold starts, because serverless
- Cold starts occur, even with steady traffic

## Set default project for further commands:

```$ gcloud config set project PROJECT_ID```

Set the run/region:

```$ gcloud config set run/region us-west1```

If you have a POSIX terminal (linux / mac / windows wsl)  you can save off the project ID for later use:

```$ export PROJECT_ID=$(gcloud config get-value project)```


## Build and push Docker image

First clone the repo from git

``` $ git clone repo_name```

There should be three files

``` $ ls ```
###### app.py bootstrap.py Dockerfile

``` $ gcloud builds submit --tag=us.gcr.io/$PROJECT_ID/distilbert:latest ```


## Deploy the CloudRun service

``` 
$ gcloud run deploy distilbert \
--image=gcr.io/$PROJECT_ID/distilbert:latest \
--platform=managed \
--concurrency=1 \
--set-env-vars=MKL_NUM_THREADS=2,OMP_NUM_THREADS=2,NUMEXPR_NUM_THREADS=2 \
--cpu=2 \
--memory=3G \
--allow-unauthenticated 
```

## Test the service

Get the CloudRun service url 


```
gcloud run services list --platform managed | awk 'NR==2 {print $4}'
```

Let's test the service

```
curl -d '{"context":"Roses are red, Violets are blue, Sugar is sweet, And so are you.", "question":"What am I?"}' -H "Content-Type: application/json" -X POST https://distilbert-moahrschhq-uw.a.run.app/predict
```

