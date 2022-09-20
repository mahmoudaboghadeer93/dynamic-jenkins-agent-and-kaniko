FROM --platform=linux/amd64 ubuntu:20.04
# set environment variables
ENV ANDROID_HOME="/opt/cmdline-tools/latest/bin/"
#ENV PATH="/home/linuxbrew/.linuxbrew/bin:${PATH}"
WORKDIR /root/
# to run dpkg (behind other tools like Apt) without interactive dialogue
ARG DEBIAN_FRONTEND=noninteractive

# tools and packages versions
ARG WM_VERSION=WM_VERSION_V
ARG NPM_VERSION=NPM_VERSION_V
ARG GRADLE_VERSION=GRADLE_VERSION_V
ARG ANDROID_SDK_VERSION=ANDROID_SDK_VERSION_V
ARG SDK_BUILD_TOOLS_VERSION_1=SDK_BUILD_TOOLS_VERSION_1_V
ARG SDK_BUILD_TOOLS_VERSION_2=SDK_BUILD_TOOLS_VERSION_2_V
ARG SKD_PLATFORMS_1=SKD_PLATFORMS_1_V
ARG SKD_PLATFORMS_2=SKD_PLATFORMS_2_V
ARG NODE_VERSION=NODE_VERSION_V
ARG JAVA_VERSION=JAVA_VERSION_V
#ARG KOTLIN_VERSION=KOTLIN_VERSION_V
ARG SONAR_SCANNER_VERSION=SONAR_SCANNER_VERSION_V
####install needed packages
RUN apt-get install -y unzip wget vim hub python3 python3-venv python3-pip npm ghp-import ruby-full

# install n & node
RUN npm install -g n
RUN n install ${NODE_VERSION}
# install mkdocs-material
RUN pip install mkdocs
RUN pip install mkdocs-material

# install appcenter 2.10.6
RUN npm install -g appcenter-cli
# install github_changelog_generator
RUN gem install github_changelog_generator
# install watchman
RUN wget https://github.com/facebook/watchman/releases/download/$WM_VERSION/watchman-$WM_VERSION-linux.zip && \
unzip watchman-$WM_VERSION-linux.zip && \
cd watchman-$WM_VERSION-linux && \
mkdir -p /usr/local/{bin,lib} /usr/local/var/run/watchman && \
cp bin/* /usr/local/bin && \
cp lib/* /usr/local/lib && \
chmod 755 /usr/local/bin/watchman && \
chmod 2777 /usr/local/var/run/watchman && \
cd .. && \
rm -fr watchman-$WM_VERSION-linux.zip watchman-$WM_VERSION-linux
# install JAVA
RUN apt install -y ${JAVA_VERSION}
# install sonar scanner
RUN apt-get -y install nodejs
RUN mkdir /opt/sonarscanner && cd /opt/sonarscanner && wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/$SONAR_SCANNER_VERSION.zip && unzip $SONAR_SCANNER_VERSION.zip && rm $SONAR_SCANNER_VERSION.zip && chmod +x sonar-scanner-3.2.0.1227-linux/bin/sonar-scanner && ln -s /opt/sonarscanner/sonar-scanner-3.2.0.1227-linux/bin/sonar-scanner /usr/local/bin/sonar-scanner
RUN cd /opt && wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.7.0.2747-linux.zip && unzip sonar-scanner-cli-4.7.0.2747-linux.zip && rm sonar-scanner-cli-4.7.0.2747-linux.zip && mv sonar-scanner-4.7.0.2747-linux sonar-scanner-4 && chmod 777 sonar-scanner-4/bin/sonar-scanner
# install gradle 7.3.1
RUN mkdir /opt/gradle && cd /opt/gradle && wget https://services.gradle.org/distributions/$GRADLE_VERSION-all.zip && unzip -d /opt/gradle $GRADLE_VERSION-all.zip && rm /opt/gradle/$GRADLE_VERSION-all.zip && ln -s /opt/gradle/${GRADLE_VERSION}/bin/gradle /usr/local/bin/gradle
# install android-sdk
WORKDIR /opt
RUN wget https://dl.google.com/android/repository/$ANDROID_SDK_VERSION.zip
RUN unzip ${ANDROID_SDK_VERSION}.zip && rm ${ANDROID_SDK_VERSION}.zip
RUN cd cmdline-tools && mkdir latest && mv NOTICE.txt  bin  lib  source.properties latest && cd latest/bin/ && \
    yes | ./sdkmanager --licenses --sdk_root=/opt/cmdline-tools/latest/bin  && \
    ./sdkmanager --list  && \
    ./sdkmanager "tools" "platform-tools" && \
    ./sdkmanager "build-tools;${SDK_BUILD_TOOLS_VERSION_1}" "build-tools;${SDK_BUILD_TOOLS_VERSION_2}" && \
    ./sdkmanager "platforms;${SKD_PLATFORMS_1}" "platforms;${SKD_PLATFORMS_2}"
RUN chmod 777 /opt/cmdline-tools/latest/bin
# install kotlin
#RUN brew extract --version=${KOTLIN_VERSION} kotlin homebrew/cask
#RUN brew install kotlin@${KOTLIN_VERSION}

####################
RUN apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN echo 'root:root' |chpasswd
RUN sed -ri 's/^PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config
#######################
ARG user=ado-agent
ARG group=ado-agent
ARG uid=1000
ARG gid=1000
ARG AGENT_HOME=/home/${user}

ENV AGENT_HOME ${AGENT_HOME}

RUN groupadd -g ${gid} ${group} \
    && useradd -d "${AGENT_HOME}" -u "${uid}" -g "${gid}" -m -s /bin/bash "${user}"
RUN echo "ado-agent:ado-agent" | chpasswd
#COPY ./npmrc /home/ado-agent/.npmrc
#COPY ./npmrcs /home/ado-agent/.npmrcs
#RUN chmod 777 /home/ado-agent/.npmrc /home/ado-agent/.npmrcs /home/ado-agent/.npmrcs/*

RUN mkdir -p /opt/sonar/sonar-scanner-3.1 && cp -R sonarscanner/sonar-scanner-3.2.0.1227-linux/* sonar/sonar-scanner-3.1/ && chmod 777 sonar/sonar-scanner-3.1/bin sonar/sonar-scanner-3.1/bin/* && rm -rf sonarscanner
RUN chmod 777 /home/jenkins/
#RUN cp -R /home/ado-agent/.npmrc /home/ado-agent/.npmrcs /root/ && npm install -g npmrc
RUN npm install -g npmrc
USER jenkins
#RUN mv /home/ado-agent/.npmrcs /home/ado-agent/.npmrcs_bek && npmrc && cp -R /home/ado-agent/.npmrcs_bek/* /home/ado-agent/.npmrcs && rm -rf /home/ado-agent/.npmrcs_bek
# configure git
RUN git config --global credential.helper store
COPY ./git-credentials /home/ado-agent/.git-credentials
USER root
#install penv and yarn
RUN npm i -g penv
RUN npm install -g yarn
RUN npm install -g npm@${NPM_VERSION}

###########################################################################################
RUN DEBIAN_FRONTEND=noninteractive apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y -qq --no-install-recommends \
    apt-transport-https \
    apt-utils \
    ca-certificates \
    curl \
    git \
    iputils-ping \
    jq \
    lsb-release \
    software-properties-common

RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# Can be 'linux-x64', 'linux-arm64', 'linux-arm', 'rhel.6-x64'.
ENV TARGETARCH=linux-x64

WORKDIR /azp

COPY ./start.sh .
RUN chmod +x start.sh

ENTRYPOINT [ "./start.sh" ]
