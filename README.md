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
            └── values-$ARGOCD_CLUSTER_2_NAME.yaml```
