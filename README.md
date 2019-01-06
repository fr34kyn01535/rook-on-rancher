This is small collection of usefull YAML files to setup Rook 0.9 on Rancher.

The most important thing to change is **dataDirHostPath: /storage** in 2-cluster.yaml. 
The scripts will create a Ceph Operator and Cluster that will base its data in the **dataDirHostPath**.

You can simply run these scripts with kubectl (kubectl create -f 1-operator.yaml).
Run them in order when you install, and in reverse when you remove the cluster.

Once you ran the first two scripts you can see the two namespaces rook-ceph and rook-ceph-system in https://<your rancher>/c/<your cluster>/projects-namespaces.
Assign them to a new project "Storage" for simple access. 

Once the Cluster is up you can look at the Ceph Dashboard at:
https://<your rancher>/k8s/clusters/<your cluster>/api/v1/namespaces/rook-ceph/services/https:rook-ceph-mgr-dashboard:8443/proxy/#/login

The username will be admin, the password is available in a secret called rook-ceph/rook-ceph-dashboard-password.

You can execute **ceph status** in the rook-ceph/rook-ceph-tools workload in order to see the status of your cluster, and configure it in that container.

4.1-block-pool

This will create a block pool inside of ceph and a provisioner storage class to automatically satisfy volume requests. 
In order to set the storage class to default (and not having to assign it manually) execute :
> kubectl patch storageclass rook-ceph-block -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

4.2-shared-filesystem

Out of the box, cephfs doesn't come with a provisioner.

In order to bind the volume add a ephemeral via ui with the driver ceph.rook.io/rook, filesystem type ceph and the options:
- clusterNamespace = rook-ceph
- fsName = myfs 
- path = / being the relative path inside the cephfs
  
You can also edit the workloads YAML directly and specify the volumes there (useful for fixing workloads that have a volumeclaims and also pretty easy to migrate bind mounts to cephfs).

> apiVersion: apps/v1beta2
kind: StatefulSet
spec:
  template:
    spec:
      volumes:
      - flexVolume:
          driver: ceph.rook.io/rook
          fsType: ceph
          options:
            clusterNamespace: rook-ceph
            fsName: myfs
            path: /some/path/inside/cephfs
        name: some-volume-name




Further usefull links:

Rook documentation (Make sure to navigate to the correct version and NOT master): https://rook.github.io/docs/rook/v0.9/

Automatic CephFS provisioner
https://github.com/kubernetes-incubator/external-storage/tree/master/ceph/cephfs
I have no tested it yet. @whereisaaron from the rook slack proposed it as automatic provisioner and hinted that the cephfs provisioner, by default, expects to read the admin secret from 'kube-system' and create PVC secrets in the same namespace as each PVC. 
But the sample RBAC doesn't actually include that permission. You can either set `--secret-namespace="cephfs"` and keep everything in the one namespace, or you can extend the ClusterRole to allow Secret creation elsewhere.
