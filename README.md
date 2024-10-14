# K3s Deployment Install/Upgrade and Uninstall Playbook

This two Ansible playbook automates the installation and management of my k3s Mini Cluster homelab. It is designed to handle both the master and worker nodes, ensuring the k3s services are installed, configured, and running correctly across all nodes in the cluster. It also includes procedures for handling the k3s service, upgrading the system packages, and ensuring the proper configuration of the Kubernetes client (kubeconfig) on the master node.

Features:

Master Node Setup:
    - Updates and upgrades packages (Debian and Red hat distro)
    - Verifies if the k3s service is running and handles the process if necessary
    - Installs or upgrades k3s while disable Traefik and ServiceLB
    - Creates and configures the kubeconfig file, ensuring it is accessible for the kube_user
    - Copy token from the master node and passes it to worker nodes for authentication and cluster joining.
    - Restarts the k3s service after installation or upgrades
        
Worker Node Setup:
    - Updates and upgrades packages.
    - Installs or upgrades k3s on worker nodes and joins them to the master node using the provided token.
    - Restarts the k3s-agent service for proper operation.

## Requirements

   - Ansible (version 2.x or higher).
   - At least one master node and multiple worker nodes.
   - Access to each node (either via SSH or local execution).

## Variables

Example Variables

    home_directory: "/home/kube_user"
    kube_user: "kube_user"

### Usage

Inventory File Example:

    [master]
    192.168.x.xxx ansible_python_interpreter=/usr/bin/python3.12 ansible_become_pass=your_password ansible_user=your_username home_directory=/home/your_home_directory kube_user=your_username ansible_host=192.168.x.xxx

    [workers]
    192.168.x.xxx ansible_python_interpreter=/usr/bin/python3.12 ansible_become_pass=your_password ansible_user=your_username 

### Running the Playbook

To run the playbook, use the following command:

    ansible-playbook -i inventory -k k3s-install-playbook.yaml

This will execute the playbook and apply the necessary configurations to both master and worker nodes.
Ensure that the ansible_user has SSH access to all nodes and the correct permissions to run commands as root (via sudo or similar).

## Uninstall K3s from Master and Worker Nodes Playbook

To uninstall master and all the worker nodes, use the following command:

    ansible-playbook -i inventory -k k3s-uninstall-playbook.yaml

Master Node:

   - Run k3s-uninstall.sh script
   - If not found, notify that the script is missing.
   - Remove KUBECONFIG export from ~/.bashrc.
   - Delete the ~/.kube directory.

Worker Nodes:

   - Ensure k3s-agent-uninstall.sh script is present, then run it if available.
   - If not found, notify that the agent script is missing.

### Notes:

The playbook assumes that the k3s installation script is fetched from the official URL (https://get.k3s.io).
The playbook also assumes that your nodes have internet access to fetch the required binaries and updates.
