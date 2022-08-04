# Web Terminal Operator

The Web Terminal Operator provides users with the ability to create a terminal instance embedded in the OpenShift Console.

**Note:** The OpenShift console integration that allows easily creating web terminal instances and logging in automatically is available in OpenShift 4.5.3 and higher. In previous versions of OpenShift, the operator can be installed but web terminals will have to be created and accessed manually.

### Where to report bugs or propose features?

If you face any issues with installing the Web Terminal Operator or using it, have any ideas about what could be done better, please let us know by creating a jira issue at [https://issues.redhat.com/browse/WTO](https://issues.redhat.com/browse/WTO)

### Deploying next operator from next images
After every commit in master index and bundle images are built and pushed to:
[quay.io/repository/wto/web-terminal-operator-index:next](https://quay.io/repository/wto/web-terminal-operator-index?tab=tags)
[quay.io/repository/wto/web-terminal-operator-metadata:next](https://quay.io/repository/wto/web-terminal-operator-metadata?tab=tags)

To try them you can just do
```
make install
```
and wait until Operator is installed on the cluster.

## Deploying the operator from `next` images
After every commit in master, the index and bundle images are built and pushed to
[quay.io/repository/wto/web-terminal-operator-index:next](https://quay.io/repository/wto/web-terminal-operator-index?tab=tags) and
[quay.io/repository/wto/web-terminal-operator-metadata:next](https://quay.io/repository/wto/web-terminal-operator-metadata?tab=tags)

This repo includes a `Makefile` to simplify deploying the Operator to a cluster:

| Makefile rule | Purpose |
|---|---|
| `make install` | Register the CatalogSource and install the operator on the cluster. |
| `make register_catalogsource` | Register the CatalogSource but do not install the operator. This enables the operator to be installed manually through OperatorHub. |
| `make unregister_catalogsource` | Remove the CatalogSource from the cluster. |
| `make uninstall` | uninstalls the Web Terminal Operator Subscription and related ClusterServiceVersion |
| `make uninstall_v1_2` | Remove the installed WTO 1.2 from the cluster |

The commands above require being logged in to the cluster as a `cluster-admin`. See `make help` for a full list of supported environment variables and rules available.

## Deploying the operator from local sources
In order to deploy the operator from this repo directly, you need to first create the olm bundle and index, push that to a docker registry, and then create a CatalogSource referencing those images.

This can be done in one step using the Makefile:
```bash
WTO_IMG=#<your controller image>
BUNDLE_IMG=#<your bundle image>
INDEX_IMG=#<your index image>
make build_controller_image build_install
```
This will build and push images defined by the environment variables `WTO_IMG`, `BUNDLE_IMG` and `INDEX_IMG`, and register a CatalogSource on the cluster. You may need to set the repos used for the index and bundle to be public to ensure they can be accessed from the cluster.

If you already have the index image pushed to your registry, then you can use the `make install` or `make register_catalogsource` rules with the environment variables defined above to install those images on the cluster.

## Configuring the custom default container

Upon startup, the Web Terminal Operator creates two `DevWorkspaceTemplate` custom resources in its namespace, named `web-terminal-tooling` and `web-terminal-exec`. These templates are referenced by Web Terminals in the cluster, and define the images used to run the DevWorkspaceTemplate:

* `web-terminal-tooling` defines the container that contains cluster tooling (e.g. kubectl, oc, etc.) and is where the shell will be opened when creating a web terminal
* `web-terminal-exec` is a sidecar container that allows the OpenShift cluster to communicate with the Web Terminal, allowing for the requesting user to be automatically logged into the cluster, etc.

To customize Web Terminal Operator functionality, these DevWorkspaceTemplates can be modified, and these changes will be propagated to all Web Terminals on the cluster the next time that terminal is started. To change the image used for the tooling container, for example, it's sufficient to edit the image field in the `web-terminal-tooling` DevWorkspaceTemplate on the cluster. You can find the default tools container source code in [this repository](https://github.com/redhat-developer/web-terminal-tooling). 

By default, the Web Terminal Operator will overwrite any modifications to its DevWorkspaceTemplates when it starts. In order to persist custom changes between Web Terminal Operator versions, the Kubernetes annotation
```yaml
web-terminal.redhat.com/unmanaged-state: "true"
```
should be added to the DevWorkspaceTemplate.

For example, run the following command to change the tools container image:

```bash
oc patch DevWorkspaceTemplate/web-terminal-tooling --type='json' \
-p='[{"op": "replace", "path": "/spec/components/0/container/image", "value": "quay.io/youruser/custom-web-terminal:latest"}]' \
-n openshift-operators
```

and the following command to persist the change:

```bash
oc annotate devworkspacetemplate web-terminal-tooling \
'web-terminal.redhat.com/unmanaged-state=true' \
-n openshift-operators
```


## Removing the operator from a cluster

To remove the WebTerminal Operator and the CatalogSource use
```bash
make uninstall unregister_catalogsource
```

## Manifests generation

Prereq:
The latest compatible operator sdk https://github.com/operator-framework/operator-sdk/releases/tag/v0.17.2

Execute
```
  make gen_terminal_csv
```

### Related Projects
- The Web Terminal Operator is powered by the [devworkspace-operator](https://github.com/devfile/devworkspace-operator)

### Source Syncing
| source | destination | sync job |
| --- | --- | --- |
| [devworkspace-controller](https://github.com/devfile/devworkspace-operator/) | [web-terminal](http://pkgs.devel.redhat.com/cgit/containers/web-terminal) | [Jenkins job](https://codeready-workspaces-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/job/web-terminal-sync-web-terminal-operator/) |
| [web-terminal-operator](https://github.com/redhat-developer/web-terminal-operator) | [web-terminal-dev-operator-metadata](http://pkgs.devel.redhat.com/cgit/containers/web-terminal-dev-operator-metadata) | [Jenkins job](https://codeready-workspaces-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/job/web-terminal-sync-web-terminal-operator-metadata/)
