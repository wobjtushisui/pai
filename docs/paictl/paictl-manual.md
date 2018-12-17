# paictl

A tool to manage your pai cluster.

## Index
- [Manage cluster configuration](#Config)
    - [Set external storage configuration to k8s](#External_Set)
    - [Generate cluster configuration with machine list](#Config_Generate)
    - [Update the cluster configuration in the k8s](#Config_Update)
    - [Get the cluster configuration from the k8s](#Config_Get)
- [ Maintain machines ](#Machine)
    - [ Add machines to the cluster ](#Machine_Add)
    - [ Remove machines from the cluster ](#Machine_Remove)
    - [ Fix crashed etcd node](#etcd_fix)
- [ Maintain your service ](#Service)
    - [ Start service(s) ](#Service_Start)
    - [ Stop service(s) ](#Service_Stop)
    - [ Delete service(s) ](#Service_Delete)
    - [ Refresh service(s) ](#Service_Refresh)
- [ Bootstrap your cluster ](#Cluster)
    - [ Bootstrap Kubernetes ](#Cluster_K8s_Boot)
    - [ Stop Kubernetes ](#Cluster_K8s_Stop)
    - [ Generate the cluster-configuration template from a machine list ](#Cluster_Conf_Generate)
- [ Appendix: An example of the `machine-list` file ](#Machine_Nodelist_Example)



## Manage Cluster Configuration <a name="Config"></a>


### Set external storage configuration to k8s <a name="External_Set"></a>

```
python paictl.py config -e external-config-update [ -c kube-config ] 
```

- External storage is responsible for storing your cluster configuration. And it is not a part of OpenPai's service. Now OpenPai supports 2 method to integrate.
    - git: Admin could manage their cluster configuration with git such as VSTS, Github or Gitlab. Of course, you could setup your own git server.  
    - local: If you don't have a git repo to manage your cluster configuration, you can store it in the local disk. 

- External storage configuration
```yaml
#################
#     Git       #
#################

type: git
url: https://github.com/example/example
branch: example_branch
# relative path in the project source code
path: a/b/c
```    
```yaml
##################
# Local storage. #
##################

type: local
# absolute path in your system
path: /a/b/c
```

- kube-config: The configuration for kubectl and other kubernetes client to access to the server. The default value is ```~/.kube/config```. For more detail information, please refer to the [link](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/#the-kubeconfig-environment-variable)

### Generate cluster configuratin from quick-start.yaml
```yaml
python paictl.py config generate -i /pai/deployment/quick-start/quick-start/quick-start.yaml -o ~/pai-config -f
```
- quick-start.yaml: Admin could generate a complete cluster configuration with quick-start.yaml. More detailed about this file please refer to this [link](../pai-management/doc/cluster-bootup.md#c-step-1).
- More infomation about this command please refer to this [link](../pai-management/doc/cluster-bootup.md#c-step-2). 

### Update the cluster configuration in the k8s

###### 1. Update cluster configuration from local path

```
python paictl config update -p /path/to/local/configuration [-c kubeconfig-path ]
```
- Same as local storage.
###### 2. Update cluster configuration from the external storage (Get the external storage from local)
```
python paictl config update -e external-storage-config [-c kubeconfig-path]
```
- Configure the external storage configuration and pass the configuration file with the parameter ```-e```.

###### 3. Update cluster configuration from the external storage (Get the external storage from k8s)
```
python paictl config update [-c kubeconfig-path]
```
- Note: Please ensure that you have upload the external storage configuration to k8s with the [command](#External_Set).


### Get the cluster configuration from the k8s
```yaml
paictl.py config get -o /path/to/output [-c kube-config]
```

## Maintain machines <a name="Machine"></a>


### Add machines to the cluster <a name="Machine_Add"></a>

```
python paictl.py machine add -p /path/to/cluster-configuration/dir -l machine-list.yaml
```

- See an example of the machine list [here](#Machine_Nodelist_Example).

### Remove machines from the cluster <a name="Machine_Remove"></a>

```
python paictl.py machine remove -p /path/to/cluster-configuration/dir -l machine-list.yaml
```

- See an example of the machine list [here](#Machine_Nodelist_Example).


### Fix crashed etcd node <a name="etcd_fix"></a>


```
python paictl.py machine etcd-fix -p /path/to/cluster-configuration/dir -l machine-list.yaml
```

- See an example of the machine list [here](#Machine_Nodelist_Example).
- Note: The opertion could only fix one node each time.


## Maintain infrastructure services <a name="Service"></a>

### Start service(s) <a name="Service_Start"></a>

```
python paictl.py service start -p /path/to/cluster-configuration/dir [ -n service-name ]
```

1) Start all services by default.
2) If the option `-n` is set, only the specified service will be started.

### Stop service(s) <a name="Service_Stop"></a>

```
python paictl.py service stop -p /path/to/cluster-configuration/dir [ -n service-name ]
```

- Stop all services by default.
- If the option `-n` is set, only the specified service will be stopped.

### Delete service(s) <a name="Service_Delete"></a>

```
python paictl.py service delete -p /path/to/cluster-configuration/dir [ -n service-name ]
```

- 'Delete' a service means to stop that service and then delete all of its persisted data in HDFS, Yarn, ZooKeeper, etc.
- Delete all services by default.
- If the option `-n` is set, only the specified service will be deleted.

### Refresh service(s) <a name="Service_Refresh"></a>

```
python paictl.py service refresh -p /path/to/cluster-configuration/dir [ -n service-name ]
```

- Refresh all the labels on each node.
- If the option `-n` is set, only the specified service will be upgrade.


## Maintain your cluster <a name="Cluster"></a>

### Bootstrap Kubernetes <a name="Cluster_K8s_Boot"></a>

```
python paictl.py cluster k8s-bootup -p /path/to/cluster-configuration/dir
```

- Install kubectl in the deployment box.
- Bootstrap Kubernetes in the specified cluster.

### Stop Kubernetes <a name="Cluster_K8s_Stop"></a>

```
python paictl.py cluster k8s-clean -p /path/to/cluster-configuration/dir
```

- Stop Kubernetes in the specified cluster.

### Generate cluster-configuration template files from a machine list <a name="Cluster_Conf_Generate"></a>

```
python paictl.py config generate -p /path/to/machinelist.csv
```

- The machine list should be provided in CSV format.
- All configuration files of cluster-configuration will be generated to the local folder, and then be used for bootstrapping the whole cluster.
- By default, in the generated configuration, a single-master Kubernetes is configured by default.
- Advanced users or developers can fine-tune the content of the generated configuration files according to specific environments.

## Appendix: An example of the `machine-list.yaml` file <a name="Machine_Nodelist_Example"></a>

```yaml
machine-list:

    - hostname: host1 # echo `hostname`
      hostip: 192.168.1.11
      machine-type: D8SV3
      etcdid: etcdid1
      #sshport: PORT (Optional)
      username: username
      password: password
      k8s-role: master
      dashboard: "true"
      zkid: "1"
      pai-master: "true"

    - hostname: host2
      hostip: 192.168.1.12
      machine-type: NC24R
      #sshport: PORT (Optional)
      username: username
      password: password
      k8s-role: worker
      pai-worker: "true"
```