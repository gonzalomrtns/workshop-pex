#Build IRIS image
#Use a newer IRIS Community image instead of the previously pinned version
#Note that latest-em is a mutable tag and may require compatibility checks when updated.
ARG IMAGE=containers.intersystems.com/intersystems/iris-community:latest-em

FROM ${IMAGE}

#Create  "/opt/pex" directory and assign ownership to irisowner so it can write and compile Java classes,
#fixing a permission denied issues.
USER root
RUN mkdir -p /opt/pex && chown ${ISC_PACKAGE_MGRUSER}:${ISC_PACKAGE_IRISGROUP} /opt/pex

USER ${ISC_PACKAGE_MGRUSER}

#Java PEX Compilation
#Copy java source code

COPY --chown=${ISC_PACKAGE_MGRUSER}:${ISC_PACKAGE_IRISGROUP} \
    java/src /tmp/java/src

#Compile Java classes using IRIS Java libraries, create FirstDemo.jar and delete temporary files.

RUN mkdir -p /opt/pex/build &&\
    javac --release 11 \
    -cp "/usr/irissys/dev/java/lib/1.8/*" \
    -d /opt/pex/build \
    /tmp/java/src/workshop/pex/*.java && \
    jar -cvf /opt/pex/FirstDemo.jar -C /opt/pex/build . && \
    rm -rf /opt/pex/build /tmp/java

# IRIS Configuration

COPY --chown=$ISC_PACKAGE_MGRUSER:$ISC_PACKAGE_IRISGROUP iris/irissession.sh /
RUN chmod +x /irissession.sh

# copy iris source code
COPY --chown=$ISC_PACKAGE_MGRUSER:$ISC_PACKAGE_IRISGROUP iris/src /tmp

SHELL ["/irissession.sh"]

RUN \
  zn "%SYS" \
  #  Create a Java External Language Server named PEXJavaServer using Config.Gateways, which starts and manages the Java Gateway process
  set gateway("Type") = "Java" \
  #Use port 44445 for the Java Gateway process.
  set gateway("Port") = 44445 \
  #Disable shared memory communication and use TCP sockets
  #between IRIS and the Java Gateway.
  set gateway("UseSharedMemory") = 0 \
  # Leave JavaHome empty to use the JDK shipped in the IRIS image
  set gateway("JavaHome") = ""\
  # Configure the classpath to include the application JAR and the InterSystems Java libraries.
  # This allows the Java Gateway to load the PEX classes provided by InterSystems, including BusinessService, BusinessProcess, and BusinessOperation.
  set gateway("ClassPath") = "/opt/pex/FirstDemo.jar:/usr/irissys/dev/java/lib/1.8/*"\
  set sc = ##class(Config.Gateways).Create("PEXJavaServer", .gateway) \
  if 'sc { do $SYSTEM.Status.DisplayError(sc) do ##class(%SYSTEM.Process).Terminate(, 1) } \
  zn "USER" \
  # load & compile source code
  do $SYSTEM.OBJ.LoadDir("/tmp", "ck", , 1) \
  do ##class(%File).RemoveDirectoryTree("/tmp") \
  # start production
  set production = "Workshop.PEX.FirstDemo.Production" \
  set ^Ens.Configuration("csp","LastProduction") = production \
  do ##class(Ens.Director).SetAutoStart(production) \
  set sc = 1

# bringing the standard shell back
SHELL ["/bin/bash", "-c"]
