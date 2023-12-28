---
description: Install Hyperswitch on K8s using Helm
---

# Deploy on Kubernetes using Helm

This section outlines cloud-provider agnostic deployment steps for easy installation of the Hyperswitch stack on your K8s cluster

## Prerequisites

1. Active Redis service
2. Create a Postgres database and run the schema migration using the below commands

```bash
git clone https://github.com/juspay/hyperswitch.git
diesel migration --database-url postgres://{{user}}:{{password}}@{{host_name}}:5432/{{db_name}} run
```

## Installation

### Step 1 - Clone repo and Update Configurations

Clone the [hyperswitch-helm](https://github.com/juspay/hyperswitch-helm) repo and start updating the configs

```
git clone https://github.com/juspay/hyperswitch-helm.git
cd hyperswitch-helm
```

To deploy the Helm chart, you need to update following values for each service in `values.yaml`

<table><thead><tr><th width="140.33333333333331">Service</th><th width="298">Configuration Key</th><th>Description</th></tr></thead><tbody><tr><td>App Server</td><td><code>application.server.server_base_url</code></td><td>Set to the hostname of your Hyperswitch backend for redirection scenarios.</td></tr><tr><td></td><td><code>application.server.secrets.admin_api_key</code></td><td>Used for all admin operations. Replace <code>"admin_api_key"</code> with your actual admin API key.</td></tr><tr><td></td><td><code>application.server.locker.host</code></td><td><a href="https://opensource.hyperswitch.io/going-live/pci-compliance/card-vault-installation">Card Vault</a> Hostname</td></tr><tr><td></td><td><code>redis.host</code></td><td>Hostname of your redis service. it should run in default port 6379</td></tr><tr><td></td><td><code>db.name</code></td><td>Postgres Database name.</td></tr><tr><td></td><td><code>db.host</code></td><td>Database Host name</td></tr><tr><td></td><td><code>db.user_name</code></td><td>Database username</td></tr><tr><td></td><td><code>db.password</code></td><td>Database password</td></tr><tr><td>Control Center</td><td><code>application.dashboard.env.apiBaseUrl</code></td><td>Set to the hostname of your Hyperswitch backend, so that Control center can access the Hyperswitch backend.</td></tr><tr><td></td><td><code>application.dashboard.env.sdkBaseUrl</code></td><td>Set to the URL of your hosted Hyperloader, so that you can test Hyperswitch Web SDK in the Control Center.<br>Eg: https://{{your_host}}/0.5.6/v0/HyperLoader.js</td></tr><tr><td>Hyperswitch Demo Store</td><td></td><td></td></tr><tr><td></td><td></td><td></td></tr></tbody></table>

### Step 2 - Install Hyperswitch

Use below command to install hyperswitch services with above configs

```bash
helm install hyperswitch-v1 . -n hyperswitch
```

{% hint style="success" %}
That's it! Hyperswitch should be up and running on your AWS account  :tada: :tada:
{% endhint %}

## Post-Deployment Checklist

After deploying the Helm chart, you should verify that everything is working correctly

### App Server

* [ ] &#x20;Check that `hyperswitch_server/health` returns `health is good`

### Control Center

* [ ] &#x20;Verify if you are able to sign in or sign up
* [ ] &#x20;Verify if you are able to [create API key](https://opensource.hyperswitch.io/run-hyperswitch-locally/account-setup/using-hyperswitch-control-center#user-content-create-an-api-key)
* [ ] &#x20;Verify if you are able to [configure a new payment processor](https://opensource.hyperswitch.io/run-hyperswitch-locally/account-setup/using-hyperswitch-control-center#add-a-payment-processor)

## Test a payment

Hyperswitch Demo store will mimic the behavior of your checkout page. Please follow below steps to initiate demo app

### Step 1 - Deploy card vault

### Card Vault Installation

If you intend to save cards of your customers for future usage then you need a Card Vault. This helm chart doesn't cover inbuilt card vault support as it will violate PCI compliance. You can install manually by following the steps [here](https://opensource.hyperswitch.io/going-live/pci-compliance/card-vault-installation) or use [this doc to deploy card vault in aws](https://opensource.hyperswitch.io/hyperswitch-open-source/deploy-hyperswitch-on-aws/deploy-card-vault)

### Step 2 - Configure below details again in your `values.yaml`

| Service                | Configuration Key                                   | Description                                                                                                          |
| ---------------------- | --------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Hyperswitch Demo Store | `application.sdkDemo.env.hyperswitchServerUrl`      | Set to the hostname of your Hyperswitch backend to access the Hyperswitch backend.                                   |
|                        | `application.sdkDemo.env.hyperSwitchClientUrl`      | <p>Set to the URL of your hosted Hyperloader to access the Hyperswitch SDK.<br>Eg:https://{{your_host}}/0.5.6/v0</p> |
|                        | `application.sdkDemo.env.hyperswitchPublishableKey` | This should be set to your merchant publishable key. You will get this once you create a merchant.                   |
|                        | `application.sdkDemo.env.hyperswitchSecretKey`      | This should be set to your merchant secret key. You can create this from the control center or via the REST API.     |

### Step 3 - Run helm upgrade to restart pods with updated config

```
helm upgrade --install hyperswitch-v1 . -n hyperswitch -f values.yaml
```

### Step 4 - Make a payment using our Demo App

Use the Hyperswitch Demo app and [make a payment with test card](https://opensource.hyperswitch.io/hyperswitch-open-source/test-a-payment).

Refer our [postman collection](https://www.postman.com/hyperswitch/workspace/hyperswitch/folder/25176183-0103918c-6611-459b-9faf-354dee8e4437) to try out REST APIs

### Get Repo Info
```bash
helm repo add hyperswitch-helm https://juspay.github.io/hyperswitch-helm
helm repo update
```
### Contribution guidelines
When you want others to use the changes you have added you need to package it and then index it
```bash
helm package .
helm repo index . --url https://juspay.github.io/hyperswitch-helm
```
