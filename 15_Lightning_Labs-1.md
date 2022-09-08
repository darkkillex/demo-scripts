# Lightining Lab 1

This is the solution I tried to solve the LL-1

1. Upgrade the current version of kubernetes from 1.19 to 1.20.0 exactly using the kubeadm utility. Make sure that the upgrade is carried out one node at a time starting with the master node. To minimize downtime, the deployment gold-nginx should be rescheduled on an alternate node before upgrading each node.
Upgrade controlplane node first and drain node node01 before upgrading it. Pods for gold-nginx should run on the controlplane node subsequently.
   
   Use below commands step-by-step as mentioned:

   <details>

   ```
   On CONTROLPLANE Node:-
   
   Set shorthand alias for kubectl that also works with completion:-
   
   alias k=kubectl
   complete -o default -F __start_kubectl k

   apt update
   apt-cache madison kubeadm
   kubectl drain controlplane --ignore-daemonsets
   apt-get update && apt-get install -y --allow-change-held-packages kubeadm=1.20.0-00
   kubeadm  upgrade plan
   kubeadm  upgrade apply v1.20.0
   apt-get update && apt-get install -y --allow-change-held-packages kubelet=1.20.0-00 kubectl=1.20.0-00
   systemctl daemon-reload
   systemctl restart kubelet
   kubectl uncordon controlplane
   kubectl drain node01 --ignore-daemonsets
   
   
   On WORKER Node:- ssh node01
   
   apt-get update && apt-get install -y --allow-change-held-packages kubeadm=1.20.0-00
   kubeadm upgrade node
   apt-get update && apt-get install -y --allow-change-held-packages kubelet=1.20.0-00 kubectl=1.20.0-00
   systemctl daemon-reload
   systemctl restart kubelet     

   Back on CONTROLPLANE Node:- logout
   
   kubectl uncordon node01
   kubectl get pods -o wide | grep -i gold-nginx (make sure this is scheduled on controlplane node)
   ```
   </details>


2. Print the names of all deployments in the admin2406 namespace in the following format:
   DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE
   <deployment name> <container image used> <ready replica count> <Namespace>
   . The data should be sorted by the increasing order of the deployment name. 
   Example:
   DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE
   deploy0 nginx:alpine 1 admin2406
   Write the result to the file /opt/admin2406_data.:
   <details>
   
   ```
   To see all the deployments in admin2406 namespace in JSON format:
   kubectl -n admin2406 get deployments.apps  -o json
   
   To check every query:-
   kubectl -n admin2406 get deployment
   kubectl -n admin2406 get deployment -o=jsonpath='{.items[*].metadata.name}'
   kubectl -n admin2406 get deployment -o=jsonpath='{.items[*].spec.template.spec.containers[*].image}'
   kubectl -n admin2406 get deployment -o=jsonpath='{.items[*].status.readyReplicas}'
   kubectl -n admin2406 get deployment -o=jsonpath='{.items[*].metadata.namespace}'
   
   ...and finally:-
   kubectl -n admin2406 get deployment --sort-by=.metadata.name -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[*].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace > /opt/admin2406_data
   ```
   
   </details>
   
   
3. A kubeconfig file called admin.kubeconfig has been created in /root/CKA. There is something wrong with the configuration. 
   Troubleshoot and fix it.:

      <details>

      ```
      Check the kube-apiserver:

      Change port of the server: by default the Kubernetes API server listens on port 6443
      
      vi /root/CKA/admin.kubeconfig

      Now replace the port with 6443
      
      Run the following command to check the fixed file:
      
      kubectl cluster-info --kubeconfig /root/CKA/admin.kubeconfig
      ```
      </details>
   
   
4. In default namespace create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. 
   Next upgrade the deployment to version 1.17 using rolling update:

      <details>

      ```
      kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=1
   
      N.B. rolling updates must already be configured in the manifest of the deploy 
      (are usually by default)
   
      kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
      ```
      </details>

   
5. A new deployment called alpha-mysql has been deployed in the alpha namespace. 
   However, the pods are not running. Troubleshoot and fix the issue. 
   The deployment should make use of the persistent volume alpha-pv to be mounted at 
   /var/lib/mysql and should use the environment variable MYSQL_ALLOW_EMPTY_PASSWORD=1 
   to make use of an empty root password.Important: Do not alter the persistent volume.
   
   Apply/refer below yaml to create a PersistentVolumeClaim:

      <details>

      ```
      Check the status (kubectl describe ...) of PODs and you will see an issue about 
      PersistentVolumeClaims. Let's create that:
   
      apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
        name: mysql-alpha-pvc
        namespace: alpha
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
        storageClassName: slow
      ```
      </details>

   
6. Take the backup of ETCD at the location /opt/etcd-backup.db on the controlplane node.
   Execute below command for etcd backup:

      <details>

      ```
      OPTIONAL: Let's check the settings of ETCD
      kubectl get all -n kube-system
      kubectl describe pods -n kube-system etcd-controlplane
      OPTIONAL: L'et's check the health of ETCD (here we can also observe the endpoint. the default one is 127.0.0.1:2379)
      kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl endpoint health"
      
      Put ETCDCTL_API='3' before etcdctl snapshot ...
      OR enter the following two commands   
      export ETCDCTL_API=3
      etcdctl snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 /opt/etcd-backup.db
      ```
      </details>

   
7. Create a pod called secret-1401 in the admin1401 namespace using the busybox image. The container within the pod 
   should be called secret-admin and should sleep for 4800 seconds.
   The container should mount a read-only secret volume called secret-volume at the path /etc/secret-volume. 
   The secret being mounted has already been created for you and is called dotfile-secret.
   Apply below manifest for the solution:

   <details>

      ```
      apiVersion: v1
      kind: Pod
      metadata:
        creationTimestamp: null
        labels:
          run: secret-1401
        name: secret-1401
        namespace: admin1401
      spec:
        volumes:
        - name: secret-volume
          secret:
            secretName: dotfile-secret
        containers:
        - command:
          - sleep
          args:
          - "4800"
          image: busybox
          name: secret-admin
          volumeMounts:
          - name: secret-volume
            readOnly: true
            mountPath: "/etc/secret-volume"     
      ```
   </details>


