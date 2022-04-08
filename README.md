# starterkit-devops-ansible-playbooks

Post Windows installation playbooks for different Windows environments.

<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->
[![All Contributors](https://img.shields.io/badge/all_contributors-1-orange.svg?style=flat-square)](#contributors)
<!-- ALL-CONTRIBUTORS-BADGE:END -->

## Built With

The main tools this repository will use are:

- [Ansible](https://docs.ansible.com), a tool that connects to the Windows machines using winRM protocol
- [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10), as Ansible just works in Unix machines
- [Chocolatey](https://chocolatey.org), a Windows package manager.

## Features

- Ansible is a tool that manages the configuration of any host/server. It also handles states, so once a machine is configured using Ansible, any software and/or configuration MUST BE done through the playbook process (even configuration and/or application removal). This allows to have every transaction in the git repository, and avoids configuration drifts.
- Will speed-up the ramp-up role for your project

## Getting Started

In the following section you will find out how to setup your enviroment so that later on you are able to clone this repository and run your playbooks.

## Prerequisites

Ansible works also under Windows Subsystem for Linux (WSL). Follow this guide to install it in Windows 10: <https://docs.microsoft.com/en-us/windows/wsl/install-win10>

## Installation

### Prepare the host where to run the Playbook

In both hosts the approach is the same but it uses different commands. Requirements are:

- python3

#### Debian 10 WSL

Install python 3 and pip (python package manager) if not existing:

```
sudo apt-get update
sudo apt-get install python3 python3-pip -y
```

Start installing all the packages needed for running ansible against the windows hosts. You will need to install:

- ansible
- pywinrm
- pywinrm[credssp]

```
pip3 install ansible
pip3 install pywinrm
pip3 install pywinrm[credssp]
```

Make sure Ansible is installed by running:

```
ansible --version
```

### Prepare a Windows host to apply the recipe locally from WSL

To enable your Windows environment to be configured locally by using Ansible from WSL, you will need to follow the next steps:

```
# Configure Remoting for Ansible 
iex ((New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1'))
# Enable WsMan
Enable-WSManCredSSP -Role Server -Force
```

After that, you should be able to connect from your WSL by using `localhost`

## Structure

The playbooks are organized as follows:

```
├── roles
│   ├── packages
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── files
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── meta
│   │   │   └── main.yml
│   │   ├── README.md
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   ├── tests
│   │   │   ├── inventory
│   │   │   └── test.yml
│   │   └── vars
│   │       └── main.yml
│   ├── folders
│   ├── users_management
│   └── env_vars
├── devops.yml
├── hosts.yml
└── README.md

```

There is 1 main folder, and sub-groups inside it:

- `roles`: where tasks and variables related to the group of tasks are defined.

Inside of each role folder, there is a subset of folders for the following purposes:

- `tasks`: Contains the tasks which do the actual work.
- `handlers`: Contains handlers, which, when notified, trigger actions.
- `defaults`: Role specific variables, which have a default, but are intended to get overridden from the role user (a play in a playbook).
- `vars`: Role specific variables, which are not intended to get overridden.
- `templates`: The default directory for jinja2 templated files. The [template module](http://www.markusz.io/posts/2017/11/24/ansible-playbook-roles/#templatemod) searches in that directory.
- `files`: The default directory for static files used by the [copy module](http://www.markusz.io/posts/2017/11/24/ansible-playbook-roles/#copymod).
- `meta`: Meta information for this role. This is the place where you can define dependencies to other roles, for example.
- `tests`: Contains everything necessary to test this role in isolation. I haven’t yet used this like I should. I’m going to explore this in another post later.

Finally, there are main files grouping roles by type. Those files are:

- `devops.yml`: groups the roles needed to be applied in the configuration for a DevOps role

## Run the playbooks

Now that the setup is done in both sides, let's check that the desired host(s) can be reached to be configured.

First thing to do is set the credentials at `hosts` file. You need to set `ansible_user` and `ansible_password` at the `<machine>:vars`. Done that, run the following:

```
ansible -i hosts.yml locahost -m win_ping
```

This should return the following:

```
<machine_ip> | SUCCESS => {
    "changed": false,
    "invocation": {
        "module_args": {
            "data": "pong"
        }
    },
    "ping": "pong"
}
```

### Test the playbook(s)

At this point, we have to Run the whole playbook(s) in `dry-run` mode. It is *MANDATORY* to run this command prior to applying any change in the server, to avoid configuration drifts and/or issues. This command runs the playbook in "simulation mode", and returns the result without applying any change.

> **IMPORTANT** Since now on we have encrypted variables in the repo, we need to run all the playbooks with the flag `--ask-vault-pass`.

```
ansible-playbook -i hosts.yml localhost devops.yml --check
```

### Run the playbook(s)

Once the configuration is tested and everything works as expected, apply the configuration:

```
ansible-playbook -i hosts.yml localhost devops.yml
```

## Troubleshooting and resources

- [Ansible Windows remote management guide](https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html)
- [Understanding and troubleshooting WinRM connection and authentication: a thrill seeker's guide to adventure](http://www.hurryupandwait.io/blog/understanding-and-troubleshooting-winrm-connection-and-authentication-a-thrill-seekers-guide-to-adventure)  --> HIGHLY RECOMMENDED
- [setting-up-wsl-ansible-to-control-windows-host](https://github.com/hclpandv/setting-up-wsl-ansible-to-control-windows-host)
- [Configure your dev Windows machine with Ansible](https://dev.to/gmarokov/configure-your-dev-windows-machine-with-ansible-41aj)
- [Ansible playbook: Post Windows installation](https://github.com/gmarokov/ansible-playbook-postinstall-win)
- [win_feature – Installs and uninstalls Windows Features on Windows Server](https://docs.ansible.com/ansible/latest/modules/win_feature_module.html)
- [win_chocolatey – Manage packages using chocolatey](https://docs.ansible.com/ansible/latest/modules/win_chocolatey_module.html)
- [Desired State Configuration](https://docs.ansible.com/ansible/latest/user_guide/windows_dsc.html)
- [Windows Modules](https://docs.ansible.com/ansible/latest/index.html#stq=win_&stp=1)
- [win_optional_feature – Manage optional Windows features](https://docs.ansible.com/ansible/latest/modules/win_optional_feature_module.html)
- [Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

## Contributing

Please see our [Contribution Guide](CONTRIBUTING.md) to learn how to contribute.

## License

![MIT](https://img.shields.io/badge/License-MIT-blue.svg)

(LICENSE) © 2022 [ERNI - Swiss Software Engineering](https://www.betterask.erni)

## Code of conduct

Please see our [Code of Conduct](CODE_OF_CONDUCT.md)

## Stats

![https://repobeats.axiom.co/api/embed/6d0c51994bd8584be9d34473b53945b99ab65e23.svg](https://repobeats.axiom.co/api/embed/6d0c51994bd8584be9d34473b53945b99ab65e23.svg)

## Follow us

[![Twitter Follow](https://img.shields.io/twitter/follow/ERNI?style=social)](https://www.twitter.com/ERNI)
[![Twitch Status](https://img.shields.io/twitch/status/erni_academy?label=Twitch%20Erni%20Academy&style=social)](https://www.twitch.tv/erni_academy)
[![YouTube Channel Views](https://img.shields.io/youtube/channel/views/UCkdDcxjml85-Ydn7Dc577WQ?label=Youtube%20Erni%20Academy&style=social)](https://www.youtube.com/channel/UCkdDcxjml85-Ydn7Dc577WQ)
[![Linkedin](https://img.shields.io/badge/linkedin-31k-green?style=social&logo=Linkedin)](https://www.linkedin.com/company/erni)

## Contact

📧 [esp-services@betterask.erni](mailto:esp-services@betterask.erni)

Alberto Martín  - [@albertinisg](https://twitter.com/albertinisg) - alberto.mcasado@erni-espana.es

## Contributors ✨

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tr>
    <td align="center"><a href="https://github.com/albertinisg"><img src="https://avatars.githubusercontent.com/u/8257425?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Alberto Martín</b></sub></a><br /><a href="https://github.com/ERNI-Academy/starterkit-devops-ansible-playbooks/commits?author=albertinisg" title="Code">💻</a> <a href="#content-albertinisg" title="Content">🖋</a> <a href="https://github.com/ERNI-Academy/starterkit-devops-ansible-playbooks/commits?author=albertinisg" title="Documentation">📖</a> <a href="#design-albertinisg" title="Design">🎨</a> <a href="#ideas-albertinisg" title="Ideas, Planning, & Feedback">🤔</a> <a href="#maintenance-albertinisg" title="Maintenance">🚧</a> <a href="https://github.com/ERNI-Academy/starterkit-devops-ansible-playbooks/commits?author=albertinisg" title="Tests">⚠️</a> <a href="#example-albertinisg" title="Examples">💡</a> <a href="https://github.com/ERNI-Academy/starterkit-devops-ansible-playbooks/pulls?q=is%3Apr+reviewed-by%3Aalbertinisg" title="Reviewed Pull Requests">👀</a></td>
  </tr>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->
This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!
