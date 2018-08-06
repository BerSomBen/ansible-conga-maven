# wcm_io_devops.conga_maven

This role generates the configuration for an [CONGA](http://devops.wcm.io/conga/) environment by running Maven.
The role can either use a configuration definition (e.g. in the same repository as the Ansible setup) or a configuration Git repository which the role with clone and update as required.

## Requirements

This role requires Maven to be installed and optionally Git if you want the role to use a Git repository for the configuration. Both Maven and Git need to be properly pre-configured to be able to access the repositories and artifacts used in the CONGA setup.
Currently this role has to be run on the Ansible host (i.e. localhost) since [wcm_io_devops.conga_facts](https://github.com/wcm-io-devops/ansible-conga-facts) is implemented as an action plugin which is always executed locally. So for the plugin to be able to read the configuration generated by this role, this role needs to run locally as well.

## Role Variables

Available variables are listed below, along with their default values (see `defaults/main.yml`):

    # Name of the CONGA environment to generate the configuration for
    conga_environment

By default the role uses the `conga_environment` variable to only compile the configuration for the environment specified by this variable.
However, the general idea is that you use this variable throughout your Ansible-CONGA setup to target a single environment.

    # Base directory of the CONGA configuration
    conga_basedir

When not using a Git configuration repository, the `conga_basedir` variable has to point to the base directory of the CONGA configuration definition (where the pom.xml is located). Otherwise it is automatically build from the `conga_maven_git_root` and `conga_maven_root` variables.

    # URI of the configuration Git repo
    conga_maven_git_repo: "https://github.com/wcm-io-devops/conga.git"
    # Directory to clone the configuration into
    conga_maven_git_root: /tmp/git
    # Branch to checkout
    conga_maven_git_branch: develop

If you want to use a Git configuration repository, you need to at least set the `conga_maven_git_repo` variable. If it is not set the Git part of the role will be skipped and you have to set `conga_basedir` variable manually.

    # Root path of the CONGA configuration
    conga_maven_root: configuration
    # Maven command to execute
    conga_maven_cmd: mvn
    # Maven options (run in batch mode, update snapshots and only compile a single environment)
    conga_maven_opts: "-B -U -Dconga.environments={{ conga_environment }}"
    # Path of a custom settings file to use when running Maven
    conga_maven_settings: ~/.m2/settings.xml

These variables let you customize the way Maven is executed, e.g. supplying a full path for the Maven executable or supplying a custom settings file.

Dependencies
------------

This role doesn't directly depend on other roles but it's supposed to be used in combination with the [wcm_io_devops.conga_facts](https://github.com/wcm-io-devops/ansible-conga-facts) role/plugin which parses the CONGA configuration model and exposes it as Ansible variables. For this to work the `conga_basedir` variable needs to point to the root of the CONGA configuration directory.

Example Playbook
----------------

1) Generate [CONGA example configuration](https://github.com/wcm-io-devops/conga/tree/develop/example) for environment `prod`:

        - hosts: localhost
          vars:
            conga_environment: prod
          roles:
            - { role: wcm_io_devops.conga_maven,
                conga_maven_git_repo: "https://github.com/wcm-io-devops/conga.git",
                conga_maven_git_branch: master,
                conga_maven_root: example }

2) Generate CONGA configuration for environment `dev` from local directory `config-definition`:

        - hosts: localhost
          vars:
            conga_environment: dev
          roles:
            - { role: wcm_io_devops.conga_maven,
                conga_basedir: config-definition }