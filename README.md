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

Plain galaxy roles like `geerlingguy.docker` need no `requirements.yml` — just list them under `roles:` in `config.yml`. Reach for `requirements.yml` only when you need to pin a version or pull from GitHub/GitLab.

## More

Full reference: [Blueprint Sources](https://docs.ludus.cloud/docs/using-ludus/blueprint-sources).
