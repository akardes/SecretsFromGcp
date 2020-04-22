# SecretsFromGCP - A Kustomize plugin for creating Kubernetes secrets from Secret Manager

The purpose of this plugin is to generate Kubernetes secret objects with Kustomize 
by pulling the secret contents from Google Secret Manager: https://cloud.google.com/secret-manager

## Requirements
* `kustomize` (2.10 or higher).
* Google Cloud SDK (281.0 or higher)
* A GCP project with enabled Secret Manager API and some secrets in it
* The whole directory structure with the plugin (`kustomize/plugin/akardes/v1/secretsfromgcp/SecretsFromGCP`) somewhere locally

NOTE: Plugins don't work with `kubectl kustomize` or `kubectl apply -k .` at this time.

## Configuration
Create a YAML file that will hold the confguration of the secret, e.g. `my-kustom-secret.yml`:
```yaml
# apiVersion and kind must match the location of the plugin ('kustomize/plugin/akardes/v1/secretsfromgcp/SecretsFromGCP')
apiVersion: akardes/v1
kind: SecretsFromGCP
metadata:
  # matches the name of the secret that is going to be generated
  name: my-test-secret
# namespace of the Kubernes secret
namespace: default
config:
  # The GCP project of the Secret Manager api holding the secrets
  project: my-gcp-project
  # Type of the secret (currently only generic supported)
  type: generic
# Mapping the Secret Manager secret-id to the Kubernetes secret key.
# In this example the plugin will query Secret Manager for 'sm-password-1'
# and store the result in the Kubernetes secret under a key named 'kube-password-1'
literal-mappings:
  sm-password-1: kube-password-1
  sm-password-2: kube-password-2
```

In your `kustomization.yml`, reference to the configuration as a generator:
```yaml
generators:
- my-kustom-secret.yml
```

The resulting Kubernetes secret looks as follows:
```yaml
apiVersion: v1
kind: Secret
data:
  kube-password-1: Q3VyaW9zaXR5IGtpbGxlZA==
  kube-password-2: dGhlIGNhdA==
metadata:
  name: my-test-secret
  namespace: default
type: Opaque
```

## Usage
Since plugins are an alpha feature of kustomize, you'll need to use the `--enable_alpha_plugins` flag
as well as setting the kustomize plugins home path (the `kustomize` folder) in the env variable `XDG_CONFIG_HOME`.

In the following examples the plugin is located in `../kustomize/plugin/akardes/v1/secretsfromgcp/SecretsFromGCP` relative to where `kustomize` is run.

Example:
```bash
# Build yaml with kustomize 
XDG_CONFIG_HOME=../ kustomize build --enable_alpha_plugins .

# Build and apply to the cluster (instead of "kubectl apply -k .")
XDG_CONFIG_HOME=../ kustomize build --enable_alpha_plugins . | kubectl apply -f -
```
