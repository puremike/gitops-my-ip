# My IP Client and Server

This repository contains the configuration files for deploying a client and server application using Kubernetes and Argo CD.

## Overview

The client and server applications are defined in separate Helm charts, with configurations stored in `values-prod.yaml`, `values-dev.yaml`, and `values-staging.yaml` files.

## Components

* `my-ip-client`: A client application with a deployment, service, and ingress configuration.
* `my-ip-server`: A server application with a deployment, configmaps, service, and ingress configuration.

## Deployment

The applications are deployed using Argo CD, with configurations stored in the `argoCD` directory. The `my-ip-client` and `my-ip-server` applications are deployed to separate namespaces.

## Configuration

The applications are configured using Helm values files. The values files contain settings for the application, such as the image, port, and resource requests.

## Database

The applications use an external Postgres database hosted on Render. The database connection settings are stored in the Helm values files.

## Monitoring

The cluster is monitored using the `kube-prometheus-stack` Helm chart, which provides a comprehensive monitoring solution for Kubernetes. The monitoring setup includes:

* Prometheus for metrics collection
* Alertmanager for alerting
* Grafana for visualization

## Usage

To deploy the applications, follow these steps:

1. Install Argo CD on your Kubernetes cluster.
2. Create a new application in Argo CD, pointing to the `my-ip-client` or `my-ip-server` directory.
3. Configure the application settings using the Helm values files.
4. Deploy the application to your Kubernetes cluster.
5. Install the `kube-prometheus-stack` Helm chart to enable monitoring.
6. Configure the Postgres database connection settings in the Helm values files.

## Contributing

Contributions are welcome! Please submit pull requests with changes to the configuration files or documentation.