# Register or Unregister a RHEL Node Using Ansible

When spinning up a cluster of RHEL servers, a common repetitive task includes registering a node with Red Hat Subscription Manager, attaching a pool, enabling repositories and updating the system. The register and unregister playbooks provide a means of automating the steps as Ansible plays.

## Assumptions

The playbooks are configured to retrieve a group of hosts called `rhel-servers`. This repository contains three user variables in `group_vars/all.yml`: namely, `rhn_username`, which is a username argument for the Red Hat network; `rhn_password`, which is a password argument for the Red Hat network; and, `rhn_pool_id`, which is a pool ID argument for a Red Hat SKU.

The `register.yml` play requires a Red Hat Network pool ID. If you know the SKU, but not the pool ID, you may register a single node, refresh the system manually.

## Instructions

To use the `register.yml` or `unregister.yml` plays, create a `[rhel-servers]` group in the inventories file (for example, `/etc/ansible/hosts`). Alternatively, modify the `- hosts: rhel-servers` entry in the the `register.yml` or `unregister.yml` files such that the `rhel-servers` portion reflects the name of the group of hosts targetted for the plays.

Registering a RHEL node requires a valid Red Hat Network username and password, and a valid pool ID associated to the user. Add a username, password and pool ID to `group_vars/all.yml`. 

To register a node, execute the `register.yml` play as `root`:

`# ansible-playbook register.yml`

To unregister a node, execute the `unregister.yml` play as `root`: 

`# ansible-playbook unregister.yml`

## `register.yml` Details

The `register.yml` play performs the following steps using `command` or the `yum` module:

. `# subscription-manager status`: A status check to determine if the node is registered. Enables skipping the node registration steps if the node is already registered.

. `# subscription-manager register --force`: Takes a Red Hat Network username and password and registers the node. It forces re-registration if the node is already registered to simplify error handling.

. `# subscription-manager refresh`: Refreshes the list of available repositories. That is often unnecessary, but nice to have.

. `# subscription-manager attach --pool=<pool-id>`: Attaches a pool with the pool ID specified in `group_vars/all.yml`.

. `# subscription-manager repos --disable= '*'`: Disables all of the enabled repositories.

. `# subscription-manager repos --enable=rhel-7-server-rpms`: Enables the `rhel-7-server-rpms` repository.

. `# subscription-manager repos --enable=rhel-7-server-extras-rpms`: Enables the `rhel-7-server-extras-rpms` repository.

. Updates the system. Equivalent to `# yum update -y`.

## `unegister.yml` Details

The `unregister.yml` play performs the following steps using `command`:

. `# subscription-manager unregister`: Unregisters the node. Ignores errors.

. `# subscription-manager repos --disable= '*'`: Disables all of the enabled repositories.

