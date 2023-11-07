Table of Contents
=================

* [Table of Contents](#table-of-contents)
* [Automate certification process by utilizing DCI to interact with the certification portal backend](#automate-certification-process-by-utilizing-dci-to-interact-with-the-certification-portal-backend)
   * [Prerequisites](#prerequisites)     
   * [Automation Container Certification Flow](#automation-container-certification-flow)
      * [Auto Publish Preparations](#auto-publish-preparations)
      * [Auto Publish Settings Configuration](#auto-publish-settings-configuration)
   * [Recertify Certification Container Projects](#recertify-certification-container-projects)
      * [Recertify Settings Configuration](#recertify-settings-configuration)
   * [E2E Automation Certification Of Operator Bundle Project](#e2e-automation-certification-of-operator-bundle-project)
      * [E2E Certification Settings Of Operator Bundle](#e2e-certification-settings-of-operator-bundle)
   * [Automate creation of the Openshift-cnf project for Vendor Validated](#automate-creation-of-the-openshift-cnf-project-for-vendor-validated)
      * [Global Variables](#global-variables)
      * [Variables to define for each cnf_to_certify](#variables-to-define-for-each-cnf_to_certify)
      * [Variables to define for project settings under cert_listings main variable](#variables-to-define-for-project-settings-under-cert_listings-main-variable)
      * [Example Configuration of Openshift-cnf certification project creation](#example-configuration-of-openshift-cnf-certification-project-creation)
   * [Run Chart Verifier and TNF Suite Test Together](#run-chart-verifier-and-tnf-suite-test-together)
      * [Example of chart-verifier and TNF Test Suite Settings Configuration](#example-of-chart-verifier-and-tnf-test-suite-settings-configuration)
   * [How to Run DCI Auto-publish, Recertify and Openshift-cnf Vendor validated](#how-to-run-dci-auto-publish-recertify-and-openshift-cnf-vendor-validated)
   * [Known Issues](#known-issues)
   * [Links](#links)

# Automate certification process by utilizing DCI to interact with the certification portal backend
This repository provides an automated certification process for container projects, along with their automatic publication. It encompasses both new container certification and container new release recertification by leveraging the new features offered by DCI. It also includes new procedure of `Openshift-cnf` certification project auto creation for Vendor Validate and CNF certification.

The process for certifying a new container involves creating container projects, adding or modifying mandatory project parameters, testing and scanning container images, attaching to a product listing, and auto publishing container projects once all requirements are fulfilled.

The process for recertifying existing container projects involves retesting and rescanning new release container images, updating the container's version, its digest and tag, and republishing to the catalog.

Finally, an integrated process for creating the Openshift-cnf certification project has been introduced into the workflow. Previously, the partner CNF Vendor Validate required a separate steps. However, with the new enhancement implemented in DCI, once the container and helm chart projects are certified, the Openshift-cnf project will be automatically created.
 
 
## Prerequisites
- Prepare a jumphost to install and run DCI then following this [link](https://doc.distributed-ci.io/dci-openshift-agent/#installation-of-dci-jumpbox)
- Minimum DCI Openshift APP Agent version (Recommended to use latest DCI software version)
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
- DCI Control server credential
  [create remote-ci credentials](https://www.distributed-ci.io/remotecis)
- Prepare settings.yml for container and CNF projects information  
  The details of each container certification project type are shown on next sections
- Set `auto-publish` parameter to `on` under container project settings tab

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

  - container_image: "quay.io/avu0/auto-publish-ubi8-nginx-demo2:v120"
    create_container_project: true
    short_description: "I am doing a full-automation e2e auto-publish for following image auto-publish-ubi8-nginx-demo2:v120"

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
  attach_product_listing: true

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
  
Please note that these are just general steps and may vary depending on the specific project. And also basic and general requirements to run `recertify` the container projects please use follow this [section](https://github.com/ansvu/cnf-certification-automation-with-dci/tree/main#auto-publish-preparations)

### Recertify Settings Configuration
```yaml
---
dci_topic: OCP-4.11
dci_name: Testing DCI to Recertify the certification container projects
dci_configuration: Using DCI to Recertify the Certification container Project
check_for_existing_projects: true
organization_id: 12345678
dci_config_dirs: [/etc/dci-openshift-agent]
partner_creds: "/var/lib/dci-openshift-app-agent/auth.json"
preflight_containers_to_certify:
  - container_image: "quay.io/avu0/auto-publish-ubi8-nginx-demo1:v121"

  - container_image: "quay.io/avu0/auto-publish-ubi8-nginx-demo2:v121"

pyxis_apikey_path: "/var/lib/dci-openshift-app-agent/pyxis-apikey.txt"
dci_gits_to_components: []
...
```
## E2E Automation Certification Of Operator Bundle Project
This is a new improvement of E2E certification of operators with DCI, where update and attach the product-listing are now automated. Tatiana from DCI team created this blog which have some details of the pre-requisite and settings, please click [end-to-end-certification-of-operators-with-dci](https://blog.distributed-ci.io/preflight-integration-in-dci.html#end-to-end-certification-of-operators-with-dci).

*Note:* There are some new changes on settings which will provide on next section. And a new join blog with DCI team about these new certification improvement of DCI.

### E2E Certification Settings Of Operator Bundle
Similar settings configuration requirements of container certification project with a slightly differences suchas `github_token_path` as additional to `pyxis_product_list_identifier`, `partner_creds`(auth.json), `organization_id` and `pyxis_apikey_path`. 

Name                     | Default                                                                    | Description
-------------------      | ------------                                                               | -------------
create_operator_project  | false                                                                      | If set to true, it would create a new a new "Operator Bundle Image" and submit test results in it
attach_product_listing   | false                                                                      | If e2e certification then this parameters need to set to `true` to attach the product-listing ID to project.
create_pr                | none                                                                       | If defined, Optional; use it to open the certification PR automatically in the certified-operators repository
merge_pr                 | false                                                                      | If defined, Optional; use it to merge the certification PR automatically in the certified-operators repository
page_size                | 200                                                                        | If defined, Optional; in case there are many archived or active projects are in the account, update if >= 200
github_token_path        | none                                                                       | Optional; if create_pr sets to true then it requires to generate the github-token, follow [generate a github token](https://github.com/redhatci/ansible-collection-redhatci-ocp/tree/main/roles/create_certification_project#github-token)
check_for_existing_projects | false                                                                   | If create_operator_project sets to `true`, then this parameter `check_for_existing_projects` need to set to `true` to check if project is already existed.
organization_id          |                                                                            | If create_operator_project sets to `true`, then `organization_id` need to be defined with the account associates to this organization ID. 
auto_publish             | false                                                                      | If e2e auto-publish the operator certification then set to `true` under `cert_settings`
published                | false                                                                      | Similar to auto_publish purpose if e2e certification is the purpose then set to `true` under `cert_listings`

Note: Parameters from cert_settings and cert_listings are mandatory 
```yaml
---
dci_topic: OCP-4.13
dci_name: Testing e2e creation operator bundle project and PR
dci_configuration: Using DCI to scan simple operator image.
check_for_existing_projects: true
dci_config_dirs: [/etc/dci-openshift-app-agent]
partner_creds: "/var/lib/dci-openshift-app-agent/demo-auth.json"
organization_id: 15451045
do_must_gather: false
preflight_run_health_check: false
page_size: 300
github_token_path: "/var/lib/dci-openshift-app-agent/dummy-git-token.txt"
# you could provide many operators at once.
preflight_operators_to_certify:
  - bundle_image: "quay.io/avu0/ava4-simple-demo-operator-bundle:v0.0.6"
    index_image: "quay.io/avu0/ava4-ark-simple-demo-operator-catalog:v0.0.6"
    short_description: "Describe specific about your operator bundle image"
    create_operator_project: true
    create_pr: true
    merge_pr: false

cert_settings:
   auto_publish: false
   registry_override_instruct: "<p>How to get this operator image.</p>"
   email_address: "me@redhat.com"
   application_categories: "Networking"
   privileged: false
   repository_description: "Repository Description"

cert_listings:
  published: false
  type: "container stack"
  pyxis_product_list_identifier: "639b4bfd27b76af009e324cb"
  attach_product_listing: true

pyxis_apikey_path: "/var/lib/dci-openshift-app-agent/demo-pyxis-apikey.txt"
dci_gits_to_components: []
...
```

## Automate creation of the Openshift-cnf project for Vendor Validated
This feature automatically generates an OpenShift-CNF certification project when this option `create_cnf_project` set to `true`. The new feature reuses some tasks from the `create-certification-project` role, and the associated templates are stored within this feature. Currently, there are no mandatory parameters that need to be updated in the new feature.

Please note that the feature openshift-cnf is currently a basic automation setup. It will undergo further updates once additional options are available from the backend REST API, such as `{"certification_level":"Certified"}` parameter for starting the Certify the functionality of your CNF on Red Hat OpenShift step within the project.


### Global Variables
As the new role openshift-cnf reuses some existing tasks, please refer to the description in the `create-certification-project` role for information on the shared global variables.


### Variables to define for each cnf_to_certify

Name                     | Default                                                                    | Description
-------------------      | ------------                                                               | -------------
create_cnf_project       | false                                                                      | If set to true, it would create a new Openshift-cnf certification project.
cnf_name                 | None                                                                       | If defined, it would create Openshift-cnf certification project for vendor validated, cnf_name format: `CNF25.8 + OCP4.12`

### Variables to define for project settings under `cert_listings` main variable

Name                          | Default                              | Description
----------------------------- | ------------------------------------ | -------------
attach_product_listing        | false                                | If set to true, it would attach product-listing to Openshift-cnf certification project.
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

  - cnf_name: "test-upf23.5 OCP4.11.5"
    create_cnf_project: true

cert_listings:
  attach_product_listing: true
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

## Run Chart Verifier and TNF Suite Test Together
This section will show to use DCI to run chart-verifier with option `-c` or `--skip-cleanup` so CNF helm chart deploys and leave CNF PODs running then using DCI to test the TNF suite base CNF namespace. 

### Example of chart-verifier and TNF Test Suite Settings Configuration
```yaml
---
dci_topic: OCP-4.11
dci_name: Chart-verifier with TNF Suite testing
dci_configuration: DCI Chart-verifier SampleChart + TNF Suite testing
dci_openshift_app_image: quay.io/testnetworkfunction/cnf-test-partner:latest
do_chart_verifier: true
dci_openshift_app_ns: avachart
dci_teardown_on_success: false
do_must_gather: false
check_workload_api: false
dci_disconnected: false
partner_name: telcoci SampleChart
partner_email: telco.sample@redhat.com
github_token_path: "/opt/cache/token.txt"
dci_charts:
  - name: samplechart2
    chart_file: https://github.com/ansvu/samplechart2/releases/download/samplechart-0.1.3/samplechart-0.1.3.tgz
    #chart_values: https://github.com/ansvu/samplechart2/releases/download/samplechart-0.1.3/values.yaml
    #install: true
    deploy_chart: true
    flags: "-c -W"
    create_pr: false

do_cnf_cert: true
tnf_labels: common,telco,extended
tnf_log_level: trace
tnf_config:
  - namespace: avachart
    targetNameSpaces:
      - avachart
    operators_regexp:
    exclude_connectivity_regexp:
test_network_function_version: v4.2.4
dci_gits_to_components: []
...
```
Note: Some cases where partner wants to deploy CNF and leave it running and do not want to teardown. So with new feature from chart-verifier `1.12.1`, it adds `-c` option to allow users to skip the `cleanup` e.g. `helm uninstall`. 

Result can be seen from DCI Job Server then click [here](https://www.distributed-ci.io/jobs/c65dae62-d2bb-4b28-becf-ff0975130851)

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

- Must gather log disable

For partners with larger OpenShift clusters and disconnected labs, running DCI for container image automation and recertification can pose challenges. Uploading the must-gather log, which collects OCP logging data, becomes problematic due to its size. This can cause DCI to get stuck for over 2 hours as it retries unsuccessfully through the partner's proxy.

A solution is to disable must-gather log collection, the set `do_must_gather: false` to settings.yml. This allows DCI to proceed without uploading the large logs and avoids proxy-related issues.  

## Links
- [dci-openshift-app-agent](https://doc.distributed-ci.io/dci-openshift-app-agent/)
- [dci-packages](https://blog.distributed-ci.io/install-openshift-on-baremetal-using-dci.html#dci-packages)
