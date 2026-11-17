# End-to-End CI/CD Pipeline Implementation Using Jenkins on AWS

## 1. Project overview

This project implements a manually triggered, end-to-end CI/CD pipeline for a Java web application. Jenkins orchestrates the workflow and integrates GitHub, Maven, SonarQube, Nexus Repository Manager, and Apache Tomcat on AWS EC2 instances.

The pipeline performs five stages:

1. **GitHub checkout** – retrieves the application source code.
2. **Maven build** – compiles the application and packages it as a WAR file.
3. **SonarQube analysis** – runs static code analysis.
4. **Nexus upload** – publishes the generated WAR artifact to a hosted Maven repository.
5. **Tomcat deployment** – deploys the WAR file to the Tomcat server through the Tomcat Manager text API.

The final pipeline run completed successfully, and Tomcat confirmed deployment at context path `/maven-web-app`.

## 2. Architecture

See [`docs/architecture.png`](docs/architecture.png).

## 3. Technology stack

| Component | Role | Version / configuration used |
|---|---|---|
| AWS EC2 | Hosts the CI/CD tools | Separate EC2 instances |
| Jenkins | Pipeline orchestration | Version not recorded |
| Ubuntu | Jenkins host OS | Version not recorded |
| Amazon Linux 2023 | SonarQube, Nexus and Tomcat host OS | Amazon Linux 2023 |
| Java | Runtime | Java 21 for Jenkins and Nexus; Java 17 for SonarQube and Tomcat |
| Maven | Build tool | 3.8.7; Jenkins tool name `maven3` |
| SonarQube | Static code analysis |
| Nexus Repository Manager | Artifact repository |
| Apache Tomcat | Web application server |
| GitHub | Source code repository | `suffixscope/maven-web-app` |

## 4. Repository relationship

This repository contains the **DevOps pipeline implementation**. The Java application source code is maintained separately in:

`https://github.com/suffixscope/maven-web-app`

The Jenkinsfile checks out that application repository, builds it, analyses it, uploads its WAR artifact to Nexus, and deploys it to Tomcat.

## 5. Infrastructure and endpoints

The following endpoints reflect the environment used during the project. Public IP addresses can change if EC2 instances are stopped and restarted without Elastic IPs.

| Server | OS | Private IP | Service port |
|---|---|---:|---:|
| Jenkins | Ubuntu | `IP ADDRESS` | 8080 |
| SonarQube | Amazon Linux 2023 | `IP ADDRESS` | 9000 |
| Nexus | Amazon Linux 2023 | IP ADDRESS | 8081 |
| Tomcat | Amazon Linux 2023 | `IP ADDRESS` | 8082 |

Additional known public endpoints:
- SonarQube: `http://IP ADDRESS:9000`
- Nexus: `http://IP ADDRESS:8081`
- Tomcat application: `http://IP ADDRESS:8082/maven-web-app/`
- Tomcat Manager: `http://IP ADDRESS:8082/manager/html`

The Jenkinsfile uses the Nexus public address and the Tomcat private address. Jenkins must be able to reach the Tomcat private address within the AWS network.

## 6. Jenkins configuration prerequisites

Before running the pipeline, configure the following on the Jenkins server.

### Global tools
- Configure Maven 3.8.7 with the Jenkins tool name `maven3`.
- Ensure a compatible Java runtime is configured for Jenkins and the build.

### SonarQube server
- In **Manage Jenkins → System**, configure a SonarQube installation named `sonarqube`.
- Set its URL to `http://IP ADDRESS:9000`.
- Configure the SonarQube token securely in Jenkins Credentials and associate it with the SonarQube installation. The credential ID used in the project was `sonar-token`.

### Jenkins credentials
Create these credentials securely in Jenkins:

| Credential ID | Type | Purpose |
|---|---|---|
| `nexus-credentials` | Username/password | Upload artifact to Nexus |
| `tomcat-credentials` | Username/password | Deploy to Tomcat Manager |

The Tomcat account used by the project was `tomcat USERNAME` with the `manager-script` role. Do not store passwords or tokens in this Git repository.

### Required Jenkins plugins
- Pipeline
- Git
- SonarQube Scanner for Jenkins
- Nexus Artifact Uploader

The Nexus Artifact Uploader plugin must provide the `nexusArtifactUploader` pipeline step.

## 7. How to run

1. Confirm all EC2 instances and required services are running.
2. Confirm Jenkins can reach SonarQube, Nexus, and Tomcat over the required ports.
3. Confirm the Jenkins tool and credential IDs listed above exist.
4. Create a Pipeline job in Jenkins.
5. Configure the job to use this repository's `Jenkinsfile` (Pipeline script from SCM is recommended).
6. Run **Build Now**.
7. Review the five stage results in the Jenkins console output.
8. Verify the deployed application at `http://IP ADDRESS:8082/maven-web-app/`.

This project was manually triggered using **Build Now**. A GitHub webhook or automatic polling trigger was not configured.

## 8. Build and artifact details

- Maven output file: `target/maven-web-app.war`
- Nexus repository: `maven-releases`
- Group ID: `******`
- Artifact ID: `01-maven-web-app`
- Version: `3.0-RELEASE`
- Packaging: `war`

## 9. Security notes

- Never commit AWS access keys, private keys, passwords, SonarQube tokens, or other secrets.
- Store credentials in Jenkins Credentials and reference them by credential ID.
- Use restricted security-group rules; do not expose administrative ports to the whole internet.
- The Tomcat Manager endpoint should be restricted to trusted hosts. The project allowed the Jenkins private IP for the Manager text API.
- HTTP was used for the configured tool endpoints in this implementation. For production, use TLS and appropriate network/security controls.

## 10. Current scope and possible improvements

Implemented:
- Five-stage Jenkins pipeline.
- Maven WAR packaging.
- SonarQube static analysis.
- Nexus artifact upload.
- Tomcat deployment.
- AWS EC2-based tool hosting.

Not implemented in this project:
- GitHub webhook / automatic SCM trigger.
- Separate SonarQube Quality Gate enforcement stage.
- Automated rollback.
- Docker or Kubernetes deployment.
- Terraform-based infrastructure provisioning.

Possible future enhancements include webhook-based triggering, Quality Gate enforcement, automated rollback, infrastructure as code, containerization, and HTTPS configuration.

## 11. Troubleshooting notes

- **SonarQube unavailable:** verify the EC2 instance and SonarQube service are running and that port 9000 is reachable from Jenkins.
- **Nexus redeployment rejected:** the hosted repository used an `Allow redeploy` deployment policy for this artifact version.
- **Tomcat deployment returns HTTP 401:** verify Jenkins credential ID `tomcat-credentials` and the credential binding variables `TOMCAT_USER` and `TOMCAT_PASSWORD`.
- **Tomcat Manager access denied:** verify the Tomcat Manager script user has the `manager-script` role and that the Manager IP restriction permits the Jenkins private IP.
- **Connection refused / timeout:** check service status, listening ports, EC2 security groups, and network connectivity.

## 12. License

Add a license if you intend to publish this repository for reuse. Do not imply that the separately maintained Java application is licensed by this repository.
