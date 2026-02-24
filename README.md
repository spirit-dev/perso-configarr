# Configarr Helm Chart

This Helm chart deploys [Configarr](https://configarr.de) on a Kubernetes cluster using the Helm package manager.



<!--TOC-->

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installing the Chart](#installing-the-chart)
- [Uninstalling the Chart](#uninstalling-the-chart)
- [Parameters](#parameters)
  - [Global Parameters](#global-parameters)
  - [CronJob Parameters](#cronjob-parameters)
  - [Configuration Parameters](#configuration-parameters)
  - [Persistence Parameters](#persistence-parameters)
  - [Security Parameters](#security-parameters)
  - [Resource Parameters](#resource-parameters)
  - [Node Selection Parameters](#node-selection-parameters)
- [Configuration and Installation Details](#configuration-and-installation-details)
  - [Configuring Configarr](#configuring-configarr)
  - [Managing Secrets](#managing-secrets)
  - [Persistence](#persistence)
  - [CronJob Schedule](#cronjob-schedule)
- [Troubleshooting](#troubleshooting)
  - [Viewing Logs](#viewing-logs)
  - [Manual Job Execution](#manual-job-execution)
  - [Common Issues](#common-issues)
- [License](#license)
- [Links](#links)

<!--TOC-->
## Introduction

Configarr is a configuration management tool for Arr applications (Sonarr, Radarr, etc.) that helps you maintain consistent quality profiles and custom formats across your media automation stack.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure (if persistence is enabled)

## Installing the Chart

To install the chart with the release name `configarr`:

```bash
helm install configarr ./configarr
```

The command deploys Configarr on the Kubernetes cluster with the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `configarr` deployment:

```bash
helm uninstall configarr
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Parameters

### Global Parameters

| Name                      | Description                                     | Value                                  |
| ------------------------- | ----------------------------------------------- | -------------------------------------- |
| `image.repository`        | Configarr image repository                      | `ghcr.io/raydak-labs/configarr`        |
| `image.pullPolicy`        | Image pull policy                               | `Always`                               |
| `image.tag`               | Overrides the image tag (default is appVersion) | `""`                                   |
| `imagePullSecrets`        | Docker registry secret names as an array        | `[]`                                   |
| `nameOverride`            | String to partially override configarr.fullname | `""`                                   |
| `fullnameOverride`        | String to fully override configarr.fullname     | `""`                                   |

### CronJob Parameters

| Name                                    | Description                                      | Value         |
| --------------------------------------- | ------------------------------------------------ | ------------- |
| `cronjob.schedule`                      | Cron schedule for running Configarr              | `"0 * * * *"` |
| `cronjob.successfulJobsHistoryLimit`    | Number of successful jobs to keep                | `1`           |
| `cronjob.failedJobsHistoryLimit`        | Number of failed jobs to keep                    | `1`           |
| `cronjob.concurrencyPolicy`             | How to handle concurrent executions              | `Forbid`      |
| `cronjob.restartPolicy`                 | Restart policy for the pod                       | `Never`       |
| `cronjob.tty`                           | Enable TTY for color support in logs             | `true`        |

### Configuration Parameters

| Name                  | Description                                      | Value                                  |
| --------------------- | ------------------------------------------------ | -------------------------------------- |
| `config.configYml`    | Main Configarr configuration file content        | See `values.yaml`                      |
| `secrets.secretsYml`  | Secrets file content (API keys, etc.)            | See `values.yaml`                      |
| `env`                 | Environment variables to add to the container    | `{}`                                   |

### Persistence Parameters

| Name                          | Description                                      | Value                |
| ----------------------------- | ------------------------------------------------ | -------------------- |
| `persistence.enabled`         | Enable persistence for repository cache          | `true`               |
| `persistence.existingClaim`   | Name of an existing PVC to use                   | `""`                 |
| `persistence.storageClass`    | Storage class of backing PVC                     | `""`                 |
| `persistence.accessMode`      | PVC Access Mode                                  | `ReadWriteOnce`      |
| `persistence.size`            | PVC Storage Request                              | `1Gi`                |
| `persistence.subPath`         | Subdirectory of the volume to mount              | `configarr-repos`    |
| `persistence.annotations`     | Annotations for the PVC                          | `{}`                 |

### Security Parameters

| Name                      | Description                                      | Value         |
| ------------------------- | ------------------------------------------------ | ------------- |
| `podSecurityContext`      | Pod security context                             | `{}`          |
| `securityContext`         | Container security context                       | `{}`          |

### Resource Parameters

| Name                      | Description                                      | Value         |
| ------------------------- | ------------------------------------------------ | ------------- |
| `resources.limits`        | Resource limits for the container                | `{}`          |
| `resources.requests`      | Resource requests for the container              | `{}`          |

### Node Selection Parameters

| Name                      | Description                                      | Value         |
| ------------------------- | ------------------------------------------------ | ------------- |
| `nodeSelector`            | Node labels for pod assignment                   | `{}`          |
| `tolerations`             | Tolerations for pod assignment                   | `[]`          |
| `affinity`                | Affinity for pod assignment                      | `{}`          |

## Configuration and Installation Details

### Configuring Configarr

The main configuration is provided through the `config.configYml` value. This should contain your complete Configarr configuration. See the [Configarr documentation](https://configarr.de/docs/configuration/config-file/) for details on available options.

Example:

```yaml
config:
  configYml: |
    trashGuideUrl: https://github.com/TRaSH-Guides/Guides
    recyclarrConfigUrl: https://github.com/recyclarr/config-templates

    sonarr:
      series:
        base_url: http://sonarr:8989
        api_key: !secret SONARR_API_KEY
        quality_definition:
          type: series
        include:
          - template: sonarr-quality-definition-series
          - template: sonarr-v4-quality-profile-web-1080p
```

### Managing Secrets

API keys and other sensitive information should be stored in the `secrets.secretsYml` value:

```yaml
secrets:
  secretsYml: |
    SONARR_API_KEY: "your-actual-api-key" # pragma: allowlist secret
    RADARR_API_KEY: "your-radarr-api-key" # pragma: allowlist secret
```

**Security Note**: For production use, consider using:

- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
- [External Secrets Operator](https://external-secrets.io/)
- Your cloud provider's secret management solution

### Persistence

The chart mounts a PersistentVolumeClaim at `/app/repos` to cache downloaded repositories. This prevents repeated downloads on each run.

To use an existing PVC:

```yaml
persistence:
  enabled: true
  existingClaim: my-existing-pvc
```

To disable persistence (not recommended):

```yaml
persistence:
  enabled: false
```

### CronJob Schedule

The default schedule runs Configarr every hour. You can customize this using standard cron syntax:

```yaml
cronjob:
  schedule: "0 */6 * * *"  # Every 6 hours
```

## Troubleshooting

### Viewing Logs

To view logs from the most recent job:

```bash
kubectl logs -l app.kubernetes.io/name=configarr --tail=100
```

### Manual Job Execution

To manually trigger a job for testing:

```bash
kubectl create job --from=cronjob/configarr configarr-manual
```

### Common Issues

1. **API Connection Failures**: Ensure your Sonarr/Radarr services are accessible from the cluster and the API keys are correct.

2. **Persistence Issues**: Verify your PVC is bound:

   ```bash
   kubectl get pvc
   ```

3. **Configuration Errors**: Check the pod logs for configuration validation errors.

## License

This Helm chart is open source and available under the same license as Configarr.

## Links

- [Configarr Documentation](https://configarr.de/docs/)
- [Configarr GitHub](https://github.com/raydak-labs/configarr)
- [TRaSH Guides](https://trash-guides.info/
