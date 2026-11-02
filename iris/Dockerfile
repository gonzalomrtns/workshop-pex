ARG IMAGE=containers.intersystems.com/intersystems/iris-community:latest-em

# STAGE 1: JAVA BUILDER

FROM eclipse-temurin:8-jdk AS builder

ARG APP_HOME=/tmp/app

# Copiar el código fuente Java
COPY java/src $APP_HOME/src

# copy iris pex libraries to $APP_HOME/jgw
#COPY --from=intersystemsdc/jgw:latest /jgw/*.jar $APP_HOME/jgw/, no se ejecuta porque dicha imagen contiene una versión antigua de JDBC (3.6.1)
COPY java/dist/*.jar $APP_HOME/jgw/

# create $APP_HOME/jar
WORKDIR $APP_HOME/jar/

# compile java source code and create a jar file in $APP_HOME/jar
WORKDIR $APP_HOME/src/
RUN javac -classpath $APP_HOME/jar/*:$APP_HOME/jgw/* workshop/pex/*.java && \
    jar -cvf $APP_HOME/jar/FirstDemo.jar workshop/pex/*.class

# STAGE 2: Build IRIS image
FROM $IMAGE

USER ${ISC_PACKAGE_MGRUSER}

COPY --chown=$ISC_PACKAGE_MGRUSER:$ISC_PACKAGE_IRISGROUP iris/irissession.sh /
RUN chmod +x /irissession.sh

# copy iris source code
COPY --chown=$ISC_PACKAGE_MGRUSER:$ISC_PACKAGE_IRISGROUP iris/src /tmp

# Copy FirstDemo.jar into IRIS image
COPY --from=builder \
    --chown=${ISC_PACKAGE_MGRUSER}:${ISC_PACKAGE_IRISGROUP} \
    /tmp/app/jar/FirstDemo.jar /opt/pex/FirstDemo.jar


#IRIS and PEX Configuration 

SHELL ["/irissession.sh"]

RUN \
  zn "%SYS" \
  set gateway("Type") = "Java" \
  set gateway("Port") = 44445 \
  set gateway("UseSharedMemory") = 0 \
  set gateway("JavaHome") = "/usr/lib/jvm/temurin-11-jdk-amd64"\
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
