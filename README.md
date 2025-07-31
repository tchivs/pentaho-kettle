# Pentaho Data Integration # 

Pentaho Data Integration ( ETL ) a.k.a Kettle

### Project Structure

* **assemblies:** 
Project distribution archive is produced under this module
* **core:** 
Core implementation
* **dbdialog:** 
Database dialog
* **ui:** 
User interface
* **engine:** 
PDI engine
* **engine-ext:** 
PDI engine extensions
* **[plugins:](plugins/README.md)** 
PDI core plugins
* **integration:** 
Integration tests

How to build
--------------

Pentaho Data Integration uses the Maven framework. 


#### Pre-requisites for building the project:
* Maven, version 3.6+ (recommended: 3.9+)
* Java JDK 17 (LTS)
* This [settings.xml](https://raw.githubusercontent.com/pentaho/maven-parent-poms/master/maven-support-files/settings.xml) in your <user-home>/.m2 directory

#### System Requirements:
* **Operating System**: Windows, Linux, or macOS
* **Memory**: Minimum 4GB RAM (recommended: 8GB+)
* **Disk Space**: At least 2GB free space for build artifacts
* **Network**: Internet connection for downloading dependencies

#### Building it

This is a Maven project, and to build it use the following command:

```bash
$ mvn clean install
```

**Build Options:**
* `-Drelease` - Trigger obfuscation and/or uglification (as needed)
* `-Dmaven.test.skip=true` - Skip the tests (not recommended)
* `-DskipTests` - Skip test execution but compile test classes
* `-B` - Batch mode (non-interactive, recommended for CI/CD)

**Quick compile check (faster):**
```bash
$ mvn clean compile -B -DskipTests
```

The build result will be a Pentaho package located in ```target```.

#### Packaging / Distributing it

Packages can be built by using the following command:
```
$ mvn clean package
```

The packaged results will be in the `target/` sub-folders of `assemblies/*`.

For example, a distribution of the Desktop Client (CE) can then be found in: `assemblies/client/target/pdi-ce-*-SNAPSHOT.zip`.

#### Running the tests

__Unit tests__

This will run all unit tests in the project (and sub-modules). To run integration tests as well, see Integration Tests below.

```
$ mvn test
```

If you want to remote debug a single Java unit test (default port is 5005):

```
$ cd core
$ mvn test -Dtest=<<YourTest>> -Dmaven.surefire.debug
```

__Integration tests__

In addition to the unit tests, there are integration tests that test cross-module operation. This will run the integration tests.

```
$ mvn verify -DrunITs
```

To run a single integration test:

```
$ mvn verify -DrunITs -Dit.test=<<YourIT>>
```

To run a single integration test in debug mode (for remote debugging in an IDE) on the default port of 5005:

```
$ mvn verify -DrunITs -Dit.test=<<YourIT>> -Dmaven.failsafe.debug
```

To skip test

```
$ mvn clean install -DskipTests
```

To get log as text file

```
$ mvn clean install test >log.txt
```

### Dependencies and Compatibility

#### Java Version Support
* **Java 17 (LTS)** - Primary supported version
* **Java 11** - Legacy support (may require additional configuration)
* **Java 8** - No longer supported

#### Key Dependencies
* **Maven**: 3.6+ (uses modern lifecycle phases)
* **Jetty**: 12.x (requires Java 17+)
* **Spring Framework**: 6.x
* **Hibernate**: 6.x
* **JAXB**: 2.3.1 (for Java 11+ compatibility)

#### Known Issues
* Some third-party libraries may show warnings with newer Java versions
* Eclipse SWT dependencies are platform-specific
* Network timeouts may occur during dependency download (retry recommended)

__IntelliJ__

* Don't use IntelliJ's built-in maven. Make it use the same one you use from the commandline.
  * Project Preferences -> Build, Execution, Deployment -> Build Tools -> Maven ==> Maven home directory

### Troubleshooting

#### Common Build Issues

**Java Version Mismatch:**
```bash
# Check your Java version
$ java -version
$ mvn -version

# Ensure JAVA_HOME points to JDK 17
$ echo $JAVA_HOME  # Linux/macOS
$ echo %JAVA_HOME% # Windows
```

**Maven Dependency Issues:**
```bash
# Clear Maven cache and retry
$ mvn dependency:purge-local-repository
$ mvn clean install

# Force update snapshots
$ mvn clean install -U
```

**Memory Issues:**
```bash
# Increase Maven memory
$ export MAVEN_OPTS="-Xmx4g -Xms1g"  # Linux/macOS
$ set MAVEN_OPTS=-Xmx4g -Xms1g       # Windows
```

**Network/Proxy Issues:**
- Configure Maven proxy settings in `~/.m2/settings.xml`
- Use `-Dmaven.wagon.http.retryHandler.count=3` for retry on network failures

For more detailed troubleshooting, see: `doc/build-troubleshooting.md`

### Contributing

1. Submit a pull request, referencing the relevant [Jira case](https://jira.pentaho.com/secure/Dashboard.jspa)
2. Attach a Git patch file to the relevant [Jira case](https://jira.pentaho.com/secure/Dashboard.jspa)

Use of the Pentaho checkstyle format (via `mvn checkstyle:check` and reviewing the report) and developing working 
Unit Tests helps to ensure that pull requests for bugs and improvements are processed quickly.

When writing unit tests, you have at your disposal a couple of ClassRules that can be used to maintain a healthy
test environment. Use [RestorePDIEnvironment](core/src/test/java/org/pentaho/di/junit/rules/RestorePDIEnvironment.java)
and [RestorePDIEngineEnvironment](engine/src/test/java/org/pentaho/di/junit/rules/RestorePDIEngineEnvironment.java)
for core and engine tests respectively.

pex.:
```java
public class MyTest {
  @ClassRule public static RestorePDIEnvironment env = new RestorePDIEnvironment();
  setUp(){}
  @Test public void testSomething() { 
    assertTrue( myMethod() ); 
  }
}
```  

### Asking for help
Please go to https://community.hitachivantara.com/community/products-and-solutions/pentaho/ to ask questions and get help.
