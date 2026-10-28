ARG IMAGE=containers.intersystems.com/intersystems/iris-community:latest-em

#Crear Java Builder para compilar código Java y FirstDemo.jar
FROM eclipse-temurin:8-jdk AS builder

ARG APP_HOME=/tmp/app

# Copiar código Java
COPY java/src $APP_HOME/src

# Copiar dependencias PEX
COPY java/dist/*.jar $APP_HOME/jgw/

# Crear directorio de salida
WORKDIR $APP_HOME/jar/

# Establecer directorio de compilación
WORKDIR $APP_HOME/src/

# Compilar código Java y generar FirstDemo.jar
RUN javac -classpath $APP_HOME/jar/*:$APP_HOME/jgw/* workshop/pex/*.java && \
    jar -cvf $APP_HOME/jar/FirstDemo.jar workshop/pex/*.class

#Contrucción de la imagen de IRIS
FROM $IMAGE

USER ${ISC_PACKAGE_MGRUSER}

COPY --chown=$ISC_PACKAGE_MGRUSER:$ISC_PACKAGE_IRISGROUP iris/irissession.sh /
RUN chmod +x /irissession.sh

# copy iris source code
COPY --chown=$ISC_PACKAGE_MGRUSER:$ISC_PACKAGE_IRISGROUP iris/src /tmp

#Copy the FirstDemo JAR generated into the IRIS image
COPY --from=builder \
    --chown=${ISC_PACKAGE_MGRUSER}:${ISC_PACKAGE_IRISGROUP} \
    /tmp/app/jar/FirstDemo.jar /opt/pex/FirstDemo.jar

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
