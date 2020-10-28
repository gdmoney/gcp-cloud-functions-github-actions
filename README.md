# GCP Cloud Functions and GitHub Actions Integration

Integrate GitHub and GCP Cloud Functions to auto deploy an existing function on code changes.

### Usage
- GitHub > Settings > Secrets >  
  - New secret > Name: `GCP_PROJECT_ID`, Value: ... > Add secret  
  - New secret > Name: `GCP_SA_KEY`, Value: ... > Add secret
  
- GitHub > Actions > New workflow > set up a workflow yourself > ...
  - modify the parameters below and then copy & paste in the editor

- GCP Cloud Functions function will be updated every time the `main.py` Python code or the `update-cloud-functions.yml` file are modified
- deployments can also be automated by scheduling to run the workflow at specific times using `schedule` and `cron` commands

```
name: update-cloud-functions

on:
  push:
    branches:
    - master
    paths:
    - 'GCP/main.py'
    - '.github/workflows/update-cloud-functions.yml'
  pull_request:
    branches:
    - master

jobs:
  
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Check out repo
      uses: actions/checkout@v2

    - name: Setup gcloud
      uses: GoogleCloudPlatform/github-actions/setup-gcloud@master
      with:
        project_id: ${{ secrets.GCP_PROJECT_ID }}
        service_account_key: ${{ secrets.GCP_SA_KEY }}
        export_default_credentials: true
     
    - name: Run
      run: |
        gcloud functions deploy product-availability-checker \
          --region us-west2 \
          --source https://source.developers.google.com/projects/product-availability-checker/repos/github_gdmoney_product-availability-checker/moveable-aliases/master/paths/GCP
```
