#!/bin/bash

# This script creates the following resources in the specified subscription:
# - Resource group
# - Network Security Group rules
# - Virtual network (vnet) and subnet
# - Network Settings with specified subnet and GitHub Enterprisedatabase ID
#
# It also registers the `GitHub.Network` resource provider with the subscription,
# delegates the created subnet to the Actions service via the `GitHub.Network/NetworkSettings`
# resource type, and applies the NSG rules to the created subnet.

# stop on failure
set -e

#set environment
export AZURE_LOCATION=YOUR_AZURE_LOCATION
export SUBSCRIPTION_ID=YOUR_SUBSCRIPTION_ID
export RESOURCE_GROUP_NAME=YOUR_RESOURCE_GROUP_NAME
export VNET_NAME=YOUR_VNET_NAME
export SUBNET_NAME=YOUR_SUBNET_NAME
export NSG_NAME=YOUR_NSG_NAME
export NETWORK_SETTINGS_RESOURCE_NAME=YOUR_NETWORK_SETTINGS_RESOURCE_NAME
export DATABASE_ID=YOUR_DATABASE_ID

# These are the default values. You can adjust your address and subnet prefixes.
export ADDRESS_PREFIX=10.0.0.0/16
export SUBNET_PREFIX=10.0.0.0/24

echo
echo login to Azure
. az login --output none

echo
echo set account context $SUBSCRIPTION_ID
. az account set --subscription $SUBSCRIPTION_ID

echo
echo Register resource provider GitHub.Network
. az provider register --namespace GitHub.Network

echo
echo Create resource group $RESOURCE_GROUP_NAME at $AZURE_LOCATION
. az group create --name $RESOURCE_GROUP_NAME --location $AZURE_LOCATION

echo
echo Create NSG rules deployed with 'actions-nsg-deployment.bicep' file
. az deployment group create --resource-group $RESOURCE_GROUP_NAME --template-file ./actions-nsg-deployment.bicep --parameters location=$AZURE_LOCATION nsgName=$NSG_NAME

echo
echo Create vnet $VNET_NAME and subnet $SUBNET_NAME
. az network vnet create --resource-group $RESOURCE_GROUP_NAME --name $VNET_NAME --address-prefix $ADDRESS_PREFIX --subnet-name $SUBNET_NAME --subnet-prefixes $SUBNET_PREFIX

echo
echo Delegate subnet to GitHub.Network/networkSettings and apply NSG rules
. az network vnet subnet update --resource-group $RESOURCE_GROUP_NAME --name $SUBNET_NAME --vnet-name $VNET_NAME --delegations GitHub.Network/networkSettings --network-security-group $NSG_NAME

echo
echo Create network settings resource $NETWORK_SETTINGS_RESOURCE_NAME
. az resource create --resource-group $RESOURCE_GROUP_NAME  --name $NETWORK_SETTINGS_RESOURCE_NAME --resource-type GitHub.Network/networkSettings --properties "{ \"location\": \"$AZURE_LOCATION\", \"properties\" : {  \"subnetId\": \"/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP_NAME/providers/Microsoft.Network/virtualNetworks/$VNET_NAME/subnets/$SUBNET_NAME\", \"businessId\": \"$DATABASE_ID\" }}" --is-full-object --output table --query "{GitHubId:tags.GitHubId, name:name}" --api-version 2024-04-02

echo
echo To clean up and delete resources run the following command:
echo az group delete --resource-group $RESOURCE_GROUP_NAME

# GitHub public roadmap

