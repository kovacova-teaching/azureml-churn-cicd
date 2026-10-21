# Churn prediction: Azure ML pipeline with GitHub Actions CI/CD

Trains a telecom churn classifier in Azure Machine Learning and deploys it to a managed online endpoint, with every step driven from GitHub Actions.

## How it works

```
GitHub Actions (workflow_dispatch)
 ├─ read-yaml              read config-infra-prod.yml (workspace, endpoint name)
 ├─ register-dataset       data/teleco_churn.csv → teleco-churn-data
 ├─ register-environment   conda envs for training and serving
 ├─ create-compute         cpu-cluster
 ├─ run-pipeline           Azure ML pipeline job
 │    ├─ prep_data         clean, label-encode, train/test split
 │    ├─ sweep_job         random hyperparameter sweep, Decision Tree, MLflow model
 │    └─ register          register the best model
 ├─ create-endpoint        managed online endpoint (create, or update if it exists)
 ├─ create-deployment      deploy the model with score.py
 └─ allocate-traffic       route 100% of traffic to the new deployment
```

## Layout

| Path | What's there |
|---|---|
| `.github/workflows/` | One reusable workflow per step, orchestrated by `pipeline-workflow.yml` |
| `data-science/src/` | `prep.py`, `train.py`, `register.py` |
| `mlops/azureml/train/` | Pipeline, training component and environment definitions |
| `mlops/azureml/deploy/online/` | Endpoint, deployment and `score.py` |
| `config-infra-prod.yml` | Workspace, resource group and endpoint names |

## Run it

1. Create a service principal and save its JSON output as the repo secret `AZURE_CREDENTIALS`:
   ```bash
   az ad sp create-for-rbac --name "ml-pipeline-sp" \
     --role contributor \
     --scopes /subscriptions/<YOUR_SUBSCRIPTION_ID> \
     --sdk-auth
   ```
2. Point `config-infra-prod.yml` at your own resource group, workspace and a unique endpoint name.
3. Run the **pipeline** workflow from the Actions tab.

## Stack

Azure Machine Learning (CLI v2), GitHub Actions, MLflow, scikit-learn, pandas.
