packages
=========

This role installs a list of packages (chocolatey and psmodules) needed on the workstation.

The list of packages is at `packages/defaults/main.yml`, and they can be overriden.


Requirements
------------

A Windows host, tested in:
- `Windows Server 2019`

Role Variables
--------------

List of packages:
```
chocolatey:
    - name: adobereader
      state: present
    - name: azure-cli
      state: present
    - name: 7zip
      state: present
    - name: google-backup-and-sync
      state: present
    - name: calibre
      state: present
    - name: drawio
      state: present
    - name: docker-desktop
      state: present
    - name: git
      state: present
    - name: graphviz
      state: present
    - name: javaruntime
      state: present
    - name: minikube
      state: present
    - name: obs-studio
      state: present
    - name: 'powershell-core'
      version: '7.0.0'
      state: present
    - name: spotify
      state: present
    - name: vscode
      state: present
    - name: zoom
      state: present
psmodule:
    - name: PowerShellGet
      state: present
    - name: PackageManagement
      state: present    
    - name: NuGet
      state: present
    - name: CertificateDsc
      state: present
    - name: NetworkingDsc
      state: present
    - name: xWebAdministration
      state: present
```

Dependencies
------------

- None

Example Playbook
----------------

Run the role:

    - hosts: servers
      roles:
         - packages