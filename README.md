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

# Running the application

## Manually deploy to local machine
1. [Set up your machine](#1-set-up-your-machine)
2. [Create IBM cloud services](#2-create-ibm-cloud-services)
3. [Create a solution](#3-create-a-solution)
4. [Clone the repository](#4-clone-the-repository)
5. [Modify the configuration files](#5-modify-the-configuration-files)
6. [Run the application](#6-run-the-application)

### 1. Set up your machine

Install the following dependencies -

- [Docker](https://www.docker.com/): Go to the Docker website and download the installer. After installation, run Docker.
- [git](https://git-scm.com/): Install `git` which is a free and open source distributed version control system.

### 2. Create IBM cloud services

* Create the [IBM Cloud Kubernetes Service](https://cloud.ibm.com/catalog/infrastructure/containers-kubernetes).  You can find the service in the `Catalog`. For this code pattern, we can use the `Free` cluster, and give it a name.  Note, that the IBM Cloud allows one instance of a free cluster and expires after 30 days.

<br>
<p align="center">
  <img src="docs/doc-gifs/1.gif">
</p>
<br>

* Create two instances of [Databases for Redis Service](https://cloud.ibm.com/catalog/services/databases-for-redis).  You can find the service in the `Catalog`.

<br>
<p align="center">
  <img src="docs/doc-gifs/2.gif">
</p>
<br>

  > Note: You can use just one instance of Redis as well. Modify the code in the server repository to allow for this.

* Create the [IBM Blockchain Service](https://cloud.ibm.com/catalog/services/ibm-blockchain-5-prod). You can find the service in the `Catalog`.

<br>
<p align="center">
  <img src="docs/doc-gifs/3.gif">
</p>
<br>

* Create the `Blockchain document store` and `Blockchain solution manager` services. These services are not currently available publicly on the `IBM cloud catalog`. You can reach out to `Rak-Joon Choi (rak-joon.choi@us.ibm.com)` to provision these services for you. Follow the service [documentation](https://cloud.ibm.com/docs/services/blockchain-document-store?topic=blockchain-document-store-getting-started#getting-started) to connect the `Blockchain document store` to the `Blockchain service`.

<br>
<p align="center">
  <img src="docs/doc-gifs/4.gif">
</p>
<br>

### 3. Create a solution

* After configuring your services in the previous step, we now move on to creating a solution using our custom swagger url for the `blockchain solution manager` service. Go to the `Patch endpoint (/v1/solutions)` under `Solution` and authorize using the api by going to the `/v1/logins` url in a new tab, logging in as `Administrator`, and getting the JWT. Add the token prepended by `bearer` such that it looks like `bearer <JWT>`. After authorization, click on `try it out` to execute the api, and paste the following JSON in the `on-boarding` section. Give the name `medrec_demo` to the solution.

<br>
<p align="center">
  <img src="docs/doc-gifs/5.gif">
</p>
<br>

* After creating the solution successfully, add yourself as the admin of the solution. Go to the `Post endpoint (/v1/solutions/{solutionId}/administrators)` under `Solution` and authorize using the api by going to the `/v1/logins` url in a new tab, logging in as `Administrator`, and getting the JWT. Add the token prepended by `bearer` such that it looks like `bearer <JWT>`. After authorization, click on `try it out` to execute the api, and type your email id under `solutionAdministrators` in the JSON object. Provide `medrec_demo` as the `solutionId`.

<br>
<p align="center">
  <img src="docs/doc-gifs/6.gif">
</p>
<br>

### 4. Clone the repository

```
git clone https://github.com/ash7594/private-data-collections-on-fabric.git
cd private-data-collections-on-fabric
```

### 5. Modify the configuration files

* Modify the redis config file:
  - Go to the previously provisioned redis services on IBM Cloud.
  - Click on `Service credentials`.
  - Click on `New credential` button.
  - Once the new credentials are created, click on `view credentials`.
  - From the JSON object, extract the URI from `connection.rediss.composed[0]`.
  - From the JSON object, extract the certificate from `connection.rediss.certificate.certificate_base64`.
  - Navigate to the `server/config.json` file in the cloned repository.
  - Replace the URI and certificate values in the marked places.
  - Repeat the steps for the second provisioned service, and enter it in the second spot in the config file.

<br>
<p align="center">
  <img src="docs/doc-gifs/7.gif">
</p>
<br>

* Modify the blockchain config file:
  - Go to the `/v1/logins` url for your blockchain document store service.
  - Login as administrator.
  - Extract the `iss` field from the decoded JWT and remove `/onboarding` string from it.
  - Navigate to the `src/secrets/config.json` file in the cloned repository.
  - Replace the `iss` field with the extracted value above.
  - Replace the `blockchain_channel` field with the name of the channel provided during connecting the blockchain service to the document store.

<br>
<p align="center">
  <img src="docs/doc-gifs/8.gif">
</p>
<br>

### 6. Run the application

* Running the application locally:
  - To run the application on the local system, execute the `run-application.sh` file.
  - Go to `localhost:8080` to see the running application.

<br>
<p align="center">
  <img src="docs/doc-gifs/9.gif">
</p>
<br>

* Running the application on kubernetes:
  - Navigate to server directory - `cd server`.
  - Build the docker image for the server - `docker build -t <DOCKERHUB_USERNAME>/medrec-server .`
  - Replace the image name in `manifest.yml`, where indicated.
  - Apply the manifest to the previously provisioned kubernetes cluster.
  - Navigate to `/src/apis/RedisApi.js` and replace the `baseURL` value with the Kubernetes load balancer IP.
  - Build and run the Vue application by executing the below in the repository home.
  - Go to `localhost:8080` to see the running application.

```
docker build -t medrec-vue .
docker run -d --restart always --name medrec-vue -p 8080:8080 medrec-vue
```

> Note: You can also deploy the Vue App to Kubernetes, by modifying the manifest.yml to support two pods.

# License

This code pattern is licensed under the Apache Software License, Version 2.  Separate third-party code objects invoked within this code pattern are licensed by their respective providers pursuant to their own separate licenses. Contributions are subject to the [Developer Certificate of Origin, Version 1.1 (DCO)](https://developercertificate.org/) and the [Apache Software License, Version 2](http://www.apache.org/licenses/LICENSE-2.0.txt).

[Apache Software License (ASL) FAQ](http://www.apache.org/foundation/license-faq.html#WhatDoesItMEAN)
