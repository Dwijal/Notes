### `kubectl rollout status <deployment-name>` Outputs

This command tracks the progress of a Kubernetes Deployment rollout.

#### 1. **Rollout In Progress (Healthy)**

Waiting for deployment "my-app" rollout to finish: X of Y updated replicas are available...

* **Meaning:** New pods are being created and becoming ready; old pods are terminating. Progressing as expected.

#### 2. **Rollout Completed Successfully**

deployment "my-app" successfully rolled out

* **Meaning:** All desired new pods are running and ready; all old pods terminated. Deployment is stable.

#### 3. **Rollout Stuck / Progressing Slowly**

Waiting for deployment "my-app" rollout to finish: 0 of Y updated replicas are available...

*(This line repeats indefinitely)*
* **Meaning:** New pods are created but are **not becoming ready** (e.g., `CrashLoopBackOff`, readiness probe failures, resource issues, bad config). Rollout is stalled.

#### 4. **Rollout Timed Out**
error: deployment "my-app" exceeded its progress deadline

* **Meaning:** The rollout failed to complete within the `progressDeadlineSeconds`. Indicates a serious issue with the new version.

#### 5. **Rollout Paused**
deployment "my-app" rollout paused

* **Meaning:** The deployment's rollout has been manually halted.

#### 6. **Deployment Not Found**
Error from server (NotFound): deployments.apps "non-existent-app" not found

* **Meaning:** The specified Deployment does not exist.

---

**Debugging Tip:** If a rollout is stuck or timed out, immediately check `kubectl get pods`, 
