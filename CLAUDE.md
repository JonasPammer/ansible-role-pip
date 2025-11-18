# CLAUDE.md

## Repository Overview

This is an Ansible role (`jonaspammer.pip`) that installs and manages pip (Python package installer) on target systems. The role ensures pip is at the latest compatible version for the target Python version and can optionally install pip packages to system or virtualenv.

## Testing and Development

### Running Tests

Tests use Molecule with Docker containers to test across multiple distributions and Ansible versions:

```bash
# Run all tests (includes linting + all Ansible versions)
tox

# Test specific distribution
MOLECULE_DISTRO=ubuntu2204 tox

# Test specific Ansible version
tox -e py3-ansible-9

# Test specific combination
MOLECULE_DISTRO=debian12 tox -e py3-ansible-8

# Keep container running for debugging
MOLECULE_DESTROY=never MOLECULE_DISTRO=rockylinux9 tox -e py3-ansible-9

# Display installed package versions (useful for debugging)
CI=true tox
```

Tested distributions (via geerlingguy Docker images):
- ubuntu2004, ubuntu2204
- debian11, debian12
- rockylinux8, rockylinux9
- fedora39

Tested Ansible versions: 6 (core 2.13), 7 (core 2.14), 8 (core 2.15), 9 (core 2.16)

### Debugging Molecule Containers

When `MOLECULE_DESTROY=never` is set and tests fail:

```bash
# Find container
docker ps

# Enter container
docker exec -it <container_id> /bin/bash

# Debug files are available at:
# - /var/tmp/vars.yml (hostvars)
# - /var/tmp/environment.yml

# Cleanup when done
docker stop <container_id>
docker container rm <container_id>
```

### Linting

```bash
# Run all pre-commit hooks
pre-commit run --all-files

# Install pre-commit hooks locally
pre-commit install

# Specific linters
yamllint .
ansible-lint
```

## Architecture

### Role Structure

Standard Ansible role layout:
- `defaults/main.yml` - Default variables with OS-specific overrides
- `vars/main.yml` - Internal variables (rarely overridden)
- `tasks/main.yml` - Main task execution flow
- `tasks/assert.yml` - Variable validation
- `tasks/determine-default-pip-version.yml` - Python version detection logic
- `handlers/main.yml` - Event handlers (currently empty)
- `meta/main.yml` - Role metadata and Galaxy info
- `molecule/` - Molecule test scenarios
- `templates/` - Jinja2 templates (currently none)

### Key Task Flow

1. **Validate variables** (`assert.yml`) - Ensures `pip_state` is valid
2. **Install pip package** - Uses system package manager to install `pip_package` (default: `python3-pip`)
3. **Determine default pip version** (`determine-default-pip-version.yml`):
   - Detects Python version via `pip_python_executable`
   - Sets `pip__default_pip_version` based on compatibility:
     - Python 3.3: `<18`
     - Python 3.4: `<19.2`
     - Python 2 or 3.5: `<21`
     - Python 3.6: `<22`
     - Python 3.7+: `>=22`
4. **Upgrade pip** - Uses pip to install/upgrade pip to desired version
5. **Install virtualenv packages** (conditional) - Only if virtualenv is needed
6. **Install pip packages** - Installs packages from `pip_install_packages` list

### OS-Specific Variable Pattern

The role uses a dictionary-based pattern for OS-specific variables:

```yaml
_pip_virtualenv_packages:
  default: [virtualenv]
  CentOS_7: [python-virtualenv]
  RedHat_9: []

pip_virtualenv_packages: "{{
  _pip_virtualenv_packages[ansible_distribution ~ '_' ~ ansible_distribution_major_version]|default(
  _pip_virtualenv_packages[ansible_os_family ~ '_' ~ ansible_distribution_major_version])|default(
  _pip_virtualenv_packages[ansible_distribution])|default(
  _pip_virtualenv_packages[ansible_os_family])|default(
  _pip_virtualenv_packages['default']) }}"
```

This pattern searches in order: `Distribution_Version` → `OSFamily_Version` → `Distribution` → `OSFamily` → `default`

### Python Version Detection

The role handles Python 2's quirk of outputting version to stderr (Python 2 bug #18338). The detection logic:

1. Runs `{{ pip_python_executable }} --version`
2. Detects if output is Python 2 (checks stderr)
3. Extracts version from appropriate stream
4. Parses major/minor version numbers
5. Sets appropriate pip version constraint

### Molecule Test Architecture

- **dependency**: Uses `galaxy` with `requirements.yml` to install role dependencies (jonaspammer.bootstrap)
- **driver**: Docker with geerlingguy service-enabled images
- **platforms**: Single instance per test, name includes `${TOX_ENVNAME}` and `${MOLECULE_DISTRO}`
- **provisioner**: Ansible with `prepare.yml` for system setup, `converge.yml` for role execution
- **verifier**: Ansible tasks in `verify.yml` validate correct installation

Test scenario (`molecule/default/converge.yml`) installs:
- `pre-commit==1.0.0` (specific version test)
- `awscli` (latest version test)
- `ipaddress` to `/virtualenv` (virtualenv test)

Verification checks:
- `/virtualenv` directory exists
- `awscli` is in `pip3 freeze` output
- `pre-commit==1.0.0` exact version installed

## Important Configuration Files

- `.ansible-lint` - Skips `name` rule, excludes `molecule/resources/*`
- `.yamllint` - Max line length 140, ignores `.tox/`
- `tox.ini` - Defines test matrix: Python 3 × Ansible 6,7,8,9
- `.pre-commit-config.yaml` - Runs commitlint, prettier, yamllint, Python linters (black, flake8, mypy, etc.)

## CookieCutter Template

This role is generated from https://github.com/JonasPammer/cookiecutter-ansible-role and kept in sync using cruft:

```bash
# Update from template
cruft update

# Check for template updates
cruft check
```

When making changes, consider if they should apply to the template instead of this role specifically.

## CI/CD

GitHub Actions workflows:
- `ci.yml` - Runs lint + molecule tests on PR/push/schedule
- `gh-pages.yml` - Generates documentation
- `release-to-galaxy.yml` - Publishes to Ansible Galaxy on tag push
- `issue-label-manager.yml` - Manages issue labels
- `label-pr-sizes.yml` - Auto-labels PR sizes

Versioning: Uses git tags (without 'v' prefix) for Ansible Galaxy releases.

## Development Dependencies

Installed via `requirements-dev.txt`:
- `cruft` - CookieCutter template sync
- `pre-commit` - Git hook framework
- `tox` - Test orchestration

System requirements:
- Python 3.10+
- Docker (for Molecule tests)

## Role Variables

Key variables (see `defaults/main.yml` and README.adoc for full documentation):

- `pip_package: python3-pip` - System package to install
- `pip_python_executable` - Auto-detected from pip_package (python3/python)
- `pip_executable` - Auto-detected from pip_package (pip3/pip)
- `pip_version` - Auto-determined based on Python version, or override with pip requirement specifier
- `pip_state: forcereinstall` - One of: forcereinstall, latest, present
- `pip_virtualenv_packages` - OS-specific, typically [virtualenv]
- `pip_install_packages: []` - List of packages to install (supports name, version, virtualenv, requirements file, etc.)

## Commit Convention

Uses Conventional Commits for core contributors. PRs are squash-merged, so individual commits in PRs don't need to follow the convention.
