# Maven multi-modules for SonarQube

Sample project to help the ticket investigation.

This is a Maven multi-modules with a Spring Boot 2 `backend` an Angular `frontend`.

With this configuration I've noted a strange behavior since the concept of module have removed in the release **7.6**,
see [MMF-365](https://jira.sonarsource.com/browse/MMF-365)

The `sonar-project.properties` is defined globally in the root of the project, and loaded with the Maven `properties-maven-plugin`.
The Angular frontend module is Mavenized with the [frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin).

# Versions

* SonarQube version: **8.6.1**
* SonarQube Maven plugging version: **3.8.0.2131**

# How to run

If you want start a local SonarQube instance run the following docker command:

    docker-compose -f docker/sonar.yml up -d

Then start a sonar analysis with the maven command:

    ./mvnw clean verify sonar:sonar 

# Issues

## Source paths frontend module

During the `frontend` module indexing the source paths is `pom.xml` instead of the global configuration `sonar.sources=src/`:

```
[INFO]   Base dir: /sonar-multimodule-maven/frontend
[INFO]   Source paths: pom.xml
[INFO]   Excluded sources: src/main/webapp/content/**/*.*, src/main/webapp/i18n/*.js, target/classes/static/**/*.*
```

=> Only the `pom.xml` file as source path ?

### Workaround

Override the source paths for the `frontend` module in the pom properties or in a sonar-project.properties:

```xml
<properties>
<node.version>v12.16.1</node.version>
<npm.version>6.14.5</npm.version>
<!-- Uncomment to override global sonar configuration to handle this module source... -->
<sonar.sources>src/</sonar.sources>
</properties>
```

## JaCoCo is run for all child modules

Even if JaCoCo is not applicable for the `frontend` and `parent` modules, the sensor JaCoCo XML Report is running.

Especially for frontends we use the [lcov report](https://wiki.documentfoundation.org/Development/Lcov) (via `sonar.typescript.lcov.reportPaths`)

```
[WARNING] No coverage report can be found with sonar.coverage.jacoco.xmlReportPaths='target/jacoco/test/jacoco.xml,target/jacoco/integrationTest/jacoco.xml'. Using default locations: target/site/jacoco/jacoco.x
ml,target/site/jacoco-it/jacoco.xml,build/reports/jacoco/test/jacocoTestReport.xml
```

=> How to skip the JaCoCo sensor for some modules?

## Define global ignore issue

For large projects or in the corporate world, a simple way to control what can
and cannot be ignored by project teams in reports is to use rules in a "super pom" or just in the parent pom.

Example to ignore globally the **S3437** rule with the **resourceKey** `rc/main/java/**/*`:

```properties
# Rule https://sonarcloud.io/coding_rules?open=squid%3AS3437&rule_key=squid%3AS3437 is ignored, as a JPA-managed field cannot be transient
sonar.issue.ignore.multicriteria.S3437.resourceKey=src/main/java/**/*
sonar.issue.ignore.multicriteria.S3437.ruleKey=squid:S3437
```

However, this raises warnings because it is necessary to specify all the relative paths:

```
[WARNING] Specifying module-relative paths at project level in property 'sonar.issue.ignore.multicriteria' is deprecated. To continue matching files like 'backend/src/main/java/com/example/myapp/backend/Backend
Application.java', update this property so that patterns refer to project-relative paths.
```

I have noted the [SONAR-11587](https://jira.sonarsource.com/browse/SONAR-11587) recommendations,
but in the context of a global "super pom" company configuration this is not applicable. 
Do you have a solution ?
