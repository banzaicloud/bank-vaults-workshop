# bank-vaults-workshop

## Infrastructure Setup

You can use the Kubernetes cluster of your choice, this guide walks you through
to use the one inside a Vagrantbox distributed on the pendrives.

1. Copy the content of the pendrive to your workstation
2. Open up a terminal and enter that directory where you copied the content.
3. Install the `vbguest` Vagrant plugin with:

    `vagrant plugin install vagrant-vbguest`

4. Add the Vagrant Box:

    `vagrant box add --name banzaibox banzaibox.box`

5. Start up the workshop VM with Vagrant:

    `vagrant up`

6. There are two ways to do this (work in the VM or outside the VM):
  - a. External: setup `kubectl` to point to the Kubernetes cluster inside the VM:

    `export KUBECONFIG=$PWD/kubeconfig.yaml`

  - b. Internal: SSH into the VM:

    `vagrant ssh`
    
    In this case the examples are mounted to `/vagrant/bank-vaults`

## Banzai Vault Operator and Webhook install

1. Add the `Banzai Cloud` Helm chart repository:

    `helm repo add banzaicloud-stable https://kubernetes-charts.banzaicloud.com/`

2. Install the `Banzai Cloud Vault Operator` into the `vault-infra` namespace:

     `helm upgrade --install vault-infra banzaicloud-stable/vault-operator --namespace vault-infra`

3. Install the `Vault Secrets Webhook` also to the `vault-infra` namespace:

     `helm upgrade --install vault-secrets-webhook banzaicloud-stable/vault-secrets-webhook --namespace vault-infra`

4. Check that everything has been installed correctly:

     `kubectl get pods -w -n vault-infra`


At this point we have a fully working Vault provisioner operator and the secret injecting webhook installed!

## Installing Vault

1. Checkout the workshop repository:

    ```
    git clone git@github.com:banzaicloud/bank-vaults-workshop.git
    cd bank-vaults-workshop
    ```

3. Now can deploy a Vault instance on top of Kubernetes:

    1. Create the RBAC rules for the Vault instance to work properly:

        `kubectl apply -f 01-vault/01-rbac.yaml`

    2. Install the Vault instance itself:

        `kubectl apply -f 01-vault/02-vault.yaml`

    3. Check that all Vault related Pods have been started:

        `kubectl get pods -w`

4. Get the Vault root token, we will use it later on ;)

    `kubectl get secrets vault-unseal-keys -o jsonpath={.data.vault-root} | base64 --decode`

    NOTE on how to access this Vault instance:

        ```
        kubectl port-forward statefulset/vault 8200 &
        export VAULT_ADDR="https://127.0.0.1:8200"
        export VAULT_TOKEN="the token from above"
        vault secrets list
        ```

## Inject some secrets from Vault!


1. Inject secrets into a Pod:

    `kubectl apply -f 02-examples/01-deployment.yaml`
    
    `kubectl get pods -w`

    `kubectl describe pod -l app.kubernetes.io/name=hello-secrets`
    
    `kubectl logs -f -l app.kubernetes.io/name=hello-secrets`
