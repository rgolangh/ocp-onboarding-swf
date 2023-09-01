# OCP Onboarding workflow using Sonata Serverless Workflow

## Description

This is an OCP onboarding flow to create namespace by a Jira request.

![SWF VIZ](https://raw.githubusercontent.com/ederign/ocp-onboarding-swf/patch-1/src/main/resources/ocp-onboarding.svg)


## Generated clients involved:
- Jira with bearer token, see [specs/jira.yml](src/main/resources/specs/jira.yml)
- k8 with bearer token, [specs/kube.yaml](src/main/resources/specs/kube.yaml)

### Working with yor Jira and K8s instances:
- Jira:
  - go to your Jira profile -> Security -> Manage API tokens -> create
- K8s:
  - create a service account and set its rbac:
    ```sh
      kubectl create sa sonataflow-bot
      kubectl create clusterrole ns-creator --resource=namespace --verb=get,list,create
      kubectl create clusterrolebinding sonataflow-bot-ns-creator --clusterrole=ns-creator --user=sonataflow-bot
    ```
  - create a long-lived token using a secret for a service account:
    ```sh
      kubectl apply -f - <<EOF
      apiVersion: v1
      kind: Secret
      metadata:
      name: sonataflow-bot
      annotations:
      kubernetes.io/service-account.name: sonata-bot
      type: kubernetes.io/service-account-token
      EOF
    ```
  - get the token and set it later as the bearer token
    ```sh
      kubectl get secret sonataflow-bot -o jsonpath={".data.token"} | base64 -d
    ```

- Set the keys and values under an `.env` file in the root of the project (obviously not kept in this git repo)
```console
QUARKUS_REST_CLIENT_TRUST_STORE=/etc/pki/ca-trust/extracted/java/cacerts
QUARKUS_REST_CLIENT_TRUST_STORE_PASSWORD=changeit

QUARKUS_REST_CLIENT_JIRA_YML_URL=https://your-jira-url:443
QUARKUS_OPENAPI_GENERATOR_JIRA_YML_AUTH_BASICAUTH_USERNAME=your-jira-username
QUARKUS_OPENAPI_GENERATOR_JIRA_YML_AUTH_BASICAUTH_PASSWORD=your-jira-token

QUARKUS_REST_CLIENT_KUBE_YAML_URL=https://your-k8s-fqdn:port
QUARKUS_OPENAPI_GENERATOR_KUBE_YAML_AUTH_BEARERTOKEN_BEARER_TOKEN=your-k8s-token

```
  

- If you want to deploy using a CR use this [WIP] 
```
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app: ocp-onboarding
  name: ocp-onboarding-props
data:
  application.properties: |
    quarkus.rest-client.k8s.json.url={K8S Api server Url (run `kubectl config  view --minify -o jsonpath={.clusters[0].cluster.server}` }

```


## Installing and Running

### Prerequisites
 
You will need:
  - Java 11+ installed
  - Environment variable JAVA_HOME set accordingly
  - Maven 3.8.1+ installed

### Compile and Run in Local Dev Mode

```sh
mvn clean package quarkus:dev
```

### Execute the flow using curl

```sh
curl -XPOST -H "Content-Type: application/json" http://localhost:8080/ocpob -d '{"namespace": "my-new-namespace"}'
```

## Deploying with Kogito Operator

In the [`operator`](operator) directory you'll find the custom resources needed to deploy this example on OpenShift with the [Kogito Operator](https://docs.jboss.org/kogito/release/latest/html_single/#chap_kogito-deploying-on-openshift).
