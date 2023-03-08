# get-vault-token-for-azure-user

A GitHub Action for retrieving an authentication token for [HashiCorp Vault](https://www.vaultproject.io/) using [Vault's Azure Auth Method](https://www.vaultproject.io/docs/auth/azure). It can be used in conjunction with [HashiCorp's vault-action](https://github.com/hashicorp/vault-action) to retrieve secrets.

Prerequisites:

1. Azure Auth enabled on the Vault Server.
2. A logged in Azure user that is able to authenticate as the provided Vault role.

## Index

- [get-vault-token-for-azure-user](#get-vault-token-for-azure-user)
  - [Index](#index)
  - [Inputs](#inputs)
  - [Outputs](#outputs)
  - [Example](#example)
  - [Contributing](#contributing)
    - [Incrementing the Version](#incrementing-the-version)
  - [Code of Conduct](#code-of-conduct)
  - [License](#license)
    

## Inputs
| Parameter                          | Is Required | Default | Description                                                                                                        |
| ---------------------------------- | ----------- | ------- | ------------------------------------------------------------------------------------------------------------------ |
| `vault-role`                       | true        | N/A     | The role in Vault that the outputted auth token will be provided for.                                              |
| `vault-url`                        | true        | N/A     | The base url where Vault is hosted. E.g. https://vault.myvault.com:8200                                            |
| `azure-auth-mount-path`            | false       | azure   | The path where Vault's Azure Auth Method was mounted. This should be 'azure' unless otherwise configured in Vault. |
| `output-environment-variable-name` | false       | N/A     | If set, an environment variable with the provided name will be outputted in addition to the normal output.         |

## Outputs
| Output               | Description                                                                     |
| -------------------- | ------------------------------------------------------------------------------- |
| `vault_client_token` | A Vault token that can be used in subsequent jobs/steps to interact with Vault. |

## Example

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

When creating new PRs please ensure:

1. For major or minor changes, at least one of the commit messages contains the appropriate `+semver:` keywords listed under [Incrementing the Version](#incrementing-the-version).
1. The action code does not contain sensitive information.

When a pull request is created and there are changes to code-specific files and folders, the `auto-update-readme` workflow will run.  The workflow will update the action-examples in the README.md if they have not been updated manually by the PR author. The following files and folders contain action code and will trigger the automatic updates:

- `action.yml`

There may be some instances where the bot does not have permission to push changes back to the branch though so this step should be done manually for those branches. See [Incrementing the Version](#incrementing-the-version) for more details.

### Incrementing the Version

The `auto-update-readme` and PR merge workflows will use the strategies below to determine what the next version will be.  If the `auto-update-readme` workflow was not able to automatically update the README.md action-examples with the next version, the README.md should be updated manually as part of the PR using that calculated version.

This action uses [git-version-lite] to examine commit messages to determine whether to perform a major, minor or patch increment on merge.  The following table provides the fragment that should be included in a commit message to active different increment strategies.
| Increment Type | Commit Message Fragment                     |
| -------------- | ------------------------------------------- |
| major          | +semver:breaking                            |
| major          | +semver:major                               |
| minor          | +semver:feature                             |
| minor          | +semver:minor                               |
| patch          | *default increment type, no comment needed* |

## Code of Conduct

This project has adopted the [im-open's Code of Conduct](https://github.com/im-open/.github/blob/master/CODE_OF_CONDUCT.md).

## License

Copyright &copy; 2021, Extend Health, LLC. Code released under the [MIT license](LICENSE).

[git-version-lite]: https://github.com/im-open/git-version-lite
