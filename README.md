Table of Contents
=================

* [Table of Contents](#table-of-contents)
* [Automate certification process by utilizing DCI to interact with the catalog backend](#automate-certification-process-by-utilizing-dci-to-interact-with-the-catalog-backend)
   * [Purpose](#purpose)
   * [Benefits](#benefits)
   * [Prerequisites](#prerequisites)
   * [Manual Certification Container Process](#manual-certification-container-process)
   * [Automation Container Certification Flow](#automation-container-certification-flow)
      * [Auto Publish Preparations](#auto-publish-preparations)
      * [Auto Publish Settings Configuration](#auto-publish-settings-configuration)
   * [Recertify Certification Container Projects](#recertify-certification-container-projects)
      * [Recertify Settings Configuration](#recertify-settings-configuration)
   * [Automate creation of the Openshift-cnf project for Vendor Validated](#automate-creation-of-the-openshift-cnf-project-for-vendor-validated)
      * [Global Variables](#global-variables)
      * [Variables to define for each cnf_to_certify](#variables-to-define-for-each-cnf_to_certify)
      * [Variables to define for project settings under cert_listings main variable](#variables-to-define-for-project-settings-under-cert_listings-main-variable)
      * [Example Configuration of Openshift-cnf certification project creation](#example-configuration-of-openshift-cnf-certification-project-creation)
   * [How to Run DCI Auto-publish, Recertify and Openshift-cnf Vendor validated](#how-to-run-dci-auto-publish-recertify-and-openshift-cnf-vendor-validated)
   * [Known Issues](#known-issues)
# Automate certification process by utilizing DCI to interact with the catalog backend
This repository showcases the process of automating the certification of container projects and their subsequent automatic publication. This encompasses the reevaluation of container images for recertification and the automatic creation of an `Openshift-cnf` certification project for `Vendor Validate`.

## Purpose
The purpose of this repository is to showcase the enhancement of the container certification process through the utilization of the new features provided by DCI. The process entails the creation of container projects, the modification of mandatory parameters, the testing and scanning of container images, the inclusion of a product listing, and the automation of container project publication once all requirements are met.

In the second part, we demonstrate the procedure for recertifying existing container projects in the catalog when a new software release becomes available. This involves updating the container's version, including its digest and tag, and republishing it in the catalog.

Lastly, we introduce a new process where the creation of the Openshift-cnf certification project is integrated into the workflow. Previously, the Vendor Validated partner CNF required a separate step, but now, once the container and helm chart projects are certified, the Openshift-cnf project will be automatically generated. This addition enhances the automation capabilities of DCI. 

## Benefits
Automation with DCI can save time, money, and errors in the CNF certification process. It can also improve efficiency, quality control, and scalability.

**Time savings:** Automation can streamline the certification process by eliminating manual and repetitive tasks. This can save time and resources, and allow RedHat and partners to quickly respond to new software releases.  

**Cost-effectiveness:** Automation can help reduce costs associated with manual labor and resources. This is because fewer personnel are required to manage the certification process.  

**Error reduction:** Automation can help minimize errors by executing predefined and standardized procedures consistently and accurately. This ensures that all necessary steps are followed correctly, improving the reliability and accuracy of the certification process.    

**Improved efficiency:** Automation can enable faster and more efficient execution of tasks. This can help RedHat and partners quickly respond to new software releases and keep their catalog up to date.  

**Enhanced quality control:** Automation can help enforce consistent and standardized practices throughout the certification process. This can lead to higher-quality certifications and better overall quality control.  

**Scalability:** Automation with DCI allows for easy scalability, as the system can handle a larger volume of certifications without sacrificing efficiency or quality.

## Prerequisites
- Minimum DCI Openshift APP Agent version (Recommended to use latest DCI software version)
```shellSession
$ rpm -qa|grep dci-openshift-agent-0.5.7
dci-openshift-agent-0.5.7+
```
- Upgrade or re-install the latest DCI Repo
Follow this link if upgrade/remove/install NOT workking [install-dci-packages](https://blog.distributed-ci.io/install-openshift-on-baremetal-using-dci.html#dci-packages)
```shellSession
$ sudo dnf upgrade --refresh --repo dci -y
 ```
- if upgrade gets issue then try following:  
```shellSession
$ sudo dnf remove dci-openshift-app-agent -y
$ sudo dnf install dci-openshift-app-agent -y
```

## Manual Certification Container Process
![Manual Process](img/manual-process.png)

## Automation Container Certification Flow
![Automation Container Cert Workflow](img/automation-container-certification-flow.png)

### Auto Publish Preparations

- OCP and DCI ENV (included dcirc.sh,install.yml etc..)  
Any OCP Cluster and a helper node with DCI RPM packages installed
 
- **Settings.yml**  
A Settings with DCI auto-publish enhancement feature
 
- **Auth.json**    
Docker Authentication to access container repo on private registry server  
Login to your private registry server:
```shellSession
$ podman login -u testuser quay.io
$ echo $XDG_RUNTIME_DIR
/run/user/4205315/containers/auth.json
```
  
- **Pyxis-apikey.txt**    
A token to access specific partner Pyxis catalog data using REST API. [Create Pyxis API Key](https://connect.redhat.com/account/api-keys)
 
- **Kubeconfig**    
A kubeconfig that can access the OCP cluster
 
- **Product-listing ID**    
Before a container or helm chart/operator can be publicly listed into RedHat catalog, a Product-Listing must be created, it only need to create once according to CNF type.
Follow this link to [Create-Product-Listing](https://connect.redhat.com/manage/products)
 
- **Organization ID**    
Mandatory when using create_container_project. Company ID will be used for the verification of container certification project Organization-ID Company-Profile.
![Get Redhat OrgID](img/redhat-org-id.png)

### Auto Publish Settings Configuration
```yaml
---
dci_topic: OCP-4.11
dci_name: Testing DCI to create certification Project Automatic and Update mandatory settings and publish
dci_configuration: Using DCI create project,update,submit result and auto-publish
preflight_test_certified_image: true
check_for_existing_projects: true
ignore_project_creation_errors: true
dci_config_dirs: [/etc/dci-openshift-agent]
partner_creds: "/var/lib/dci-openshift-app-agent/auth.json"
organization_id: 12345678
preflight_containers_to_certify:
  - container_image: "quay.io/avu0/auto-publish-ubi8-nginx-demo1:v120"
    create_container_project: true
    short_description: "I am doing a full-automation e2e auto-publish for following image auto-publish-ubi8-nginx-demo1:v120"
    attach_product_listing: true

  - container_image: "quay.io/avu0/auto-publish-ubi8-nginx-demo2:v120"
    create_container_project: true
    short_description: "I am doing a full-automation e2e auto-publish for following image auto-publish-ubi8-nginx-demo2:v120"
    attach_product_listing: true

cert_settings:
   auto_publish: true
   build_categories: "Standalone image"
   registry_override_instruct: "<p>This is an instruction how to get the image link.</p>"
   email_address: "whoami@redhat.com"
   application_categories: "Networking"
   os_content_type: "Red Hat Universal Base Image (UBI)"
   privileged: false
   release_category: "Generally Available"
   repository_description: "This is a test for Demo how to automate to create project, SCAN and update settings"

cert_listings:
  published: true
  type: "container stack"
  pyxis_product_list_identifier: "yyyyyyyyyyyyyyyyy"  # product list id for container projects

pyxis_apikey_path: "/var/lib/dci-openshift-app-agent/pyxis-apikey.txt"
dci_gits_to_components: []
...
```
## Recertify Certification Container Projects
In the auto-publish process, we have seen the effectiveness of using DCI to communicate with the catalog backend, which enables full end-to-end automation for container certification.

In the next recertification process, we can use a similar approach to recertify a new release of a certified container under an existing project. DCI is also able to recertify certification projects that are already active and published with a new release version and tag.

- Identify the certification projects that need to be recertified  
- Update the certification project's release version and tag to the new release version and tag  
- Run DCI to embed Preflight to retest the container of new release and submit new report to current container project in partner portal  
- Confirm and verify the project information under the project, then publish it to the catalog for completion recertification.  
  
Please note that these are just general steps and may vary depending on the specific project.

### Recertify Settings Configuration
```yaml
---
dci_topic: OCP-4.11
dci_name: Testing DCI to Recertify the certification container projects
dci_configuration: Using DCI to Recertify the Certification container Project
preflight_test_certified_image: true
check_for_existing_projects: true
ignore_project_creation_errors: true
organization_id: 12345678
dci_config_dirs: [/etc/dci-openshift-agent]
partner_creds: "/var/lib/dci-openshift-app-agent/auth.json"
preflight_containers_to_certify:
  - container_image: "quay.io/avu0/auto-publish-ubi8-nginx-demo1:v121"
    create_container_project: true
 
  - container_image: "quay.io/avu0/auto-publish-ubi8-nginx-demo2:v121"
    create_container_project: true

pyxis_apikey_path: "/var/lib/dci-openshift-app-agent/pyxis-apikey.txt"
dci_gits_to_components: []
...
```
As you may have noticed, the `create_container_project` parameter is now set to `true` even when recertifying container projects. This is a new change that was recently made. The backend will now reject the creation of a container project if it is already in the active state, but the DCI will ignore this and continue with the recertification process.

## Automate creation of the Openshift-cnf project for Vendor Validated
This feature automatically generates an OpenShift-CNF certification project when this option `create_cnf_project` set to `true`. The new feature reuses some tasks from the `create-certification-project` role, and the associated templates are stored within this feature. Currently, there are no mandatory parameters that need to be updated in the new feature.

Please note that the feature openshift-cnf is currently a basic automation setup. It will undergo further updates once additional options are available from the backend REST API, such as `{"certification_level":"Certified"}` parameter for starting the Certify the functionality of your CNF on Red Hat OpenShift step within the project.


### Global Variables
As the new role openshift-cnf reuses some existing tasks, please refer to the description in the `create-certification-project` role for information on the shared global variables.


### Variables to define for each cnf_to_certify

Name                     | Default                                                                    | Description
-------------------      | ------------                                                               | -------------
attach_product_listing   | false                                                                      | If set to true, it would attach product-listing to Openshift-cnf certification project.
create_cnf_project       | false                                                                      | If set to true, it would create a new Openshift-cnf certification project.
cnf_name                 | None                                                                       | If defined, it would create Openshift-cnf certification project for vendor validated, cnf_name format: `CNF25.8 + OCP4.12`

### Variables to define for project settings under `cert_listings` main variable

Name                          | Default                              | Description
----------------------------- | ------------------------------------ | -------------
pyxis_product_list_identifier | None                                 | Product-listing ID, it has to be created before. [See doc](https://redhat-connect.gitbook.io/red-hat-partner-connect-general-guide/managing-your-account/product-listing)
published                     | false                                | Boolean to enable publishing list of products
type                          | "container stack"                    | String. Type of product list
email_address                 | "mail@example.com"                   | String. Email address is needed for creating openshift-cnf project

### Example Configuration of Openshift-cnf certification project creation
```yaml
---
dci_topic: OCP-4.11
dci_name: Testing Openshift-cnf auto creation and attach
dci_configuration: Using DCI create cnf project and attach product-list
check_for_existing_projects: true
ignore_project_creation_errors: true
dci_config_dirs: [/etc/dci-openshift-agent]
partner_creds: "/var/lib/dci-openshift-app-agent/auth.json"
organization_id: 12345678
#cnf_name is a free-text but format: CNF-version + OCP-version e.g "CNF23.5 OCP4.12.9"
cnf_to_certify:
  - cnf_name: "test-smf23.5 OCP4.11.5"
    create_cnf_project: true
    attach_product_listing: true

  - cnf_name: "test-upf23.5 OCP4.11.5"
    create_cnf_project: true
    attach_product_listing: true

cert_listings:
  #email_address is mandatory when creating openshift-cnf for vendor validation but does not hurt to define it
  email_address: "email@example.com"
  published: false
  type: "container stack"
  pyxis_product_list_identifier: "yyyyyyyyyyyyyyyyy" #7GC UDM

pyxis_apikey_path: "/var/lib/dci-openshift-app-agent/pyxis-apikey.txt"
dci_gits_to_components: []
...
```
**Note:** New and recertified container image projects can be included in the same settings.yml file, but new container certification projects require more detailed descriptions and additional parameters in the cert_setting section. This is so that partners can easily distinguish between new and recertified images.

For recertified container projects, if partners have not yet enabled auto-publish for the projects from the portal Gui, they must manually enable it before using the DCI to automate the certification process.

## How to Run DCI Auto-publish, Recertify and Openshift-cnf Vendor validated
- Login to DCI user  
```shellSession
$ su - dci-openshift-app-agent
```
- Prepare a settings file for different type of certification projects accordingly
  - [Auto Publish New Container Certification Project Settings](https://github.com/ansvu/cnf-certification-automation-with-dci/tree/main#auto-publish-settings-configuration)
  - [Recertify Container Certification Project Settings](https://github.com/ansvu/cnf-certification-automation-with-dci/tree/main#recertify-settings-configuration)
  - [Openshift-cnf Certification Project Vendor Validated Settings](https://github.com/ansvu/cnf-certification-automation-with-dci/tree/main#example-configuration-of-openshift-cnf-certification-project-creation)  
- Export KUBECONFIG  
```shellSession
$ export KUBECONFIG=/var/lib/dci-openshift-app-agent/kubeconfig
```
- Start RUN DCI OpenShift App Agent  
```shellSession
$ dci-openshift-app-agent-ctl -s -- -vv
```
## Known Issues
- Must Gather Log Disable  
For partners with larger OpenShift clusters and disconnected labs, running DCI for container image automation and recertification can pose challenges. Uploading the must-gather log, which collects OCP logging data, becomes problematic due to its size. This can cause DCI to get stuck for over 2 hours as it retries unsuccessfully through the partner's proxy. A solution is to To disable must-gather log collection, the set `do_must_gather: false` to settings.yml. This allows DCI to proceed without uploading the large logs and avoids proxy-related issues.  

