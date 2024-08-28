Yes, the event payload from the GCS event source in Argo Events will include the file name, along with other relevant metadata such as the bucket name, event type, and more. When a file is added or removed from the GCS bucket, this information is included in the event payload, which can be used in your workflows or triggers.

### Example of an Event Payload

When a file is added or removed, the event payload typically looks like this:

```json
{
  "bucket": "my-gcs-bucket",
  "name": "my-folder/my-file.txt",
  "timeCreated": "2024-08-28T12:34:56.789Z",
  "updated": "2024-08-28T12:34:56.789Z",
  "size": "1024",
  "contentType": "text/plain",
  "eventType": "OBJECT_FINALIZE" // or "OBJECT_DELETE"
}
```

- **`bucket`**: The name of the GCS bucket.
- **`name`**: The file name (including the path within the bucket).
- **`eventType`**: The type of event (e.g., `OBJECT_FINALIZE` for file creation/modification or `OBJECT_DELETE` for file deletion).

### Using a Service Account Instead of JSON Credentials

If you want to use a **Kubernetes Service Account** instead of JSON credentials, you can configure the GCS event source to use **Workload Identity** or **GCP IAM Roles**. Here’s how you can achieve that:

#### 1. Enable Workload Identity

1. **Enable Workload Identity on your GKE Cluster** (if not already enabled).

   ```bash
   gcloud container clusters update [CLUSTER_NAME] \
     --workload-pool=[PROJECT_ID].svc.id.goog
   ```

2. **Create a Kubernetes Service Account (KSA)** and bind it to a GCP Service Account (GSA) with the necessary permissions (like `Storage Object Viewer` for GCS).

   ```bash
   kubectl create serviceaccount argo-gcs-sa
   ```

3. **Bind the KSA to a GSA**:

   ```bash
   gcloud iam service-accounts add-iam-policy-binding \
     [GSA_NAME]@[PROJECT_ID].iam.gserviceaccount.com \
     --role roles/iam.workloadIdentityUser \
     --member "serviceAccount:[PROJECT_ID].svc.id.goog[YOUR_NAMESPACE/argo-gcs-sa]"
   ```

4. **Annotate the Kubernetes Service Account**:

   ```bash
   kubectl annotate serviceaccount \
     argo-gcs-sa \
     iam.gke.io/gcp-service-account=[GSA_NAME]@[PROJECT_ID].iam.gserviceaccount.com
   ```

#### 2. Update the GCS Event Source

Modify your GCS event source YAML to reference the Kubernetes Service Account:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: EventSource
metadata:
  name: gcs-events
spec:
  gcs:
    example-gcs:
      bucket: my-gcs-bucket
      # Use the Kubernetes Service Account instead of the JSON credentials
      serviceAccountName: argo-gcs-sa
      events:
        - OBJECT_FINALIZE
        - OBJECT_DELETE
      prefix: my-folder/
```

### Deploy Updated Configuration

Deploy the updated event source to your cluster:

```bash
kubectl apply -f gcs-event-source.yaml
```

### Summary

- The event payload will include the file name and other metadata.
- You can use a Kubernetes Service Account (via Workload Identity) instead of JSON credentials to authenticate with GCS.

---

**a.** Would you like to set up a sample workflow to process the file name from the event payload?

**b.** Do you want guidance on setting up the necessary IAM permissions for the GCP Service Account?
