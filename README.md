# RHEL 8 CIS

![Build Status](https://img.shields.io/github/workflow/status/ansible-lockdown/RHEL8-CIS/CommunityToDevel?label=Devel%20Build%20Status&style=plastic)
![Build Status](https://img.shields.io/github/workflow/status/ansible-lockdown/RHEL8-CIS/DevelToMain?label=Main%20Build%20Status&style=plastic)
![Release](https://img.shields.io/github/v/release/ansible-lockdown/RHEL8-CIS?style=plastic)

Configure RHEL/Rocky/AlmaLinux machine to be [CIS](https://www.cisecurity.org/cis-benchmarks/) compliant

Based on [CIS RedHat Enterprise Linux 8 Benchmark v2.0.0 - 02-23-2022 ](https://www.cisecurity.org/cis-benchmarks/)

## Join us

On our [Discord Server](https://discord.io/ansible-lockdown) to ask questions, discuss features, or just chat with other Ansible-Lockdown users

## Caution(s)

This role **will make changes to the system** which may have unintended consequences. This is not an auditing tool but rather a remediation tool to be used after an audit has been conducted.

This role was developed against a clean install of the Operating System. If you are implementing to an existing system please review this role for any site specific changes that are needed.

To use release version please point to main branch

## Documentation

- [Getting Started](https://www.lockdownenterprise.com/docs/getting-started-with-lockdown)
- [Customizing Roles](https://www.lockdownenterprise.com/docs/customizing-lockdown-enterprise)
- [Per-Host Configuration](https://www.lockdownenterprise.com/docs/per-host-lockdown-enterprise-configuration)
- [Getting the Most Out of the Role](https://www.lockdownenterprise.com/docs/get-the-most-out-of-lockdown-enterprise)
- [Wiki](https://github.com/ansible-lockdown/RHEL8-CIS/wiki)
- [Repo GitHub Page](https://ansible-lockdown.github.io/RHEL8-CIS/)

### Running level1 or level2 only

While the defaults/main.yml has the options this is used for auditing purposes.
In order to run specific level(1|2)-server level(1|2)-workstation  This is carried out via tags. Otherwise, leaving out the tag defaults to running all tasks.

e.g.

``` shell
ansible-playbook site.yml -t level1-server
```

Note that the host inventory file, `inventory.ini`, does not need to be explicitly defined upon invoking `ansible-playbook` command. Ansible will look for an inventory file to parse until it finds a correctly formatted  _inventory source_, as seen in the following log output:

```shell
host_list declined parsing /etc/ansible/hosts as it did not pass its verify_file() method
script declined parsing /etc/ansible/hosts as it did not pass its verify_file() method
auto declined parsing /etc/ansible/hosts as it did not pass its verify_file() method
Parsed /etc/ansible/hosts inventory source with ini plugin
```


## Auditing (new)

This can be turned on or off within the defaults/main.yml file with the variable rhel8cis_run_audit. The value is false by default, please refer to the wiki for more details.

This is a much quicker, very lightweight, checking (where possible) config compliance and live/running settings.

A new form of auditing has been developed, by using a small (12MB) go binary called [goss](https://github.com/aelsabbahy/goss) along with the relevant configurations to check. Without the need for infrastructure or other tooling.
This audit will not only check the config has the correct setting but aims to capture if it is running with that configuration also trying to remove [false positives](https://www.mindpointgroup.com/blog/is-compliance-scanning-still-relevant/) in the process.

Refer to [RHEL8-CIS-Audit](https://github.com/ansible-lockdown/RHEL8-CIS-Audit).

## Requirements

RHEL/AlmaLinux/Rocky/Oracle 8 - Other versions are not supported.

- AlmaLinux/Rocky Has been tested on 8.4 enabling crypto (sections 1.10&1.11) breaks updating or installs 01Jul2021
- Access to download or add the goss binary and content to the system if using auditing (other options are available on how to get the content to the system.)

## General

- Basic knowledge of Ansible, below are some links to the Ansible documentation to help get started if you are unfamiliar with Ansible
  - [Main Ansible documentation page](https://docs.ansible.com)
  - [Ansible Getting Started](https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html)
  - [Tower User Guide](https://docs.ansible.com/ansible-tower/latest/html/userguide/index.html)
  - [Ansible Community Info](https://docs.ansible.com/ansible/latest/community/index.html)
- Functioning Ansible and/or Tower Installed, configured, and running. This includes all of the base Ansible/Tower configurations, needed packages installed, and infrastructure setup.
- Please read through the tasks in this role to gain an understanding of what each control is doing. Some of the tasks are disruptive and can have unintended consiquences in a live production system. Also familiarize yourself with the variables in the defaults/main.yml file or the [Main Variables Wiki Page](https://github.com/ansible-lockdown/RHEL8-CIS/wiki/Main-Variables).

## Dependencies

- Python3
- Ansible 2.9+
- python-def (should be included in RHEL 8)
- libselinux-python

## Role Variables

This role is designed that the end user should not have to edit the tasks themselves. All customizing should be done via the defaults/main.yml file or with extra vars within the project, job, workflow, etc. These variables can be found [here](https://github.com/ansible-lockdown/RHEL8-CIS/wiki/Main-Variables) in the Main Variables Wiki page. All variables are listed there along with descriptions.

## Tags

There are many tags available for added control precision. Each control has it's own set of tags noting what level, if it's scored/notscored, what OS element it relates to, if it's a patch or audit, and the rule number.

Below is an example of the tag section from a control within this role. Using this example if you set your run to skip all controls with the tag services, this task will be skipped. The opposite can also happen where you run only controls tagged with services.

```txt
      tags:
      - level1-server
      - level1-workstation
      - scored
      - avahi
      - services
      - patch
      - rule_2.2.4
```

## Example Audit Summary

This is based on a vagrant image with selections enabled. e.g. No Gui or firewall.
Note: More tests are run during audit as we check config and running state.

```txt

ok: [default] => {
    "msg": [
        "The pre remediation results are: ['Total Duration: 5.454s', 'Count: 338, Failed: 47, Skipped: 5'].",
        "The post remediation results are: ['Total Duration: 5.007s', 'Count: 338, Failed: 46, Skipped: 5'].",
        "Full breakdown can be found in /var/tmp",
        ""
    ]
}

PLAY RECAP *******************************************************************************************************************************************
default                    : ok=270  changed=23   unreachable=0    failed=0    skipped=140  rescued=0    ignored=0
```

## Pipeline Testing

uses:

- ansible-core 2.12
- ansible collections - pulls in the latest version based on requirements file
- runs the audit using the devel branch
- This is an automated test that occurs on pull requests into devel

## Local Testing

Molecule can be used to work on this role and test in distinct _scenarios_.

**examples**

```bash
molecule test -s default
molecule converge -s wsl -- --check
molecule verify -s localhost
```

local testing uses:

- ansible 2.13.3
- molecule 4.0.1
- molecule-docker 2.0.0
- molecule-podman 2.0.2
- molecule-vagrant 1.0.0
- molecule-azure 0.5.0

## known-issues

cloud0init - due to a bug this will stop working if noexec is added to /var.
rhel8cis_rule_1_1_3_3

[bug 1839899](https://bugs.launchpad.net/cloud-init/+bug/1839899)

## Support

This is a community project at its core and will be managed as such. Please provide as much information as possible and utilise the community [Discord Server](https://discord.io/ansible-lockdown).

Refer to linked below drop us a message for further information

- [Lockdown Enterprise](https://www.lockdownenterprise.com)
  - support for individual repository benchmarks
    - advice on how to use and adopt and priority issue adoption
- [Ansible Counselor](https://www.mindpointgroup.com/cybersecurity-products/ansible-counselor)
  - support for all available repos and enhanced support around ansible usage

Bespoke automation support - ansible and otrher products

- Please enquire for specific requirements
