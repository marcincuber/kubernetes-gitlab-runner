# kubernetes-gitlab-runner
Gitlab runners configuration yamls to run in Kubernetes

## General

Configuration deployed using provided yamls in [k8s-templates](./k8s-templates) will create runners in gitlab namespace. It has configuration for S3 bucket caching (make sure to update bucket name in deployment.yaml).

## Kubernetes addons used for deployments

In my configuration I am making use of the following addons:

* [External Secrets](https://github.com/godaddy/kubernetes-external-secrets) - This is used to pull gitlab token from AWS SSM parameter store and populate it as a secret
* [Reloader](https://github.com/stakater/Reloader) - this is extremely useful as it watches changes in ConfigMap and Secret and performs rolling upgrades on Pods

In the [external-secret.yaml](./k8s-templates/external-secret.yaml) you can modify which SSM paremeter should be used by editing `key`.

### Gitlab global cache

S3 bucket caching setup can all be configured to your needs in [deployment.yaml](./k8s-templates/deployment.yaml). See the following block:

```
- name: CACHE_TYPE
  value: s3
- name: CACHE_PATH
  value: gitlab-caches
- name: CACHE_SHARED
  value: "true"
- name: CACHE_S3_SERVER_ADDRESS
  value: s3.amazonaws.com
- name: CACHE_S3_BUCKET_NAME
  value: kubernetes-gitlab-cache
- name: CACHE_S3_BUCKET_LOCATION
  value: eu-west-1
- name: RUNNER_CACHE_DIR
  value: /opt/gitlab-runner/cache/
- name: RUNNER_BUILDS_DIR
  value: /opt/gitlab-runner/builds/
```

### Gitlab registration token

As previously stated, token is stored in AWS SSM parameter store. Following is the configuration which points at the generated secret from external-secrets:

```
- name: init-runner-secrets
  projected:
    defaultMode: 420
    sources:
    - secret:
        items:
        - key: runner-registration-token
          path: runner-registration-token
        name: gitlab-runner-token
``` 