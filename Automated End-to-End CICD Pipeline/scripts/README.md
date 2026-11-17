# Scripts directory

The final pipeline uses the commands in the root `Jenkinsfile`; no separate installation script was part of the recorded final pipeline.

This directory is reserved for optional, environment-specific helper scripts, such as EC2 installation scripts, service start/stop scripts, connectivity checks, and maintenance scripts.

Only add scripts after validating them in your AWS environment. Do not add passwords, API keys, private keys, or other secrets. Keep credentials in Jenkins Credentials or an approved secret manager.

The pipeline's Tomcat deployment command is already defined in the root `Jenkinsfile`.
