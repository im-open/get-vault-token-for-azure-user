# get-vault-token-for-azure-user

A GitHub Action for retrieving an authentication token for [HashiCorp Vault](https://www.vaultproject.io/) using [Vault's Azure Auth Method](https://www.vaultproject.io/docs/auth/azure). It can be used in conjunction with [HashiCorp's vault-action](https://github.com/hashicorp/vault-action) to retrieve secrets.

Prerequisites:

1. Azure Auth enabled on the Vault Server.
2. A logged in Azure user that is able to authenticate as the provided Vault role.

## Index <!-- omit in toc -->

- [get-vault-token-for-azure-user](#get-vault-token-for-azure-user)
  - [Inputs](#inputs)
  - [Outputs](#outputs)
  - [Usage Examples](#usage-examples)
  - [Contributing](#contributing)
    - [Incrementing the Version](#incrementing-the-version)
    - [Source Code Changes](#source-code-changes)
    - [Updating the README.md](#updating-the-readmemd)
  - [Code of Conduct](#code-of-conduct)
  - [License](#license)

## Inputs

| Parameter                          | Is Required | Default | Description                                                                                                        |
|------------------------------------|-------------|---------|--------------------------------------------------------------------------------------------------------------------|
| `vault-role`                       | true        | N/A     | The role in Vault that the outputted auth token will be provided for.                                              |
| `vault-url`                        | true        | N/A     | The base url where Vault is hosted. E.g. <https://vault.myvault.com:8200>                                          |
| `azure-auth-mount-path`            | false       | azure   | The path where Vault's Azure Auth Method was mounted. This should be 'azure' unless otherwise configured in Vault. |
| `output-environment-variable-name` | false       | N/A     | If set, an environment variable with the provided name will be outputted in addition to the normal output.         |

## Outputs

| Output               | Description                                                                     |
|----------------------|---------------------------------------------------------------------------------|
| `vault_client_token` | A Vault token that can be used in subsequent jobs/steps to interact with Vault. |

## Usage Examples

```yml
jobs:
  get-vault-secrets:
    runs-on: ubuntu-20.04
    steps:
      - name: AZ Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_LOGIN_CREDENTIALS }}

      - name: Get Vault Token
        id: vault_token
        # You may also reference just the major or major.minor version
        uses: im-open/get-vault-token-for-azure-user@v1.1.1
        with:
          vault-role: 'my-role'
          vault-url: 'https://vault.myvault.com:8200'
          azure-auth-mount-path: 'azure'
          output-environment-variable-name: 'VAULT_CLIENT_TOKEN'

      
      - name: Import Secrets
        id: vault-secrets
        uses: hashicorp/vault-action@v2.4.0
        with:
          url: 'https://vault.myvault.com:8200'
          token: '${{ steps.vault_token.outputs.client_token }}' # you could also use ${{ env.VAULT_CLIENT_TOKEN }} since output-environment-variable-name is set in the vault_token step
          secrets: |
            secret/data/ci/auth-creds username ;
            secret/data/ci/auth-creds password

```

## Contributing

When creating PRs, please review the following guidelines:

- [ ] The action code does not contain sensitive information.
- [ ] At least one of the commit messages contains the appropriate `+semver:` keywords listed under [Incrementing the Version] for major and minor increments.
- [ ] The README.md has been updated with the latest version of the action.  See [Updating the README.md] for details.

### Incrementing the Version

This repo uses [git-version-lite] in its workflows to examine commit messages to determine whether to perform a major, minor or patch increment on merge if [source code] changes have been made.  The following table provides the fragment that should be included in a commit message to active different increment strategies.

| Increment Type | Commit Message Fragment                     |
|----------------|---------------------------------------------|
| major          | +semver:breaking                            |
| major          | +semver:major                               |
| minor          | +semver:feature                             |
| minor          | +semver:minor                               |
| patch          | *default increment type, no comment needed* |

### Source Code Changes

The files and directories that are considered source code are listed in the `files-with-code` and `dirs-with-code` arguments in both the [build-and-review-pr] and [increment-version-on-merge] workflows.  

If a PR contains source code changes, the README.md should be updated with the latest action version.  The [build-and-review-pr] workflow will ensure these steps are performed when they are required.  The workflow will provide instructions for completing these steps if the PR Author does not initially complete them.

If a PR consists solely of non-source code changes like changes to the `README.md` or workflows under `./.github/workflows`, version updates do not need to be performed.

### Updating the README.md

If changes are made to the action's [source code], the [usage examples] section of this file should be updated with the next version of the action.  Each instance of this action should be updated.  This helps users know what the latest tag is without having to navigate to the Tags page of the repository.  See [Incrementing the Version] for details on how to determine what the next version will be or consult the first workflow run for the PR which will also calculate the next version.

## Code of Conduct

This project has adopted the [im-open's Code of Conduct](https://github.com/im-open/.github/blob/main/CODE_OF_CONDUCT.md).

## License

Copyright &copy; 2023, Extend Health, LLC. Code released under the [MIT license](LICENSE).

<!-- Links -->
[Incrementing the Version]: #incrementing-the-version
[Updating the README.md]: #updating-the-readmemd
[source code]: #source-code-changes
[usage examples]: #usage-examples
[build-and-review-pr]: ./.github/workflows/build-and-review-pr.yml
[increment-version-on-merge]: ./.github/workflows/increment-version-on-merge.yml
[git-version-lite]: https://github.com/im-open/git-version-lite
