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
