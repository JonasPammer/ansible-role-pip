[![Version on Galaxy](https://img.shields.io/badge/available%20on%20ansible%20galaxy-jonaspammer.pip-brightgreen)](https://galaxy.ansible.com/jonaspammer/pip) [![Testing CI](https://github.com/JonasPammer/ansible-role-pip/actions/workflows/ci.yml/badge.svg)](https://github.com/JonasPammer/ansible-role-pip/actions/workflows/ci.yml)

An Ansible role for installing the latest pip (the python package installer) to the system.

By default, this role ensures that pip’s version always matches the latest version that work’s with version of the given python executable (something which just `pip --upgrade pip` 'ing for whatever reason doesn’t do).

This role can also be used to install a given set of pip packages to either the system or in a virtualenv.

# 🔎 Metadata

Below you can find information on…

- the role’s required Ansible version

- the role’s supported platforms

- the role’s [role dependencies](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#role-dependencies)

**[meta/main.yml](meta/main.yml)**

    ---
    galaxy_info:
      role_name: "pip"
      description: "An ansible role for installing the latest pip (python package installer) to the system."
      standalone: true

      author: "jonaspammer"
      license: "MIT"

      min_ansible_version: "2.11"
      platforms:
        # note: text after "actively tested: " represent the docker image name
        - name: EL # (Enterprise Linux)
          versions:
            - "8" # actively tested: rockylinux8
            - "9" # actively tested: rockylinux9
        - name: Fedora
          versions:
            - "35" # actively tested: fedora35
            - "36" # actively tested: fedora36
            - "37" # actively tested: fedora37
            - "38" # actively tested: fedora38
        - name: Debian
          versions:
            - buster # actively tested: debian10
            - bullseye # actively tested: debian11
        - name: Ubuntu
          versions:
            - xenial # actively tested: ubuntu1604
            - bionic # actively tested: ubuntu1804
            - focal # actively tested: ubuntu2004
            - jammy # actively tested: ubuntu2204

      galaxy_tags: []

    dependencies: []

# 📌 Requirements

The Ansible User needs to be able to `become`.

The [`community.general` collection](https://galaxy.ansible.com/community/general) must be installed on the Ansible controller.

# 📜 Role Variables

    pip_version: [latest possible version which supports the python version of behind `pip_python_executable`]

The version of pip to install using pip after pip was installed in form of a [pip requirement specifier](https://pip.pypa.io/en/stable/cli/pip_install/#requirement-specifiers).

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">Version behind <code>pip_python_executable</code></th>
<th style="text-align: left;">Default <code>pip_version</code></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;"><p>no match</p></td>
<td style="text-align: left;"><p>&gt;=22</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>3.6</p></td>
<td style="text-align: left;"><p>&lt;22</p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>3.5 or 2 (.7 assumed)</p></td>
<td style="text-align: left;"><p>&lt;21</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>3.4</p></td>
<td style="text-align: left;"><p>&lt;19.2</p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>3.3</p></td>
<td style="text-align: left;"><p>&lt;18</p></td>
</tr>
</tbody>
</table>

Checkout [pip’s pypi release history](https://pypi.org/project/pip/#history) for available `pip`-versions.

Also, if interested, checkout [the pip documentation](https://pip.pypa.io/en/stable/news/) for a human-generated change log of the changes to the `pip` project _(Maintained as described in their [Contribution Guidelines](https://pip.pypa.io/en/latest/development/contributing/#news-entries))_.

    pip_state: "forcereinstall"

State to pass to `pip install pip`. One of: `forcereinstall`, `latest`, `present`.

If `forcereinstall` is passed this role tries to keep idemponency by disabling `changed_when` on the install and instead executing another task that compares the before/after result of `pip --version`.

If `latest` is passed `pip_version` has no effect.

    pip_package: "python3-pip"

The name of the package(s) to install to get `pip` on the system. For older systems that don’t have Python 3 available, you can set this to `python-pip`.

    pip_virtualenv_packages: [OS-specific by default, see /defaults directory]

The name of the package(s) to install to get `virtualenv` on the system.

    pip_executable: "{{ 'pip3' if pip_package.startswith('python3') else 'pip' }}"

The `pip_executable` passed to the `ansible.builtin.pip` modules issued by this role.

The role will try to autodetect the pip executable based on the `pip_package`. You may also override this explicitly, e.g. `pip_executable: pip3.6`, in case .

    pip_install_packages: []

A list of packages to install with [pip](https://docs.ansible.com/ansible/2.9/modules/pip_module.html).

Each entry may either be a simple string (shorthand for `- name: …`) or an own object with below properties:

_chdir_
`cd` into this directory before running the command.

name
The name of a Python library to install or the url(bzr+,hg+,git+,svn+) of the remote package.

version
The version number to install of the Python library specified in the name parameter.

requirements
Instead of using `name` and `version` to define a single package inline you may also use this option to reference to a path of a pip requirements file, which should be local to the _remote_ system. File can be specified as a relative path if using the `chdir` option.

state
The state of the pip module (i.e. absent / forcereinstall / latest / **present**)

umask
_Defaults to `pip_install_packages_umask` if exists._

The system umask to apply before installing the pip package. This is useful, for example, when installing on systems that have a very restrictive umask by default (e.g., "0077") and you want to pip install packages which are to be used by all users. Note that this requires you to specify desired umask mode as an octal string, (e.g., "0022").

virtualenv
_Defaults to `pip_install_packages_virtualenv` if exists._

Path to a virtualenv directory to install into. If the virtualenv does not exist, it will be created before installing packages. The optional `virtualenv_command`, and `virtualenv_python` options affect the creation of the virtualenv.

virtualenv_command
_Defaults to `pip_install_packages_virtualenv_command` if exists._

The command or a pathname to the command to create the virtual environment with. For example `pyvenv`, **`virtualenv`**, `virtualenv2`, `~/bin/virtualenv`, `/usr/local/bin/virtualenv`.

virtualenv_python
_Defaults to `pip_install_packages_virtualenv_python` if exists._

The Python executable used for creating the virtual environment. For example python3.5, python2.7. When not specified, the Python version used to run the ansible module is used. This parameter should not be used when `virtualenv_command` is using `pyvenv` or the `-m venv` module.

virtualenv_site_packages
_Defaults to `pip_install_packages_virtualenv_python` if exists._

Whether the virtual environment will inherit packages from the global site-packages directory. Note that if this setting is changed on an already existing virtual environment it will not have any effect - the environment must be deleted and newly created.

extra_args
_Defaults to `pip_install_packages_extra_args` if exists._

Extra arguments passed to pip.

environment
_Defaults to `pip_install_packages_environment` if exists._

Environment Variables passed to the pip module.

<!-- -->

    pip_python_executable: "{{ 'python3' if pip_package.startswith('python3') else 'python' }}"

This variable is being used to determine the default value of `pip_version`.

The role will try to autodetect the python executable based on the `pip_package`.

# 📜 Facts/Variables defined by this role

Each variable listed in this section is dynamically defined when executing this role (and can only be overwritten using `ansible.builtin.set_facts`) _and_ is meant to be used not just internally.

# 🏷️ Tags

Tasks are tagged with the following [tags](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html#adding-tags-to-roles):

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">Tag</th>
<th style="text-align: left;">Purpose</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td colspan="2" style="text-align: left;"><p>This role does not have officially documented tags yet.</p></td>
</tr>
</tbody>
</table>

You can use Ansible to skip tasks, or only run certain tasks by using these tags. By default, all tasks are run when no tags are specified.

# 👫 Dependencies

# 📚 Example Playbook Usages

This role is part of [ many compatible purpose-specific roles of mine](https://github.com/JonasPammer/ansible-roles).

The machine needs to be prepared. In CI, this is done in `molecule/default/prepare.yml` which sources its soft dependencies from `requirements.yml`:

**[molecule/resources/prepare.yml](molecule/resources/prepare.yml)**

    ---
    - name: prepare
      hosts: all
      become: true
      gather_facts: false

      roles:
        - role: jonaspammer.bootstrap
        #    - role: jonaspammer.core_dependencies

The following diagram is a compilation of the "soft dependencies" of this role as well as the recursive tree of their soft dependencies.

![requirements.yml dependency graph of jonaspammer.pip](https://raw.githubusercontent.com/JonasPammer/ansible-roles/master/graphs/dependencies_pip.svg)

    ---
    roles:
      - role: jonaspammer.pip

    ---
    roles:
      - role: jonaspammer.pip

    vars:
      - pip_package: "python-pip"

    ---
    roles:
      - role: jonaspammer.pip

    vars:
      pip_install_packages:
      # Specify names and versions.
      - name: docker
        version: "1.2.3"
      - name: awscli
        version: "1.11.91"

      # Or specify bare packages to get the latest release (on first install).
      - docker
      - awscli

      # Or uninstall a package.
      - name: docker
        state: absent

      # Or update a package to the latest version.
      - name: docker
        state: latest

      # Or force a reinstall.
      - name: docker
        state: forcereinstall

# 🧪 Tested Distributions

A role may work on different **distributions**, like Red Hat Enterprise Linux (RHEL), even though there is no test for this exact distribution.

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">OS Family</th>
<th style="text-align: left;">Distribution</th>
<th style="text-align: left;">Distribution Release Date</th>
<th style="text-align: left;">Distribution End of Life</th>
<th style="text-align: left;">Accompanying Docker Image</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;"><p>Rocky</p></td>
<td style="text-align: left;"><p>Rocky Linux 8 (<a href="https://www.howtogeek.com/devops/is-rocky-linux-the-new-centos/">RHEL/CentOS 8 in disguise</a>)</p></td>
<td style="text-align: left;"><p>2021-06</p></td>
<td style="text-align: left;"><p>2029-05</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-rockylinux8-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-rockylinux8-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>Rocky</p></td>
<td style="text-align: left;"><p>Rocky Linux 9</p></td>
<td style="text-align: left;"><p>2022-07</p></td>
<td style="text-align: left;"><p>2032-05</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-rockylinux9-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-rockylinux9-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>RedHat</p></td>
<td style="text-align: left;"><p>Fedora 35</p></td>
<td style="text-align: left;"><p>2021-11</p></td>
<td style="text-align: left;"><p>2022-11</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-fedora35-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-fedora35-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>RedHat</p></td>
<td style="text-align: left;"><p>Fedora 36</p></td>
<td style="text-align: left;"><p>2022-05</p></td>
<td style="text-align: left;"><p>2023-05</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-fedora36-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-fedora36-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>RedHat</p></td>
<td style="text-align: left;"><p>Fedora 37</p></td>
<td style="text-align: left;"><p>2022-11</p></td>
<td style="text-align: left;"><p>2023-12</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-fedora37-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-fedora37-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>RedHat</p></td>
<td style="text-align: left;"><p>Fedora 38</p></td>
<td style="text-align: left;"><p>2023-03</p></td>
<td style="text-align: left;"><p>2024-05</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-fedora38-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-fedora38-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>Debian</p></td>
<td style="text-align: left;"><p>Ubuntu 1604</p></td>
<td style="text-align: left;"><p>2016-04</p></td>
<td style="text-align: left;"><p>2026-04</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-ubuntu1604-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-ubuntu1604-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>Debian</p></td>
<td style="text-align: left;"><p>Ubuntu 1804</p></td>
<td style="text-align: left;"><p>2018-04</p></td>
<td style="text-align: left;"><p>2028-04</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-ubuntu1804-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-ubuntu1804-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>Debian</p></td>
<td style="text-align: left;"><p>Ubuntu 2004</p></td>
<td style="text-align: left;"><p>2021-04</p></td>
<td style="text-align: left;"><p>2030-04</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-ubuntu2004-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-ubuntu2004-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>Debian</p></td>
<td style="text-align: left;"><p>Ubuntu 2204</p></td>
<td style="text-align: left;"><p>2022-04</p></td>
<td style="text-align: left;"><p>2032-04</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-ubuntu2204-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-ubuntu2204-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>Debian</p></td>
<td style="text-align: left;"><p>Debian 10</p></td>
<td style="text-align: left;"><p>2019-07</p></td>
<td style="text-align: left;"><p>2022-08</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-debian10-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-debian10-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>Debian</p></td>
<td style="text-align: left;"><p>Debian 11</p></td>
<td style="text-align: left;"><p>2021-08</p></td>
<td style="text-align: left;"><p>2024-07~</p></td>
<td style="text-align: left;"><p><a href="https://github.com/geerlingguy/docker-debian11-ansible/actions?query=workflow%3ABuild"><img src="https://github.com/geerlingguy/docker-debian11-ansible/workflows/Build/badge.svg?branch=master" alt="CI" /></a></p></td>
</tr>
</tbody>
</table>

# 🧪 Tested Ansible versions

The tested ansible versions try to stay equivalent with the [ support pattern of Ansible’s `community.general` collection](https://github.com/ansible-collections/community.general#tested-with-ansible). As of writing this is:

- 2.11 (Ansible 4)

- 2.12 (Ansible 5)

- 2.13 (Ansible 6)

# 📝 Development

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org) [![pre-commit.ci status](https://results.pre-commit.ci/badge/github/JonasPammer/ansible-role-pip/master.svg)](https://results.pre-commit.ci/latest/github/JonasPammer/ansible-role-pip/master)

## 📌 Development Machine Dependencies

- Python 3.9 or greater

- Docker

## 📌 Development Dependencies

Development Dependencies are defined in a [pip requirements file](https://pip.pypa.io/en/stable/user_guide/#requirements-files) named `requirements-dev.txt`. Example Installation Instructions for Linux are shown below:

    # "optional": create a python virtualenv and activate it for the current shell session
    $ python3 -m venv venv
    $ source venv/bin/activate

    $ python3 -m pip install -r requirements-dev.txt

## ℹ️ Ansible Role Development Guidelines

Please take a look at my [ Ansible Role Development Guidelines](https://github.com/JonasPammer/cookiecutter-ansible-role/blob/master/ROLE_DEVELOPMENT_GUIDELINES.adoc).

If interested, I’ve also written down some [ General Ansible Role Development (Best) Practices](https://github.com/JonasPammer/cookiecutter-ansible-role/blob/master/ROLE_DEVELOPMENT_TIPS.adoc).

## 🔢 Versioning

Versions are defined using [Tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging), which in turn are [recognized and used](https://galaxy.ansible.com/docs/contributing/version.html) by Ansible Galaxy.

**Versions must not start with `v`.**

When a new tag is pushed, [ a GitHub CI workflow](https://github.com/JonasPammer/ansible-role-pip/actions/workflows/release-to-galaxy.yml) (![Release CI](https://github.com/JonasPammer/ansible-role-pip/actions/workflows/release-to-galaxy.yml/badge.svg)) takes care of importing the role to my Ansible Galaxy Account.

## 🧪 Testing

Automatic Tests are run on each Contribution using GitHub Workflows.

The Tests primarily resolve around running [Molecule](https://molecule.readthedocs.io/en/latest/) on a [varying set of linux distributions](#tested-distributions) and using [various ansible versions](#tested-ansible-versions).

The molecule test also includes a step which lints all ansible playbooks using [`ansible-lint`](https://github.com/ansible/ansible-lint#readme) to check for best practices and behaviour that could potentially be improved.

To run the tests, simply run `tox` on the command line. You can pass an optional environment variable to define the distribution of the Docker container that will be spun up by molecule:

    $ MOLECULE_DISTRO=ubuntu2204 tox

For a list of possible values fed to `MOLECULE_DISTRO`, take a look at the matrix defined in [.github/workflows/ci.yml](.github/workflows/ci.yml).

### 🐛 Debugging a Molecule Container

1.  Run your molecule tests with the option `MOLECULE_DESTROY=never`, e.g.:

        $ MOLECULE_DESTROY=never MOLECULE_DISTRO=ubuntu1604 tox -e py3-ansible-5
        ...
          TASK [ansible-role-pip : (redacted).] ************************
          failed: [instance-py3-ansible-5] => changed=false
        ...
         ___________________________________ summary ____________________________________
          pre-commit: commands succeeded
        ERROR:   py3-ansible-5: commands failed

2.  Find out the name of the molecule-provisioned docker container:

        $ docker ps
        30e9b8d59cdf   geerlingguy/docker-debian10-ansible:latest   "/lib/systemd/systemd"   8 minutes ago   Up 8 minutes                                                                                                    instance-py3-ansible-5

3.  Get into a bash Shell of the container, and do your debugging:

        $ docker exec -it 30e9b8d59cdf /bin/bash

        root@instance-py3-ansible-2:/#
        root@instance-py3-ansible-2:/# python3 --version
        Python 3.8.10
        root@instance-py3-ansible-2:/# ...

    If the failure you try to debug is part of `verify.yml` step and not the actual `converge.yml`, you may want to know that the output of ansible’s modules (`vars`), hosts (`hostvars`) and environment variables have been stored into files on both the provisioner and inside the docker machine under: \* `/var/tmp/vars.yml` \* `/var/tmp/hostvars.yml` \* `/var/tmp/environment.yml` `grep`, `cat` or transfer these as you wish!

    You may also want to know that the files mentioned in the admonition above are attached to the **GitHub CI Artifacts** of a given Workflow run.
    This allows one to check the difference between runs and thus help in debugging what caused the bit-rot or failure in general.

    <figure>
    <img src="https://user-images.githubusercontent.com/32995541/178442403-e15264ca-433a-4bc7-95db-cfadb573db3c.png" alt="178442403 e15264ca 433a 4bc7 95db cfadb573db3c" />
    </figure>

4.  After you finished your debugging, exit it and destroy the container:

        root@instance-py3-ansible-2:/# exit

        $ docker stop 30e9b8d59cdf

        $ docker container rm 30e9b8d59cdf
        or
        $ docker container prune

### 🐛 Debugging installed package versions locally

Although a standard feature in tox 3, this [now](https://github.com/tox-dev/tox/pull/2794) only happens when tox recognizes the presence of a CI variable. For example:

    $ CI=true tox

## 🧃 TIP: Containerized Ideal Development Environment

This Project offers a definition for a "1-Click Containerized Development Environment".

This Container even enables one to run docker containers inside of it (Docker-In-Docker, dind), allowing for molecule execution.

To use it:

1.  Ensure you fullfill the [ the System requirements of Visual Studio Code Development Containers](https://code.visualstudio.com/docs/remote/containers#_system-requirements), optionally following the _Installation_-Section of the linked page section.
    This includes: Installing Docker, Installing Visual Studio Code itself, and Installing the necessary Extension.

2.  Clone the project to your machine

3.  Open the folder of the repo in Visual Studio Code (_File - Open Folder…_).

4.  If you get a prompt at the lower right corner informing you about the presence of the devcontainer definition, you can press the accompanying button to enter it. **Otherwise,** you can also execute the Visual Studio Command `Remote-Containers: Open Folder in Container` yourself (_View - Command Palette_ → _type in the mentioned command_).

I recommend using `Remote-Containers: Rebuild Without Cache and Reopen in Container` once here and there as the devcontainer feature does have some problems recognizing changes made to its definition properly some times.

You may need to configure your host system to enable the container to use your SSH/GPG Keys.

The procedure is described [ in the official devcontainer docs under "Sharing Git credentials with your container"](https://code.visualstudio.com/remote/advancedcontainers/sharing-git-credentials).

## 🍪 CookieCutter

This Project shall be kept in sync with [the CookieCutter it was originally templated from](https://github.com/JonasPammer/cookiecutter-ansible-role) using [cruft](https://github.com/cruft/cruft) (if possible) or manual alteration (if needed) to the best extend possible.

> <figure>
> <img src="https://raw.githubusercontent.com/cruft/cruft/master/art/example_update.gif" alt="Official Example Usage of `cruft update`" />
> </figure>

### 🕗 Changelog

When a new tag is pushed, an appropriate GitHub Release will be created by the Repository Maintainer to provide a proper human change log with a title and description.

## ℹ️ General Linting and Styling Conventions

General Linting and Styling Conventions are [**automatically** held up to Standards](https://stackoverflow.blog/2020/07/20/linters-arent-in-your-way-theyre-on-your-side/) by various [`pre-commit`](https://pre-commit.com/) hooks, at least to some extend.

Automatic Execution of pre-commit is done on each Contribution using [`pre-commit.ci`](https://pre-commit.ci/)[\*](#note_pre-commit-ci). Pull Requests even automatically get fixed by the same tool, at least by hooks that automatically alter files.

Not to confuse: Although some pre-commit hooks may be able to warn you about script-analyzed flaws in syntax or even code to some extend (for which reason pre-commit’s hooks are **part of** the test suite), pre-commit itself does not run any real Test Suites. For Information on Testing, see [🧪 Testing](#testing).

Nevertheless, I recommend you to integrate pre-commit into your local development workflow yourself.

This can be done by cd’ing into the directory of your cloned project and running `pre-commit install`. Doing so will make git run pre-commit checks on every commit you make, aborting the commit themselves if a hook alarm’ed.

You can also, for example, execute pre-commit’s hooks at any time by running `pre-commit run --all-files`.

# 💪 Contributing

[![Open in Visual Studio Code](https://img.shields.io/static/v1?logo=visualstudiocode&label=&message=Open%20in%20Visual%20Studio%20Code&labelColor=2c2c32&color=007acc&logoColor=007acc)](https://open.vscode.dev/JonasPammer/ansible-role-pip) ![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)

The following sections are generic in nature and are used to help new contributors. The actual "Development Documentation" of this project is found under [📝 Development](#development).

## 🤝 Preamble

First off, thank you for considering contributing to this Project.

Following these guidelines helps to communicate that you respect the time of the developers managing and developing this open source project. In return, they should reciprocate that respect in addressing your issue, assessing changes, and helping you finalize your pull requests.

## 🍪 CookieCutter

This Project owns many of its files to [the CookieCutter it was originally templated from](https://github.com/JonasPammer/cookiecutter-ansible-role).

Please check if the edit you have in mind is actually applicable to the template and if so make an appropriate change there instead. Your change may also be applicable partly to the template as well as partly to something specific to this project, in which case you would be creating multiple PRs.

## 💬 Conventional Commits

A casual contributor does not have to worry about following [_the spec_](https://github.com/JonasPammer/JonasPammer/blob/master/demystifying/conventional_commits.adoc) [_by definition_](https://www.conventionalcommits.org/en/v1.0.0/), as pull requests are being squash merged into one commit in the project. Only core contributors, i.e. those with rights to push to this project’s branches, must follow it (e.g. to allow for automatic version determination and changelog generation to work).

## 🚀 Getting Started

Contributions are made to this repo via Issues and Pull Requests (PRs). A few general guidelines that cover both:

- Search for existing Issues and PRs before creating your own.

- If you’ve never contributed before, see [ the first timer’s guide on Auth0’s blog](https://auth0.com/blog/a-first-timers-guide-to-an-open-source-project/) for resources and tips on how to get started.

### Issues

Issues should be used to report problems, request a new feature, or to discuss potential changes **before** a PR is created. When you [ create a new Issue](https://github.com/JonasPammer/ansible-role-pip/issues/new), a template will be loaded that will guide you through collecting and providing the information we need to investigate.

If you find an Issue that addresses the problem you’re having, please add your own reproduction information to the existing issue **rather than creating a new one**. Adding a [reaction](https://github.blog/2016-03-10-add-reactions-to-pull-requests-issues-and-comments/) can also help be indicating to our maintainers that a particular problem is affecting more than just the reporter.

### Pull Requests

PRs to this Project are always welcome and can be a quick way to get your fix or improvement slated for the next release. [In general](https://blog.ploeh.dk/2015/01/15/10-tips-for-better-pull-requests/), PRs should:

- Only fix/add the functionality in question **OR** address wide-spread whitespace/style issues, not both.

- Add unit or integration tests for fixed or changed functionality (if a test suite already exists).

- **Address a single concern**

- **Include documentation** in the repo

- Be accompanied by a complete Pull Request template (loaded automatically when a PR is created).

For changes that address core functionality or would require breaking changes (e.g. a major release), it’s best to open an Issue to discuss your proposal first.

In general, we follow the "fork-and-pull" Git workflow

1.  Fork the repository to your own Github account

2.  Clone the project to your machine

3.  Create a branch locally with a succinct but descriptive name

4.  Commit changes to the branch

5.  Following any formatting and testing guidelines specific to this repo

6.  Push changes to your fork

7.  Open a PR in our repository and follow the PR template so that we can efficiently review the changes.

# 🗒 Changelog

Please refer to the [Release Page of this Repository](https://github.com/JonasPammer/ansible-role-pip/releases) for a human changelog of the corresponding [Tags (Versions) of this Project](https://github.com/JonasPammer/ansible-role-pip/tags).

Note that this Project adheres to Semantic Versioning. Please report any accidental breaking changes of a minor version update.

# ⚖️ License

**[LICENSE](LICENSE)**

    MIT License

    Copyright (c) 2022, Jonas Pammer

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
