# Prometheus-MSTeams Alert Integration

This project provides a setup for integrating Prometheus alerts with Microsoft Teams notifications using the Prometheus-MSTeams integration. The integration allows you to receive alerts directly in your Microsoft Teams channels, enhancing your monitoring and response capabilities.

## Overview

The integration involves the following key steps:

1. **Adding the Helm Repository**: This step involves adding the Prometheus-MSTeams Helm repository to your local Helm configuration.
2. **Installing the Prometheus-MSTeams Chart**: You will install the Prometheus-MSTeams chart using Helm, which sets up the necessary components in your Kubernetes cluster.
3. **Creating Secrets**: A Kubernetes secret will be created to store the Microsoft Teams webhook URL securely.
4. **Updating Configuration**: You will update the Prometheus configuration to include the alertmanager settings for sending notifications to Microsoft Teams.
5. **Restarting the Stateful Set**: Finally, the Alertmanager Stateful Set will be restarted to apply the new configuration.

## Setup Instructions

For detailed setup instructions, please refer to the `src/setup.md` file.

## Configuration

The `values.yaml` file is used to configure the Prometheus-MSTeams deployment. Ensure that you specify the correct webhook URL and any other relevant parameters in this file.

## Troubleshooting

If you encounter any issues during the setup or operation of the integration, please check the logs of the Alertmanager and the Prometheus-MSTeams deployment for any error messages. You can also refer to the official documentation for additional troubleshooting tips.

## License

This project is licensed under the MIT License. See the LICENSE file for more details.