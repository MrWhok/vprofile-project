# Use a Tomcat version compatible with your JDK 17 Jenkins environment
FROM tomcat:9-jdk17-temurin-jammy

# Clean up default Tomcat applications to avoid conflicts
RUN rm -rf /usr/local/tomcat/webapps/*

# CRITICAL FIX: Copy the .war file that Jenkins just built.
# Your Jenkinsfile 'Build' stage creates this in the 'target' folder.
# This ensures your 'TESTTTTTT' changes are included in the image.
COPY target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]