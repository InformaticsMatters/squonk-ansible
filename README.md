# Squonk (Ansible)

![lint](https://github.com/InformaticsMatters/squonk-ansible/workflows/lint/badge.svg)

![GitHub tag (latest SemVer)](https://img.shields.io/github/v/tag/informaticsmatters/squonk-ansible)

[![CodeFactor](https://www.codefactor.io/repository/github/informaticsmatters/squonk-ansible/badge)](https://www.codefactor.io/repository/github/informaticsmatters/squonk-ansible)

An Ansible role to deploy the Informatics Matters [Squonk] application
to [Kubernetes].

Ideally you'll start from a Python 3 virtual environment
(and tested using Python 3.7.6) and then install
the required modules, roles and collections: -

    $ python -m venv venv
    $ source ./venv/bin/activate
    $ pip install --upgrade pip

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
deploy Squonk by specifying the deployment name in the `sq_parameter_vault`
variable (i.e. to deploy the 'im-main' site): -

    $ ansible-playbook -e sq_parameter_vault=im-main site-squonk.yaml \
        --vault-password-file vault-pass.txt

### Plays
The following plays are supported, captured in corresponding `site*.yaml`
playbook files: -

-   `site-squonk` (for the main Squonk deployment)
-   `site-squon_update-website` (to update a deployed website)
-   `site-chemcentral-database` (for the Squonk ChemCentral Database deployment)
-   `site-chemcentral-database_run-loader` (to run a ChemCentral DB loader Job)
-   `site-pipeline` (for pipeline deployment)

## Image pull secrets
Refer to the Informatics Matters inter-project (developer) documentation
for details on how to create image pull secrets.

## Deleting Squonk
The following play deletes Squonk and any deployed pipelines: -

    $ ansible-playbook -e sq_parameter_vault=im-main unsite-squonk.yaml\
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
[private registry]: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
[squonk]: https://squonk.it
[vault]: https://docs.ansible.com/ansible/latest/user_guide/vault.html
