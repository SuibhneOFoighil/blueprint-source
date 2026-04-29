# Ludus Blueprint Source Template

A blueprint source is an external git repo that ships [Ludus](https://docs.ludus.cloud) range configs (blueprints) and the Ansible roles and Packer templates they need. This repo is a starting point for publishing your own.

Click **Use this template**, edit the files below, push, then run:

```bash
ludus blueprint source add https://github.com/<you>/<repo>
ludus blueprint apply <repo>/example
ludus range deploy
```

## Files

```
LICENSE                           MIT placeholder — replace with your own
source.yml                        repo metadata: name, maintainer, homepage
blueprints/example/
  blueprint.yml                   one blueprint's display metadata
  config.yml                      the range config — same shape `ludus range config get` returns
scripts/validate.py               manifest schema check; extend with your own rules
.github/workflows/validate.yml    runs scripts/validate.py on every push
```

Add when you need them:

```
blueprints/<id>/requirements.yml  pinned versions, or roles hosted off galaxy.ansible.com
blueprints/<id>/templates/<n>/    custom OS image (Packer config) only this blueprint uses
blueprints/<id>/roles/<n>/        local Ansible role only this blueprint uses
templates/<n>/                    custom OS image shared across blueprints in this source
roles/<n>/                        local Ansible role shared across blueprints in this source
```

Plain galaxy roles like `geerlingguy.docker` need no `requirements.yml` — just list them under `roles:` in `config.yml`. Reach for `requirements.yml` only when you need to pin a version or pull from GitHub/GitLab:

```yaml
# blueprints/<id>/requirements.yml
roles:
  - name: geerlingguy.docker
    version: 7.4.4                                  # pin a galaxy role
  - name: badsectorlabs.ludus_adcs                  # off-galaxy: name + src
    src: https://github.com/badsectorlabs/ludus_adcs
    version: v1.2.0
```

Names must match what `roles:` in `config.yml` references — otherwise Ludus installs one role and tries to use another.

### Custom Packer templates

Each `templates/<name>/` directory is a standard Ludus Packer template — the same shape as the [builtin templates](https://gitlab.com/badsectorlabs/ludus/-/tree/main/ludus-server/packer):

```
templates/my-debian-base/
  my-debian-base.pkr.hcl   the Packer build config
  http/                    Linux: preseed.cfg / kickstart served at install time
  Autounattend.xml         Windows only: unattended install answer file
```

After `ludus blueprint source add`, run `ludus templates build` to produce the actual VM image.

### Custom Ansible roles

Each `roles/<name>/` directory is a standard [Ansible role](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html):

```
roles/my_helper/
  tasks/main.yml           the role's tasks (typical entry point)
  defaults/main.yml        optional — default variables
  handlers/main.yml        optional — handlers
  meta/main.yml            optional — role metadata, dependencies
```

Reference the role by directory name (`my_helper`) under `roles:` in `config.yml`. If a local role shares a name with a galaxy role, Ludus skips the galaxy install entirely — only the local role gets installed.

## More

Full reference: [Blueprint Sources](https://docs.ludus.cloud/docs/using-ludus/blueprint-sources).
