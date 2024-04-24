

---

**Setting Up Metrics, Logs, and Traces  on Elastic**

---



**Step 1: Deploy Fleet server**

# Deploying Fleet with Helm

To configure preferences for Fleet for use in Helm, including secret names, MySQL and Redis hostnames, and TLS certificates, download the `values.yaml`[values.yaml](https://raw.githubusercontent.com/fleetdm/fleet/main/charts/fleet/values.yaml)   and change the settings to match your configuration.

Please note you will need all dependencies configured prior to installing the Fleet Helm Chart as it will try and run database migrations immediately.

Once you have those configured, run the following:

```bash
helm upgrade --install fleet fleet \
  --repo https://fleetdm.github.io/fleet/charts \
  --values values.yaml
```



## Installing infrastructure dependencies with Helm



### MySQL

The MySQL that we will use for this tutorial is not replicated and it is not Highly Available. I suggest using their MySQL on cloudsql.  we will use a non-replicated instance of MySQL.

To install MySQL from Helm, run the following command. Note that there are some options that need to be defined:

- There should be a fleet database created
- The default user's username should be `fleet`

**Helm v2**

```bash
helm install \
  --name fleet-database \
  --set auth.username=fleet,auth.database=fleet \
  oci://registry-1.docker.io/bitnamicharts/mysql
```

**Helm v3**

```bash
helm install fleet-database \
  --set auth.username=fleet,auth.database=fleet \
  oci://registry-1.docker.io/bitnamicharts/mysql 
```

This helm package will create a Kubernetes Service which exposes the MySQL server to the rest of the cluster on the following DNS address:

`fleet-database-mysql:3306`

We will use this address when we configure the Kubernetes deployment and database migration job

### Database migrations

**Note:** this step is handled automatically.



### Redis

**Helm v2**

```bash
helm install \
  --name fleet-cache \
  --set persistence.enabled=false \
  oci://registry-1.docker.io/bitnamicharts/redis
```

**Helm v3**

```bash
helm install fleet-cache \
  --set persistence.enabled=false \
  oci://registry-1.docker.io/bitnamicharts/redis
```

This helm package will create a Kubernetes Service which exposes the Redis server to the rest of the cluster on the following DNS address:

`fleet-cache-redis:6379`

We will use this address when we configure the Kubernetes deployment, but if you're not using a Helm-installed Redis in your deployment, you'll have to change this in your Kubernetes config files. If you are using the Fleet Helm Chart, this will also be used in the `values.yaml` file.

## Setting up and installing Fleet

### A note on container versions

The Kubernetes files referenced use the Fleet container tagged at `1.0.5`. The tag is something that should be consistent across the migration job and the deployment specification. 
### Create server secrets



### TLS certificate & key

Consider using Lets Encrypt to easily generate your TLS certificate. For examples on using lego, the command-line Let's Encrypt client, see the documentation. Consider the following example, which may be useful if you're a GCP user:

```bash
GCE_PROJECT="acme-gcp-project" GCE_DOMAIN="acme-co" \
  lego --email="username@acme.co" \
    -x "http-01" \
    -x "tls-sni-01" \
    --domains="fleet.acme.co" \
    --dns="gcloud" --accept-tos run
```

going the  route of a more traditional CA-signed certificate, you'll have to generate a TLS key and a CSR (certificate signing request):

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout tls.key -out tls.csr
```

Now w'll have to give this CSR to a Certificate Authority, and they will give a file called `tls.crt`. We will then have to add the key and certificate as Kubernetes secrets.

```bash
kubectl create secret tls fleet-tls --key=./tls.key --cert=./tls.crt
```

### Deploying Fleet

First, we must deploy the instances of the Fleet webserver. The Fleet webserver is described using a Kubernetes deployment object. To create this deployment, run the following:

```bash
kubectl apply -f ./docs/Deploy/_kubernetes/fleet-deployment.yml
```

You should be able to get an instance of the webserver running via `kubectl get pods` and you should see the following logs:



### Deploying the load balancer

Now that the Fleet server is running on our cluster, we have to expose the Fleet webservers to the internet via a load balancer. To create a Kubernetes Service of type LoadBalancer, run the following:

```bash
kubectl apply -f ./docs/Deploy/_kubernetes/fleet-service.yml
```

### Configure DNS

Finally, we must configure a DNS address for the external IP address that we now have for the Fleet load balancer. Run the following to show some high-level information about the service:

```bash
kubectl get services fleet-loadbalancer
```



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

