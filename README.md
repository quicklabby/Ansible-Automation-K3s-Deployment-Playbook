# Ansible Automation: K3s Deployment Install/Uninstall

This Ansible playbook automates the installation and management of my k3s Mini Cluster homelab. It is designed to handle both the master and worker nodes, ensuring the k3s services are installed, configured, and running correctly across all nodes in the cluster. It also includes procedures for handling the k3s service, upgrading the system packages, and ensuring the proper configuration of the Kubernetes client (kubeconfig) on the master node.
Features:

    Master Node Setup:
        Updates and upgrades packages.
        Checks if the k3s service is running and handles the process if it is.
        Installs or upgrades k3s, disabling unnecessary services (Traefik and ServiceLB).
        Creates and configures the kubeconfig file for the master node, making it accessible for the kube_user.
        Provides a token for worker nodes to join the cluster.
        Restarts the k3s service after installation or upgrades.

    Worker Node Setup:
        Updates and upgrades packages.
        Installs or upgrades k3s on each worker node and joins them to the master node using the token.
        Restarts the k3s-agent service to ensure proper operation.

    Global Token Management:
        A token is dynamically created on the master node and passed to worker nodes for authentication and cluster joining.
        The token is cleared from the local Ansible variables once all worker nodes are processed.

## Requirements

    Ansible (version 2.x or higher).
    At least one master node and multiple worker nodes.
    Access to each node (either via SSH or local execution).
    The k3s binary and associated scripts (k3s-killall.sh) should be available on the nodes, or they will be fetched from the official k3s installation script.

## Variables

The playbook uses the following variables:

    home_directory: The home directory for the kube_user (typically /home/kube_user or /root).
    kube_user: The user that will own the kubeconfig and other related Kubernetes files.
    ansible_env.SHELL: Ensures that the script is using /bin/bash for bash-specific commands.

Example Variables

```home_directory: "/home/kube_user"
kube_user: "kube_user"```

Playbook Breakdown
1. Master Node Setup (hosts: master)

    Update and upgrade system packages: Ensures that all packages are up-to-date on the master node.
    Check if k3s service is running: Checks the status of the k3s service to determine whether it is already active. If active, it will be killed and reinstalled.
    Install/Upgrade k3s: Installs or upgrades the k3s service with disabled Traefik and ServiceLB components.
    Generate token for worker nodes: The token is retrieved from the master node, enabling the worker nodes to join the cluster.
    Configure kubeconfig: Creates a .kube/config file for the kube_user, enabling them to interact with the Kubernetes cluster using kubectl.
    Restart k3s service: Restarts the k3s service to ensure that all changes take effect.

2. Worker Node Setup (hosts: workers)

    Update and upgrade system packages: Updates packages on worker nodes.
    Install/Upgrade k3s: Installs k3s and joins each worker node to the master node using the token created by the master node.
    Restart k3s-agent: Ensures that the k3s-agent service is restarted to apply any necessary changes after installation.

3. Clear Token (hosts: localhost)

    Clear token variable: Once all worker nodes have been processed, the TOKEN variable is cleared to prevent security leaks or unintended access.

Usage
Inventory File Example

An example Ansible inventory file (hosts.ini) could look like this:

ini

[master]
master1 ansible_host=192.168.1.100 ansible_user=root

[workers]
worker1 ansible_host=192.168.1.101 ansible_user=root
worker2 ansible_host=192.168.1.102 ansible_user=root

Running the Playbook

To run the playbook, use the following command:

bash

ansible-playbook -i hosts.ini k3s-cluster-setup.yml

This will execute the playbook and apply the necessary configurations to both master and worker nodes.
Ansible Configuration Example

Ensure that the ansible_user has SSH access to all nodes and the correct permissions to run commands as root (via sudo or similar).
Notes:

    The playbook assumes that the k3s installation script is fetched from the official URL (https://get.k3s.io).
    The playbook also assumes that your nodes have internet access to fetch the required binaries and updates.
    serial: 1 ensures that nodes are processed one by one, which is useful in a controlled environment like a Kubernetes cluster setup where the master needs to be prepared before the workers.
