

---

**Setting Up Metrics, Logs, and Traces  on Elastic**

---

**Step 0: Create an Account on Elastic Cloud**

we have a self hosted elastic cloud.

---

**Step 1: Deploy application on a Kubernetes Cluster**

1. Here the application has been deployed we just need to monitor

**Step 2: Enable kube-state-metrics**

1. Clone the kube-state-metrics repository:

   ```
   git clone https://github.com/kubernetes/kube-state-metrics.git
   ```

2. Apply the standard configuration from the examples directory:

   ```
   kubectl apply -f ./kube-state-metrics/examples/standard
   ```

   This will deploy kube-state-metrics, and you should see a pod running in the kube-system namespace.

---

**Step 3: Install Elastic Agent with Kubernetes Integration**

1. Add Kubernetes Integration:
   - In Elastic, navigate to integrations and select the Kubernetes Integration. Add Kubernetes.
   - Provide a name for the integration and turn on kube-state-metrics in the configuration screen.
   - Give the configuration a name in the "new-agent-policy-name" text box and save it.

2. Select the policy created in the previous step and follow the instructions to add the agent:
   - Copy or download the manifest provided in the third step of Add Agent instructions.
   - Save the manifest as "elastic-agent-managed-kubernetes.yaml" and apply it using the following command:

   ```
   kubectl apply -f elastic-agent-managed-kubernetes.yaml
   ```

   This will deploy Elastic Agents as part of a DaemonSet in the kube-system namespace.

---

**Step 4: Explore Elastic Dashboards and Logs**

Once configured, metrics will flow into Elastic dashboards. To view logs for specific pods:
- Access the Discover feature in Kibana.
- Search for the logs of a specific pod.

---

