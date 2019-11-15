# Private Data Collections on Hyperledger Fabric

In the Medical Supply Chain, there are multiple entities such as the drug manufacturers, wholesellers, pharmacies and patients. These entities share data about the pill as it moves through the chain. However, there are cases where entities want to keep some data about the pill hidden from the other entities. Consider the instance where a manufacturer have negotiated different price rates with the wholesellers. They wouldn't want the different wholesellers to be able to see the various drug rates negotiated. Having all entities on the same channel of the blockchain would inherently make every transaction between any two entities, visible to every other entity. With the introduction of private data collections, certain data parts associated with a given transaction, can be kept private from other entities.

In this pattern, we showcase 1 manufacturer, 2 wholesalers, 1 pharmacy and 1 patient connected on the same channel on a blockchain ledger. The manufacturer generates a new drug pill, and sells it at different prices to the two wholesellers. Only the manufacturer and the patient have visibility to the two negotiated prices for this example.

This code pattern is for developers who want to learn how to use the private data collections feature introduced into Hyperledger Fabric. When you have completed it, you will understand how to:

* Create multiple organizations with the IBM Blockchain Platform.
* Create a VueJS web app that has multiple dashboards on a Single Page Application, which can communicate in realtime with each other.
* Create a NodeJS server that is deployed to Kubernetes on IBM Cloud.
* Use private data collections to enforce data privacy between organizations on the same channel.

# Architecture flow

![Architecture flow](docs/doc-images/arch-flow.png?raw=true)

1. The blockchain operator creates a IBM Kubernetes Service cluster and an IBM Blockchain Platform 2.0 service.
1. The IBM Blockchain Platform 2.0 creates a Hyperledger Fabric network on an IBM Kubernetes Service, and the operator installs and instantiates the smart contract on the network.
1. The Node.js application server uses the Fabric SDK to interact with the deployed network on IBM Blockchain Platform 2.0.
1. The React UI uses the Node.js application API to interact and submit transactions to the network.
1. The user interacts with the supply chain application web interface to update and query the blockchain ledger and state.

# Included components

