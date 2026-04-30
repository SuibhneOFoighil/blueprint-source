# Ludus Blueprint Source Template

A **blueprint source** is an external repo that ships one or more [Ludus](https://docs.ludus.cloud) **blueprints** — range configs plus the Ansible roles and Packer templates they need. One repo, many labs. This template is a starting point for publishing your own.

Click **Use this template**, edit the files below, push, then run:

```bash
ludus blueprint source add https://github.com/<you>/<repo>
ludus blueprint apply <repo>/example
ludus range deploy
```

Any git host works (GitHub, GitLab, self-hosted). You can also feed `source add` a local tarball/zip (`source add ./source.tar.gz`) or a local directory (`source add -d ./my-source`) — see the [docs](https://docs.ludus.cloud/docs/using-ludus/blueprint-sources) for the full list.

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

Each `templates/<name>/` directory is a standard Ludus Packer template — the same shape as the [Ludus template catalog](https://gitlab.com/badsectorlabs/ludus/-/tree/main/templates):

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

## Required fields

The validator and the server enforce these:

- **`source.yml`** — `manifest_version`. Everything else is optional. The whole file is optional too.
- **`blueprint.yml`** — `manifest_version`, `id`, `name`, `description`, `version` (semver), `config`. Optional: `author`, `homepage`, `license`, `tags`, `thumbnail`, `min_ludus_version`.

The example files have these annotated inline.

## Versioning

Two separate fields:

- **`manifest_version`** is the schema version of the manifest file. Ludus bumps it when the format changes incompatibly. Leave it at `1`.
- **`version`** is *your* semver for the blueprint. Bump it any time you change a blueprint and want users to see it as new. Push to your repo, then users run:

```bash
ludus blueprint source sync <repo>     # pull latest manifests + reinstall any new role deps
ludus blueprint info <repo>/example    # see the new version
ludus blueprint apply <repo>/example   # write the new config to their range
ludus range deploy                     # rebuild
```

`ludus blueprint apply` always writes whatever's currently in the source — there's no automatic upgrade prompt. The `version` field is for display and changelog purposes; pin it to a git tag (`source update <repo> --ref v1.2.0`) if you need users locked to a specific release.

## More

Full reference: [Blueprint Sources](https://docs.ludus.cloud/docs/using-ludus/blueprint-sources).
