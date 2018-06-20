# hello-spring

This is a very basic "Hello World!" web app using the
Spring Boot frameork:
[http://spring.io](http://spring.io).

**hello-spring** inteded for use with OpenShift workshops, but can be used however you want. 
This project also demonstrates JUnit testing.

This README file will provide instructions for building 
and running locally, as well as instructions for 
deploying to OpenShift using S2I (source to image) methods

*On a side note, you can easily spin up a Spring Boot project yourself 
from scratch by using the ["SPRING INITIALIZR"](http://start.spring.io). 
This will generate the maven pom.xmlfile and initiial 
directory structure that can compile and run right away, 
including setup for JUnit testing, which is demonstrated 
here in `hello-spring`.*

This project is set up to use maven to build the jar 
executable, so ensure that is installed on your build 
system ([maven](https://maven.apache.org/install.html)). 

**Note**:  `yum`, `apt-get`, `brew install`, 
`chocolatey`, etc.. can all be used to install maven.

## Building and running locally
Once you clone the project to your local system, you 
can build and test locally by running:

`mvn clean test` - for unit testing

`mvn clean package` - to compile a *jar executable 
which you can run locally

After successful compilation, you will find the executable jar in the `target/` directory.

You can runn `hello` with the following command from the project root directory:

`java -jar target/*.jar`

Once running, point your web browser to `localhost:8080` or run the command:

`curl localhost:8080` 

on the command line to see the welcome message.

## Unit testing
The test class `HelloControllerTest` contains one unit test to ensure the application returns the string "Hello World!"

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


To see the unit test fail, edit the string in the `index(...)` function in the `HelloController` class.

Example:

~~~
@RestController
public class HelloController {

    @RequestMapping("/")
    public String index() { return "Hello Foo!"; }

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

create a new OpenShift project to run your hello-spring

`oc new-project hello-spring`

If your OpenShift environment doesn't already have a java image (as in some versions of minishift lack, for some strange reason) you can create one in the OpenShift project by downloading the `openjdk-s2i-imagestream.json` file from here and running"

##TODO - NEED TO VALIDATE EVERYTHING BELOW THIS LINE!!!

`oc create -f openjdk-s2i-imagestream.json`

The Java S2I image enables developers to automatically build, deploy and run java applications on demand, in OpenShift Container Platform, by simply specifying the location of their application source code or compiled java binaries. In many cases, these java applications are bootable “fat jars” that include an embedded version of an application server and other frameworks (wildfly-swarm in this instance). 

Before we start using the Java S2I image we need to tell OpenShift how to find it. This is done by creating an image stream. The image stream definition can be downloaded and used. To add the image stream to your project run the following command:

`$ oc create -f openjdk-s2i-imagestream.json`

Now you can deploy the service from github

`$ oc new-app https://github.com/bugbiteme/catalog-spring-boot.git --name catalog --image-stream=redhat-openjdk18-openshift`

A build gets created and starts building the Node.js Web UI container image. You can see the build logs using OpenShift Web Console or OpenShift CLI:

`$ oc logs -f bc/catalog`

In order to access the Web UI from outside (e.g. from a browser), it needs to get added to the load balancer. Run the following command to add the Web UI service to the built-in HAProxy load balancer in OpenShift.

~~~
$ oc expose svc/catalog
$ oc get route catalog
~~~

While you are waiting for the deploy command to complete, you can log into the OpenShift web consol and check the progress of your deployment, and even view the build and deployment logs, which should look very similar to the messages seen when running the service locally.


### Validate 
Once the service has been deployed, you can get the url by running

`$ oc get route`

validate it is running using curl (or a web browser)

`curl http://CATALOGSERVICEURL/api/catalog 
 output

