This is small collection of usefull YAML files to setup Rook on Rancher.

The most important thing to change is *dataDirHostPath: /storage* in 2-cluster.yaml. 
The scripts will create a Ceph Operator and Cluster that will base its data in the dataDirHostPath.

You can simply run these scripts with kubectl (kubectl create -f 1-operator.yaml).
Run them in order when you install, and in reverse when you remove the cluster.

Once you ran the first two scripts you can see the two namespaces rook-ceph and rook-ceph-system in https://<your rancher>/c/<your cluster>/projects-namespaces.
Assign them to a new project "Storage" for simple access. 

Once the Cluster is up you can look at the Ceph Dashboard at:
https://<your rancher>/k8s/clusters/<your cluster>/api/v1/namespaces/rook-ceph/services/https:rook-ceph-mgr-dashboard:8443/proxy/#/login

The username will be admin, the password is available in a secret called rook-ceph/rook-ceph-dashboard-password.

You can execute *ceph status* in the rook-ceph/rook-ceph-tools workload in order to see the status of your cluster, and configure it in that container.