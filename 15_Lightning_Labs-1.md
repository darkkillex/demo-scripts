# Lightining Lab 1

1. Use below commands step-by-step as mentioned:

   <details>

   ```
   On CONTROLPLANE Node:-
   
   Set shorthand alias for kubectl that also works with completion:-
   
   alias k=kubectl
   complete -o default -F __start_kubectl k

   apt update
   apt-cache-madison kubeadm
   kubectl drain controlplane --ignore-daemonsets
   apt-get update && apt-get install -y --allow-change-held-packages kubeadm=1.20.0-00
   kubeadm  upgrade plan
   kubeadm  upgrade apply v1.20.0
   apt-get update && apt-get install -y --allow-change-held-packages kubelet=1.20.0-00 kubectl=1.20.0-00
   systemctl daemon-reload
   systemctl restart kubelet
   kubectl uncordon controlplane
   kubectl drain node01 --ignore-daemonsets
   
   
   On WORKER Node:-
   
   apt-get update && apt-get install -y --allow-change-held-packages kubeadm=1.20.0-00
   kubeadm upgrade node
   apt-get update && apt-get install -y --allow-change-held-packages kubelet=1.20.0-00 kubectl=1.20.0-00
   systemctl daemon-reload
   systemctl restart kubelet     

   Back on CONTROLPLANE Node:-
   
   kubectl uncordon node01
   kubectl get pods -o wide | grep gold (make sure this is scheduled on controlplane node)
   ```
   </details>


2. Ciao:
   <details>
   
   ```
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


