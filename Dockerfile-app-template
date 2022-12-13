FROM adoptopenjdk/openjdk11:jre-11.0.6_10-alpine
WORKDIR /opt/tomcat
RUN wget -qO- https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.70/bin/apache-tomcat-9.0.70.tar.gz | tar xvz --strip-components 1
CMD /opt/tomcat/bin/catalina.sh run