+ [IBM Blockchain Platform](https://console.bluemix.net/docs/services/blockchain/howto/ibp-v2-deploy-iks.html#ibp-v2-deploy-iks) gives you total control of your blockchain network with a user interface that can simplify and accelerate your journey to deploy and manage blockchain components on the IBM Cloud Kubernetes Service.
+ [IBM Cloud Kubernetes Service](https://www.ibm.com/cloud/container-service) creates a cluster of compute hosts and deploys highly available containers. A Kubernetes cluster lets you securely manage the resources that you need to quickly deploy, update, and scale applications.

## Featured technologies

* [Nodejs](https://www.nodejs.org/) is an open-source, cross-platform JavaScript run-time environment that executes JavaScript code server-side.
* [Vuejs](https://vuejs.org/) is a progressive framework for building user interfaces.
* [Bootstrap](https://getbootstrap.com/) is a free and open-source front-end Web framework. It contains HTML and CSS-based design templates for typography, forms, buttons, navigation and other interface components, as well as optional JavaScript extensions.
* [Docker](https://www.docker.com/) is a computer program that performs operating-system-level virtualization, also known as Containerization.

## Prerequisites

* [IBM Cloud account](https://cloud.ibm.com/registration/?target=%2Fdashboard%2Fapps)
* [Docker](https://www.docker.com/products) - latest
* [Docker Compose](https://docs.docker.com/compose/overview/) - latest
* [NPM](https://www.npmjs.com/get-npm) - latest
* [nvm]() - latest
* [Node.js](https://nodejs.org/en/download/) - Node v8.9.x
* [Git client](https://git-scm.com/downloads) - latest

# Steps (Cloud Network)

1. [Create IBM Cloud services](#step-1-Create-IBM-Cloud-services)
2. [Build a network - Certificate Authority](#step-2-Build-a-network---Certificate-Authority)
3. [Build a network - Create MSP Definitions](#step-3-Build-a-network---Create-MSP-Definitions)
4. [Build a network - Create Peers](#step-4-build-a-network---Create-Peers)
5. [Build a network - Create Orderer](#step-5-Build-a-network---Create-Orderer)
6. [Build a network - Create and Join Channel](#step-6-Build-a-network---Create-and-Join-Channel)
7. [Deploy Insurance Smart Contract on the network](#step-7-Deploy-Insurance-Smart-Contract-on-the-network)
8. [Connect application to the network](#step-8-Connect-application-to-the-network)
9. [Enroll App Admin Identities](#step-9-Enroll-App-Admin-Identities)
10. [Run the application](#step-10-Run-the-application)

**Important Note:** This pattern is more advanced because it uses four organizations. For this reason, you will likely
have to get a paid kubernetes cluster to run this pattern on the cloud, since a free cluster will not have the CPU/storage 
necessary to deploy all of the pods that we need to run this pattern. There are other patterns that leverage a free
Kubernetes cluster (and only two organizations), so if you want to try that one out first, go [here](https://github.com/IBM/blockchainbean2). 

## Step 1. Create IBM Cloud services

* Create the [IBM Cloud Kubernetes Service](https://cloud.ibm.com/catalog/infrastructure/containers-kubernetes).  You can 
find the service in the `Catalog`. Note that <b>for this code pattern, we need to use the 32CPU, 32GB RAM cluster.</b>

* Once you reach the <b> create a new cluster page </b> you will need to do the following:
  - Choose <b>standard</b> cluster type
  - Fill out cluster name
  - choose Geography: <b>North America</b>
  - Choose Location and availability: <b>Multizone</b>
  - Choose Metro: <b>Dallas</b>
  - Choose Worker nodes: <b>Dallas 10 only</b>
  - Choose Master service endpoint: <b>Both private & public endpoints</b>
  - Choose Default worker pool: <b>1.12.7 (Stable, Default)</b>
  - Choose Master service endpoint: <b>Both private & public endpoints</b>
  - Choose Flavor <b>32 Cores 32GB RAM, Ubuntu 18</b>
  - Choose Encrypt local disk <b>Yes</b>
  - Choose Worker nodes <b>3</b>
  - Click on <b>create cluster. </b>
<b>The cluster takes around 15-20
minutes to provision, so please be patient!</b>

<br>
<p align="center">
  <img src="images/gifs/createCluster.gif">
</p>
<br>

* After your kubernetes cluster is up and running, you can deploy your IBM Blockchain Platform V2 Beta on the cluster.  The service walks through few steps and finds your cluster on the IBM Cloud to deploy the service on.

<br>
<p align="center">
  <img src="images/gifs/deploy-blockchain-on-cluster.gif">
</p>
<br>

* Once the Blockchain Platform is deployed on the Kubernetes cluster, you can launch the console to start operating on your blockchain network.

<br>
<p align="center">
  <img src="images/gifs/launch-ibm-blockchain.gif">
</p>
<br>

## Step 2. Build a network - Certificate Authority

We will build a network as provided by the IBM Blockchain Platform [documentation](https://console.bluemix.net/docs/services/blockchain/howto/ibp-console-build-network.html#ibp-console-build-network).  This will include creating a channel with a single peer organization with its own MSP and CA (Certificate Authority), and an orderer organization with its own MSP and CA. We will create the respective identities to deploy peers and operate nodes.

* #### Create your insurance organization CA 
  - Click <b>Add Certificate Authority</b>.
  - Click <b>IBM Cloud</b> under <b>Create Certificate Authority</b> and <b>Next</b>.
  - Give it a <b>Display name</b> of `Insurance CA`.  
  - Specify an <b>Admin ID</b> of `admin` and <b>Admin Secret</b> of `adminpw`.

<br>
<p align="center">
  <img src="images/gifs/insuranceIBP2.gif">
</p>
<br>

* #### Create your shop organization CA (process is same as shown in gif above)
  - Click <b>Add Certificate Authority</b>.
  - Click <b>IBM Cloud</b> under <b>Create Certificate Authority</b> and <b>Next</b>.
  - Give it a <b>Display name</b> of `Shop CA`.  
  - Specify an <b>Admin ID</b> of `admin` and <b>Admin Secret</b> of `adminpw`.

* #### Create your repair shop organization CA (process is same as shown in gif above)
  - Click <b>Add Certificate Authority</b>.
  - Click <b>IBM Cloud</b> under <b>Create Certificate Authority</b> and <b>Next</b>.
  - Give it a <b>Display name</b> of `Repair Shop CA`.  
  - Specify an <b>Admin ID</b> of `admin` and <b>Admin Secret</b> of `adminpw`.

* #### Create your police organization CA (process is same as shown in gif above)
  - Click <b>Add Certificate Authority</b>.
  - Click <b>IBM Cloud</b> under <b>Create Certificate Authority</b> and <b>Next</b>.
  - Give it a <b>Display name</b> of `Police CA`.  
  - Specify an <b>Admin ID</b> of `admin` and <b>Admin Secret</b> of `adminpw`.


* #### Use your CA to register insurance identities
  - Select the <b>Insurance CA</b> Certificate Authority that we created.
  - First, we will register an admin for our Insurance Organization. Click on the <b>Register User</b> button.  Give an <b>Enroll ID</b> of `insuranceAdmin`, and <b>Enroll Secret</b> of `insuranceAdminpw`.  Click <b>Next</b>.  Set the <b>Type</b> for this identity as `client` and select `org1` from the affiliated organizations drop-down list. We will leave the <b>Maximum enrollments</b> and <b>Add Attributes</b> fields blank.
  - We will repeat the process to create an identity of the peer. Click on the <b>Register User</b> button.  Give an <b>Enroll ID</b> of `insurancePeer`, and <b>Enroll Secret</b> of `insurancePeerpw`.  Click <b>Next</b>.  Set the <b>Type</b> for this identity as `peer` and select `org1` from the affiliated organizations drop-down list. We will leave the <b>Maximum enrollments</b> and <b>Add Attributes</b> fields blank.

<br>
<p align="center">
  <img src="images/gifs/registerInsuranceAdminPeer.gif">
</p>
<br>

* #### Use your CA to register shop identities (process is same as shown in gif above)
  - Select the <b>Shop CA</b> Certificate Authority that we created.
  - First, we will register an admin for our Insurance Organization. Click on the <b>Register User</b> button.  Give an <b>Enroll ID</b> of `shopAdmin`, and <b>Enroll Secret</b> of `shopAdminpw`.  Click <b>Next</b>.  Set the <b>Type</b> for this identity as `client` and select `org1` from the affiliated organizations drop-down list. We will leave the <b>Maximum enrollments</b> and <b>Add Attributes</b> fields blank.
  - We will repeat the process to create an identity of the peer. Click on the <b>Register User</b> button.  Give an <b>Enroll ID</b> of `shopPeer`, and <b>Enroll Secret</b> of `shopPeerpw`.  Click <b>Next</b>.  Set the <b>Type</b> for this identity as `peer` and select `org1` from the affiliated organizations drop-down list. We will leave the <b>Maximum enrollments</b> and <b>Add Attributes</b> fields blank.

* #### Use your CA to register repair shop identities (process is same as shown in gif above)
  - Select the <b>Repair Shop CA</b> Certificate Authority that we created.
  - First, we will register an admin for our Insurance Organization. Click on the <b>Register User</b> button.  Give an <b>Enroll ID</b> of `repairShopAdmin`, and <b>Enroll Secret</b> of `repairShopAdminpw`.  Click <b>Next</b>.  Set the <b>Type</b> for this identity as `client` and select `org1` from the affiliated organizations drop-down list. We will leave the <b>Maximum enrollments</b> and <b>Add Attributes</b> fields blank.
  - We will repeat the process to create an identity of the peer. Click on the <b>Register User</b> button.  Give an <b>Enroll ID</b> of `repairShopPeer`, and <b>Enroll Secret</b> of `repairShopPeerpw`.  Click <b>Next</b>.  Set the <b>Type</b> for this identity as `peer` and select `org1` from the affiliated organizations drop-down list. We will leave the <b>Maximum enrollments</b> and <b>Add Attributes</b> fields blank.

* #### Use your CA to register police shop identities (process is same as shown in gif above)
  - Select the <b>Police CA</b> Certificate Authority that we created.
  - First, we will register an admin for our Insurance Organization. Click on the <b>Register User</b> button.  Give an <b>Enroll ID</b> of `policeAdmin`, and <b>Enroll Secret</b> of `policeAdminpw`.  Click <b>Next</b>.  Set the <b>Type</b> for this identity as `client` and select `org1` from the affiliated organizations drop-down list. We will leave the <b>Maximum enrollments</b> and <b>Add Attributes</b> fields blank.
  - We will repeat the process to create an identity of the peer. Click on the <b>Register User</b> button.  Give an <b>Enroll ID</b> of `policePeer`, and <b>Enroll Secret</b> of `policePeerpw`.  Click <b>Next</b>.  Set the <b>Type</b> for this identity as `peer` and select `org1` from the affiliated organizations drop-down list. We will leave the <b>Maximum enrollments</b> and <b>Add Attributes</b> fields blank.


## Step 3. Build a network - Create MSP Definitions

* #### Create the insurance MSP definition
  - Navigate to the <b>Organizations</b> tab in the left navigation and click <b>Create MSP definition</b>.
  - Enter the <b>MSP Display name</b> as `Insurance MSP` and an <b>MSP ID</b> of `insurancemsp`.
  - Under <b>Root Certificate Authority</b> details, specify the peer CA that we created `Insurance CA` as the root CA for the organization.
  - Give the <b>Enroll ID</b> and <b>Enroll secret</b> for your organization admin, `insuranceAdmin` and `insuranceAdminpw`. Then, give the Identity name, `Insurance Admin`.
  - Click the <b>Generate</b> button to enroll this identity as the admin of your organization and export the identity to the wallet. Click <b>Export</b> to export the admin certificates to your file system. Finally click <b>Create MSP definition</b>.

<br>
<p align="center">
  <img src="images/gifs/insuranceMSP.gif">
</p>
<br>

* #### Create the shop MSP definition (same process as shown in gif above)
  - Navigate to the <b>Organizations</b> tab in the left navigation and click <b>Create MSP definition</b>.
  - Enter the <b>MSP Display name</b> as `Shop MSP` and an <b>MSP ID</b> of `shopmsp`.
  - Under <b>Root Certificate Authority</b> details, specify the peer CA that we created `Shop CA` as the root CA for the organization.
  - Give the <b>Enroll ID</b> and <b>Enroll secret</b> for your organization admin, `shopAdmin` and `shopAdminpw`. Then, give the Identity name, `Shop Admin`.
  - Click the <b>Generate</b> button to enroll this identity as the admin of your organization and export the identity to the wallet. Click <b>Export</b> to export the admin certificates to your file system. Finally click <b>Create MSP definition</b>.

* #### Create the repair shop MSP definition (same process as shown in gif above)
  - Navigate to the <b>Organizations</b> tab in the left navigation and click <b>Create MSP definition</b>.
  - Enter the <b>MSP Display name</b> as `Repair Shop MSP` and an <b>MSP ID</b> of `repairshopmsp`.
  - Under <b>Root Certificate Authority</b> details, specify the peer CA that we created `Repair Shop CA` as the root CA for the organization.
  - Give the <b>Enroll ID</b> and <b>Enroll secret</b> for your organization admin, `repairShopAdmin` and `repairShopAdminpw`. Then, give the Identity name, `Repair Shop Admin`.
  - Click the <b>Generate</b> button to enroll this identity as the admin of your organization and export the identity to the wallet. Click <b>Export</b> to export the admin certificates to your file system. Finally click <b>Create MSP definition</b>.

* #### Create the police MSP definition (same process as shown in gif above)
  - Navigate to the <b>Organizations</b> tab in the left navigation and click <b>Create MSP definition</b>.
  - Enter the <b>MSP Display name</b> as `Police MSP` and an <b>MSP ID</b> of `policemsp`.
  - Under <b>Root Certificate Authority</b> details, specify the peer CA that we created `Police CA` as the root CA for the organization.
  - Give the <b>Enroll ID</b> and <b>Enroll secret</b> for your organization admin, `policeAdmin` and `policeAdminpw`. Then, give the Identity name, `Police Admin`.
  - Click the <b>Generate</b> button to enroll this identity as the admin of your organization and export the identity to the wallet. Click <b>Export</b> to export the admin certificates to your file system. Finally click <b>Create MSP definition</b>.


## Step 4. Build a network - Create Peers

* Create an insurance peer
  - On the <b>Nodes</b> page, click <b>Add peer</b>.
  - Click <b>IBM Cloud</b> under Create a new peer and <b>Next</b>.
  - Give your peer a <b>Display name</b> of `Insurance Peer`.
  - On the next screen, select `Insurance CA` as your <b>Certificate Authority</b>. Then, give the <b>Enroll ID</b> and <b>Enroll secret</b> for the peer identity that you created for your peer, `insurancePeer`, and `insurancePeerpw`. Then, select the <b>Administrator Certificate (from MSP)</b>, `Insurance MSP`, from the drop-down list and click <b>Next</b>.
  - Give the <b>TLS Enroll ID</b>, `admin`, and <b>TLS Enroll secret</b>, `adminpw`, the same values are the Enroll ID and Enroll secret that you gave when creating the CA.  Leave the <b>TLS CSR hostname</b> blank.
  - The last side panel will ask you to <b>Associate an identity</b> and make it the admin of your peer. Select your peer admin identity `Insurance Admin`.
  - Review the summary and click <b>Submit</b>.

<br>
<p align="center">
  <img src="images/gifs/addInsurancePeer.gif">
</p>
<br>
 
* Create a shop peer (same process as shown in gif above)
  - On the <b>Nodes</b> page, click <b>Add peer</b>.
  - Click <b>IBM Cloud</b> under Create a new peer and <b>Next</b>.
  - Give your peer a <b>Display name</b> of `Shop Peer`.
  - On the next screen, select `Shop CA` as your <b>Certificate Authority</b>. Then, give the <b>Enroll ID</b> and <b>Enroll secret</b> for the peer identity that you created for your peer, `shopPeer`, and `shopPeerpw`. Then, select the <b>Administrator Certificate (from MSP)</b>, `Shop MSP`, from the drop-down list and click <b>Next</b>.
  - Give the <b>TLS Enroll ID</b>, `admin`, and <b>TLS Enroll secret</b>, `adminpw`, the same values are the Enroll ID and Enroll secret that you gave when creating the CA.  Leave the <b>TLS CSR hostname</b> blank.
  - The last side panel will ask you to <b>Associate an identity</b> and make it the admin of your peer. Select your peer admin identity `Shop Admin`.
  - Review the summary and click <b>Submit</b>.

* Create a repair shop peer (same process as shown in gif above)
  - On the <b>Nodes</b> page, click <b>Add peer</b>.
  - Click <b>IBM Cloud</b> under Create a new peer and <b>Next</b>.
  - Give your peer a <b>Display name</b> of `Repair Shop Peer`.
  - On the next screen, select `Repair Shop CA` as your <b>Certificate Authority</b>. Then, give the <b>Enroll ID</b> and <b>Enroll secret</b> for the peer identity that you created for your peer, `repairShopPeer`, and `repairShopPeerpw`. Then, select the <b>Administrator Certificate (from MSP)</b>, `Repair Shop MSP`, from the drop-down list and click <b>Next</b>.
  - Give the <b>TLS Enroll ID</b>, `admin`, and <b>TLS Enroll secret</b>, `adminpw`, the same values are the Enroll ID and Enroll secret that you gave when creating the CA.  Leave the <b>TLS CSR hostname</b> blank.
  - The last side panel will ask you to <b>Associate an identity</b> and make it the admin of your peer. Select your peer admin identity `Repair Shop Admin`.
  - Review the summary and click <b>Submit</b>.

* Create a police peer (same process as shown in gif above)
  - On the <b>Nodes</b> page, click <b>Add peer</b>.
  - Click <b>IBM Cloud</b> under Create a new peer and <b>Next</b>.
  - Give your peer a <b>Display name</b> of `Police Peer`.
  - On the next screen, select `Police CA` as your <b>Certificate Authority</b>. Then, give the <b>Enroll ID</b> and <b>Enroll secret</b> for the peer identity that you created for your peer, `policePeer`, and `policePeerpw`. Then, select the <b>Administrator Certificate (from MSP)</b>, `Police MSP`, from the drop-down list and click <b>Next</b>.
  - Give the <b>TLS Enroll ID</b>, `admin`, and <b>TLS Enroll secret</b>, `adminpw`, the same values are the Enroll ID and Enroll secret that you gave when creating the CA.  Leave the <b>TLS CSR hostname</b> blank.
  - The last side panel will ask you to <b>Associate an identity</b> and make it the admin of your peer. Select your peer admin identity `Police Admin`.
  - Review the summary and click <b>Submit</b>.

## Step 5. Build a network - Create Orderer

* #### Create your orderer organization CA
  - Click <b>Add Certificate Authority</b>.
  - Click <b>IBM Cloud</b> under <b>Create Certificate Authority</b> and <b>Next</b>.
  - Give it a unique <b>Display name</b> of `Orderer CA`.  
  - Specify an <b>Admin ID</b> of `admin` and <b>Admin Secret</b> of `adminpw`.

<br>
<p align="center">
  <img src="images/gifs/createOrderer.gif">
</p>
<br>

* #### Use your CA to register orderer and orderer admin identities (shown in gif above)
  - In the <b>Nodes</b> tab, select the <b>Orderer CA</b> Certificate Authority that we created.
  - First, we will register an admin for our organization. Click on the <b>Register User</b> button.  Give an <b>Enroll ID</b> of `ordereradmin`, and <b>Enroll Secret</b> of `ordereradminpw`.  Click <b>Next</b>.  Set the <b>Type</b> for this identity as `client` and select `org1` from the affiliated organizations drop-down list. We will leave the <b>Maximum enrollments</b> and <b>Add Attributes</b> fields blank.
  - We will repeat the process to create an identity of the orderer. Click on the <b>Register User</b> button.  Give an <b>Enroll ID</b> of `orderer1`, and <b>Enroll Secret</b> of `orderer1pw`.  Click <b>Next</b>.  Set the <b>Type</b> for this identity as `peer` and select `org1` from the affiliated organizations drop-down list. We will leave the <b>Maximum enrollments</b> and <b>Add Attributes</b> fields blank.

* #### Create the orderer organization MSP definition
  - Navigate to the <b>Organizations</b> tab in the left navigation and click <b>Create MSP definition</b>.
  - Enter the <b>MSP Display name</b> as `Orderer MSP` and an <b>MSP ID</b> of `orderermsp`.
  - Under <b>Root Certificate Authority</b> details, specify the peer CA that we created `Orderer CA` as the root CA for the organization.
  - Give the <b>Enroll ID</b> and <b>Enroll secret</b> for your organization admin, `ordereradmin` and `ordereradminpw`. Then, give the <b>Identity name</b>, `Orderer Admin`.
  - Click the <b>Generate</b> button to enroll this identity as the admin of your organization and export the identity to the wallet. Click <b>Export</b> to export the admin certificates to your file system. Finally click <b>Create MSP definition</b>.

<br>
<p align="center">
  <img src="images/gifs/ordererMSP.gif">
</p>
<br>

* #### Create an orderer
  - On the <b>Nodes</b> page, click <b>Add orderer</b>.
  - Click <b>IBM Cloud</b> and proceed with <b>Next</b>.
  - Give your peer a <b>Display name</b> of `Orderer`.
  - On the next screen, select `Orderer CA` as your <b>Certificate Authority</b>. Then, give the <b>Enroll ID</b> and <b>Enroll secret</b> for the peer identity that you created for your orderer, `orderer1`, and `orderer1pw`. Then, select the <b>Administrator Certificate (from MSP)</b>, `Orderer MSP`, from the drop-down list and click <b>Next</b>.
  - Give the <b>TLS Enroll ID</b>, `admin`, and <b>TLS Enroll secret</b>, `adminpw`, the same values are the Enroll ID and Enroll secret that you gave when creating the CA.  Leave the <b>TLS CSR hostname</b> blank.
  - The last side panel will ask to <b>Associate an identity</b> and make it the admin of your peer. Select your peer admin identity `Orderer Admin`.
  - Review the summary and click <b>Submit</b>.

<br>
<p align="center">
  <img src="images/gifs/createOrdererNode.gif">
</p>
<br>

* #### Add organizations as Consortium Member on the orderer to transact
  - Navigate to the <b>Nodes</b> tab, and click on the <b>Orderer</b> that we created.
  - Under <b>Consortium Members</b>, click <b>Add organization</b>.
  - From the drop-down list, select `Insurance MSP`.
  - Click <b>Submit</b>.
  - Repeat the same steps, but add `Shop MSP`, `Repair Shop MSP`, and `Police MSP` as well.

<br>
<p align="center">
  <img src="images/gifs/addConsortium.gif">
</p>
<br>

## Step 6. Build a network - Create and Join Channel

* #### Create the channel
  - Navigate to the <b>Channels</b> tab in the left navigation.
  - Click <b>Create channel</b>.
  - Give the channel a name, `mychannel`.
  - Select the orderer you created, `Orderer` from the orderers drop-down list.
  - Select the MSP identifying the organization of the channel creator from the drop-down list. This should be `Insurance MSP (insurancemsp)`.
  - Associate available identity as `Insurance Admin`.
  - Click <b>Add</b> next to the insurance organization. Make the insurance organization an <b>Operator</b>.
  - Click <b>Add</b> next to the shop organization. Make the shop organization an <b>Operator</b>.
  - Click <b>Add</b> next to the repair shop organization. Make the repair shop organization an <b>Operator</b>.
  - Click <b>Add</b> next to the police organization. Make the insurance organpoliceization an <b>Operator</b>.
  - Click <b>Create</b>.

<br>
<p align="center">
  <img src="images/gifs/createChannel.gif">
</p>
<br>


* #### Join your peer to the channel
  - Click <b>Join channel</b> to launch the side panels.
  - Select your `Orderer` and click <b>Next</b>.
  - Enter the name of the channel you just created. `mychannel` and click <b>Next</b>.
  - Select which peers you want to join the channel, click `Insurance Peer`, `Shop Peer`, `Repair Shop Peer`, and `Police Peer`.
  - Click <b>Submit</b>.

<br>
<p align="center">
  <img src="images/gifs/joinChannel.gif">
</p>
<br>

* #### Add anchor peers to the channel
  - In order to communicate between organizations, we need to enroll anchor peers.
  - From the channels tab, click on the channel you have created, `mychannel`.
  - From the channel overview page, click on `channel details`. Scroll all the way down until you see `Anchor peers`.
  - Click `Add anchor peer` and add the Insurance, Police, Repair Shop,
    and Shop peers.
  - Select which peers you want to join the channel, click `Insurance Peer`, `Shop Peer`, `Repair Shop Peer`, and `Police Peer`.
  - Click <b>Add anchor peer</b>.
  - If all went well, your channel Anchor peers should look like below:

<br>
<p align="center">
  <img src="images/gifs/addAnchorPeer.gif">
</p>
<br>

## Step 7. Deploy Insurance Smart Contract on the network

* #### Install a smart contract
* Clone the repository:
  ```bash
  git clone https://github.com/IBM/build-blockchain-insurance-app
  ```
  - Click the <b>Smart contracts</b> tab to install the smart contract.
  - Click <b>Install smart contract</b> to upload the insurance smart contract package file.
  - Click on <b>Add file</b> and find your packaged smart contract. It is the file in the `build-blockchain-insurance-app/chaincodePackage` directory. 
  - Select all peers - we need to install the contract on each peer.
  - Once the contract is uploaded, click <b>Install</b>.


<br>
<p align="center">
  <img src="images/gifs/installSmartContract.gif">
</p>
<br>

* #### Instantiate smart contract
  - On the smart contracts tab, find the smart contract from the list installed on your peers and click <b>Instantiate</b> from the overflow menu on the right side of the row.
  - On the side panel that opens, select the channel, `mychannel` to instantiate the smart contract on. Click <b>Next</b>.
  - Select the organization members to be included in the policy, `insurancemsp`, `shopmsp`, `repairshopmsp`, `policemsp`.  Click <b>Next</b>.
  - Give <b>Function name</b> of `Init` and leave <b>Arguments</b> blank.
  - Click <b>Instantiate</b>.

<br>
<p align="center">
  <img src="images/gifs/instantiateContract.gif">
</p>
<br>

## Step 8. Connect application to the network

* #### Connect with sdk through connection profile
  - Under the Instantiated Smart Contract, click on `Connect with SDK` from the overflow menu on the right side of the row.
  - Choose from the dropdown for <b>MSP for connection</b>, `insurancemsp`.
  - Choose from <b>Certificate Authority</b> dropdown, `Insurance CA`.
  - Download the connection profile by scrolling down and clicking <b>Download Connection Profile</b>.  This will download the connection json which we will use soon to establish connection.
  - You can click <b>Close</b> once the download completes.

<br>
<p align="center">
  <img src="images/gifs/downloadConnection.gif">
</p>
<br>

* #### Create insurance application admin
  - Go to the <b>Nodes</b> tab on the left bar, and under <b>Certificate Authorities</b>, choose your <b>Insurance CA</b>.
  - Click on <b>Register user</b>.
  - Give an <b>Enroll ID</b> and <b>Enroll Secret</b> to administer your application users, `insuranceApp-admin` and `insuranceApp-adminpw`.
  - Choose `client` as <b>Type</b>.
  - You can leave the <b>Use root affiliation</b> box checked.
  - You can leave the <b>Maximum enrollments</b> blank.
  - Under <b>Attributes</b>, click on <b>Add attribute</b>.  Give attribute as `hf.Registrar.Roles` = `*`.  This will allow this identity to act as registrar and issues identities for our app.  Click <b>Add-attribute</b>.
  - Click <b>Register</b>.

<br>
<p align="center">
  <img src="images/gifs/appAdmin.gif">
</p>
<br>

* #### Create shop application admin (same process as shown above in the gif)
  - Go to the <b>Nodes</b> tab on the left bar, and under <b>Certificate Authorities</b>, choose your <b>Shop CA</b>.
  - Click on <b>Register user</b>.
  - Give an <b>Enroll ID</b> and <b>Enroll Secret</b> to administer your application users, `shopApp-admin` and `shopApp-adminpw`.
  - Choose `client` as <b>Type</b>.
  - You can leave the <b>Use root affiliation</b> box checked.
  - You can leave the <b>Maximum enrollments</b> blank.
  - Under <b>Attributes</b>, click on <b>Add attribute</b>.  Give attribute as `hf.Registrar.Roles` = `*`.  This will allow this identity to act as registrar and issues identities for our app.  Click <b>Add-attribute</b>.
  - Click <b>Register</b>.

* #### Create repair shop application admin (same process as shown above in the gif)
  - Go to the <b>Nodes</b> tab on the left bar, and under <b>Certificate Authorities</b>, choose your <b>Repair Shop CA</b>.
  - Click on <b>Register user</b>.
  - Give an <b>Enroll ID</b> and <b>Enroll Secret</b> to administer your application users, `repairShopApp-admin` and `repairShopApp-adminpw`.
  - Choose `client` as <b>Type</b>.
  - You can leave the <b>Use root affiliation</b> box checked.
  - You can leave the <b>Maximum enrollments</b> blank.
  - Under <b>Attributes</b>, click on <b>Add attribute</b>.  Give attribute as `hf.Registrar.Roles` = `*`.  This will allow this identity to act as registrar and issues identities for our app.  Click <b>Add-attribute</b>.
  - Click <b>Register</b>.

* #### Create police application admin (same process as shown above in the gif)
  - Go to the <b>Nodes</b> tab on the left bar, and under <b>Certificate Authorities</b>, choose your <b>Police CA</b>.
  - Click on <b>Register user</b>.
  - Give an <b>Enroll ID</b> and <b>Enroll Secret</b> to administer your application users, `policeApp-admin` and `policeApp-adminpw`.
  - Choose `client` as <b>Type</b>.
  - You can leave the <b>Use root affiliation</b> box checked.
  - You can leave the <b>Maximum enrollments</b> blank.
  - Under <b>Attributes</b>, click on <b>Add attribute</b>.  Give attribute as `hf.Registrar.Roles` = `*`.  This will allow this identity to act as registrar and issues identities for our app.  Click <b>Add-attribute</b>.
  - Click <b>Register</b>.

 #### Update application connection
  - Copy the connection profile you downloaded into the `web/www/blockchain` directory.
  - Copy and paste everything in the connection profile, and overwrite
  the **ibpConnection.json**. 

  <p align="center">
    <img src="images/gifs/ibpConnection.gif">
  </p>

## Step 9. Enroll App Admin Identities

* #### Enroll insurnaceApp-admin
  - First, navigate to the `web/www/blockchain` directory.
    ```bash
    cd web/www/blockchain/
    ```
  - Open the `config.json` file, and update the caName with the URL 
    of the <b>insurance</b> certificate authority from your `ibpConnection.json` file. Save the file.  
  
  - Run the `enrollAdmin.js` script
    ```bash
    node enrollAdmin.js
    ```

  - You should see the following in the terminal:
    ```bash
    msg: Successfully enrolled admin user insuranceApp-admin and imported it into the wallet
    ```
<p align="center">
  <img src="images/gifs/enrollAdmin.gif">
</p>

* Enroll shopApp-admin
  - First, change the appAdmin, appAdminSecret, and caName properties in your `config.json` file, 
  so that it looks something like this (your caName should be different than mine):

    ```js
    {
        "connection_file": "ibpConnection.json",
        "appAdmin": "shopApp-admin",
        "appAdminSecret": "shopApp-adminpw",
        "orgMSPID": "shopmsp",
        "caName": "https://fa707c454921423c80ec3c3c38d7545c-caf2e287.horeainsurancetest.us-south.containers.appdomain.cloud:7054",
        "userName": "shopUser",
        "gatewayDiscovery": { "enabled": true, "asLocalhost": false }
    }
    ```
  - To find the other CA urls, you will need to click on the `Nodes` tab in IBM Blockchain Platform, then on 
    the Shop CA, and on the settings cog icon at the top of the page. That will take you to the certificate 
    authority settings, as shown in the picture below, and you can copy that endpoint URL into your `config.json` **caName**
    field. 

<p align="center">
  <img src="images/gifs/enrollShopAdmin.gif">
</p>  
<br>

  - Run the `enrollAdmin.js` script
    
    ```bash
    node enrollAdmin.js
    ```

  - You should see the following in the terminal:

    ```bash
    msg: Successfully enrolled admin user shopApp-admin and imported it into the wallet
    ```

* #### Enroll repairShopApp-admin (same process as shown in gif above)
  - First, change the appAdmin, appAdminSecret, and caName properties in your `config.json` file, 
  so that it looks something like this (your caName should be different than mine):

    ```js
    {
        "connection_file": "ibpConnection.json",
        "appAdmin": "repairShopApp-admin",
        "appAdminSecret": "repairShopApp-adminpw",
        "orgMSPID": "repairshopmsp",
        "caName": "https://fa707c454921423c80ec3c3c38d7545c-caf2e287.horeainsurancetest.us-south.containers.appdomain.cloud:7054",
        "userName": "repairUser",
        "gatewayDiscovery": { "enabled": true, "asLocalhost": false }
    }
    ```
  - Run the `enrollAdmin.js` script
      ```bash
      node enrollAdmin.js
      ```

  - You should see the following in the terminal:
    ```bash
    msg: Successfully enrolled admin user repairShopApp-admin and imported it into the wallet
    ```

* #### Enroll policeApp-admin (same process as shown in gif above)
  - First, change the appAdmin, appAdminSecret, and caName properties in your `config.json` file, 
  so that it looks something like this (your caName should be different than mine):

    ```js
    {
        "connection_file": "ibpConnection.json",
        "appAdmin": "policeApp-admin",
        "appAdminSecret": "policeApp-adminpw",
        "orgMSPID": "policemsp",
        "caName": "https://fa707c454921423c80ec3c3c38d7545c-caf2e287.horeainsurancetest.us-south.containers.appdomain.cloud:7054",
        "userName": "policeUser",
        "gatewayDiscovery": { "enabled": true, "asLocalhost": false }
    }
    ```

  - Run the `enrollAdmin.js` script
      ```bash
      node enrollAdmin.js
      ```

  - You should see the following in the terminal:
      ```bash
      msg: Successfully enrolled admin user policeApp-admin and imported it into the wallet
      ```
  
## Step 10. Run the application

Navigate to the directory blockchain directory which contains the [config.js file](https://github.com/IBM/build-blockchain-insurance-app/blob/ubuntu/local-fix/web/www/blockchain/config.js):
  ```bash
  cd build-blockchain-insurance-app/web/www/blockchain/
  ```

In the editor of choice, change [line 8](https://github.com/IBM/build-blockchain-insurance-app/blob/ubuntu/local-fix/web/www/blockchain/config.js#L8) of the `config.js` file to `isCloud: true` as 
shown in the image below: 

![Is Cloud](images/isCloud.png)

If you are using Mac, save the changes. Otherwise, if you are using an Ubuntu system, change
# License

This code pattern is licensed under the Apache Software License, Version 2.  Separate third-party code objects invoked within this code pattern are licensed by their respective providers pursuant to their own separate licenses. Contributions are subject to the [Developer Certificate of Origin, Version 1.1 (DCO)](https://developercertificate.org/) and the [Apache Software License, Version 2](http://www.apache.org/licenses/LICENSE-2.0.txt).

[Apache Software License (ASL) FAQ](http://www.apache.org/foundation/license-faq.html#WhatDoesItMEAN)
