# helm-appsets
---
POC that demonstrates how to structure a repo to deploy N charts with defaults/overrides, and values file for env and cluster

Until ArgoCD version 2.6 there is a tradeoff between using a remote Helm chart directly, and referencing a remote Helm chart as a dependency. 

## As a dependency
---  

### The Good
The benefit of this approach is deploying additional templates packaged with the remote Helm chart is trivial, and passing override values files is also lightweight.
### The Bad
As a dependency, Helm is able to retrieve the remote Helm chart just fine, but dependent charts can't have the chart version as a variable. This means a different Chart.yaml ( that references which version of the dependent chart is needed ) and since Helm MUST have a file named exactly Chart.yaml, multiple Chart.yaml have to be stored separately.
### The Informative
Initially I setup subfolders in charts/metrics-server for each environment and then created a Chart.yaml that set the chart version. I wasn't a fan because the approach seemed redundant so I settled on creating subfolders for each chart version. When an application is generated, chart version is interpolated, and the corresponding Chart.yaml is used. This enables us to easily add additional versions, or even additional templates to a specific version.

## As a remote Helm chart
---  

### The Good
When the remote helm chart is sourced by the application, version can be set directly. This means defaulting the underlying Helm chart is incredibly easy.
### The Bad
Until v2.6 of ArgoCD is released, an application can only have ONE source. This means override values files can not be sourced from a separate repository and used. You can work around this by either getting the values into the source as inline values, or by creating a custom Helm plugin that runs a command to retrieve the necessary values files and then using them to template the application.
### The Informative
v2.6 will change this entirely and we will be able to use a remote Helm Chart directly, while storing values file in our own repo. Once this is a thing, we should aim to use remote charts from Jfrog exclusively. And for charts with additional templates beyond the base chart, we should publish these to Jfrog as well and retrieve them from there. 

With that said, even after v2.6 I recommend using the dir pattern I proposed. All charts inside charts/ should be published to Jfrog and retrieved from there. All values files inside values/ should remain local, and be referenced in the application spec.

ArgoCD Github Issue - [Helm chart + values files from Git](https://github.com/argoproj/argo-cd/issues/2789)  
ArgoCD Github PR - [Multiple sources for applications](https://github.com/argoproj/argo-cd/pull/10432)

## dir structure
---
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
---
Each chart being deployed as an application has a seperate subdir that contains a folder per version, defaults.yaml, and overrides.yaml.

Defaults instruct ArgoCD what settings to configure for every application being generated, and the overrides file is to set different values per environment.

### Values dir
---
Each chart being deployed as an application has a seperate subdir that contains all values files besides the default. The applicationSet yaml passes both env and cluster values file to helm as optional. This means each application doesn't require both, or either. It enables multiple levels of configuration. 