:sparkle: View the [official GitHub public product roadmap](https://github.com/orgs/github/projects/4247)[^1]

Our product roadmap is where you can learn about what features we're working on, what stage they're in, and when we expect to bring them to you. Have any questions or comments about items on the roadmap? Share your feedback via [GitHub public feedback discussions](https://github.com/github/feedback/discussions). 

[^1]:We've adopted the latest [beta features of GitHub projects](https://github.com/features/issues) for the [public roadmap](https://github.com/orgs/github/projects/4247). The [roadmap project board](https://github.com/github/roadmap/projects/1) within this repository is now closed. 

The roadmap repository is for communicating GitHub’s roadmap. Existing issues are currently read-only, and we are locking conversations, as we get started. Interaction limits are also in place to ensure issues originate from GitHub. We’re planning to iterate on the format of the roadmap itself, and we see potential to engage more in discussions about the future of GitHub products and features. If you have feedback about this roadmap repository itself, such as how the issues are presented, let us know through [general feedback in GitHub public feedback discussions](https://github.com/github/feedback/discussions/new?category=General-Feedback&title=[Public%20roadmap]%20).


## Guide to the roadmap

Every item on the roadmap is an issue, with a label that indicates each of the following:

- A **release phase** that describes the next expected phase of the roadmap item. See below for a guide to release phases. 

- A **feature area** that indicates the area of the product to which the item belongs. For a list of current product areas, see below.

- A **feature** that indicates the feature or product to which the item belongs. For a list of current features, see below. 

- One or more **product SKU** labels that indicate which product SKUs we expect the feature to be available in. For a list of current product SKUs, see below.

- One or more **deployment models** (cloud, server, and/or ae). Where not stated, features will generally come out Cloud first, and follow on Server and GHAE at or soon after GA.

- Once a feature is delivered, the **shipped** label will be applied to the roadmap issue and the issue will be closed with a comment linking to the relevant [Changelog](https://github.blog/changelog/) post.

## Release phases

Release phases indicate the stages that the product or feature goes through, from early testing to general availability.

- **alpha:** *Primarily for testing and feedback*\
Limited availability, requires pre-release agreement. Features still under heavy development, and subject to change. Not for production use, and no documentation, SLAs or support provided.

- **beta:** *Publicly available in full or limited capacity*\
Features mostly complete and documented. Timeline and requirements for GA usually published. No SLAs or support provided.

- **ga:** *Generally available to all customers*\
Ready for production use with associated SLA and technical support obligations. Approximately 1-2 quarters from Beta.

Some of our features may still be in the exploratory stages, and have no timeframe available. These are included in the roadmap only for early feedback. These are marked as follows: 

- **in design:**\
Feature in discovery phase. We have decided to build this feature, but are still figuring out _how_.

- **exploring:**\
Feature under consideration. We are considering building this feature, and gathering feedback on it.

## Release phases - For GHES

Some features may be marked with a GHES 3.X label, which indicates that the feature will also become available for Github Enterprise Server customers. Below are the release version numbers and expected release quarters (_Note: these dates are subject to change_). 

**GHES release version dates**:
| **Version Number** | **Release Quarter** | **Release Notes** |
|-|-|-|
| 3.5 | Q2 2022 | [Release Notes](https://docs.github.com/en/enterprise-server@3.5/admin/release-notes) |
| 3.6 | Q3 2022 | [Release Notes](https://docs.github.com/en/enterprise-server@3.6/admin/release-notes) |
| 3.7 | Q4 2022 | - |
| 3.8 | Q1 2023 | - |

## Roadmap stages

The roadmap is arranged on a project board to give a sense for how far out each item is on the horizon. Every product or feature is added to a particular project board column according to the quarter in which it is expected to ship next. Be sure to read the [disclaimer](#disclaimer) below since the roadmap is subject to change, especially further out on the timeline.  You'll also find an **Exploratory** column, which is used in conjunction with the **in design** and **exploring** release phase labels for when no timeframe is yet available.

GitHub Enterprise Server has major releases on a quarterly basis, and minor releases on a monthly basis. Once we know what version we are delivering a feature, we will update the issue to indicate that information.

## Feature Areas

The following is a list of our current product areas:

- **code:** Code experiences (Repositories, Pull Requests, Gists)
- **planning:** Planning and tracking tools (Issues, Projects)
- **code-to-cloud:** Code-to-cloud DevOps (Actions, Packages)
- **collaboration:** Collaboration features (Pages, Wikis, Discussions)
- **security & compliance:** Code security and compliance features
- **admin-server:** Administrative features specific to GitHub Enterprise Server
- **admin-cloud:** Administrative features specific to GitHub Cloud
- **communities:** Community and social features
- **ecosystem:** Ecosystem and API features
- **learning:** Education and learning features
- **insights:** Continuous learning and insights features
- **client-apps:** Client applications (Desktop, Mobile)
- **other:** Other features

## Feature

The following is a list of our current features and products, with distinct labels for filtering:

- **actions:** GitHub Actions
- **docs:** GitHub Docs
- **packages:** GitHub Packages
- **pages:** GitHub Pages

_More labels will be added in the future as needed._

## Product SKUs 

The following is a list of our current product SKUs. 

- **all:** Available to all users, including a free tier. Different SKUs may have different levels of functionality.
- **github team:** GitHub Team
- **github enterprise:** GitHub Enterprise (Cloud and Server)
- **github one:** GitHub One (Cloud and Server)
- **github ae:** GitHub AE (GHAE)
- **github advanced security:** GitHub Advanced Security (add-on to GHE)
- **github insights:** GitHub Insights (add-on to GHE)
- **github learning lab:** GitHub Learning Lab (add-on to GHE)

## Disclaimer 

Any statement in this repository that is not purely historical is considered a forward-looking statement. Forward-looking statements included in this repository are based on information available to GitHub as of the date they are made, and GitHub assumes no obligation to update any forward-looking statements. The forward-looking product roadmap does not represent a commitment, guarantee, obligation or promise to deliver any product or feature, or to deliver any product and feature by any particular date, and is intended to outline the general development plans. Customers should not rely on this roadmap to make any purchasing decision.
