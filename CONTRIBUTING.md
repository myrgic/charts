# Contributing to CogOS Charts

Thanks for your interest. This repo holds Helm charts and deployment manifests for running CogOS on Kubernetes. The charts are **experimental** (see README) — PRs that move them toward production-ready are especially welcome.

## Development setup

```sh
git clone https://github.com/myrgic/charts.git
cd charts
helm version  # 3.13+
```

Requirements: Helm 3.13+, optionally kind or minikube for local testing.

## Validating changes

Before opening a PR, lint every chart you touch:

```sh
helm lint charts/cogos-node
helm lint charts/cogos-kernel
helm lint charts/cogos-mod3
```

And render the templates against the default values to catch template errors early:

```sh
helm template test charts/cogos-node --debug
```

If you have a cluster handy, a dry-run install is the highest-confidence check:

```sh
helm install --dry-run --debug test charts/cogos-node
```

## Chart layout

- `charts/cogos-node/` — umbrella chart (the primary entry point)
- `charts/cogos-kernel/` — kernel-only subchart
- `charts/cogos-mod3/` — mod³ voice subchart

Each chart has its own `values.yaml`, `templates/`, and `Chart.yaml`.

## Submitting changes

1. Fork the repo and create a branch from `main`
2. Make your changes and bump the chart `version` in `Chart.yaml` if the contract changes
3. Run `helm lint` + template dry-run
4. Note the change in `CHANGELOG.md` (add one if missing) under the Unreleased section
5. Open a pull request using the org PR template — include the commands you used to validate

## Reporting issues

Use the org-level [Bug Report](https://github.com/myrgic/charts/issues/new?template=bug.yml) or [Feature Request](https://github.com/myrgic/charts/issues/new?template=feature.yml) forms. Please note your cluster version and Helm version.

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](LICENSE).
