# SimpleLogin Ansible Roles

This repository is a collection of ansible roles designed to help deploy a self-hosted version of [SimpleLogin](https://simplelogin.io/), an open-source email aliasing service. This deployment optionally supports automatic backups of the postgres database to backblaze and optionally automatically sets any user that is created to lifetime premium. It uses docker compose to run the simplelogin stack.

## Why this repository?

The official SimpleLogin documentation, while comprehensive, was difficult to navigate and contained outdated sections at the time we attempted to use it. For example, which database migration, with what commands, in what order, had to be run, etcetera. As a result, we created this repository to streamline and partly automate the process of setting up SimpleLogin with Ansible.

The goal of these roles is to:
- Automate the deployment of a self-hosted SimpleLogin instance.
- Simplify the configuration and installation process.
- Create a reproduceable deployment of simplelogin on selfhosted hardware.
- Provide a reusable and customizable solution for those looking to host their own email aliasing service, built by [SimpleLogin](https://simplelogin.io/).

We have been experimenting with deploying this ourselves for email privacy apps we are building at [Codemoon](https://codemoon.io/). Ultimately we ran into uncertainties regarding email deliverability and opted to use another email provider to handle this. However, for those self-hosting, SimpleLogin should work well enough and this ansible playbook can assist in deploying it efficiently and repeatably.

## Features
- Automated installation of dependencies and setup of simplelogin.
- Configures nginx tls certificate with let's encrypt, databases, and SimpleLogin backend/frontend components.
- Automatically (optionally) sets your newly registered users to lifetime premium.
- Extensible roles that can be adapted for specific configurations and setups.

## What the ansible roles do:

| Role      | Description |
| ----------- | ----------- |
| dkim      | This creates dkim files (for spf and dmarc records) on your target VM. On subsequent runs, it won't overwrite existing dkim records.       |
| packages   | This role installs packages dnsutils that you can use for debugging dns records |
| firewall | This role installs ufw and configures the firewall of your VM |
| dirs | This role creates the directories sl, sl/db, sl/pgp and sl/upload with the correct permissions. | 
| docker | This role installs docker, creates the docker network and adds your user to the docker group. |
| postgresdocker | This role installs postgres with a docker container and runs the necessary migrations for simplelogin. Optionally it also creates a backup cronjob and uploads the result to backblaze if enabled. Optionally a postgres trigger is added that automatically sets new users to premium. |
| postfix | This role automatically installs postfix with the correct installation steps, installs the package postfix-pgsql and creates the required configuration files for postfix. |
| sl | This role creates the docker compose file and creates all docker containers of the simplelogin stack. |
| nginx | This role creates an nginx reverse proxy in front of your simplelogin app and automatically configures a TLS certificate signed by let's encrypt. In case you have your own reverse proxy like traefik or caddy you can skip this step by e.g. commenting it away. |
| finalize | This role echoes a message to your console with the link to your simplelogin instance. |

## Prerequisites

- Basic understanding of ansible, linux, ssh
- A (sub)domain where you can manage the DNS records.
- An Ubuntu 22.04+ or Debian 12 VM with 1 vCPU, 10G disk and 2G of RAM minimum. This VM should have a publicly reachable IPv4 address and outbound traffic should be allowed on port 25.
- SSH access to the VM with ssh key
- Linux user added to sudoers group
- Ansible-core >= 2.17.4 on your controller machine (tested with ansible-core 2.17.4)
- Ansible collections installed: 
    - `ansible-galaxy collection install geerlingguy.docker`
    - `ansible-galaxy collection install community.general`
    - `ansible-galaxy collection install community.docker`
    - `ansible-galaxy collection install community.postgresql`
- Optional: backblaze bucket for postgres database backups (recommended)

## How to install

0. Acquire prerequisites

1. Clone this repository:
   ```bash
   git clone https://github.com/codemoon-io/simplelogin-ansible.git
   cd simplelogin-ansible
   ```

2. Customize the `vars/main.yml` file to your values.

3. Customize the `inventory.yml` file to match your target VM.

4. Set an A record of the prefix.domain to your VM's IPv4 address. 

5. Run the Ansible playbook:
   ```bash
   ansible-playbook deploy-sl.yml -i inventory.yml
   ```

6. Setup the MX, SPF and DMARC records as specified in [simplelogin's documentation](https://github.com/simple-login/app?tab=readme-ov-file#dns), using the DKIM files that were created on the VM when you ran the ansible playbook.

7. Go to https://prefix.domain to register an account on your self hosted instance and enjoy your self-hosted email forwarding aliases 🎉! 

## Notes

- While we were able to deploy SimpleLogin using these roles, we eventually switched to another mail provider for better deliverability. We decided this because we didn't want to actively have to manage deliverability of emails, but for a self-hosted instance, SimpleLogin should work well enough.

- If you find that one of the roles fail during deployment on your system, you can comment some roles so that they won't run again during subsequent tries.
  
- These roles are provided "as-is" and may require adjustments based on the specific server environment or changes in SimpleLogin's requirements. We don't provide support on the deployments but feel free to create a pull request if there are bugs.

- If you wish to make further adjustments to the deployment you could do so via the simplelogin.env file that is created on the VM on /home/{{ linuxuser }}/simplelogin.env. See [simplelogin's documentation](https://github.com/simple-login/app/blob/master/example.env) about how to configure the environment file for further customization options.

- We recommend UpCloud to get a VM, as this is what simplelogin says they use too (clean IP ranges, cost-effective hardware). In case you would like to support yourself (and us too), here is an affiliate link so you get 25€ hosting credits for free which will also give us some free hosting credits. [Affiliate link](https://upcloud.com/signup/?promo=2CT92F). You will still have to ask them to open port 25 for you.

## Contributing

If you'd like to improve this repository, feel free to submit a pull request or open an issue. Any contributions that can further streamline or improve the deployment process are welcome! For example, we see some areas for improvement, like putting the vars files in each role instead of globally. We thought it was easier this way but it may be less configurable like this.

## License

This repository is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.

## Acknowledgements

Thanks goes out to SimpleLogin and Jeff Geerling for their contributions to open source.