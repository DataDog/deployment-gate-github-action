# Datadog Deployment Gate GitHub Action

Deployment Gates allow you to reduce the likelihood and impact of incidents caused by deployments. This action provides a simple way to integrate Datadog Deployment Gates into your CI/CD pipeline.

You can learn more in the Deployment Gates documentation: https://docs.datadoghq.com/deployment_gates/

## Prerequisites

Before using this action, you need to:

1. Set up Deployment Gates in your Datadog account. If you have not, join the preview here: https://www.datadoghq.com/product-preview/deployment-gates/
2. Have a Datadog API key
3. Have a Datadog Application key with at least the `deployment_gates_evaluate` scope

## Usage

### Basic Example

This example evaluates a gate using an inline configuration file, without requiring any prior setup in Datadog.

```yaml
name: Deploy with Datadog Deployment Gate

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v5

      - name: Deploy Canary
        run: |
          echo "Deploying canary release for service:'my-service' in 'production'. Version 1.0.1"
          # Your deployment commands here

      - name: Evaluate Deployment Gate
        uses: DataDog/deployment-gate-github-action@v1.0.0
        env:
          DD_API_KEY: ${{ secrets.DD_API_KEY }}
          DD_APP_KEY: ${{ secrets.DD_APP_KEY }}
        with:
          service: my-service
          env: production
          config: .github/gate-config.json
      
      - name: Deploy
        run: |
          echo "Deployment Gate passed, proceeding with deployment"
          # Your deployment commands here
```

With `.github/gate-config.json`:

```json
{
  "dryRun": false,
  "rules": [
    {
      "type": "monitor",
      "name": "error rate monitors",
      "options": {
        "query": "service:payments-backend env:prod",
        "duration": 300
      }
    }
  ]
}
```

### Example with pre-configured gate

This example evaluates a gate that has been created beforehand. The gate must exist in Datadog before running this workflow — it can be created via the [Datadog UI](https://app.datadoghq.com/ci/deployment-gates), [API](https://docs.datadoghq.com/api/latest/deployment-gates/), or [Terraform](https://registry.terraform.io/providers/DataDog/datadog/latest/docs).

```yaml
name: Deploy with Datadog Deployment Gate

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v5

      - name: Deploy Canary
        run: |
          echo "Deploying canary release for service:'my-service' in 'production'. Version 1.0.1"
          # Your deployment commands here

      - name: Evaluate Deployment Gate
        uses: DataDog/deployment-gate-github-action@v1.0.0
        env:
          DD_API_KEY: ${{ secrets.DD_API_KEY }}
          DD_APP_KEY: ${{ secrets.DD_APP_KEY }}
        with:
          service: my-service
          env: production
          identifier: default
      
      - name: Deploy
        run: |
          echo "Deployment Gate passed, proceeding with deployment"
          # Your deployment commands here
```


## Environment Variables

The following environment variables are required:

| Variable | Description | Required |
|----------|-------------|----------|
| `DD_API_KEY` | Datadog API key | ✅ |
| `DD_APP_KEY` | Datadog application key | ✅ |
| `DD_SITE` | [Datadog site](https://docs.datadoghq.com/getting_started/site) (e.g., `datadoghq.com`, `datadoghq.eu`) | ❌ (defaults to `datadoghq.com`) |

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `service` | Name of the service being deployed | ✅ | |
| `env` | Deployment environment (e.g., staging, production) | ✅ | |
| `version` | Version being deployed (required for APM Faulty Deployment Detection rules) | ❌ | |
| `identifier` | Custom identifier for the Deployment Gate evaluation | ❌ | |
| `apm-primary-tag` | Primary tag to scope down APM analysis for APM Faulty Deployment Detection rules (e.g., `region:us-central-1`) | ❌ | |
| `timeout` | Command timeout in seconds | ❌ | |
| `fail-on-error` | When true, the script will consider the gate as failed when timeout is reached or unexpected errors occur calling the Datadog APIs. | ❌ | `false` |
| `config` | Path to a JSON file containing gate rule definitions. See the [setup documentation](https://docs.datadoghq.com/deployment_gates/setup) for the expected format. | ❌ | |
| `datadog-ci-version` | Version of datadog-ci to install. Use a major version like `v5` to get the latest release within that major version, or a specific tag like `v5.6.0` to pin. | ❌ | `v5` |


## Outputs

This action uses the native `datadog-ci` command which handles all output internally. The action will:
- ✅ **Succeed** if the Deployment Gate passes
- ❌ **Fail** if the Deployment Gate fails or encounters an error

This action provides detailed output in the logs, including:
- Gate evaluation status
- Individual rule results
- Links to view results in Datadog UI
- Evaluation timing and polling information

## Error Handling

The action leverages the native `datadog-ci` error handling:
- ✅ **Pass** if the Deployment Gate evaluation succeeds
- ❌ **Fail** if the Deployment Gate evaluation fails
- ❌ **Fail** if there are authentication or configuration errors
- 🔄 **Automatic polling** until evaluation completes
- ⏱️ **Built-in timeout handling** with sensible defaults

## Troubleshooting

### Common Issues

1. **Authentication Errors**
   - Verify your `DD_API_KEY` and `DD_APP_KEY` are correct
   - Ensure the Application Key have the necessary `deployment_gates_evaluate` permission
   - Ensure you have access to the Deployment Gates preview: https://app.datadoghq.com/ci/deployment-gates/getting-started

2. **Gate Not Found**
   - Verify the service name and environment match your Datadog configuration
   - Check that Deployment Gates are properly configured in Datadog


## Contributing

To contribute to this project, see the [CONTRIBUTING](CONTRIBUTING.md) file.


## License

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.

## Support

For issues related to:
- **This GitHub Action**: Open an issue in this repository
- **Datadog Deployment Gates**: Contact [Datadog Support](https://help.datadoghq.com/)
- **datadog-ci CLI**: Check the [datadog-ci repository](https://github.com/DataDog/datadog-ci)
