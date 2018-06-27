# hello-spring

This is a very basic "Hello World!" web app using the Spring Boot java frameork:
[http://spring.io](http://spring.io).

I intended **hello-spring** to be used with OpenShift workshops, but this can be used however you want. 
This project also demonstrates JUnit testing.

This README file will provide instructions for building and running the service locally, as well as instructions for deploying to OpenShift using S2I (source to image) and other methods.

*On a side note, you can easily spin up a Spring Boot project yourself from scratch by using the ["SPRING INITIALIZR"](http://start.spring.io). This will generate the maven pom.xml file and initial project directory structure that can compile and run right away, including setup for JUnit testing, which is demonstrated here in `hello-spring`.*

This project is set up to use maven to build the jar executable, so ensure that it is installed on your build system ([maven](https://maven.apache.org/install.html)). 

**Note**:  `yum`, `apt-get`, `brew install`, 
`chocolatey`, etc.. can all be used to install maven.

## Building and running locally
Once you clone the project to your local system, you can build and test locally by running:

`mvn clean test` - for unit testing

`mvn clean package` - to compile a *jar executable 
which you can run locally

`mvn spring-boot:run` - to build and run your code in one step.

After successful compilation, you will find the executable jar in the `target/` directory.

You can also run the `hello` service with the following command from the project root directory:

`java -jar target/*.jar`

Once the service is running, point your web browser to `localhost:8080` or run the command:

`curl localhost:8080` 

on the command line to see the welcome message.

Note: Try `localhost:8080/api` as well!

## Unit testing
The test class `HelloControllerTest` contains a unit test to ensure the application returns the string "Hello World!"

~~~
@Test
    public void getHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/").accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("Hello World!")));
    }
~~~

When you use maven to package or test, you can see if the test passes or fails (passes by defualt).

`mvn test`

~~~
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 8.661 s
[INFO] Finished at: 2018-06-20T07:49:20-07:00
[INFO] ------------------------------------------------------------------------

~~~


To see the unit test fail, edit `String greeting`string in the `HelloController` class.

Example:

~~~
@RestController
public class HelloController {

	String greeting = "Hello Foo!";

    @RequestMapping("/")
    public String index() { return greeting; }
    ...

}
~~~

If you run your test again with maven, you will get an error:

`mvn test`

~~~
[ERROR] Failures: 
[ERROR]   HelloControllerTest.getHello:29 Response content
Expected: "Hello World!"
     but: was "Hello Foo!"
[INFO] 
[ERROR] Tests run: 1, Failures: 1, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.799 s
[INFO] Finished at: 2018-06-20T08:03:55-07:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.21.0:test (default-test) on project hello: There are test failures.
~~~

Fix your code and verify it will pass testing once again.

## Deploy code to OpenShift using S2I

create a new OpenShift project to run your hello-spring application/service

`oc new-project hello-spring`

The Java S2I image enables developers to automatically build, deploy and run java applications on demand, in OpenShift Container Platform, by simply specifying the location of their application source code or compiled java binaries. In many cases, these java applications are bootable “fat jars” that include an embedded version of an application server and other frameworks (spring-boot in this instance).

### Getting the image stream 

If your OpenShift environment doesn't already have a java image present in its catalog (as in some versions of minishift, for some reason) you can create one in the OpenShift project by importing an image stream. 

### Getting the image stream from an official source

Go to [https://access.redhat.com/containers](https://access.redhat.com/containers)

*Builder Image -> Java Application -> Get Latest Image -> Choose your platform: Red Hat OpenShift*

From here you can copy and paste the import command provided:

example `oc import-image my-redhat-openjdk-18/openjdk18-openshift --from=registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift —confirm`

This will tell OpenShift how to find the Java S2I image. 

validate the name of the image stream:

`oc get is`

~~~
NAME                  DOCKER REPO                                       TAGS      UPDATED
openjdk18-openshift   172.30.1.1:5000/hello-maven/openjdk18-openshift   latest    20 minutes ago
~~~

Now you can deploy the service from github

`oc new-app https://github.com/bugbiteme/hello-spring.git --name hello --image-stream=openjdk18-openshift`

A build gets created and starts compiling and creating the java container from the image. You can follow the build logs using OpenShift Web Console or OpenShift CLI command:

`oc logs -f bc/hello`

Some of this should look similar from when you built the applicatin locally, including unit test status!

Once the container image has been built, it can be deployed, scaled and added to your CI/CD pipeline.

In order to access the `hello` app (e.g. from a browser), it needs to be exposed and added to the load balancer. Run the following command to add the Web UI service to the built-in HAProxy load balancer in OpenShift.

~~~
oc expose svc/hello
oc get route hello
~~~
 
 Output:
 
~~~
route "hello" exposed
NAME      HOST/PORT                                  PATH      SERVICES   PORT       TERMINATION   WILDCARD
hello     hello-hello-spring.192.168.99.100.nip.io             hello      8080-tcp                 None
~~~

### Validate 
validate it is running using curl (or a web browser)

`curl hello-hello-spring.<IP ADDRESS>`
 
 Output:
 
 `Hello World!`
 

Try the rest API:

`curl hello-hello-spring.<IP ADDRESS>/api`
 
 Output:
 
 `{"greeting":"Hello World!"}`

## Deploying prebuilt jar from local build

In some instances, you may want to compile and test the code locally and deploy the localy built jar/war, in this case, from a new OpenShift project with no application deployed, use the maven fabric8 plugin to build and deploy your application/service:

`mvn fabric8:deploy`

There are directives in `pom.xml` which will tell allow for the fabric8 plugin to be used:

~~~
   <properties>
		...
		<!--fabric8 info-->
		<fabric8.version>2.3.4</fabric8.version>
		<spring.k8s.bom.version>0.2.0.RELEASE</spring.k8s.bom.version>
		<k8s.client.version>2.4.1</k8s.client.version>
		<fabric8.maven.plugin.version>3.5.32</fabric8.maven.plugin.version>
		...
	</properties>
~~~

and

~~~
           <!--fabric8 info-->
			<plugin>
				<groupId>io.fabric8</groupId>
				<artifactId>fabric8-maven-plugin</artifactId>
				<version>${fabric8.maven.plugin.version}</version>
				<executions>
					<execution>
						<goals>
							<goal>resource</goal>
							<goal>build</goal>
						</goals>
					</execution>
				</exec>
				...
			</plugin>
~~~

## Setting up a Jenkins pipeline (BELOW IS A WORK IN PROGRESS AND NOT INTENDED FOR USE!!!)
 
Since minishift cannot be access from external sources (such as public github) we will first start off by setting up local git repository using [Gogs](https://gogs.io)

`oc new-project gogs`
`oc new-app -f http://bit.ly/openshift-gogs-template --param=HOSTNAME=gogs-gogs.<YOUR IP>.nip.io`
`oc get route gogs`

Once deployed:

- Register a new account

- From the **+** in the upper right corner, select "New Migration"

- Clone this repo ("Clone Address"): 

  - `https://github.com/bugbiteme/hello-spring.git`

- Repository Name: "hello-spring"

Now create a new subdirectory for your new repo

~~~
mkdir gogs
cd gogs
git clone http://gogs-gogs.<IP-ADDRESS>.nip.io/<PROFILE>/hello-spring.git 
cd hello-spring
~~~

Build and run code locally to validate (requires maven to build)

~~~
mvn spring-boot:run 
curl localhost:8080/api
{"greeting":"Hello World!"}
~~~

ctrl-c to kill app

Any changes you commit will go to the repo on gogs

Create a new verion of hello-spring project

`oc new-project hello-spring-pipeline`

Deploy the app from the gogs repo:

~~~
oc new-app http://gogs-gogs.<IP-ADDRESS>.nip.io/<PROFILE>/hello-spring.git --name hello --image-stream=redhat-openjdk18-openshift:1.2
~~~

`oc expose svc/hello`

`oc get route hello`

~~~
NAME      HOST/PORT                                    PATH      SERVICES   PORT       TERMINATION   WILDCARD
hello     hello-hello-pipeline.192.168.99.100.nip.io             hello      8080-tcp                 None
~~~

Verify it is running correctly and that the route is accessible:

`curl hello-hello-spring-pipeline.192.168.99.100.nip.io/api`

`{"greeting":"Hello World!"}`

### Create an OpenShift Pipeline

OpenShift Pipelines enable creating deployment pipelines using the Jenkinsfile format.

OpenShift automates deployments using deployment triggers that react to changes to the container image or configuration. Since you want to control the deployments instead from the pipeline, you should remove the "hello" app deploy triggers so that building a new "hello" container image won’t automatically result in a new deployment. This will allow the pipeline to decide when a deployment should occur.

Remove the hello deployment triggers:

`oc  set triggers dc/hello --manual`

`deploymentconfig "hello" updated`

Deploy a Jenkins server using the provided template and container image that comes out-of-the-box with OpenShift:

~~~
oc new-app openshift/jenkins-2-centos7
oc logs -f dc/jenkins-2-centos7

--> Scaling jenkins-2-centos7-1 to 1
--> Success
~~~

*Note: it may take a few minutes before the Jenkens server is accessible.*

*Note: If this doesn't work, try to deploy via the OpenShift Admin GUI*

Ensure the Jenkens container/server is up and running before you proceed.

Take a look at the Jenkinsfile included with this cloned project

`cat Jenkinsfile`

~~~
pipeline {
  agent {
      label 'maven'
  }
  stages {
    stage('Build JAR') {
      steps {
        sh "mvn package"
        stash name:"jar", includes:"target/hello-0.0.1-SNAPSHOT.jar"
      }
    }
    stage('Build Image') {
      steps {
        unstash name:"jar"
        script {
          openshift.withCluster() {
            openshift.startBuild("hello", "--from-file=target/hello-0.0.1-SNAPSHOT.jar", "--wait")
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        script {
          openshift.withCluster() {
			def dc = openshift.selector("dc", "hello")
			dc.rollout().latest()
            dc.rollout().status()
          }
        }
      }
    }
  }
}
~~~

This pipeline has three stages:

- Build JAR: to build and test the jar file using Maven
- Build Image: to build a container image ("hello") from the JAR archive using OpenShift S2I
- Deploy: to deploy the "hello" container image in the current project

The next steps will deploy the Jenkinsfile to the CI/CD configuration for this project:

After Jenkins is deployed and is running (verify in web console), create a deployment pipeline by running the following command within the hello-spring folder:

oc new-app . --name=hello-pipeline --strategy=pipeline

~~~
    * A pipeline build using source code from http://gogs-gogs.192.168.99.100.nip.io/leon/hello-spring.git#master will be created
      * Use 'start-build' to trigger a new build

--> Creating resources ...
    buildconfig "hello-pipeline" created
--> Success
    Build scheduled, use 'oc logs -f bc/hello-pipeline' to track its progress.
    Run 'oc status' to view your app.
~~~
