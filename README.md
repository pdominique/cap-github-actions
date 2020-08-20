# DevOps: CAP CI/CD with GitHub Actions

## Initial Setup

### Create a new cap project
```bash
cds init cap-github-actions --add samples
cds watch
```

### Enhance Project Configuration for SAP HANA Cloud
Add the following in the package.json file:
```json
{
  "cds": {
    "requires": {
      "db": {
        "kind": "sql"
      }
    },
    "hana": {
      "deploy-format": "hdbtable"
    }
  }
}
```

Add the hdb driver for SAP HANA as a dependency to the project:
```bash
npm add hdb --save
```

## Build and Deploy from your local machine

### Add the MTA development descriptor file (mta.yaml)
```bash
cds add mta
```

### Build the MTA
```bash
mbt build -p cf
```

### Deploy the MTA to Cloud Foundry
```bash
cf login
cf deploy mta_archives/cap-github-actions_1.0.0.mtar
```

## Build and Deploy Using GitHub Actions

### Encrypted Secrets
As we don't want to expose the details of our Cloud Foundry account (organization, space and so on) we need first to create some encrypted secrets in GitHub. This can be done in our repository Settings on GitHub.

### Create GitHub Workflow
Now we need to create a GitHub Workflow to build and deploy our project to Cloud Foundry when an event occurs (i.e push to master branch for instance). We don't have to build everything from scratch and we can just reuse an existing action to deploy to CF: https://github.com/guerric-p/cf-cli-action

To create a new Workflow, just click on the Actions tab and then on the setup a workflow yourself link. This will open an editor where we can define our own workflow. Replace the content of the editor with the following code:
```yaml
# Build Multitarget Application & Deploy it to Cloud Foundry
name: Build MTA & Deploy it to CF

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js
      uses: actions/setup-node@v1
      with:
        node-version: 12
    - name: Install MTA Build Tool
      run: npm install -g mbt
    - name: Build MTA
      run: mbt build -p cf
    - name: Upload Artifact
      uses: actions/upload-artifact@master	
      with:	
        name: mta
        path: ./mta_archives/cap-github-actions_1.0.0.mtar
        
  deploy:
     needs: build
     runs-on: ubuntu-latest
     steps:
       - name: Download Artifact
         uses: actions/download-artifact@master
         with:
           name: mta
           path: ./
       - name: Deploy to Cloud Foundry
         uses: guerric-p/cf-cli-action@master
         with:
           cf_api: ${{ secrets.CF_API }}
           cf_username: ${{ secrets.CF_USERNAME }}
           cf_password: ${{ secrets.CF_PASSWORD }}
           cf_org: ${{ secrets.CF_ORG }}
           cf_space: ${{ secrets.CF_SPACE }}
           command: deploy ./cap-github-actions_1.0.0.mtar -f
```

### Test the Workflow
Now we need just need to push to the master branch to trigger the new workflow and to automatically build & deploy our application. :-) 