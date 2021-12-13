# Pipeline deployments

The *Secured Data Warehouse Module* creates a secured Bigquery Data Warehouse infrastructure.
To use this infrastructure to deploy new Dataflow Flex Pipelines use the instructions in the following sections.

We assume you are familiar with [Deploying a Pipeline](https://cloud.google.com/dataflow/docs/guides/deploying-a-pipeline).

## *Secured Data Warehouse Module* deployment

It is necessary configure some controls for the deployment of the Secured Data Warehouse to allow user to create Dataflow Flex pipelines.

The Secured Data Warehouse module uses [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs/service-perimeters).

The identity deploying the Dataflow job must be in the [access level](https://cloud.google.com/access-context-manager/docs/create-basic-access-level#members-example) of the perimeter. You can add it using the input `perimeter_additional_members` of the *Secured Data Warehouse Module*.

To use a private template repository outside of the perimeter, the identity deploying the Dataflow job must be in a egress rule that allow the Dataflow templates to be fetched. In the *Secured Data Warehouse Module* you configure it using the appropriated list below.

- For the **confidential perimeter**, the identity needs to be added in the input `confidential_data_dataflow_deployer_identities` of the *Secured Data Warehouse Module*.
- For the **data ingestion perimeter**, the identity needs to be added in the input `data_ingestion_dataflow_deployer_identities` of the *Secured Data Warehouse Module*.

<!--
Introdução
Before you start - vpc sc
Requirements - apis, roles
Boas práticas - o que o usuário deve seguir?
Como fazer - como fazer o deploy usando tudo que nós mencionamos

-->

## Requirements

### Apis

All the required APIs to deploy the module had to be enabled. See the list of [APIs](../README.md#apis) in the README file.
Ensured that all the additional APIs your Dataflow pipeline needs are enabled too.

### Service Accounts Roles

You may need grant additional roles for Dataflow Controller Service Accounts created by the module to be able run your Dataflow Pipeline.

You can check the current roles associated with the Services Accounts in the files linked below:

- Data ingestion Dataflow Controller Service Account [roles](../modules/data-ingestion/service_accounts.tf)
- Confidential Data Dataflow Controller Service Account [roles](../modules/confidential-data/service_accounts.tf)

### Subnetwork

The subnetwork is a [requirement](https://cloud.google.com/dataflow/docs/guides/specifying-networks#specifying_a_network_and_a_subnetwork) to deploy the *Dataflow Pipelines* in the *Secured Data Warehouse Module*.

We do not recommend the usage of [Default Network](https://cloud.google.com/vpc/docs/vpc#default-network) in the *Secured Data Warehouse Module*.

If you are using Shared VPC, make sure to add them as Trusted subnetworks using `trusted_subnetworks` variable. You can check more about it in the *Secured Data Warehouse Module* [inputs](../README.md#inputs) section.

The subnetwork must be [configured for Private Google Access](https://cloud.google.com/vpc/docs/configure-private-google-access). Make sure you have configured all the [firewall rules](#firewall-rules) and [DNS configurations](#dns-configurations) listed in the sections below.

#### Firewall rules

- [All the egress should be denied](https://cloud.google.com/vpc-service-controls/docs/set-up-private-connectivity#configure-firewall).
- [Allow only Restricted API Egress by TPC at 443 port](https://cloud.google.com/vpc-service-controls/docs/set-up-private-connectivity#configure-firewall).
- [Allow only Private API Egress by TPC at 443 port](https://cloud.google.com/vpc-service-controls/docs/set-up-private-connectivity#configure-firewall).
- [Allow ingress Dataflow workers by TPC at ports 12345 and 12346](https://cloud.google.com/dataflow/docs/guides/routes-firewall#example_firewall_ingress_rule).
- [Allow egress Dataflow workers by TPC at ports 12345 and 12346](https://cloud.google.com/dataflow/docs/guides/routes-firewall#example_firewall_egress_rule).

#### DNS configurations

- [Restricted Google APIs](https://cloud.google.com/vpc-service-controls/docs/set-up-private-connectivity#configure-routes).
- [Private Google APIs](https://cloud.google.com/vpc/docs/configure-private-google-access).
- [Restricted gcr.io](https://cloud.google.com/vpc-service-controls/docs/set-up-gke#configure-dns).
- [Restricted Artifact Registry](https://cloud.google.com/vpc-service-controls/docs/set-up-gke#configure-dns).

<!--Is it opinionated or rather best practice -->
<!-- Some of the points in this section must be done, if the user does not follow the instruction he may face errors during deploy new jobs-->
## Best Practice for Dataflow Flex Template Usage

The *Secured Data Warehouse Module* provides resources to deploy secured Dataflow Pipeline.
We highly recommend you to use them to deploy your Dataflow Pipeline.

<!--avoid use or / . use same term in public doc -->
<!--Some template asked to use the Temporary and Staging Location. -->
### Temporary and Staging Location

Use the appropriated [output](../README.md#outputs) of the main module as the Temporary and Staging Location bucket in the [pipeline options](https://cloud.google.com/dataflow/docs/guides/setting-pipeline-options#setting_required_options):

- Data ingestion project: `data_ingestion_dataflow_bucket_name`.
- Confidential Data project: `confidential_data_dataflow_bucket_name`.

### Dataflow Worker Service Account

Use the appropriated [output](../README.md#outputs) of the main module as the [Dataflow Controller Service Account](https://cloud.google.com/dataflow/docs/concepts/security-and-permissions#specifying_a_user-managed_worker_service_account):

- Data ingestion project: `dataflow_controller_service_account_email`.
- Confidential Data project: `confidential_dataflow_controller_service_account_email`.

### Customer Managed Encryption Key

Use the appropriated [output](../README.md#outputs) of the main module as the [Dataflow KMS Key](https://cloud.google.com/dataflow/docs/guides/customer-managed-encryption-keys):

- Data ingestion project: `cmek_data_ingestion_crypto_key`
- Confidential project: `cmek_reidentification_crypto_key`

### Disable Public IPs

[Disabling Public IPs helps to better secure you data processing infrastructure.](https://cloud.google.com/dataflow/docs/guides/routes-firewall#turn_off_external_ip_address). Make sure you have your subnetwork configured as [Subnetwork section](#subnetwork) details.

### Enable Streaming Engine

Enabling Streaming Engine it is important to ensure all the performance benefits of the infrastructure. You can learn more about it in the [documentation](https://cloud.google.com/dataflow/docs/guides/deploying-a-pipeline#streaming-engine).

## Deploying with Terraform

Use the Dataflow Flex Job Template [submodule](../modules/dataflow-flex-job/README.md). See [Tutorial Standalone example](../examples/tutorial-standalone/README.md) for details.

<!--
i'd like user focused actions described as additional topics . consider adding:
I suggest to scope this down to

can't we just point to the cloud build file. don't need to rewrite the code in docs, right?
 -->

### Deploying with `gcloud` Command

**Flex Template** https://cloud.google.com/dataflow/docs/guides/templates/using-flex-templates

You can run the following commands to create a **Java** Dataflow Flex Job using the **gcloud command**:

```sh

export PROJECT_ID=<PROJECT_ID>
export DATAFLOW_BUCKET=<DATAFLOW_BUCKET>
export DATAFLOW_KMS_KEY=<DATAFLOW_KMS_KEY>
export SERVICE_ACCOUNT_EMAIL=<SERVICE_ACCOUNT_EMAIL>
export SUBNETWORK=<SUBNETWORK>

gcloud dataflow flex-template run "TEMPLATE_NAME`date +%Y%m%d-%H%M%S`" \
    --template-file-gcs-location="TEMPLATE_NAME_LOCATION" \
    --project="${PROJECT_ID}" \
    --staging-location="${DATAFLOW_BUCKET}/staging/" \
    --temp-location="${DATAFLOW_BUCKET}/tmp/" \
    --dataflow-kms-key="${DATAFLOW_KMS_KEY}" \
    --service-account-email="${SERVICE_ACCOUNT_EMAIL}" \
    --subnetwork="${SUBNETWORK}" \
    --region="us-east4" \
    --disable-public-ips \
    --enable-streaming-engine

```

For more details about `gcloud dataflow flex-template` see the command [documentation](https://cloud.google.com/sdk/gcloud/reference/dataflow/flex-template/run).

**Classic Template** https://cloud.google.com/dataflow/docs/guides/templates/running-templates#using-gcloud

*********:

```sh

COMMAND

```

For more details about `gcloud dataflow jobs run` see the command [documentation](https://cloud.google.com/sdk/gcloud/reference/dataflow/jobs/run).

## Common tasks

### how do I rerun the job for new data (let's day add 10k more records?

### how do i send streaming data through and how to check it's been de-id

### what if I have my own flex templates in a different project?