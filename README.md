# bank-vaults-workshop

## Setup

1. Copy the content of the Pendrive to your workstation
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

7. Add the `Banzai Cloud` Helm chart repository:

    `helm repo add banzaicloud-stable https://kubernetes-charts.banzaicloud.com/`

8. Install the `Banzai Cloud Vault Operator` into the `vault-infra` namespace:

     `helm upgrade --install vault-infra banzaicloud-stable/vault-operator --namespace vault-infra`

9. Install the `Vault Secrets Webhook` also to the `vault-infra` namespace:

     `helm upgrade --install vault-secrets-webhook banzaicloud-stable/vault-secrets-webhook --namespace vault-infra`
