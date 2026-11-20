# Maven Web App – Jenkins, Nexus, Docker and Amazon EKS

This package contains CI/CD and Kubernetes configuration files to add to the existing
`maven-web-app` application repository. It does NOT replace the Java source code or
the existing Maven `pom.xml`.

## Files

- `Jenkinsfile` – Declarative Jenkins pipeline.
- `Dockerfile` – builds a Tomcat-based image containing `target/maven-web-app.war`.
- `k8s/deployment.yaml` – Deployment; Jenkins replaces `__IMAGE__` at build time.
- `k8s/service.yaml` – LoadBalancer Service, port 80 to container port 8080.
- `k8s/hpa.yaml` – CPU-based Horizontal Pod Autoscaler (2 to 5 replicas).
- `docs/architecture.png` – architecture diagram.

## Before running the pipeline

1. Keep the existing Java source folders and `pom.xml` from the original project. Configure the Jenkins Pipeline job to use this GitHub repository and its branch; `checkout scm` checks out that configured repository.
2. Check `pom.xml` and replace `REPLACE_WITH_POM_GROUP_ID` in `Jenkinsfile` with its exact
   `<groupId>` value. Confirm the generated WAR is named `maven-web-app.war`.
3. Create a Nexus hosted Maven repository named `maven-releases` and ensure it permits uploads.
4. This pipeline expects Nexus to run on the same Jenkins EC2 host with port 8081 published.
5. Create Jenkins credentials:
   - `nexus-credentials` – Nexus username/password.
   - `dockerhub-credentials` – Docker Hub username/access token.
   - `eks-management-ssh` – SSH private-key credential for the Management Host.
6. Configure Maven in Jenkins Global Tool Configuration with the exact name `Maven-3`.
7. Install/configure Pipeline, Git, Credentials Binding, SSH Agent and JUnit plugins. The
   Jenkins agent also needs Docker CLI/daemon access, curl, SSH and SCP.
8. Set `EKS_HOST` as a Jenkins global environment variable to the CURRENT private IP or
   reachable DNS name of the EKS Management Host. Do not use an old/terminated EC2 IP.
9. Ensure the Management Host has a working kubeconfig for the intended EKS cluster.
10. Ensure the Docker Hub image is public, or configure Kubernetes imagePullSecrets.
11. Ensure the EKS cluster has Metrics Server or another working metrics API for HPA.
12. Ensure the EKS node group has capacity for the Pods.

## Implementation notes

- HPA is a separate manifest; it is not inserted into the Deployment YAML.
- The Deployment omits `spec.replicas` so HPA can control the replica count.
- No HTTP probes are included because the correct application health URL has not yet
  been confirmed. Add probes only after testing the endpoint.
- `kubectl top pods` is not ignored; verification fails if resource metrics are unavailable.
- The WAR is uploaded to Nexus and downloaded again before Docker image creation.
- The Dockerfile uses `tomcat:latest`, consistent with the original WAR-based project.
- The `build-<BUILD_NUMBER>` image tag is deployed; `latest` is also pushed.
- This package has not been run against your AWS account. Test it before claiming the
  enhanced pipeline is successfully implemented.

## Add these files to the existing Git repository

Copy `Jenkinsfile`, `Dockerfile`, `k8s/` and `docs/` into the root of the existing
application repository. Keep its existing `pom.xml` and Java source folders.

Then run:

```bash
git status
git add Jenkinsfile Dockerfile k8s docs
git commit -m "Add Jenkins Nexus Docker and EKS CI/CD pipeline"
git push origin main
```

If your default branch is not `main`, use its actual name.
