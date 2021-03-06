# set-namespace

### Overview

<!--mdtogo:Short-->

Update or add namespace.

<!--mdtogo-->

### FunctionConfig

<!--mdtogo:Long-->

To add a namespace `color` to all resources, configure the function with a
simple ConfigMap like this:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  namespace: color
```

There is another advanced way to configure the function by a custom resource
which can provide more flexibility than using ConfigMap.

Example:

```yaml
apiVersion: fn.kpt.dev/v1alpha1
kind: SetNamespaceConfig
metadata:
  name: my-config
namespace: color
```

The values for `apiVersion` and `kind` must match the values in the example
above.

You can use key `fieldSpecs` to specify the resource selector you
want to use. By default, the function will use this field spec:

```yaml
- path: metadata/namespace
  create: true
```

This means a `metadata/namespace` field will be added to all resources with
namespaceable kinds. Whether a resource is namespaceable is determined by the
Kubernetes API schema. If the API path for that kind contains
`namespaces/{namespace}` then the resource is considered namespaceable.
Otherwise, it's not. Currently this function is using API version 1.19.1.

For more information about API schema used in this function, please take a look at
https://github.com/kubernetes-sigs/kustomize/tree/master/kyaml/openapi

To support your own CRDs you will need to add more items to fieldSpecs list.

In addition to simple updating the `metadata/namespace` fields for resources,
it's also possible that references to namespace names exist in some resources.
In this case, these references will also need to be updated at the same time. In
Kubernetes native resources, the references to namespace in `subject` fields in
`RoleBinding` and `ClusterRoleBinding` will be handled implicitly by this
function. If you have references to namespaces in you CRDs, use the field spec
described above to update them.

Fieldspec has following fields:

- group: Select the resources by API version group. Will select all groups
  if omitted.
- version: Select the resources by API version. Will select all versions
  if omitted.
- kind: Select the resources by resource kind. Will select all kinds
  if omitted.
- path: Specify the path to the field that the value will be updated. This field
  is required.
- create: If it's set to true, the field specified will be created if it doesn't
  exist. Otherwise the function will only update the existing field.

Example:

To add a namespace `color` to `spec/selector/namespace` in `MyCRD` resource:

```yaml
apiVersion: fn.kpt.dev/v1alpha1
kind: SetNamespaceConfig
metadata:
name: my-config
namespace: color
fieldSpecs:
- path: spec/selector/namespace
  kind: MyCRD
  version: v1
  group: example.com
  create: true
```

For more information about fieldSpecs, please see
https://kubectl.docs.kubernetes.io/guides/extending_kustomize/builtins/#arguments-4

<!--mdtogo-->
