Ansible roles for a variety of common server setups, including maintenance and reporting roles. Intended to be simple for admins unfamiliar with ansible. Roles designed modularly, reducing conflicts if used standalone or together.

Created as a project by a linux admin interested in security and devops, and further learning.

Please see "Features" for a variety of notices and additional information.

-----
Features:

V0.7 (Current)

- Debian OS role with hardening through DevSec and additional logging and security software installed and configured (Including SELinux, firewalld, tcpdump, AIDE, and others).
- SSH role with hardening through DevSec and public key authentication.
- Webserver role which uses systemd to manage, restart (if stopped) and update an automatically updated hardened docker image (Default image from Chainguard).
- Standalone docker role for manual configuration and use of docker if desired.
- VPN server roles (OpenVPN and Wireguard) using best practice configurations from Nyr.
- Maintenance roles for most of the above roles to prevent configuration drift and perform updates/alterations. Reporting/monitoring roles in progress/testing.

Testing,
- Nginx reverse proxy role (Default nginx), using aforementioned features and linking with webserver.
- Additional webserver tasks to create 2 load-balanced containers on demand-for future role and systemd management service.
- Kicksecure role to distro-morph a Debian server into a Kicksecure server for additional permission hardening, surface area reduction, and security features.

Also see:
	- Devsec (https://github.com/dev-sec)
	- Kicksecure (https://www.kicksecure.com and https://github.com/Kicksecure)
	- Chainguard (https://www.chainguard.dev/)
	- Nyr (https://github.com/Nyr)	

All tested roles were tested on Vultr VPS with the following details:
	-Debian 12 (Bookworm)
	-High frequency CPU
	-Cheapest settings imaginable.

V0.8 (In progress)

General,
- Implement reporting and monitoring roles to produce common reports/logs for troubleshooting.
	- Rewrite some Setup_ tasks to be more intuitive for Maint_ tasks (Consider replacing lineinfile tasks with template modules and fully modified configuration files, or modifying lineinfile tasks).
	- Add more reporting options, such as current processes, kernel modules loaded, and so on (as well as comparison tasks/scripts)
	- More effective reporting and monitoring inside the docker container.
- Disable unnecessary outgoing connections (securely while maintaining functionality).
- Separate host and docker bind port variables for all related roles.

V1

General,
- Implement end-to-end VPS creation with Ansible (ie no manual interaction with the VPS provider) and (possibly) populating server IPs automatically into playbook.
- General simplifying of some roles/combinations, such as connecting webservers and reverse proxies.

OS Hardening,
- Add audit=1 to the kernel automatically.
- pam_ssh_agent_auth to use sudo with ssh key (Rather than password) for custom user.
- Better solution for rkhunter --check and aideinit tasks (Decrease playbook time + better choice of task).
- Better solution for SELinux - preferrably place audit2allow and semodule -i in a handler which executes after all tasks in a playbook are finished.

Docker,
- Additional hardening (Such as closing off docker daemon, using unix socket, rootless docker).
- 2 load-balanced containers through webserver role to decrease possible downtime.

Webserver,
- Implement Lets Encrypt certificates and https to webserver-related roles automatically.

Later/Maybe/To do,

SSH role:	
- Optionally generate unique public key(s) per server?
- Optionally randomize ssh port within a specific range per-server?

Debian role:
- Lock down/remove more user commands by default.
- Look for additional packages/compilers/interpreters to disable/remove.
- More security applications and configurations, including many Kicksecure features (such as attack surface reduction and permission hardening) for those who would prefer not to fully distromorph.
- ps aux + diff + filtering cron to log new processes?

Reverse proxy and/or webserver role:
- Set up rotating ipv4 and/or ipv6 reverse proxies automatically if possible.

Other:
- Alpine/Wolfi/Arch role?
- Postfix role?
- Separate Nginx and Apache roles?
- Nginx load balancer role?
- Logging server role? Link server(s) together automatically?
- Allow roles to operate without interruption on 1 hosts file alone and no extra variables or repositioning of roles if possible (Or otherwise as simple as possible).

-----
Instructions

V0.7,

Requirements:
Ansible (latest)
Ansible-galaxy (latest) with devsec hardening roles (ansible-galaxy collection install devsec.hardening).

To configure a new/unconfigured server: ansible-playbook -i [Host file location] [Playbook location] -k -e ansible_user=root -e ansible_port=22
Otherwise: ansible-playbook -i [Host file location] [Playbook location]

Several examples of playbooks are in the main directory. The SSH role will likely need to be executed on it's own before all other playbooks to maintain connectivity through entire playbook.

Explanation of some possible options,

-i identifies the hosts folder (used to determine which servers to target, as well as some variables).
-k prompts for the SSH password to connect to the server. This is typically provided by the VPS provider or the default operating system settings.
-e ansible_user and ansible_port set the user and port ansible attempts to connect with.

A separate ./hosts_ssh.yml has been provided for first-time ssh setups on servers, if you'd prefer not to enter the -k and -e options into the command line.

Using ansible vault for any password or sensitive information is suggested. (See: https://docs.ansible.com/ansible/latest/vault_guide/index.html)
If you have not defined them in the hosts file, you may also consider using -K for the become (sudo/doas) password, and/or --key-file to specify your private key location.

Notes:
- OS Hardening role begins an unfiltered packet capture on all ports through TCPdump. You may want to consider disabling, altering, or filtering this depending on your circumstances and preferences.
- Commands/tasks exists to create a environment variables in /etc/profile for the logging folder location(s) (OS Hardening role), however these are currently disabled to allow users to optionally enable if preferred.
- By role defaults,
	Mailing is not configured outside of defaults.
	Logs and configurations are otherwise untouched/default outside of the configurations mentioned in the ./hosts file(s).
	SELinux is configured to targeting at the end of the OS_Hardening role, and enforce if Setup_SELinux is ran (typically at the end of all other roles, with an assumed trusted/clean system). For security and functionality purposes, please be aware of this when running these playbooks.
- Variables which must be defined for the playbooks to function are currently located in the hosts_example file located in the "Playbooks" directory, along with an explanation of the variables.
- Per the devsec OS hardening manual: "By default, any process that starts before the `auditd` daemon will have an AUID of `4294967295`. To improve this and provide more accurate logging, it's recommended to add the kernel boot parameter `audit=1` to you configuration. Without doing this, you will find that your `auditd` logs fail to properly audit all processes."
	Consider adding "audit=1" to the end of the GRUB_CMDLINE_LINUX_DEFAULT variable in /etc/default/grub.d, performing update-grub, and rebooting. This may be added automatically in future versions.
