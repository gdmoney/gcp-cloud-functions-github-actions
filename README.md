# GCP Cloud Functions and GitHub Actions Integration

Integrates **GitHub Actions** and **GCP Cloud Functions** to automate updating and redeployment an **existing** Python based function on code changes.

This example assumes that the GitHub repo containing the function code and its dependencies, is synced with GCP **Cloud Source Repositories**. The `--source` parameter points to the GCP CSR directory containing the `main.py` code. If the repos were not synced, the source would be a local Zip file containing the `main.py` code.

### Usage
In the example below, the workflow will run and update the GCP Cloud Functions function every time `main.py`, `urls.py`, or `update-cloud-functions.yml` files are modified and changes are pushed to the `master` branch. It can also be triggerred manually with the `workflow_dispatch` event.

- GitHub > Settings > Secrets >  
  - New secret > Name: `GCP_PROJECT_ID`, Value: `...` > Add secret  
  - New secret > Name: `GCP_SA_KEY`, Value: `...` > Add secret
  
- GitHub > Actions > New workflow > set up a workflow yourself > *modify the parameters below and copy & paste in the editor*

### Parameters
- **`gcloud functions deploy`** - name of the existing function
- **`--region`** - region of the existing function 
- **`--source`** - Cloud Source Repository URL (Cloud Functions > click on the function > Details > Equivalent REST)

```
name: update-cloud-functions

on:
  push:
    branches:
    - master
    paths:
    - 'GCP/main.py'
    - 'GCP/urls.py'
    - '.github/workflows/update-cloud-functions.yml'
  workflow_dispatch:

jobs:
  
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Check out repo
      uses: actions/checkout@v2

    - name: Setup gcloud
      uses: google-github-actions/setup-gcloud@master
      with:
        project_id: ${{ secrets.GCP_PROJECT_ID }}
        service_account_key: ${{ secrets.GCP_SA_KEY }}
    
    - name: Setup Python 3.8
      uses: actions/setup-python@v2
      with:
        python-version: 3.8

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        
    - name: Run
      run: |
        gcloud functions deploy product-availability-checker \
          --region us-west2 \
          --source https://source.developers.google.com/projects/product-availability-checker/repos/github_gdmoney_product-availability-checker/moveable-aliases/master/paths/GCP
```
