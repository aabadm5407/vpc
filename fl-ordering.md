---

copyright:
  years: 2019, 2020
lastupdated: "2020-04-22"

keywords: flow logs, ordering, getting started

subcollection: vpc
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:term: .term}
{:tip: .tip}
{:important: .important}
{:external: target="_blank_" .external}
{:generic: data-hd-programlang="generic"}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Creating a flow log collector
{: #ordering-flow-log-collector}

You can order and provision a flow log collector for a specific Virtual Private Cloud (VPC), subnet, instance, or interface. Before you begin, make sure that you review the use cases listed in [About flow logs](/docs/vpc?topic=vpc-fl-prereq) and satisfy the following prerequisites. 

When provisioning a flow log collector, keep in mind that [the finest granularity wins](/docs/vpc?topic=vpc-flow-logs#flow-logs-granularity-wins).
{: tip}

## Prerequisites
{: #fl-before-you-begin}
 
Prior to creating a flow log collector, ensure that you have met the following prerequisites: 
 
1. Make sure that at least one VPC, a subnet, and a virtual server instance exist. For instructions, see [Creating a VPC and subnet](/docs/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpc-and-subnet) and [Creating a virtual server instance](/docs/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vsi).
2. Make sure that a COS instance with a bucket exists for your flow logs. To create a COS bucket, see the [Cloud Object Storage](https://cloud.ibm.com/catalog/services/cloud-object-storage) ordering page. For more information, see [Getting started with IBM Cloud Object Storage](/docs/cloud-object-storage?topic=cloud-object-storage-getting-started).

   The COS bucket must be a single-region bucket in the same region as the target resource.
   {: important}
   
3. [Authorize](/docs/iam?topic=iam-serviceauth#create-auth) resources of type **Flow Log Collector** in the VPC infrastructure to use the COS instance identified in Step 2.

## Using the UI
{: #fl-ordering-ui}

To create a flow log collector by using the IBM Cloud console, follow these steps:

1. Navigate to the [Virtual Private Cloud](https://cloud.ibm.com/vpc/) page in the IBM Cloud console.
2. Select **Gen 2 Compute**.
3. In the left navigation pane, click **Flow Log Collectors (beta)**. The flow logs list appears.

  ![Flow Log Collector List View](./images/screenshots/flow-log-collectors/list-view-01.png "Flow Log Collector List View")

4. Click **Create flow log collector** to go to the flow logs provisioning page.
5. Enter values for the following fields:

  * **Name** - Type a unique name for your flow log collector.
  * **Resource group** - Select a resource group for your flow log collector.

6. From the **Attach the flow log connector to** menu, choose a "target type" for the flow log. Depending on your selection, additional fields might be required.   

  * **VPC** - Select a VPC. All network traffic within the selected VPC is logged. 
  * **Subnet** -  Select a VPC and a subnet within the selected VPC. All traffic within the selected subnet is logged. 
  * **Instance** - Select a VPC and a VSI that exists within the selected VPC. All traffic for the VSI is logged. 
  * **Interface** - Select a VPC, a VSI within the selected VPC, and a specific network interface for the selected VSI. All traffic for the selected network interface is logged.

  ![Example Network Interface Target](./images/flow-log-provision-interface-target-example.png "Example Network Interface Target")

8. Specify where the logs are written. Flow logs are written to a COS bucket, which must be created as a single-region bucket in the same region as the target resource.

  * **Cloud Object Storage Instance** - The COS instance that the wanted bucket resides in.
  * **Location** - This input is unavailable because it is directly tied to the region the target resource resides in.
  * **Bucket** - The wanted Cloud Object Storage (COS) bucket that the flow log collector service writes to. 

## Using the CLI
{: #fl-ordering-cli}

To create a flow log collector by using the CLI, run the following command:

  ``` 
  ibmcloud is flow-log-create \
    --bucket STORAGE_BUCKET_NAME \
    --target TARGET_ID [--name NAME] \
    [--resource-group-id RESOURCE_GROUP_ID | --resource-group-name RESOURCE_GROUP_NAME] \
    [--json]
  ```

Where...

* **--bucket** is the name of the COS bucket.
* **--target** is the target for the flow log.
* **--name** is the new name for the flow log.
* **--resource-group-id** is the ID of the resource group. This option is mutually exclusive with **--resource-group-name**.
* **--resource-group-name** is the name of the resource group. This option is mutually exclusive with **--resource-group-id**.
* **--json** formats the output in JSON.

## Using the API
{: #fl-ordering-api}

To create a flow log collector by using the API, follow these steps:

1. Provision these variables with the appropriate values:

   * `token` - Use the following command:

      ```
      export token="$(ibmcloud iam oauth-tokens | awk '{ print $4 }')"
      ```
      {: pre}

   *  `api_endpoint` - Set the environment end point. For example, for a production in `us-south` use:
    
      ```
      export api_endpoint=https://us-south.iaas.cloud.ibm.com
      ```
      {: pre}

   * `ResourceGroupId` - First, get your resource group and then populate the variable:

      ```
      ibmcloud resource groups
      export ResourceGroupId=
      ```
      {: pre}

   * `VpcId` - Find by using the **list vpc** command (with the preceding variables) and then populate the variable based on the provided ID:
   
      ```
      curl -k -sS -H "Authorization: Bearer ${token}" $api_endpoint/v1/vpcs?version=2019-10-03  | jq
      export VpcId=
      ```
      {: pre}

2. When all variables are initiated, provision a flow log collector for the specific VPC:

   ```
   curl -X POST 
     -sH "Authorization:${token}" 
     $api_endpoint/v1/flow_log_collectors?version=2019-10-03 \
     -d  '{ \
          "name": "flow-logs-1", \
          "resource_group": { "id": "'$ResourceGroupId'"  }, \
          "storage_bucket": { "name": "riastestbucket" }, \
          "target": { "id": "'$VpcId'" } \
          }' | jq
   ```
   {: pre}

3. To provision a collector that targets a subnet, VSI, or VNIC, you must provide a subnet ID, VSI ID, or VNIC ID as collector targets. For example, the following request creates a collector that targets a VSI ID:

   ```bash
   export VsiId=

   curl -X POST \
     -sH "Authorization:${token}" \
     $api_endpoint/v1/flow_log_collectors?version=2019-10-03 \
     -d '{ \
      	 "name": "flow-logs-1", \
         "resource_group": { "id": "'$ResourceGroupId'"  }, \
         "storage_bucket": { "name": "riastestbucket" }, \
         "target": { "id": "'$VsiId'" } \
         }' | jq    
   ```
   {: pre}
   
## Next steps

* [Viewing flow log records](/docs/vpc?topic=vpc-fl-analyze)
* Working with flow logs
   * [Managing access](/docs/vpc?topic=vpc-fl-iam)
   * [Listing flow log collectors](/docs/vpc?topic=vpc-listing-all-flow-log-collectors)
   * [Suspending and resuming flow log collectors](/docs/vpc?topic=vpc-managing-flow-log-collectors_activate)
   * [Deleting a flow log collector](/docs/vpc?topic=vpc-deleting-a-flow-log-collector)