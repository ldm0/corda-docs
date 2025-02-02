---
aliases:
- /head/corda-testnet-intro.html
- /HEAD/corda-testnet-intro.html
- /corda-testnet-intro.html
date: '2020-04-07T12:00:00Z'
menu:
  corda-os-4-6:
    identifier: corda-os-4-6-corda-testnet-intro
    parent: corda-os-4-6-deploy-to-testnet-index
    weight: 1000
tags:
- corda
- testnet
- intro
title: Joining Corda Testnet
---


# Joining Corda Testnet


The Corda Testnet is an open public network of Corda nodes on the internet. It is designed to be a complement to *the* Corda Network where any entity can transact real world value with any other counterparty in the context of any application. The Corda Testnet is designed for “non-production” use in a genuine global context of Corda nodes, including but not limited to CorDapp development, multi-party testing, demonstration and showcasing of applications and services, learning, training and development of the Corda platform technology and specific applications of Corda.

The Corda Testnet is based on the same underlying technology as the main Corda Network, however it can be joined on a self-service basis through the automated provisioning system described below.

{{< warning >}}

The Corda Testnet services are a demonstration version only - they are supported on a "best-effort" basis and there are no commitments with regards to their uptime.

Users looking for long-lasting supported networks should consider the [Corda Network](https://corda.network/) for UAT and Production Networks, or refer to [the Corda Enterprise Network Manager documentation](https://docs.corda.net/docs/cenm/index.html) to evaluate or license the software recommended to operate a production Network on Corda.

{{< /warning >}}


## Deploying a Corda node to the Corda Testnet

The Corda Testnet is accessible via [https://marketplace.r3.com/](https://marketplace.r3.com/).

[{{< figure alt="testnet landing" zoom="/en/images/testnet-landing.png" >}}
Click on “Join the Corda Testnet” to begin joining the network.

This will create an account with the Testnet on-boarding application which will enable you to provision and manage multiple Corda nodes on Testnet. You will log in to this account to view and manage you Corda Testnet identity certificates.

{{< figure alt="testnet signup" zoom="/en/images/testnet-signup.png" >}}
Fill in the registration form and please read and accept the terms of use, then click Next Step; this will take you to the Testnet dashboard where you can see a list of your nodes as well as create new ones.

When creating a new node you can now choose how to deploy it to the Corda Testnet. We strongly recommend hosting your Corda node on a public cloud resource.

Select the cloud provider you wish to use for documentation on how to specifically configure Corda for that environment.

{{< figure alt="testnet platform" zoom="/en/images/testnet-platform.png" >}}
Once your cloud instance is set up you can install and run your Testnet pre-provisioned Corda node by clicking on “Copy” and pasting the one time link into your remote cloud terminal.

The installation script will download the Corda binaries as well as your PKI certificates, private keys and supporting files and will install and run Corda on your fresh cloud VM. Your node will register itself with the Corda Testnet when it first runs and be added to the global network map and be visible to counterparties after approximately 5 minutes.

Hosting a Corda node locally is possible but will require manually configuring firewall and port forwarding on your local router. If you want this option then click on the “Download” button to download a Zip file with a pre-configured Corda node.

{{< note >}}
If you host your node on your own machine or a corporate server you must ensure it is reachable from the public internet at a specific IP address. Please follow the instructions here: [Deploying Corda to Corda Testnet from your local environment](deploy-locally.md).

{{< /note >}}

## A note on identities on Corda Testnet

Unlike the main Corda Network, which is designed for verified real world identities, The Corda Testnet automatically assigns a “distinguished name” as your identity on the network. This is to prevent name abuse such as the use of offensive language in the names or name squatting. This allows the provision of a node to be automatic and instantaneous. It also enables the same user to safely generate many nodes without accidental name conflicts. If you require a human readable name then please contact support and a partial organisation name can be approved.
