ARG IMAGE=containers.intersystems.com/intersystems/iris-community:latest-em
FROM $IMAGE

USER ${ISC_PACKAGE_MGRUSER}

COPY --chown=$ISC_PACKAGE_MGRUSER:$ISC_PACKAGE_IRISGROUP irissession.sh /
RUN chmod +x /irissession.sh

# copy iris source code
COPY --chown=$ISC_PACKAGE_MGRUSER:$ISC_PACKAGE_IRISGROUP src /tmp

SHELL ["/irissession.sh"]

RUN \
  zn "%SYS" \
  set gateway("Type") = "Remote" \
  set gateway("Address") = "jgw" \
  set gateway("Port") = 44444 \
  set gateway("UseSharedMemory") = 0 \
  set sc = ##class(Config.Gateways).Create("WorkshopJava", .gateway) \
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
