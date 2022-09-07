# Argocd Example Ops

## Getting started
Dies ist das Ops Repository für das [ArgoCD Example Application Project](https://gitlab.puzzle.ch/pitc-cicd/argocd-example-dev)

## Set Up ArgoCD
Da die beiden für das Deployment verwendeten Projekte private sind, muss vorgängig ArgoCD konfiguriert werden damit die auf Helm Charts und das Ops Git-Repository zugegriffen werden kann.

### Helm Repository verbinden

Damit auf die Private Package Regisrty des Projekts zugegriffen werden kann,
muss zuerst im GitLab ein Deploy Token erstellt werden.

Unter `Settings` → `Repository` → `Deploy tokens` ein neues Deploy Token erfassen.

Dazu Name und Username ausfüllen, und als Scope **read_repository** und **read_package_registry** auswählen und anschliessend das Token erzeugen.
Das Token ist nur bei der erstellung Sichtbar und kann zu einem späteren Zeitpunkt nicht mehr angezeigt werden.

Zum schluss das Helm Repository mit ArgoCD verbinden
```
argocd repo add https://gitlab.puzzle.ch/api/v4/projects/2643/packages/helm/stable --name argocd-exmaple-helm \
--type helm  --username <tokne_name> --password <token> --ssh-private-key-path deploy_key 
```

### ArgoCD Example Application Ops Repository verbinden 

Falls in GitLab noch keine Deploy Keys existieren müssen diese zuerst erstellt werden.
```
ssh-keygen -f deploy_key -C argocd
```

Anschliessend den Public Key `deploy_key.pub` in GitLab als Deploy Key hinterlegen.
Dazu im Projekt unter `Settings` → `Repository` → `Deploy Keys` den Key hinzufügen.  

Danach kann mit der ArgoCD CLI das Git Repository hinzugefügt werden

```
argocd repo add https://gitlab.puzzle.ch/pitc-cicd/argocd-example-ops.git  --ssh-private-key-path deploy_key --insecure-ignore-host-key
```

## Deplyoment Structure
Es existieren drei verschiedene Umgebungen in welche via ArgoCD deployt wird.

| Umgebung            | Branch | Helm Values File |
|---------------------|--------|------------------|
| Dev                 | dev    | values-dev.yaml  |
| Integration/Staging | int    | values-int.yaml  |
| Production          | prod   | values-prod.yaml |

Für jede Umgebung wird in ArgoCD eine Applikation erstellt welche auf dieses Repository und den 
entsprechenden Branch und Values File verweist.

## ApplicationSets
Für ein Multi Environment Deployment kann auch folgendes ApplicationSet verwendet werden. Dazu muss der ApplicationSet Controller installiert sein.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: exapp
spec:
  generators:
    - list:
        elements:
          - customer: a
            stage: prod
            cluster: ocp4-cloudscale-production
          - customer: b
            stage: prod
            cluster: ocp4-cloudscale-production
          - customer: a
            stage: staging
            cluster:  ocp4-cloudscale-staging
          - customer: b
            stage: staging
            cluster:  ocp4-cloudscale-staging
  template:
    metadata:
      name: exapp-{{stage}}-{{customer}}
    spec:
      source:
        repoURL: https://github.com/schlapzz/argocd-ops-example.git
        targetRevision: propagation
        path: subchart
        helm:
          valueFiles:
          - common/values.yaml
          - variants/customer-{{customer}}/values.yaml
          - variants/{{stage}}/values.yaml
          - envs/{{stage}}-{{customer}}/values.yaml
          - envs/{{stage}}-{{customer}}/version.yaml
      project: pitc-apps
      destination:
        name: "{{cluster}}"
        namespace: pitc-cicd-argocd-example-{{stage}}-{{customer}}
      syncPolicy:
        automated: # automated sync by default retries failed attempts 5 times with following delays between attempts ( 5s, 10s, 20s, 40s, 80s ); retry controlled using `retry` field.
          prune: true # Specifies if resources should be pruned during auto-syncing ( false by default ).
          selfHeal: true # Specifies if partial app sync should be executed when resources are changed only in target Kubernetes cluster and no git change detected ( false by default ).
          allowEmpty: false # Allows deleting all application resources during automatic syncing ( false by default ).
        syncOptions:     # Sync options which modifies sync behavior
        - CreateNamespace=true
```
