# helm-appsets
POC that demonstrates how to structure a repo to deploy N charts with defaults/overrides, and values file for env and cluster

## dir structure
```
.
├── charts/
│   └── metrics-server/
│       ├── version-5.11.7/
│       │   ├── Chart.yaml
│       │   └── values.yaml
│       ├── version-6.2.1/
│       │   ├── Chart.yaml
│       │   └── values.yaml
│       ├── defaults.yaml
│       └── overrides.yaml
└── values/
    └── metrics-server/
        ├── env/
        │   └── values-$ENV.yaml
        └── cluster/
            ├── values-$ARGOCD_CLUSTER_1_NAME.yaml
            └── values-$ARGOCD_CLUSTER_2_NAME.yaml
```
### Charts Dir
Each chart being deployed as an application has a seperate subdir that contains a folder per version, defaults.yaml, and overrides.yaml.

Defaults instruct ArgoCD what settings to configure for every application being generated, and the overrides file is to set different values per environment.

### Values dir
Each chart being deployed as an application has a seperate subdir that contains all values files besides the default. The applicationSet yaml passes both env and cluster values file to helm as optional. This means each application doesn't require both, or either. It enables multiple levels of configuration. 