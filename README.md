[![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.camunda/camunda-release-parent/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.camunda/camunda-release-parent)


# camunda-release-parent

> [!WARNING]
> ## Migration to Sonatype Central Portal
>
> Sonatype is deprecating the legacy [OSSRH](https://central.sonatype.org/news/20250326_ossrh_sunset/) service and moving all artifact publishing to the new [Central Portal](https://central.sonatype.org/faq/what-is-different-between-central-portal-and-legacy-ossrh/).
>
> This migration affects all **Camunda namespaces** including `io.camunda` and `org.camunda`.
>
> ❗ **Important:** The migration is **one-way**. Once a namespace is migrated, publishing via OSSRH will **no longer be possible**.
>
> ### What Changed in this Parent POM?
>
> To support the migration, this parent POM (starting from version `3.10.0`) includes:
>
> - ✅ **`central-sonatype-publish`**, a new Maven profile for publishing via the Central Portal
>   - uses `central-publishing-maven-plugin` (replaces the deprecated `nexus-staging-maven-plugin`)  
>   - reuses the same configurations as in `sonatype-oss-release` profile
> - ⚠️ **`sonatype-oss-release`**, now marked as **deprecated**
>
> 🚨 Projects can upgrade to this new minor version at any time, but the `central-sonatype-publish` profile should only be used after the namespace migration has taken place.
>
> ### What Will Happen on Migration Day?
>
> 📅 Refer to internal communication for the migration date and status.
>
> Once your namespace has been migrated:  
> - OSSRH publishing will no longer work  
>
> At the same time, we will release a new `4.0.0` version of this parent POM, which:
>
> - makes `central-sonatype-publish` the **default profile** used by `maven-release-plugin`
> - reconfigures the deprecated `sonatype-oss-release` profile and `nexus-staging-maven-plugin` to use Sonatype’s Central [Portal OSSRH Staging API](https://central.sonatype.org/publish/publish-portal-ossrh-staging-api/), which is a **partial** reimplementation of the legacy staging API
>     - this allows `nexus-staging-maven-plugin` to deploy through the new Sonatype Central Portal as a temporary (emergency) fallback; please migrate to the new profile and plugin as explained below
>
> ### What You Need to Do on Migration Day  
>
> ✅ If you use the default `maven-release-plugin` configuration:  
> - upgrade this parent POM to `4.0.0` in your `pom.xml`
> - no other changes needed, the new profile will be passed by default as an argument  
>
> ⚠️ If you explicitly use the `sonatype-oss-release` profile:  
> - Switch to `central-sonatype-publish`  
> - No further changes required
>
> 🚨 If you encounter issues with the new `central-sonatype-publish` profile:
> - as a temporary fallback, you can continue using `sonatype-oss-release` with this parent POM in version `4.0.0`
>     - it uses a partial reimplementation of the legacy staging API, exposed by Sonatype’s Central Portal, allowing the deprecated `nexus-staging-maven-plugin` to deploy artifacts
>     - this fallback is not guaranteed and should only be used in case of emergency
>     - please ensure:
>         - the new default value of `nexus.sonatype.url` property is used: https://ossrh-staging-api.central.sonatype.com
>         - `-Dsonatype-oss-release.acknowledge.deprecation=true` is set
>         - `skipStaging` for `nexus-staging-maven-plugin` is not enabled
>
> 📍 If you consume SNAPSHOT artifacts, remember to update your repository URL as per Sonatype’s new [guidelines](https://central.sonatype.org/publish/publish-portal-snapshots/#consuming-snapshot-releases-for-your-project):  
`https://central.sonatype.com/repository/maven-snapshots/`
>
> ### How to Review and Publish artifact Deployment
>
> With the new publishing flow, Maven will log a Deployment Id and a URL:
>
> ```log
> Deployment xxxx-xxxx-xxxx-xxxx-xxxx has been validated. To finish publishing visit https://central.sonatype.com/publishing/deployments
> ```
>
> Visit the provided link to:
> - ✅ Review validation results
> - 🚀 Publish to Maven Central
> - ❌ Drop the Deployment/bundle if needed
> 
> **Note:** Along with the deployment ID, a deployment name is set by default to `Deployment`, unless the maven-release-plugin is used. In that case, the deployment name is automatically set to `${project.groupId}:${project.artifactId}:${project.version}` from the top-level project where the release is performed. Users can override the deployment name at any time by configuring the `central.sonatype.deployment.name` property in their top-level module.

This POM is meant to be inherited in Camunda release builds and defines common release properties. It supports deploying artifacts to two repositories at the same time:

- [Camunda Artifactory](https://artifacts.camunda.com/)
- [Maven Central](https://central.sonatype.com/publishing/deployments) via the Sonatype Central Portal

Artifacts are deployed at the end of the build to reduce the chance of failure when interacting with external systems.


## Usage

Inherit the camunda-release-parent POM inside your project by adding the following:

```xml 
<parent>
  <groupId>org.camunda</groupId>
  <artifactId>camunda-release-parent</artifactId>
  <version><!-- Use the newest version available! --></version>
  <!-- do not remove empty tag - http://jira.codehaus.org/browse/MNG-4687 -->
  <relativePath />
</parent>
```

If you have a multi-module build, simply inherit it in your parent POM.

Specify the <scm> section for your project, eg:
```xml    
<scm>
  <url>https://github.com/camunda/MY_PROJECT_URL</url>
  <connection>scm:git:git@github.com:camunda/MY_PROJECT_URL.git</connection>
  <developerConnection>scm:git:git@github.com:camunda/MY_PROJECT_URL.git</developerConnection>
</scm>
```

## Release

### Prerequisite

Server credentials and your GPG passphrase (required for artifact signing when publishing to Maven Central) can be configured in your local `~/.m2/settings.xml` file.


```xml  
<settings>
  <servers>
    <server>
      <id>camunda-nexus</id>
      <username><!-- Camunda Nexus User (Artifactory) --></username>
      <password><!-- Camunda Nexus Password (Artifactory) --></password>
    </server>
    <server>
      <id>central</id>
      <username><!-- Sonatype Central Portal User --></username>
      <password><!-- Sonatype Central Portal Password --></password>
    </server>
  ...
  </servers>  
  <profiles>
    <profile>
      <id>central-sonatype-publish</id>
      <properties>
        <!-- GPG passphrase for signing artifacts.
            It's safer to provide it at build time using:
              mvn deploy -Dgpg.passphrase=...
            Or use an environment variable:
              <gpg.passphrase>${env.GPG_PASSPHRASE}</gpg.passphrase>
        -->
        <gpg.passphrase><!-- GPG_PRIVATE_KEY_PASSPHRASE --></gpg.passphrase>
      </properties>
    </profile>
    ...
  </profiles>
</settings>
```

### Deploy artifacts

You can use the [Maven Release Plugin](https://maven.apache.org/maven-release/maven-release-plugin/) to prepare and deploy a new release of your project. For example:

```bash
mvn \
  release:prepare release:perform \
  -B -Dresume=false \
  -Dtag=1.0.0 \
  -DreleaseVersion=1.0.0 \
  -DdevelopmentVersion=1.1.0-SNAPSHOT \
  -Darguments="..."
```

This example runs both prepare and perform in a single step, but in practice, it's common to run them separately to review the changes in between.

Use `-Darguments="..."` to pass additional command-line options to the Maven commands the release plugin runs internally. By default, the pluginManagement configuration includes `-P<a-default-sonatype-profile>` in these arguments.
    
    
### Override Default Settings

You can override certain default settings using the properties listed below. To pass these to the release plugin, include them within the `-Darguments=...` option, as mentioned above.

<table>
  <tr>
    <th>Property</th><th>Description</th>
  </tr>
  <tr>
    <td>nexus.snapshot.repository</td><td>Specify the url to your snapshot repository.<br/>Default is <strong>https://artifacts.camunda.com/artifactory/camunda-bpm-snapshots</strong>.</td>
  </tr>
  <tr>
    <td>nexus.release.repository</td><td>Specify the url to your release repository.<br/>Default is <strong>https://artifacts.camunda.com/artifactory/camunda-bpm</strong>.</td>
  </tr>
  <tr>
    <td>skip.nexus.release</td><td>When setting the value to true, skip the deployment to the release repository specified in <distributionManagement>.<br/>Default is <strong>false</strong>.</td>
  </tr>
  <tr>
    <td>skip.central.release</td><td>When setting the value to true, skip the deployment to maven central.<br/>Default is <strong>false</strong>.</td>
  </tr>
  <tr>
    <td>serverId</td><td>Points to the corresponding entry in your settings.xml like <server><id>yourServerId</id></server> to specify your login credentials.<br/>Default is <strong>central</strong> for maven central.</td>
  </tr>
</table>

### Testing

You can test your release steps by using the following Maven release command:

```bash
mvn \
release:prepare release:perform \
--batch-mode \
-DpushChanges=false -DremoteTagging=false \
-DlocalCheckout=true -Dresume=false \
-Dtag=1.0.0 -DreleaseVersion=1.0.0 -DdevelopmentVersion=1.1.0-SNAPSHOT \
-Darguments="-Dskip.camunda.release=true -Dskip.central.release=true" \
```

This command performs a release without pushing changes to remote Git repositories or deploying artifacts.
> Hint: You can use a Nexus Docker image to extend your testing by hosting a local repository.
