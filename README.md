# Deploying Operators on OpenShift (air gapped)

## Summary of steps

1. Mirror the required images with skopeo
2. Deploy the catalogsource yaml file.
3. Deploy operator group.
4. Deploy operator via Operator hub gui

### Mirror the required images with skopeo
Example command:
        
    # skopeo copy --all docker://(source registry/namespace/imagename@sha256:xxxxxxxxx) docker://(destination registry/destination namespace/imagename@shar256:xxxxxxx)

### Deploy the catalogue source yaml file
Example catalog source with secret:

    apiVersion: operators.coreos.com/v1alpha1
    kind: CatalogSource
    metadata:
        name: my-operator-catalog
        namespace: openshift-marketplace
    spec:
    sourceType: grpc
    secrets: 
    - "<secret_name_1>"
    - "<secret_name_2>"
    image: <registry>:<port>/<namespace>/<image>:<tag>
    displayName: My Operator Catalog
    publisher: <publisher_name>
    updateStrategy:
      registryPoll:
        interval: 30m

Check the running pods

    $ oc get po -n openshift-marketplace

Check the catalogsource

    $ oc get catalogsource -n openshift-marketplace

Check the package manifest

    $ oc get packagemanifest -n openshift-marketplace


### Configure image content source policy
For reference [see](https://www.ibm.com/docs/en/scalecontainernative?topic=appendix-airgap-setup-network-restricted-red-hat-openshift-container-platform-clusters)

Example image content source policy:
```
apiVersion: operator.openshift.io/v1alpha1
kind: ImageContentSourcePolicy
metadata:
  name: ibm-mq
spec:
  repositoryDigestMirrors:
  - mirrors:
    - icr.io/bofa/cp
    source: cp.icr.io/cp
  - mirrors:
    - icr.io/bofa/ibmcom
    source: docker.io/ibmcom
  - mirrors:
    - icr.io/bofa/cpopen
    source: icr.io/cpopen
  - mirrors:
    - icr.io/bofa/ibm-messaging
    source: icr.io/ibm-messaging
  - mirrors:
    - icr.io/bofa/opencloudio
    source: quay.io/opencloudio
```

### Deploy Operator group
Switch to the namespace where you want to install the operator (openshift-operators if you are installing in all namespaces, or a custom name if you are installing in a specific namespace):

    $ oc project <namespace>

Example operator group:

    apiVersion: operators.coreos.com/v1
    kind: OperatorGroup
    metadata:
      name: ibm-transadv-catalog-group
      namespace: REPLACE_NAMESPACE
    spec:
      targetNamespaces:
      - REPLACE_NAMESPACE

Apply the operator group
```
$ oc apply -f operator-group.yaml
```

Validate deployment of operator

    $ oc get csv -n <namespace>

Note: As this is an airgapped system, I am not certain if the subscription yaml file is needed to get the operator up and running.
# References 

[Install Transformation Advisor via cli](https://www.ibm.com/docs/en/cta?topic=ocp-air-gap-install)

[Installing MQ operator via cli](https://www.ibm.com/docs/en/ibm-mq/9.2?topic=iumorho-installing-mq-operator-using-red-hat-openshift-cli)  <br>
[Installing MQ operator air gap install](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=openshift-installing-mq-operator-in-airgap-environment#ctr_installing_airgap__invdload)

[Install WAS Liberty operator via cli](https://www.ibm.com/docs/en/was-liberty/core?topic=operators-installing-openshift-cli)

[Create ImageContentSourcePolicy](https://www.ibm.com/docs/en/scalecontainernative?topic=appendix-airgap-setup-network-restricted-red-hat-openshift-container-platform-clusters)

[Managing Custom Catalogs](https://docs.openshift.com/container-platform/4.9/operators/admin/olm-managing-custom-catalogs.html)

[Internal IBM Only reference](https://ibmdocs-test.mybluemix.net/docs/en/SSGT7J_22.2_test?topic=operators-installing-using-cli)

[Airgap setup for network restricted Red Hat OpenShift Container Platform clusters](https://www.ibm.com/docs/en/scalecontainernative?topic=appendix-airgap-setup-network-restricted-red-hat-openshift-container-platform-clusters)