# Example registry

Template for a GitHub repository that **hosts** a private CodeArtifact domain: one `registry.yaml`, no application code, not an environment BOM.

Open **[renglo/example-registry](https://github.com/renglo/example-registry)** on GitHub, click **Use this template**, and name the copy something like `<org>-publisher` (for example `apollo-publisher`). That name is yours; the CLI never reads it.

Then edit `registry.yaml` and deploy with `renglo` from [renglo-ops](https://github.com/renglo/renglo-ops). Field-by-field: [configuration.md](https://github.com/renglo/renglo-ops/blob/main/docs/configuration.md#registryyaml). The operator walkthrough is [project-2-registry.md](https://github.com/renglo/renglo-ops/blob/main/docs/project-2-registry.md).

## What this repo is for

| This repository | Not this repository |
| --- | --- |
| Desired state of **one** CodeArtifact stack | Environment pins (`renglo.yaml` lives in a `*-bom` repo) |
| Which GitHub repos may **publish** | Which domains an environment **reads** (`registries` in `renglo.yaml`) |
| AWS accounts allowed to install (`reader_accounts`) | Product source (`data`, `schd`, …) |

One registry can serve many environments. Keep this file out of a BOM checkout.

## `name` vs the GitHub repo

| | Example |
| --- | --- |
| GitHub repo | `acmeco/acme-publisher` |
| `name` in `registry.yaml` | `acme` |
| CloudFormation stack | `acme-publisher` |

If `name` is `acme-publisher`, the stack is `acme-publisher-publisher`. Keep `name` short.

`github_org` is the org of the **product** repos that publish, not necessarily this repo.

## After you copy the template

1. Replace `CHANGE_ME` values in `registry.yaml`.
2. Commit and push.
3. From a machine with `renglo` installed and an AWS profile for the **registry** account:

```bash
source ops/renglo-ops/.venv/bin/activate
export AWS_PROFILE=<registry-profile>
renglo registry deploy --registry /path/to/your-publisher/registry.yaml --dry-run
renglo registry deploy --registry /path/to/your-publisher/registry.yaml
```

Store the path in `.renglo/local.yaml` as `registry:` if you want to omit `--registry` later. See [project-2-registry.md](https://github.com/renglo/renglo-ops/blob/main/docs/project-2-registry.md#deploy-it).

Publishing a package is a git tag on a **product** repo, after `renglo registry connect` and GitHub Actions variables. This repository does not hold extension source.

## git-convoy

This checkout declares `role = "registry"` in `gitconvoy.toml` so a workspace that also contains product repos does not treat it as:

- **product** — feature sheets and release trains
- **ops** — `git convoy ops` (that is `renglo-ops`, not this file)
- **bom** — `git convoy bom` (that is `*-bom`)

No new train or ops commands. The role only keeps membership correct.
