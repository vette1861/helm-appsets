# helm-appsets
POC that demonstrates how to structure a repo to deploy N charts with defaults/overrides, and values file for env and cluster

Until ArgoCD version 2.6 there is a tradeoff between using a remote Helm chart directly, and referencing a remote Helm chart as a dependency. 

### As a dependency
#### The Good
The benefit of this approach is deploying additional templates packaged with the remote Helm chart is trivial, and passing override values files is also lightweight.
#### The Bad
As a dependency, Helm is able to retrieve the remote Helm chart just fine, but dependent charts can't have the chart version as a variable. This means a different Chart.yaml ( that references which version of the dependent chart is needed ) and since Helm MUST have a file named exactly Chart.yaml, multiple Chart.yaml have to be stored separately.

### As a remote Helm chart
#### The Good
When the remote helm chart is sourced by the application, version can be set directly. This means defaulting the underlying Helm chart is incredibly easy.
#### The Bad
Until v2.6 of ArgoCD is released, an application can only have ONE source. This means override values files can not be sourced from a separate repository and used. You can work around this by either getting the values into the source as inline values, or by creating a custom Helm plugin that runs a command to retrieve the necessary values files and then using them to template the application.

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