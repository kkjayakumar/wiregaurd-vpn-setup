# Setup Instructions for Prometheus-MSTeams Notification Integration

This document outlines the steps required to set up Prometheus with Microsoft Teams notifications using the Prometheus-MSTeams integration.

## Step 1: Add the Prometheus-MSTeams Helm Repository

To begin, you need to add the Prometheus-MSTeams Helm repository to your Helm configuration. Run the following command:

```bash
helm repo add prometheus-msteams https://prometheus-msteams.github.io/prometheus-msteams/
```

## Step 2: Install Prometheus-MSTeams

You can install the Prometheus-MSTeams integration using Helm. Use the command below to install it with your custom values file:

```bash
helm install prometheus-msteams prometheus-msteams/prometheus-msteams -f values.yaml -n monitoring
```

Alternatively, if you need to upgrade an existing installation, you can use:

```bash
helm upgrade --install msteams prometheus-msteams/prometheus-msteams -n monitoring
```

## Step 3: Create the Secret for Microsoft Teams Webhook

You need to create a Kubernetes secret to store your Microsoft Teams webhook URL. Use the following command to create the secret:

```bash
kubectl create secret generic alertmanager-teams-secret --from-literal=webhook_url=https://*.webhook.office.com/webhookb2/*
```

## Step 4: Update Prometheus Configuration

You will need to update the `values.yaml` file to include the necessary configuration for the Prometheus-MSTeams integration. Ensure that the webhook URL is correctly referenced in your configuration.

Alternatively, you can update the existing secret in Prometheus by modifying the key named `alertmanager.yaml`.

If your Alertmanager and prometheus-msteams are in different namespaces, use the full DNS name:

```bash
url: "http://prometheus-msteams.monitoring.svc.cluster.local:2000/teams"
```

otherwise 
```bash
url: "http://prometheus-msteams:2000/teams"
```

## Step 5: Restart the Alertmanager Stateful Set

After making the necessary updates, restart the Alertmanager Stateful Set to apply the changes. Use the following command:

```bash
kubectl rollout restart statefulset alertmanager-stable-kube-prometheus-sta-alertmanager -n monitoring
```

## Conclusion

Following these steps will set up the Prometheus-MSTeams integration, allowing you to receive alerts in Microsoft Teams. Make sure to monitor the integration for any issues and adjust the configuration as necessary.