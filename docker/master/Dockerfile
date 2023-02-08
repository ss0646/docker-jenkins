FROM jenkins/jenkins:lts-jdk11

COPY ./config/plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN set -eux && \
    jenkins-plugin-cli -f /usr/share/jenkins/ref/plugins.txt
    
EXPOSE 8080