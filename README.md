# ANSIBLE DEMO PROJECT

In this demo project, 4 Linux (Ubuntu/Alpine) Container will be created and are kept running.  
One of those containers will act as Ansible Control Node and other 3 containers will act as Managed Nodes.  
In those Managed Nodes, Traefik as Reverse Proxy and Load Balancer and two nginx web servers will be installed.

## Prerequisites

- Docker
- Docker Compose

## Project Structure

```plaintext
ansible-demo
├── ansible
│   ├── ansible.cfg           # Ansible configuration settings
│   ├── .vault_password       # Vault password file for encrypting sensitive data. 
│   ├── files
│   │   └── traefik.service   # Systemd service file for Traefik
│   ├── inventory
│   │   └── hosts.ini         # Inventory of hosts to manage
│   ├── playbooks
│   │   ├── nginx.yaml        # Playbook for deploying Nginx
│   │   └── traefik.yml       # Playbook for deploying Traefik
│   ├── templates
│   │   ├── nginx
│   │   │   ├── index.html.j2 # Jinja2 template for Nginx index page
│   │   │   └── nginx.conf.j2 # Jinja2 template for Nginx config
│   │   └── traefik
│   │       ├── services
│   │       │   └── dynamic_config.yaml.j2 # Jinja2 template for Traefik dynamic config
│   │       └── traefik.yaml.j2            # Jinja2 template for Traefik config
│   └── vars
│       ├── nginx.yaml        # Variables for Nginx deployment
│       ├── traefik.yaml      # Variables for Traefik deployment
│       └── vault.yaml        # Encrypted variables for sensitive data
├── container
│   ├── Containerfile.ansible # Dockerfile for Ansible Control Node
│   ├── Containerfile.base    # Dockerfile for Managed Nodes
│   └── ssh
│       ├── id_rsa            # Private SSH key for Ansible to connect to Managed Nodes
│       └── id_rsa.pub        # Public SSH key for Managed Nodes
├── docker-compose.yaml       # Docker Compose file to set up containers
└── README.md                 # Project documentation
```

## How to run

1. Clone this repository to your local machine.
2. Navigate to the project directory.
3. Run the following command to start the containers:  

    ```bash
    docker-compose up -d
    ```

4. Access the Ansible Control Node container:

    ```bash
    docker exec -it ansible-demo-control-node sh
    ```

5. (Optional) Test connectivity from Control Node to Managed Nodes:

    ```bash
    ping ansible-demo-server-1
    ping ansible-demo-server-2
    ping ansible-demo-server-3
    ```

6. Test ssh connectivity from Control Node to Managed Nodes:

    ```bash
    ssh -i /code/.ssh/id_rsa ansible@ansible-demo-server-1
    ssh -i /code/.ssh/id_rsa ansible@ansible-demo-server-2
    ssh -i /code/.ssh/id_rsa ansible@ansible-demo-server-3
    ```

    If you get an error "Load key \"/code/.ssh/id_rsa\": error in libcrypto", make sure the SSH keys are in Unix LF format (not Windows CRLF) and rebuild (docker compose up -d --build).

7. From the Control Node, you can run Ansible commands to manage the Managed Nodes.

## Running Playbooks

To deploy Traefik and Nginx on the Managed Nodes, run the following commands from within the Ansible Control Node container:

```bash
ansible-playbook -i inventory/hosts.ini /code/ansible/playbooks/all.yaml --vault-password-file /code/ansible/.env
```

or run them separately:

```bash
ansible-playbook -i /code/ansible/inventory/hosts.ini /code/ansible/playbooks/traefik.yaml --vault-password-file /code/ansible/.env
ansible-playbook -i /code/ansible/inventory/hosts.ini /code/ansible/playbooks/nginx.yaml --vault-password-file /code/ansible/.env
```

You can also run in check mode (dry run) to see what changes would be made without actually applying them:

```bash
ansible-playbook -i /code/ansible/inventory/hosts.ini /code/ansible/playbooks/all.yaml --vault-password-file /code/ansible/.env --check
```

## Ansible Vault

This project uses Ansible Vault to encrypt sensitive data such as passwords.  
The vault password is stored in the `ansible/.env` file.  
Make sure to keep this file secure and do not commit it to version control.  
To encrypt the vault file, use the following command:

```bash
ansible-vault encrypt /code/ansible/vars/vault.yaml --vault-password-file /code/ansible/.env
```

For decrypting or editing the vault file, use:

```bash
ansible-vault decrypt /code/ansible/vars/vault.yaml --vault-password-file /code/ansible/.env
```
