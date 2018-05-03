# Istio Routing Mission for WildFly Swarm

## Purpose

Showcase Routing in Istio with a WildFly Swarm application

## Prerequisites

* Docker installed and running
* OpenShift and Istio environment up and running (See https://github.com/openshift-istio/istio-docs/blob/master/user-journey.adoc for details)

## Launcher Flow Setup

If the Booster is installed through the Launcher and the Continuous Delivery flow, no additional steps are necessary.

Skip to the _Use Cases_ section.

## Local Source to Image Build (S2I)

### Prepare the Namespace

Create a new namespace/project:
```
oc new-project <whatever valid project name you want>
```

### Build and Deploy the Application

#### With Source to Image build (S2I)

Run the following commands to apply and execute the OpenShift templates that will configure and deploy the applications:
```bash
find . | grep openshiftio | grep application | xargs -n 1 oc apply -f

oc new-app --template=wfswarm-istio-routing-client-service -p SOURCE_REPOSITORY_URL=https://github.com/wildfly-swarm-openshiftio-boosters/wfswarm-istio-routing -p SOURCE_REPOSITORY_REF=master -p SOURCE_REPOSITORY_DIR=routing-client
oc new-app --template=wfswarm-istio-routing-service-a-service -p SOURCE_REPOSITORY_URL=https://github.com/wildfly-swarm-openshiftio-boosters/wfswarm-istio-routing -p SOURCE_REPOSITORY_REF=master -p SOURCE_REPOSITORY_DIR=routing-service-a
oc new-app --template=wfswarm-istio-routing-service-b-service -p SOURCE_REPOSITORY_URL=https://github.com/wildfly-swarm-openshiftio-boosters/wfswarm-istio-routing -p SOURCE_REPOSITORY_REF=master -p SOURCE_REPOSITORY_DIR=routing-service-b
```

## Use Cases

### Default Service load balancing

1. Retrieve the URL for the Istio Ingress route, with the below command, and open it in a web browser.
    ```
    echo http://$(oc get route istio-ingress -o jsonpath='{.spec.host}{"\n"}' -n istio-system)/
    ```
2. The user will be presented with the web page of the Booster
3. Click the "Invoke" button. You should see a message in the result box indicating which service instance was called.
4. Click "Invoke" several more times.
Notice that there is an even 50% split between service a and b.

### Transfer load between services

1. Modify the load balancing such that all requests go to service a:
    ```
    oc apply -f rules/load-balancing-rule.yaml
    ```
2. Clicking on "Invoke" in the UI you will see that all requests are now being sent to service a.





# Deploy the Application

```
oc login -u developer -p pwd
mvn clean package -pl name-service-a fabric8:build -Popenshift
mvn clean package -pl name-service-b fabric8:build -Popenshift
mvn clean package -pl greeting-service fabric8:build -Popenshift
oc create -f ./config/application.yaml
```

# Use the Application

Get ingress route (further referred as ${INGRESS_ROUTE}):

```
oc login -u system:admin
oc get route -n istio-system
```

Copy and paste the HOST/PORT value that starts with `istio-ingress-istio-system` into your browser.

You will be presented with the UI.

Click on "Invok" several times and you will a mixture of the following responses:

* "Hello World (from Service A)"
* "Hello World (from Service B)"

By default Istio routes the traffic evenly between versions of the same services.

Setup the Istio Ingress rules to route 90% of requests to Service B.

```
oc login -u system:admin
oc create -f ./rules/route-rule-name-service-b-90.yaml -n myproject
```

Open ${INGRESS_ROUTE} in a browser again, and click "Invoke" several times.
You will see a mixture of responses as before, but most of them will be "Hello World (from Service B)" as we're now 90% for Service B.

We can then delete the rules:

```
oc delete -f ./rules/route-rule-name-service-b-90.yaml -n myproject
```

If we click "Invoke" again, we revert back to even distribution of executions across the versions.
