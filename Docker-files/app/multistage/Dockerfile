FROM tomcat:9-jdk17-temurin-jammy

RUN rm -rf /usr/local/tomcat/webapps/*

# Now that the context is set to '.', this path will work perfectly
COPY target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]