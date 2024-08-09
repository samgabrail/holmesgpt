# Overview
Testing Holmes GPT by Robusta

## Add Your OpenAI Key
```bash
export OPENAI_API_KEY=<YOUR_OPENAI_KEY>
```

## Sending AlertManager Alerts to Robusta

As per [this page.](https://docs.robusta.dev/master/configuration/alertmanager-integration/alert-manager.html)

Run this:
```bash
kubectl create secret generic alertmanager-monitoring-kube-prometheus-alertmanager --from-file=alertmanager.yaml -n monitoring
```

## Demo

Each demo scenario can be demonstrated both with the holmes cli locally and with the UI.

### General Setup

1. Ensure you can run the holmes cli locally gpt-4o
2. Ensure Robusta is running in your cluster with the UI enabled and that Holmes is using OpenAI-4o (since it gives the best results) using the [instructions here](https://docs.robusta.dev/master/configuration/ai-analysis.html#ai-analysis)
3. Create a Namespace: Create a namespace for the demo:
```bash
kubectl create namespace demo
```

### Scenario 1: Crashlooping

1. Deploy the Application:
```bash
kubectl apply -f Scenario1-Crashlooping/crashloop-cert-app.yaml -n demo
```
2. **Testing this with the cli:** `holmes ask "what is wrong with the db-certs-authenticator app?"`
3. **Verify Application Status:** In the Robusta UI, navigate to the apps page and ensure `db-certs-authenticator` is running.
4. **Monitor for Errors:** Wait 1-2 minutes. You will see errors on the timeline of the apps page: `PodLifecycleWarning` followed by `CrashLoopBackoff`.
5. **Diagnose the Issue:** Click on the diamond icon for either issue and select the purple button `Find Root Cause`. You will see a certification expired error.
6. **Fixing the Application:** Apply the New Certificate:
```bash
kubectl apply -f Scenario2-Crashlooping/new-cert.yaml -n demo
```
7. **Restart the Application:** Wait about a minute for the application to restart, or manually delete the pod.
8. **Check the Certificate:** Wait another minute or two for the application to validate the certificate.
9. **Review Logs:** In the Robusta UI, open the logs and click `AI Summary`. You should see a message like: `The certificate is valid until 2027-05-03 08:01:22.`

### Scenario 2: Pending pod due to resources

1. **Deploy the Application:**
```bash 
kubectl apply -f Scenario2-PendingResources/pending_pod_resources.yaml -n demo
```
2. **Testing this with the holmes cli:** run: `holmes ask "What is the issue with user-profile-resources in the demo namespace?"`
3. **Application Status:** In the Robusta UI, open the app page for `user-profile-resources.`
4. If you wait 15 minutes a Prometheus alert will fire! Click `Find Root Cause`. You will see an error about insufficient requirements

### Scenario 3: Pending pod due to node selector

1. **Deploy the Application:**
```bash
kubectl apply -f Scenario3-PendingNodeSelector/pending_pod_node_selector.yaml -n demo
```
2. **Testing this with the holmes cli:** run: `holmes ask "What is the issue with user-profile-import in the demo namespace?"`
3. **Application Status:** In the Robusta UI, open the app page for user-profile-import.
4. If you wait 15 minutes a Prometheus alert will fire! Click `Find Root Cause`. You will see an error about node selector

### Teardown

Delete the Namespace:
```bash
kubectl delete namespace demo
```