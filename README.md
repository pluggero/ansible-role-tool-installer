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
- `dependency_roles`: Galaxy roles to execute before installation (e.g., `["pluggero.git", "pluggero.python"]`)
- `python_venv`: Create Python virtual environment (default: `false`)
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
  roles:
    - pluggero.tool_installer
```

## License

MIT / BSD

## Author Information

This role was created in 2025 by Robin Plugge.
