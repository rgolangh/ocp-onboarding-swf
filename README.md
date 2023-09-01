# OCP Onboarding workflow using Sonata Serverless Workflow

## Description

This is an OCP onboarding flow to create namespace by a Jira request.

![SWF VIZ](https://raw.githubusercontent.com/ederign/ocp-onboarding-swf/patch-1/src/main/resources/ocp-onboarding.svg)


## Generated clients involved:
- Jira with bearer token, see `src/main/resources/specs/jira.yml`
- k8 with bearer token, `src/main/resources/specs/k8s.yaml` 

### Working with yor Jira and K8s instances:
- Get your Jira api key from ? Go to your Jira profile -> Security -> Manage API tokens -> create
- Set the key under a `.env` file in the root of the project (obviously not kept in this git repo)
```
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

When using native image compilation, you will also need: 
  - [GraalVm](https://www.graalvm.org/downloads/) 20.2.0+ installed
  - Environment variable GRAALVM_HOME set accordingly
  - Note that GraalVM native image compilation typically requires other packages (glibc-devel, zlib-devel and gcc) to be installed too.  You also need 'native-image' installed in GraalVM (using 'gu install native-image'). Please refer to [GraalVM installation documentation](https://www.graalvm.org/docs/reference-manual/aot-compilation/#prerequisites) for more details.

### Compile and Run in Local Dev Mode

```sh
mvn clean package quarkus:dev
```

### Compile and Run in JVM mode

```sh
mvn clean package 
java -jar target/quarkus-app/quarkus-run.jar
```


### Compile and Run using Local Native Image
Note that this requires GRAALVM_HOME to point to a valid GraalVM installation

```sh
mvn clean package -Pnative
```
  
To run the generated native executable, generated in `target/`, execute

```sh
./target/sw-quarkus-greeting-{version}-runner
```

### Execute the flow using curl

```sh
curl -XPOST -H "Content-Type: application/json" http://localhost:8080/ocpob -d '{"namespace": "my-new-namespace"}'
```


## Deploying with Kogito Operator

In the [`operator`](operator) directory you'll find the custom resources needed to deploy this example on OpenShift with the [Kogito Operator](https://docs.jboss.org/kogito/release/latest/html_single/#chap_kogito-deploying-on-openshift).
