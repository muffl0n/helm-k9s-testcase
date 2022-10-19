# Testcase for derailed/k9s#1558
https://github.com/derailed/k9s/issues/1558

* Create chart `test`
  ```
  mkdir chart
  helm create chart/test
  ```
* Delete all in templates except for `_helpers.tpl` and `serviceaccount.yaml`
* Create namespaces `foo` and `bar`
* Install release in both namespaces
  ```
  -> % helm install -n bar test ./chart/test
  -> % helm install -n foo test ./chart/test
  ```
* List releases
  ```
  -> % helm list -n bar
  NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
  test    bar             1               2022-10-19 08:17:36.750993 +0200 CEST   deployed        test-0.1.0      1.16.0
  
  -> % helm list -n foo
  NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
  test    foo             1               2022-10-19 08:16:28.218608 +0200 CEST   deployed        test-0.1.0      1.16.0
  ```
* Check for resources
  ```
  -> % kubectl get sa -n bar test -oyaml | kubectl neat -
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    annotations:
      meta.helm.sh/release-name: test
      meta.helm.sh/release-namespace: bar
    labels:
      ...
    name: test
    namespace: bar
  secrets:
  - name: test-token-ddrbv

    -> % kubectl get sa -n foo test -oyaml | kubectl neat -
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    annotations:
      meta.helm.sh/release-name: test
      meta.helm.sh/release-namespace: foo
    labels:
      ...
    name: test
    namespace: foo
  secrets:
  - name: test-token-gqjfd
  ```
* Check for manifest
  ```
  -> % helm get manifest -n bar test
  ---
  # Source: test/templates/serviceaccount.yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: test
  labels:
    ...
  
  -> % helm get manifest -n foo test
  ---
  # Source: test/templates/serviceaccount.yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: test
  labels:
    ...
  ```
* Set default namespace to `bar`
  ```
  -> % kubectl config set-context --current --namespace bar
  Context "test" modified.
  ```
* Uninstall release in `foo`
  ```
  -> % helm uninstall -n foo test
  release "test" uninstalled
  ``` 
* List releases
  ```
  -> % helm list -n bar
  NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
  test    bar             1               2022-10-19 08:17:36.750993 +0200 CEST   deployed        test-0.1.0      1.16.0
  
  -> % helm list -n foo
  NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
  ```
* Check for resources: ServiceAccount in `foo` is deleted
  ```
  -> % kubectl get sa -n bar test -oyaml | kubectl neat -
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    annotations:
      meta.helm.sh/release-name: test
      meta.helm.sh/release-namespace: bar
    labels:
    name: test
    namespace: bar
  secrets:
  - name: test-token-ddrbv
  
  -> % kubectl get sa -n foo test -oyaml | kubectl neat -
  Error from server (NotFound): serviceaccounts "test" not found
  {}
  ```
* Reinstall release in `foo`
  ```
  -> % helm install -n foo test ./chart/test
  NAME: test
  LAST DEPLOYED: Wed Oct 19 08:28:28 2022
  NAMESPACE: foo
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None

  -> % helm list -n foo                     
  NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
  test    foo             1               2022-10-19 08:28:28.710851 +0200 CEST   deployed        test-0.1.0      1.16.0
  ``` 
* Start k9s, default namespace is still `bar`
* Change namespace to `foo`
* List helm releases
* Delete release `test`
* List releases: Release in `foo` is deleted
  ```
  -> % helm list -n bar
  NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
  test    bar             1               2022-10-19 08:17:36.750993 +0200 CEST   deployed        test-0.1.0      1.16.0
  
  -> % helm list -n foo
  NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
  ```
* Check for resources: ServiceAccount in `bar` is deleted
  ```
  -> % kubectl get sa -n bar test -oyaml | kubectl neat -  
  Error from server (NotFound): serviceaccounts "test" not found
  {}
  
  -> % kubectl get sa -n foo test -oyaml | kubectl neat -
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    annotations:
      meta.helm.sh/release-name: test
      meta.helm.sh/release-namespace: foo
    labels:
      ...
    name: test
    namespace: foo
  secrets:
  - name: test-token-ktn8x
  ```

# Versions
```
-> % k9s version
 ____  __.________
|    |/ _/   __   \______
|      < \____    /  ___/
|    |  \   /    /\___ \
|____|__ \ /____//____  >
        \/            \/

Version:    0.26.6
Commit:     c66002d9868fc172e0bb8091ad87d61be027aae2
Date:       n/a
```
```
-> % helm version
version.BuildInfo{Version:"v3.10.1", GitCommit:"9f88ccb6aee40b9a0535fcc7efea6055e1ef72c9", GitTreeState:"clean", GoVersion:"go1.19.2"}
```