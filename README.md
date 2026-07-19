# Ansible Role: Tool Installer

[![CI](https://github.com/pluggero/ansible-role-tool-installer/actions/workflows/ci.yml/badge.svg)](https://github.com/pluggero/ansible-role-tool-installer/actions/workflows/ci.yml) [![Ansible Galaxy downloads](https://img.shields.io/ansible/role/d/pluggero/tool_installer?label=Galaxy%20downloads&logo=ansible&color=%23096598)](https://galaxy.ansible.com/ui/standalone/roles/pluggero/tool_installer)

An Ansible Role that installs tools from Git repositories with support for Python virtual environments, custom installation commands, and automatic dependency management.

## Requirements

None.

## Role Variables

```yaml
tool_installer_tools: []
```

List of tools to install. Each tool supports:

**Required:**

- `name`: Tool identifier
- `url`: Git repository URL
- `version`: Branch, tag, or commit SHA

**Optional:**

- `install_scope`: `"user"` (default) or `"system"`
- `tool_installer_git_username`: HTTPS auth username (e.g. `"oauth2"` for a GitLab PAT). Requires an `http(s)://` url. Never hardcode — reference a vault variable
- `tool_installer_git_password`: HTTPS auth token/password. Never hardcode — reference a vault variable
- `tool_installer_git_ssh_key_file`: Path (on the target host) to an SSH private key, used for `ssh://`/`git@host:path` urls as `ansible.builtin.git`'s `key_file`
- `tool_installer_git_accept_hostkey`: Accept unknown SSH host keys during clone (default: `false`)
- `dependency_roles`: Galaxy roles to execute before installation (e.g., `["pluggero.git", "pluggero.python"]`)
- `python_venv`: Create Python virtual environment (default: `false`)
- `dependency_pip_packages`: pip packages/requirements to install into the tool's venv (requires `python_venv: true`). Each entry is either a plain string package spec (e.g. `"requests==2.31.0"`, no auth possible) or a dict `{requirements_file: "<path relative to the tool's cloned repo>"}`, optionally with `tool_installer_pip_index_url` / `tool_installer_pip_index_username` / `tool_installer_pip_index_password` for a private index
- `pre_install_commands`: Commands to run before installation
- `install_commands`: Main installation commands
- `post_install_commands`: Commands to run after installation
- `executables`: Binaries to symlink into PATH
  - `src`: Path relative to tool directory
  - `name`: Name in $PATH
  - `type`: `"python"`, `"binary"`, or `"script"`
- `dependency_packages`: OS packages to install
  - `Debian`: []
  - `Alpine`: []
  - `Archlinux`: []

## Vault Variables

This role never stores secret values itself. When a tool needs HTTPS git
authentication, reference Ansible Vault variables from `tool_installer_git_username` /
`tool_installer_git_password`, following the naming convention
`vault_tool_installer_{tool_name}_{attribute}`:

- `vault_tool_installer_<tool_name>_git_username`
- `vault_tool_installer_<tool_name>_git_password`
- `vault_tool_installer_<tool_name>_pip_index_username`
- `vault_tool_installer_<tool_name>_pip_index_password`

**Security note:** `ansible.builtin.git` has no native credential parameters,
so HTTPS credentials are embedded in the clone URL and will be written, at
rest, into the target repo's `.git/config` on the managed host (protected
only by that file's owner/permissions, not by Ansible's `no_log`, which only
suppresses task *output*). For repos where at-rest exposure of a token is a
concern, prefer `tool_installer_git_ssh_key_file` (an SSH deploy key) instead
— SSH auth never places credential material inside the cloned repository. If
you must use `tool_installer_git_username`/`tool_installer_git_password`, use
a scoped, revocable token (e.g. a GitLab deploy token) rather than a personal
access token.

Similarly, `ansible.builtin.pip`'s `extra_args` has no dedicated credential
parameter, so authenticated `dependency_pip_packages` entries embed their
credentials in an `--index-url` value the same way git credentials are
embedded in the clone URL. Unlike the git case, this `--index-url` value is
not persisted to any file on the managed host afterward (pip does not write
it to `pip.conf`) — `no_log: true` here exists solely to keep the credential
out of Ansible's console output/logs during the run, not to guard against
at-rest exposure.

## Dependencies

This role can execute other Galaxy roles as dependencies. Install required roles:

```bash
ansible-galaxy install -r requirements.yml
```

Common dependency roles:

- `pluggero.git` - For cloning Git repositories
- `pluggero.python` - For Python tools with virtual environments

**Note**: Dependencies are dynamically executed based on the `dependency_roles` field and automatically deduplicated.

## Example Playbook

```yaml
- hosts: all
  vars:
    tool_installer_tools:
      - name: "jwt_tool"
        url: "https://github.com/ticarpi/jwt_tool"
        version: "master"
        install_scope: "user"
        python_venv: true
        pre_install_commands:
          - "pip install -r requirements.txt"
        install_commands: []
        post_install_commands:
          - "chmod +x jwt_tool.py"
        executables:
          - src: "jwt_tool.py"
            name: "jwt_tool"
            type: "python"
        dependency_packages: {}
        dependency_roles:
          - pluggero.git
          - pluggero.python
      - name: "internal-tool"
        url: "https://gitlab.example.com/security/internal-tool"
        version: "main"
        install_scope: "user"
        tool_installer_git_username: "oauth2"
        tool_installer_git_password: "{{ vault_tool_installer_internal_tool_git_password }}"
        python_venv: true
        dependency_pip_packages:
          - requirements_file: "requirements.txt"
            tool_installer_pip_index_url: "https://pypi.example.com/simple"
            tool_installer_pip_index_username: "deploy"
            tool_installer_pip_index_password: "{{ vault_tool_installer_internal_tool_pip_index_password }}"
        install_commands: []
        executables: []
        dependency_packages: {}
        dependency_roles: []
  roles:
    - pluggero.tool_installer
```

## License

MIT / BSD

## Author Information

This role was created in 2025 by Robin Plugge.
