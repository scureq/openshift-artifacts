# AM Dockerfile
#
# Copyright (c) 2016-2017 ForgeRock AS.
#
FROM tomcat:8.5.24-jre8-alpine

# When running this, make sure you pass
#
#     JAVA_OPTS="-Xms2g -Xmx2g -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=256m -Dorg.apache.tomcat.util.buf.UDecoder.ALLOW_ENCODED_SLASH=true"
#
# either from CLI (docker run) or helm chart or OpenShift Browser UI. Eg
#
# docker run --name am-eval -e JAVA_OPTS="-Xms2g -Xmx2g -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=256m \
#    -Dorg.apache.tomcat.util.buf.UDecoder.ALLOW_ENCODED_SLASH=true" -p 8080:8080 \
#    -v $PWD/openam-configuration:/usr/local/tomcat/openam-configuration am-eval


ENV FORGEROCK_HOME /home/forgerock
# The user 'default' is created by OpenShift dynamically with /sbin/nologin as shell.
# We set user's home dir to /usr/local/tomcat.
# The OPENAM_CONFIG directory is the mount point for the OpenShift persistent volume.
ENV DEFAULT_USER_HOME_DIR="/usr/local/tomcat"
ENV OPENAM_CONFIG_DIR="$DEFAULT_USER_HOME_DIR"/openam-configuration


#ENV CATALINA_OPTS -server -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap
# Option for setting the AM home directory:
#    -Dcom.sun.identity.configuration.directory=/home/forgerock/openam
# Options for using cgroups for memory size:
#   -server -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap
# Option for sending debug output to stderr
#-Dcom.sun.identity.util.debug.provider=com.sun.identity.shared.debug.impl.StdOutDebugProvider -Dcom.sun.identity.shared.debug.file.format="%PREFIX% %MSG%\n%STACKTRACE%"

ENV CATALINA_OPTS -server -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap \
  -Dorg.apache.tomcat.util.buf.UDecoder.ALLOW_ENCODED_SLASH=true \
  -Dcom.sun.identity.util.debug.provider=com.sun.identity.shared.debug.impl.StdOutDebugProvider \
  -Dcom.sun.identity.shared.debug.file.format=\"%PREFIX% %MSG%\\n%STACKTRACE%\" \
  -Duser.home=$DEFAULT_USER_HOME_DIR \
  -Dcom.sun.identity.configuration.directory=$OPENAM_CONFIG_DIR/openam


# Make sure that log files can be accessed
ENV UMASK="0002"

#  -Dcom.iplanet.services.debug.level=error


COPY openam.war  /tmp/openam.war

RUN apk add --no-cache su-exec unzip curl bash  \
  && rm -fr /usr/local/tomcat/webapps/* \
  && unzip -q /tmp/openam.war -d  "$CATALINA_HOME"/webapps/openam \
  #  Let's use bootstrap.properties rather than default location
  && echo "configuration.dir="$OPENAM_CONFIG_DIR"/openam" >> "$CATALINA_HOME"/webapps/openam/WEB-INF/classes/bootstrap.properties \
  && rm /tmp/openam.war \
  # Add 'forgerock' to primary group 'root'. OpenShift's dynamic user also has 'root' as primary group.
  # By this the dynamic user has almost the same privs a 'forgerock'
  && adduser -s /bin/bash -h "$FORGEROCK_HOME" -u 11111 -G root -D forgerock \
  && mkdir -p "$OPENAM_CONFIG_DIR" \
  # && mkdir -p "$FORGEROCK_HOME"/.openamcfg \
  # && echo "$OPENAM_HOME" >  "$FORGEROCK_HOME"/.openamcfg/AMConfig_usr_local_tomcat_webapps_openam_ \
  && chown -R forgerock:root "$CATALINA_HOME" \
  && chown -R forgerock:root  "$FORGEROCK_HOME" \
  && chown -R forgerock:root  "$OPENAM_CONFIG_DIR" \
  && chown -R forgerock:root  /usr/local \
  && chmod -R g=u "$CATALINA_HOME"


# If you want to create an image that is ready to be bootstrapped to a
# configuration store, you can add a custom boot.json file.
# This can also be added at runtime by a ConfigMap or an init container.
#COPY boot.json /root/openam

# Generate a default keystore for SSL - only needed if you want SSL inside the cluster.
# You can mount your own keystore on the ssl/ directory to override this.
# Because of the complexity of configuring ssl, we should look at using istio.io to handle intercomponent ssl
#RUN mkdir -p /usr/local/tomcat/ssl && \
#   keytool -genkey -noprompt \
#     -keyalg RSA \
#     -alias tomcat \
#     -dname "CN=forgerock.com, OU=ID, O=FORGEROCK, L=Calgary, S=AB, C=CA" \
#     -keystore /usr/local/tomcat/ssl/keystore \
#     -storepass password \
#     -keypass password

# Custom server.xml: use this if AM is behind SSL termination.
# See the server.xml file for details.
COPY server.xml "$CATALINA_HOME"/conf/server.xml

# For debugging AM in a container, uncomment this.
# Use something like  kubectl port-forward POD 5005:5005
# ENV CATALINA_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005"

# Settings for Tomcat cache.
COPY context.xml "$CATALINA_HOME"/conf/context.xml

# Path to optional script to customize the AM web app. Use this script hook to copy in images, web.xml, etc.
# ENV CUSTOMIZE_AM /home/forgerock/customize-am.sh


# USER forgerock
# Best practice is to use uid here
USER 11111

COPY *.sh $FORGEROCK_HOME/

ENTRYPOINT ["/home/forgerock/docker-entrypoint.sh"]

CMD ["run"]
