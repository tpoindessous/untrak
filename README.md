# Untrak
Find untracked resources in Kubernetes cluster, garbage collect them.

## Why?

When you use `kubectl apply`, `kustomize build` or `helm template` for injecting manifests through you CI/CD pipeline, kubernetes don't know when an object has been deleted from you repo. Your resources are now **untracked** from your delivery process and they are still managed by your clusters.

Untrak is a tool made for finding and deleting these untracked files on your cluster.


## How it works?

![untrak-schema.png](docs/untrak-schema.png)

Via a simple config file (`untrak.yaml`), this tool will internally execute commands that output YAMLs and find resources in your clusters that are not in your SCM anymore.

In a GitOps context, this is the tool you always dreamed of.

## Example

Put a `untrak.yaml` file in you SCM.

> Note: if you have multiple environments, you would need multiple untrak config files.
```yaml
# untrak.yaml
## git sources
in:
- cmd: "cat"
  args: ["example/manifests/resources.yaml"]

## cluster manifests
out:
- cmd: "kubectl"
  args: ["get", "cm,deploy,svc,ing", "-o", "yaml", "-n", "api"]

exclude:
- namespace
```
To show untracked resources in you cluster (out) simply launch `untrak` like so:

```
$ untrak -c untrak.yaml -o text
- api/ConfigMap/django-config-b4k42gm792
- api/ConfigMap/django-config-g55mctg456
- api/Ingress/my-ingress
```

If you need to garbage collect them, you can change the output format to yaml and pipe the result in kubectl:

```
$ untrak -c untrak.yaml -o yaml | kubectl delete -f -
configmap "django-config-b4k42gm792" deleted
configmap "django-config-g55mctg456" deleted
ingress.extensions "my-ingress" deleted
```