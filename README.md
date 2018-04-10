# Install Istio command

Ensure Docker is installed and running.

Download the binary for your operating system from https://github.com/openshift-istio/origin/releases/tag/istio-3.9-0.7.1-alpha1

# Start OpenShift/Istio

```
istiooc cluster up --istio=true --istio-auth=true
```

# Prepare the Namespace

**This mission assumes that `myproject` namespace is used.**

Create the namespace if one does not exist:
```
oc new-project myproject
```

Label the namespace for Istio Injection:
```
oc login -u system:admin
oc label namespace myproject istio-injection=enabled
```

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
