# Datadog Deployment Gate GitHub Action

A GitHub Action that evaluates Datadog deployment gates to ensure deployment quality and safety. This action provides a simple way to integrate Datadog deployment gates into your CI/CD pipeline.

You can learn more in the Deployment Gates documentation: https://docs.datadoghq.com/deployment_gates/

## Prerequisites

Before using this action, you need to:

1. Set up deployment gates in your Datadog account. If you have not, join the preview here: https://www.datadoghq.com/product-preview/deployment-gates/
2. Have a Datadog API key and Application key.
3. Configure your deployment gates for your services and environments on the Datadog UI.

## Usage

### Basic Example

```yaml
name: Deploy with Datadog Gate

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
    
        - name: Deploy Canary
        run: |
          echo "Deploying canary release for service:'my-service' in 'production'. Version 1.0.1"
          # Your deployment commands here

      - name: Evaluate Deployment Gate
        uses: your-org/deployment-gate-github-action@v1
        env:
          DD_API_KEY: ${{ secrets.DD_API_KEY }}
          DD_APP_KEY: ${{ secrets.DD_APP_KEY }}
        with:
          service: 'my-service'
          env: 'production'
          version: '1.0.1'
      
      - name: Deploy
        if: success()
        run: |
          echo "Deployment gate passed, proceeding with deployment"
          # Your deployment commands here
```


## Environment Variables

The following environment variables are required:

| Variable | Description | Required |
|----------|-------------|----------|
| `DD_API_KEY` | Datadog API key | ✅ |
| `DD_APP_KEY` | Datadog application key | ✅ |
| `DD_SITE` | Datadog site (e.g., `datadoghq.com`, `datadoghq.eu`) | ❌ (defaults to `datadoghq.com`) |

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `service` | Name of the service being deployed | ✅ | |
| `env` | Deployment environment (e.g., staging, production) | ✅ | |
| `version` | Version being deployed (required for APM Faulty Deployment Detection rules) | ❌ | |
| `identifier` | Custom identifier for the deployment gate evaluation | ❌ | |
| `primary-tag` | Primary tag to scope down APM analysis for APM Faulty Deployment Detection rules (e.g., `region:us-central-1`) | ❌ | |
| `timeout` | Maximum time to wait for the script execution in seconds | ❌ | `10800` (3 hours) |
| `fail-on-error` | When false, the script will consider the gate as passed and exit with code 0 when timeout is reached or unexpected errors occur | ❌ | `false` |


### Datadog Sites

The `DD_SITE` environment variable supports the following values:
- `datadoghq.com` (US1) - default
- `us3.datadoghq.com` (US3)
- `us5.datadoghq.com` (US5)
- `datadoghq.eu` (EU1)
- `ap1.datadoghq.com` (AP1)
- `ddog-gov.com` (US1-FED)

## Outputs

This action uses the native `datadog-ci` command which handles all output internally. The action will:
- ✅ **Succeed** if the deployment gate passes
- ❌ **Fail** if the deployment gate fails or encounters an error

The `datadog-ci` command provides detailed output in the action logs, including:
- Gate evaluation status
- Individual rule results
- Links to view results in Datadog UI
- Evaluation timing and polling information

## Error Handling

The action leverages the native `datadog-ci` error handling:
- ✅ **Pass** if the deployment gate evaluation succeeds
- ❌ **Fail** if the deployment gate evaluation fails
- ❌ **Fail** if there are authentication or configuration errors
- 🔄 **Automatic polling** until evaluation completes (handled by datadog-ci)
- ⏱️ **Built-in timeout handling** with sensible defaults

## Troubleshooting

### Common Issues

1. **Authentication Errors**
   - Verify your `DD_API_KEY` and `DD_APP_KEY` are correct
   - Ensure the keys have the necessary permissions
   - Ensure you have access to the Deployment Gates preview

2. **Gate Not Found**
   - Verify the service name and environment match your Datadog configuration
   - Check that deployment gates are properly configured in Datadog


## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For issues related to:
- **This GitHub Action**: Open an issue in this repository
- **Datadog Deployment Gates**: Contact [Datadog Support](https://help.datadoghq.com/)
- **datadog-ci CLI**: Check the [datadog-ci repository](https://github.com/DataDog/datadog-ci)
