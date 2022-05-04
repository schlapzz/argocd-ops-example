# Argocd Example Ops

## Getting started
Dies ist das Ops Repository für das [ArgoCD Example Application Project](https://gitlab.puzzle.ch/pitc-cicd/argocd-example-dev)

## Set Up ArgoCD
Da die beiden für das Deployment verwendeten Projekte private sind, muss vorgängig ArgoCD konfiguriert werden damit die auf Helm Charts und das Ops Git-Repository zugegriffen werden kann.

### Helm Repository verbinden

### ArgoCD Example Application Ops Repository verbinden 
//TODO


## Deplyoment Structure
Es existieren drei verschiedene Umgebungen in welche via ArgoCD deployt wird.

| Umgebung            | Branch | Helm Values File |
|---------------------|--------|------------------|
| Dev                 | dev    | values-dev.yaml  |
| Integration/Staging | int    | values-int.yaml  |
| Production          | prod   | values-prod.yaml |

Für jede Umgebung wird in ArgoCD eine Applikation erstellt welche auf dieses Repository und den 
entsprechenden Branch und Values File verweist.