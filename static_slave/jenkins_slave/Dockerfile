FROM openjdk:8-jdk

MAINTAINER lakshmanavinod9@gmail.com  

ENV JENKINS_SWARM_VERSION 3.14
ENV JENKINS_SWARM_DOWNLOAD_SITE https://repo.jenkins-ci.org/releases/org/jenkins-ci/plugins/swarm-client
ENV JENKINS_HOME /home/jenkins
ENV JENKINS_USER jenkins
ENV MAVEN_VERSION 3.5.4
ENV MAVEN_HOME /usr/share/maven
ENV DOCKER_VERSION 17.09.1-ce
ENV HELM_VERSION v2.9.1
ENV HELM_FILENAME helm-${HELM_VERSION}-linux-amd64.tar.gz

RUN useradd -m -d "${JENKINS_HOME}" -u 1000 -U -s /sbin/nologin "${JENKINS_USER}"
############# Adding the below line to add jenkins user to root group #######
#RUN usermod -g 0 jenkins
################
RUN curl --create-dirs -sSLo /usr/share/jenkins/swarm-client-${JENKINS_SWARM_VERSION}.jar \
  ${JENKINS_SWARM_DOWNLOAD_SITE}/${JENKINS_SWARM_VERSION}/swarm-client-${JENKINS_SWARM_VERSION}.jar \
  && chmod 755 /usr/share/jenkins


ENV MAVEN_CONFIG ${JENKINS_HOME}/.m2
COPY jenkins-slave.sh /usr/local/bin/jenkins-slave.sh

RUN mkdir /docker-entrypoint-init.d
ONBUILD ADD ./*.sh /docker-entrypoint-init.d/ 
RUN curl -fsSL http://apache.osuosl.org/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz \
   | tar -xzC /usr/share \
  && mv "/usr/share/apache-maven-${MAVEN_VERSION}" "${MAVEN_HOME}" \
  && ln -s "${MAVEN_HOME}/bin/mvn" /usr/bin/mvn


#### install docker client
RUN set -x \
        && curl -fSL "https://download.docker.com/linux/static/stable/x86_64/docker-${DOCKER_VERSION}.tgz" -o docker.tgz \
        && tar -xzvf docker.tgz \
        && mv docker/* /usr/local/bin/ \
        && rmdir docker \
        && rm docker.tgz \
        && docker -v
# install helm
RUN apt-get update && apt-get install -y --no-install-recommends \
    # Install Helm
    && curl -L http://storage.googleapis.com/kubernetes-helm/helm-${HELM_VERSION}-linux-amd64.tar.gz -o /tmp/helm.tar.gz \
    && tar -zxvf /tmp/helm.tar.gz -C /tmp \
    && cp /tmp/linux-amd64/helm /usr/local/bin/helm \
    # Cleanup uncessary files
    && apt-get clean \
    && rm -rf /tmp/* ~/*.tgz
    
##### Net-tools###########
RUN apt-get update \
  && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    net-tools \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

#####Node####
RUN apt install curl
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash -
RUN  apt-get install -y nodejs
RUN node -v
RUN npm -v

#####Install Grunt CLI####
RUN npm install -g grunt-cli

### Install Python dependent modules for AWX ###
RUN apt-get update && apt-get install -y \
    python-pip

### Install Python dependent modules for AWX ###
RUN pip install requests

############ANt#############
ENV ANT_VERSION 1.10.6

RUN cd && \
    wget -q http://www.us.apache.org/dist//ant/binaries/apache-ant-${ANT_VERSION}-bin.tar.gz && \
    tar -xzf apache-ant-${ANT_VERSION}-bin.tar.gz && \
    mv apache-ant-${ANT_VERSION} /opt/ant && \
    rm apache-ant-${ANT_VERSION}-bin.tar.gz
ENV ANT_HOME /opt/ant
ENV PATH ${PATH}:/opt/ant/bin

#################################

#######################
### Installing Make###########
RUN apt-get install -y build-essential
RUN apt-get install zip xvfb libgconf2-4 -y
RUN apt-get install unzip
RUN echo 'deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main' | tee /etc/apt/sources.list.d/google-chrome.list \
  && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
  && apt update \
  && apt install -y google-chrome-stable \
  && rm -rf /var/lib/apt/lists/*
ENV PATH /usr/bin/google-chrome:$PATH


# Set up Chromedriver Environment variables
ENV CHROMEDRIVER_VERSION 76.0.3809.12
ENV CHROMEDRIVER_DIR /chromedriver
RUN mkdir $CHROMEDRIVER_DIR

RUN curl http://chromedriver.storage.googleapis.com/76.0.3809.12/chromedriver_linux64.zip -o /chromedriver/chromedriver_linux64.zip -J -L

RUN unzip $CHROMEDRIVER_DIR/chromedriver* -d $CHROMEDRIVER_DIR

RUN chmod 777 -R /chromedriver

# Put Chromedriver into the PATH
ENV PATH $CHROMEDRIVER_DIR:$PATH

USER "${JENKINS_USER}"
#USER root
ENTRYPOINT ["/usr/local/bin/jenkins-slave.sh"]
