# EOCP Operator GitOps Deployment Template

## How to use this template

This template can be used to setup an automated Operator deployment and upgrade capability that leverages GitOps using ArgoCD. This template is built using Kustomize to allow for customization of the deployment as well as specific configurations for each environment catagory to be deployed to. This includes specific configuration for Staging, Development, and Production.

To use this template:

Clone to new repo
1) Create a new repository to host the new operator within GitLab
2) Copy the contents from this repo into your new repo as the starting point or on repo creation just to copy this existing repo
3) All following code changes should be made to your copy of this content in the new repo and not to the original template content

Gather Information
1) Identify the Operator you wish to configure deployment using this GitOps Template
2) Identify the Channel you wish the operator to use
	a) this can be done by installing the operator and reviewing the channel choice in the subscription object
3) Identify the install version, and desired update version if different from install version
	a) this can be done by installing the operator in the test environment and viewing the ClusterSubscriptionVersion (CSV) file associated with the deployment
4) Identify the target namespace for the operator. The template is built around using operatorname-operator as its basis so this is the easiest format to use. i.e. amqstreams-operator
5) Identify any cusomizations you need to apply to the operator deployment. This are not usually required, but sometimes are in special cases. i.e. FIPS configuration for AMQ-Streams

Update Template
1) Update the filename of all included yaml files to be named appropriately for the desired operator. i.e. operatorname-operator-subscription.yaml -> amqstreams-operator-subscription.yaml 

2) Find all instances of "operatorname-operator" within the included yaml files and replace with the appropriate name. The following is a list of each instance of this item
base/kustomization.yaml:  - operatorname-operator-subscription.yaml
base/kustomization.yaml:  - operatorname-operatorgroup.yaml
base/kustomization.yaml:  - operatorname-operator-install-approvaljob.yaml
base/kustomization.yaml:  - operatorname-operator-upgrade-approvaljob.yaml
base/kustomization.yaml:  - operatorname-operator-rbac.yaml
base/operatorname-operator-install-approvaljob.yaml:  namespace: operatorname-operator
base/operatorname-operator-install-approvaljob.yaml:              value: "operatorname-operator"
base/operatorname-operator-rbac.yaml:  namespace: operatorname-operator
base/operatorname-operator-rbac.yaml:  namespace: operatorname-operator
base/operatorname-operator-sa.yaml:  namespace: operatorname-operator
base/operatorname-operator-subscription.yaml:  namespace: operatorname-operator
base/operatorname-operator-upgrade-approvaljob.yaml:  namespace: operatorname-operator
base/operatorname-operator-upgrade-approvaljob.yaml:              value: "operatorname-operator"
base/operatorname-operatorgroup.yaml:  namespace: operatorname-operator
base/operatorname-operatorgroup.yaml:    - operatorname-operator
overlays/Development/kustomization.yaml:namespace: operatorname-operator
overlays/Production/kustomization.yaml:namespace: operatorname-operator
overlays/Staging/kustomization.yaml:namespace: operatorname-operator

3) Update the subscription.yaml
	a) Identify the appropriate channel from the "Gather Information" stage. Often is "stable" which is the default in the template
	b) Identify the version for the "Gather Information" stage and add it to the startingCSV section replacing "operator-install-version".

4) Update the version files
	a) in each overlay directory there is an operatorname-versions files. Update the filename appropriately: operatorname-versions -> amqstreams-versions
	b) After updating the file name, open each file and add the appropriate version information found in the "Gather Information" section
	c) If your initial install and upgrade version are the same, just enter the same version information in each field

5) Add and additional cusotmizations using Kustomize Patches
	a) Examples of kustomize patches can be found in a txt file in the root of this repository
	b) These can be added to the kustomize.yaml in each of the overlay sections, or the base kustomize.yaml
	c) These allow you to edit/change/configure sections of each yaml and handle that differently for each overlay

6) Update the README here with the templated default content below to match information related to the specifics of the operator you have configured with this template

7) Push code changes to GitLab and configure your ArgoCD Application to use the newly created and configured content

# EOCP Operator GitOps Deployment Capability

## Description

This code base provides a template, that when completed, can be ingested by ArgoCD/Red Hat Git Ops to deploy and manage the configured Operator into an OpenShift cluster using GitOps.

The code is structured using kustomize to template the included yaml and allow customized deployment to multiple environemnts including Staging, Development, and Production

This code can also be used to manage the version or upgarde of the operator installed and ensure consistancy across all clusters in a particular environment category.

## Installation

To utilize this project setup a new "Application" within your ArgoCD Instance that has access to the repository you cloned this template into. Target the appropriate overlay by providing the path. This should match the environment you are deploying the operator too. Use overlay/Stagining for Staging overlay/Development for Development and overlay/Production for production.

You can verify what versions will be targetted for initial installation and the final upgraded version by checking the operator.versions file within each of the overlay folders

Once the application is created within ArgoCD use the Sync actions to advance the configuration.

Sync 1 - Installs the Operator Subscription into the target Namespace and generates an InstallPlan to do the actual operator installation  
Sync 2- Approves the initial InstallPlan and allows the cluster to finish the initial installation of the Operator into the target namespace  
Sync 3- Approves any generated InstallPlans for upgrading the Operator to the version that is found the operator.version file and allows that upgrade to complete  
  
Your Operator is now installed, upgraded, and ready for use.  

## Usage

This code is leveraged in a GitOps fashion by ArgoCD so any changes to configuration or capability are done directly within the Git Repository. Here are some potential configuration changes and how to action them:
  
Changing Install Version: in the appropriate overlay (staging, development, production) update the operator.version file to have the new initial version you wish to install. Also, update the subscription objects desiredCSV field to have the new initial version.  

Changing the Upgrade Version: in the appropriate overlay (staging, development, production) update the operator.version file to have the new upgrade version you wish to install. Run the "sync" operation and any install plans available for that version will be approved and the upgrade will roll out.  


## Features

This code works by leveraging 2 OpenShift Jobs to approve InstallPlans for when Operators are set to "Manual" Approval mode. These Jobs leverage a serviceaccount within the cluster to check for install plans that match the versions in the operator.version file and appove them as needed. Once approved OpenShift handles migrating the Opeartor to the version approved in the InstallPlan  
