Common Kubernetes Issues:- 

## Issue: ImagePullBackOff   

**Symptoms:**
- Pod is stuck in `ImagePullBackOff` or `ErrImagePull` state
- `kubectl get pods` shows `ImagePullBackOff` under STATUS

**Check:**
1. Describe the pod to see detailed error messages:    

**Command:**
kubectl describe pod <pod-name> 

Look for events at the bottom mentioning **Failed to pull image**, **unauthorized**, or **not found**.

or check in your container registry.

**Common Causes:**
- Wrong image name or tag
- Private registry without proper image pull secret
- Typo in image URL
- Missing or incorrect imagePullSecrets in pod spec
- Registry down or unreachable

**If private registry:**  
Create a secret for your registry credentials:  
============================================================================================================================================================================================================================
## 🐞 Issue: Pod Stuck in Pending State

Step 1: Describe the pod

**Command:**

kubectl describe pod <pod-name>

Focus on the Events section at the bottom for clues.

Look for messages like:

0/3 nodes are available: node(s) had taint {…}, that the pod didn't tolerate

0/3 nodes are available: node(s) didn't match pod's node selector

0/3 nodes are available: insufficient memory/cpu

no nodes available to schedule pods

**Possible Causes & Solutions:**  

Possible Causes:

Insufficient resources (CPU/memory) on nodes

No matching nodes due to:

nodeSelector

node affinity

Taints blocking scheduling

PVC (PersistentVolumeClaim) pending (if using volumes)

Unschedulable nodes (cordoned/drained)

1️⃣ NodeSelector or Node Affinity mismatch

Check if pod is requesting a specific node label that doesn’t exist.

Either fix the nodeSelector / nodeAffinity in the pod spec or label a node accordingly:

kubectl label nodes <node-name> <key>=<value>

2️⃣ Taints and No Tolerations

Node may be tainted (e.g. NoSchedule), and pod lacks toleration.

**Command:**

kubectl describe node <node-name> | grep Taint

Add a toleration in pod spec or remove the taint if unnecessary:

kubectl taint nodes <node-name> <key>-  # remove taint

3️⃣ Insufficient resources

Scheduler can't place pod due to lack of CPU or memory.

kubectl describe node <node-name> 

4️⃣ PVC issues (waiting for volumes)

Check if pod is waiting for a PersistentVolume.

kubectl get pvc

============================================================================================================================================================================================================================
## 🐞 Issue: Pod OOMKilled (Out of Memory)

Symptoms:

Pod restarts frequently or is in CrashLoopBackOff

kubectl get pods shows STATUS: OOMKilled

kubectl describe pod <pod-name> shows "OOMKilled" in Last State

Verifying the Issue

1️⃣ Check pod status:

kubectl get pod <pod-name>

Look for STATUS: CrashLoopBackOff or STATUS: Error.

Describe the pod for last state:

kubectl describe pod <pod-name>

Under Containers → Last State → Terminated → Reason, check for:

Reason: OOMKilled

🛠️ Troubleshooting & Solutions

1. Check current memory limits and requests

kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].resources}'

Verify the limits.memory setting.

2. Solution: Increase memory limit or optimize app

Edit the deployment/manifest to increase memory limit:
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"
kubectl apply -f <your-deployment-file>.yaml

✅ Summary Troubleshooting Flow:
kubectl describe pod <pod-name> → confirm OOMKilled

Check memory requests and limits → adjust if needed

Monitor actual usage with kubectl top pod

Check node memory → scale if needed

Fix app memory issue if problem persists

============================================================================================================================================================================================================================
## 🐞 Issue: Pod stuck in ContainerCreating state

Verifying
Check pod status:

kubectl get pods -n <namespace>

Describe the pod to see detailed events:
kubectl describe pod <pod-name> -n <namespace>

Check node status:
kubectl get nodes

Possible Causes & Troubleshooting
✅ Missing or failing image pull
Look for ErrImagePull or ImagePullBackOff in describe output.
→ Fix: Check image name/tag, credentials (if private registry).

✅ Volume/PVC not yet bound
Look for waiting for volume or unable to mount volume messages.
→ Fix: Check PVC status:

kubectl get pvc -n <namespace>

✅ Node issues (network, CNI plugins, kubelet)
Check events or node conditions in kubectl describe node <node-name>.
→ Fix: Restart node components, check CNI installation.

✅ Image pull timeout or slow registry
Check for timeouts in events.
→ Fix: Try pulling image manually on node or check network issues.

============================================================================================================================================================================================================================
## 🐞 Issue: StatefulSet pods stuck due to PVC issues

Verifying
Check StatefulSet and Pod status

View the pods in the StatefulSet:

kubectl get pods -n <namespace> -l app=<statefulset-name>

Check the status of PVCs associated with the StatefulSet:

kubectl get pvc -n <namespace> -l app=<statefulset-name>

Describe the StatefulSet
Look for conditions and events related to PVC or storage.

kubectl describe statefulset <statefulset-name> -n <namespace>

Check PVC and PV status

kubectl describe pvc <pvc-name> -n <namespace>
kubectl describe pv <pv-name>

Possible Causes & Troubleshooting
✅ PVC is stuck in Pending state

If PVC is not bound, check if there are available Persistent Volumes (PVs) that match the request.

Check PV availability:

kubectl get pv

✅ Storage Class misconfiguration

If you’re using a StorageClass, ensure it’s properly defined and the underlying provisioner is working.

Check StorageClass:

kubectl get storageclass

Fix:

Ensure the StorageClass is available and the provisioner is functioning correctly (e.g., dynamic provisioning might be misconfigured).

✅ Insufficient resources (e.g., storage)

The underlying storage might be unavailable or lacking sufficient resources.

Fix:

Verify the storage provider and check quotas, limits, and storage availability.
Ensure the node has sufficient resources to fulfill the storage request.

✅ Pod failures due to volume mount issues

If the PVC is bound but the pod fails due to mounting issues, look for error messages in kubectl describe pod <pod-name>.
Common errors might be "mount failed" or "timeout".

============================================================================================================================================================================================================================