# Ansible Squonk

[![Build Status](https://travis-ci.com/InformaticsMatters/squonk-ansible.svg?branch=master)](https://travis-ci.com/InformaticsMatters/squonk-ansible)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/informaticsmatters/squonk-ansible)

An Ansible role to deploy the Informatics Matters [Squonk] application
to [Kubernetes].

Ideally you'll start from a Python 3 virtual environment and then install
the required modules, roles and collections: -

>   Ansible prior to 2.9.1 will probably have issues with Python 3.8.
    So, base your environment on Python 3.7 and use Ansible 2.8.7
    (which is better because AWX runs Ansible 2.8, not 2.9).

    $ conda activate ansible-squonk
    $ pip install -r requirements.txt

>   [Ansible Galaxy] roles this project depends upon
    (those in the `requirements.yaml` file) are downloaded by this project's
    playbooks. There should be no need to install them separately.

## Cluster pre-requisites
Your cluster will need: -

1.  A working infrastructure with a **Keycloak** and **PostgreSQL** database.
1.  A **StorageClass** for the Squonk pipeline work directory.
    This must support the **RWX** access mode. If you've installed
    the Informatics Matters infrastructure you'll probably already
    have **EFS**, which will do.     
1.  To help organise Pod deployment you should have nodes
    with the label `purpose=application`. This is not mandatory,
    but recommended.
1.  Domain names should be routed to your cluster.
    You will set the actual names using parameters but you should have
    resolvable domain names for infrastructure components that will be deployed
    (e.g. domains for the Squonk **Portal**).
 
## Cluster credentials
You should be in possession of a Kubernetes configuration file. This is often
the content of the `config` file in your `~/.kube` directory. When run from
AWX, AWX will inject the following environment variables: -

-   `K8S_AUTH_HOST`
-   `K8S_AUTH_API_KEY`
-   `K8S_AUTH_VERIFY_SSL`

When running outside of AWX you need to provide values for these
where the `HOST` is the **cluster -> server** value of your control plane from
the config file and `API_KEY` is the **user-> token** value.

    $ export K8S_AUTH_HOST=https://1.2.3.4:6443
    $ export K8S_AUTH_API_KEY=kubeconfig-user-abc:00000000
    $ export K8S_AUTH_VERIFY_SSL=no

>   If you intend to use `kubectl` you will need to set `KUBECONFIG` variable
    to point to a local copy of the cluster config file. You can safely place
    the config in the root of a clone of this repository as the file
    `kubeconfig` as this is part fo the project ignore set.

    $ export KUBECONFIG=./kubeconfig

## Creating Squonk
You will need to adjust some deployment parameters to suit your needs.
Do this by creating an encrypted `parameters.vault` file in the
`roles/squonk/vars` directory. You'll find an example there that 
you can use as a starting point.

The parameter file should be called `<deployment-name>-parameters.vault`
where `<deployment-name>` is a name to identify the deployment. For example,
the _main_ Squonk deployment parameters will be found in
`im-main-parameters.vault`.

And then, to using the correct encrypted parameter file for your deployment,
deploy Squonk by specifying the deployment name in the `sq_deployment_name`
variable (i.e. to deploy the 'im-main' site): -

    $ ansible-playbook -e sq_deployment_name=im-main site-squonk.yaml \
        --vault-password-file vault-pass.txt

### Plays
The following plays are supported, captured in corresponding `site*.yaml`
playbook files: -

-   `site-squonk` (for the main Squonk deployment)
-   `site-pipeline` (for pipeline deployment)

## Deleting Squonk
The following play deletes Squonk and any deployed pipelines: -

    $ ansible-playbook -e sq_deployment_name=im-main unsite-squonk.yaml\
        --vault-password-file vault-pass.txt

## Using Ansible Vault to preserve parameters
Site parameter files can be stored in `.vault` files. These will be written
to revision control but their un-encrypted versions will not. This only works
if your sensitive (unencrypted) parameter files end with the word `parameters`.

>   You can deploy directly without having to decrypt the encrypted parameter
    file.

---

[ansible galaxy]: https://galaxy.ansible.com
[kubernetes]: https://kubernetes.io
[squonk]: https://squonk.it
[vault]: https://docs.ansible.com/ansible/latest/user_guide/vault.html
