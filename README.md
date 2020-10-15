# How to deploy an Quarkus Application on OpenShift

### Preparations (Fedora 32 Workstation)
To develop and run a Quarkus application on OpenShift we need additional software.

You need:
1.  An IDE like IntelliJ IDEA, Eclipse, VSCode ([Click for Instructions](https://www.jetbrains.com/help/idea/installation-guide.html#silent))
2.  A JDK 8 or 11+ (any distribution)  ([Click for Instructions](https://www.tecmint.com/install-java-in-fedora/))
3. (Optionally) Get GraalVM 20.2.0 for native compilation  ([Click for Instructions](https://quarkus.io/guides/building-native-image))
4. Apache Maven 3.6.2+ or Gradle (in this webinar Maven is used) ([Click for Instructions](https://tecadmin.net/install-apache-maven-on-fedora/))
5. Docker or Podman installed (in this webinar Docker is used) ([Click for Instructions](https://docs.docker.com/engine/install/fedora/))
6. A running OpenShift-Cluster ([Click to Learn More](https://www.openshift.com/try))

## 1. Deploy Quarkus Using OpenShift-Extension
To add the OpenShift extension to an existing project, enter the following command:
```bash
./mvnw quarkus:add-extension -Dextensions="openshift"
````
Editing the "src/main/resources/application.properties"
```bash
quarkus.kubernetes-client.trust-certs=true
quarkus.s2i.base-jvm-image=registry.access.redhat.com/openjdk/openjdk-11-rhel7
quarkus.openshift.expose=true
````
Login to your OpenShift-Cluster
```bash
./mvnw quarkus:add-extension -Dextensions="openshift"
````
Deploy your Quarkus Application
```bash
./mvnw clean package -Dquarkus.kubernetes.deploy=true
````

Compare with the following instructions:
* [Red Hat: Deploy Quarkus using Openshift-Extension](https://access.redhat.com/documentation/en-us/red_hat_build_of_quarkus/1.3/html-single/deploying_quarkus_applications_on_red_hat_openshift_container_platform/index#proc-using-openshfit-extension_quarkus-openshift)

## 2. Deploy Quarkus Using Source-To-Image (S2I)
##### Prepare your Quarkus Project
Create a hidden directory called .s2i at the same level as the pom.xml file. Then create a file called "environment" in the .s2i directory and add the following content:
```bash
ARTIFACT_COPY_ARGS=-p -r lib/ *-runner.jar
```
Login into your OpenShift-Cluster:
```bash
~:$ oc login -u $USER -p $PASSWORD §DOMAN_API_CLUSTER
```
Create a new project:
```bash
~:$ oc new-project $PROJECT_NAME
```
##### Deployment (JVM)
Import builder image:
```bash
~:$ oc import-image --confirm openjdk/openjdk-11-rhel7 --from=registry.access.redhat.com/openjdk/openjdk-11-rhel7
```
Create a new app:
```bash
~:$ oc new-app openjdk-11-rhel7 GIT_PATH --name=PROJECT_NAME
```

##### Deployment (Native)
Import native builder image:
```bash
~:$ oc import-image --confirm quarkus/ubi-quarkus-native --from=quay.io/quarkus/ubi-quarkus-native-s2i:20.1.0-java11
```
Create a new app:
```bash
~:$ oc new-app ubi-quarkus-native https://github.com/schmti/quarkus-webinar --name=qdemo-s2i-native
```
The native build of your application needs a lot of ressources! Scale them up in your Build-Config!
(4 cores, 6GB Memory only for small applications)
```bash
~:$ oc patch bc/qdemo-s2i-native -p '{"spec":{"resources":{"limits":{"cpu":"4", "memory":"6Gi"}}}}'
```
Compare with the following instructions:
* [Red Hat: Deploy Quarkus using S2I](https://access.redhat.com/documentation/en-us/red_hat_build_of_quarkus/1.3/html-single/deploying_quarkus_applications_on_red_hat_openshift_container_platform/index#proc-using-openshfit-extension_quarkus-openshift)
* [Quarkus: Deploying to OpenShift S2I ](https://quarkus.pro/guides/deploying-to-openshift-s2i.html)

## 3. Deploy Quarkus Using Dockerfile
Prebuild your Application an push them into your git repository.
After that, deploy your application using the Developer-View in OpenShift
##### non-native (.jar)
Use:
```bash
~:$ ./mvnw clean package
```
##### native
Use:
```bash
~:$ ./mvnw clean package -Pnative
```
## 4. Deploy Quarkus Using Code-Ready-Container
Prebuild your Application like it was described in option 3. After that login to docker registry.
```bash
~:$ docker login
```
##### Build Container & Push ro Registry with non-native (.jar) Application
Use:
```bash
~:$ ./mvnw package -Dquarkus.container-image.build=true -Dquarkus.container-image.push=true
```
##### Build Container & Push ro Registry with native Application
Use:
```bash
~:$ ./mvnw package -Pnative -Dquarkus.native.container-build=true -Dquarkus.container-image.build=true -Dquarkus.container-image.push=true
```



# webinar-demo project (Quarkus basics)

This project uses Quarkus, the Supersonic Subatomic Java Framework.

If you want to learn more about Quarkus, please visit its website: https://quarkus.io/ .

## Running the application in dev mode

You can run your application in dev mode that enables live coding using:
```shell script
./mvnw compile quarkus:dev
```

## Packaging and running the application

The application can be packaged using:
```shell script
./mvnw package
```
It produces the `webinar-demo-1.0.0-SNAPSHOT-runner.jar` file in the `/target` directory.
Be aware that it’s not an _über-jar_ as the dependencies are copied into the `target/lib` directory.
If you want to build an _über-jar_, execute the following command:
```shell script
./mvnw package build -Dquarkus.package.type=uber-jar
```

The application is now runnable using `java -jar target/webinar-demo-1.0.0-SNAPSHOT-runner.jar`.

## Creating a native executable

You can create a native executable using: 
```shell script
./mvnw package -Pnative
```

Or, if you don't have GraalVM installed, you can run the native executable build in a container using: 
```shell script
./mvnw package -Pnative -Dquarkus.native.container-build=true
```

You can then execute your native executable with: `./target/webinar-demo-1.0.0-SNAPSHOT-runner`

If you want to learn more about building native executables, please consult https://quarkus.io/guides/maven-tooling.html.

# RESTEasy JAX-RS

Guide: https://quarkus.io/guides/rest-json


