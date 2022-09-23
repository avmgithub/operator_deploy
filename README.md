# Deploying Operators on OpenShift (air gapped)

## Summary of steps

1. Mirror the required images with skopeo
2. Deploy the catalogsource yaml file.
3. Deploy operator group.

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

### Deploy Operator group
Example operator group:

    apiVersion: operators.coreos.com/v1
    kind: OperatorGroup
    metadata:
      name: ibm-transadv-catalog-group
      namespace: REPLACE_NAMESPACE
    spec:
      targetNamespaces:
      - REPLACE_NAMESPACE

# References 

[Install Transformation Advisor via cli](https://www.ibm.com/docs/en/cta?topic=ocp-air-gap-install)

[Installing MQ operator via cli](https://www.ibm.com/docs/en/ibm-mq/9.2?topic=iumorho-installing-mq-operator-using-red-hat-openshift-cli)  <br>
[Installing MQ operator air gap install](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=openshift-installing-mq-operator-in-airgap-environment#ctr_installing_airgap__invdload)

[Install WAS Liberty operator via cli](https://www.ibm.com/docs/en/was-liberty/core?topic=operators-installing-openshift-cli)

[Create ImageContentSourcePolicy](https://www.ibm.com/docs/en/scalecontainernative?topic=appendix-airgap-setup-network-restricted-red-hat-openshift-container-platform-clusters)

[Managing Custom Catalogs](https://docs.openshift.com/container-platform/4.9/operators/admin/olm-managing-custom-catalogs.html)
