# Pipeline overview

1. GitHub checkout: `https://github.com/suffixscope/maven-web-app.git`
2. Maven build: `mvn clean package` creates `target/maven-web-app.war`.
3. SonarQube: Maven Sonar Scanner plugin performs code analysis.
4. Nexus: uploads the WAR to `maven-releases` with coordinates `org.scopeindia:01-maven-web-app:3.0-RELEASE`.
5. Tomcat: deploys the WAR to context path `/maven-web-app` through the Tomcat Manager text API.

See the root `Jenkinsfile` for the executable pipeline.
