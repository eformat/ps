## OpenShift Binary Deployment using WAR and Pipeline

Deploy the apache petstore, clone this repo and:

```
oc new-project petstore --display-name="Petstore" --description='Petstore'
oc process -f ps-pipeline.yaml | oc create -f -
oc start-build pipeline
```

#### Pre-requisites:
- Openshift 3.4+
- EAP
```
oc import-image registry.access.redhat.com/jboss-eap-7/eap70-openshift --confirm -n openshift
```