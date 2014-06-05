camunda-release-parent
======================

Pom which can be inherited for camunda releases defining some common release properties
It allows to deploy to two repositories simultaneously. One is a Nexus OSS server, the other one a Nexus Enterprise server.
It will deploy the artifacts at the end of the build to keep the window of failure small when talking to external systems.

Usage
-----

Inherit the camunda-release-parent pom inside your project like so  
  
    <parent>
      <groupId>org.camunda</groupId>
      <artifactId>camunda-release-parent</artifactId>
      <version>${latestVersion}</version>
      <!-- do not remove empty tag - http://jira.codehaus.org/browse/MNG-4687 -->
     <relativePath />
    </parent>  
    
If you have a multi-module build, just inherit in your parent pom.  

Specify the <scm> section for your project eg.
    
    <scm>
      <url>https://github.com/camunda/MY_PROJECT_URL</url>
      <connection>scm:git:git@github.com:camunda/MY_PROJECT_URL.git</connection>
      <developerConnection>scm:git:git@github.com:camunda/MY_PROJECT_URL.git</developerConnection>
    </scm>

Release
-------

Prerequisite:  

  Add the following to the profiles and servers section of your local settings.xml file. This allows the signing of your artifacts during the release. <strong>Mandatory for maven central</strong>.
  
    <settings>
      ...
      
      <servers>
        ...
        
        <server>
          <id>camunda-nexus</id>
          <username>MY_CAMUNDA_NEXUS_USER</username>
          <password>MY_CAMUNDA_NEXUS_PASSWORD</password>
        </server>
        <server>
          <id>central</id>
          <username>MY_CENTRAL_USER</username>
          <password>MY_CENTRAL_PASSWORD</password>
        </server>
        
      </servers>
      
      
      <profiles>
        ...
        
        <profile>
          <id>sonatype-oss-release</id>
          <properties>
            <gpg.passphrase>YOUR_SECRET_GPG_PASSPHRASE</gpg.passphrase>
          </properties>
        </profile>
        
      </profiles>
    </settings>

To release your own project use the following command

    mvn \
    release:prepare release:perform \
    -B -Dresume=false -Dtag=1.0.0 -DreleaseVersion=1.0.0 -DdevelopmentVersion=1.1.0-SNAPSHOT \
    -Darguments="--settings=${PATH_TO_YOUR_SETTINGS_XML_FILE}" \
    --settings=${PATH_TO_YOUR_SETTINGS_XML_FILE}
    
This will trigger the `sonatpye-oss-release` profile inside the camunda-release-parent pom automatically.
    
Customization
-------------

You can override some default behaviours through the usage of the command line properties below. Add those properties to the `-Darguments="INSERT_PROPERTIES_HERE"` part of the maven release command.

<table>
  <tr>
    <th>Property</th><th>Description</th>
  </tr>
  <tr>
    <td>nexus.snapshot.repository</td><td>Specify the url to your snapshot repository.<br/>Default is <strong>https://app.camunda.com/nexus/content/repositories/camunda-bpm</strong>.</td>
  </tr>
  <tr>
    <td>nexus.release.repository</td><td>Specify the url to your release repository.<br/>Default is <strong>https://app.camunda.com/nexus/content/repositories/camunda-bpm</strong>.</td>
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
  <tr>
    <td>nexusUrl</td><td>The plain url which points to your nexus installation.<br/><strong>Must be a Nexus Enterprise Edition</strong><br/>Default is <strong>https://oss.sonatype.org</strong> aka maven central.</td>
  </tr>
</table>

Testing
-------

You can test your release steps by using the following maven release command.

    mvn \
    release:prepare release:perform \
    -B -Dresume=false -Dtag=1.0.0 -DreleaseVersion=1.0.0 -DdevelopmentVersion=1.1.0-SNAPSHOT \
    -Darguments="--settings=/Users/hawky4s/.m2/settings.xml -Dskip.central.release=true -Dnexus.release.repository=http://localhost:49081/content/repositories/camunda-bpm-test" \
    --settings=/Users/hawky4s/.m2/settings.xml \
    -DpushChanges=false -DremoteTagging=false -DlocalCheckout=true
    
This will do a release but will not push anything to the involved git repositories and will not deploy to maven central. Instead it will deploy to a local nexus repository. (Hint: I use a simple nexus docker image to do so.)
