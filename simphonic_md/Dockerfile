FROM otomato/ubuntu32:14.04

RUN locale-gen en_US.UTF-8  

ENV SWARM_VERSION=2.2 \
  JAVA_HOME=/usr/lib/jvm/java-7-openjdk-i386/ \
  ANT_VERSION=1.9.4 \
  ANT_HOME=/usr/share/ant \
  PHANTOMJS_VERSION=2.1.1 \
  PHANTOMJS_PLATFORM=linux-x86_64 \
  PATH=~/.rbenv/bin:$PATH \
  DEBIAN_FRONTEND=noninteractive \
  LANGUAGE=en_US.en \
  LANG=en_US.UTF-8 \
  LC_ALL=en_US.UTF-8 \
  DOCKER_PATH=/opt/docker/bin
  
ENV PATH="$PATH:$DOCKER_PATH"

RUN echo 'APT::Install-Recommends "0"; \n\
          APT::Get::Assume-Yes "true"; \n\
          APT::Install-Suggests "0";' > /etc/apt/apt.conf \
  && apt-get update \
  && apt-get install vim-nox curl telnet software-properties-common dnsutils net-tools less netcat \
  && add-apt-repository ppa:webupd8team/java -y \
  && apt-get update \
  && curl -o /opt/swarm-client-${SWARM_VERSION}-jar-with-dependencies.jar http://repo.jenkins-ci.org/releases/org/jenkins-ci/plugins/swarm-client/${SWARM_VERSION}/swarm-client-${SWARM_VERSION}-jar-with-dependencies.jar 

COPY swarm_slave.sh /usr/bin/swarm_slave.sh
RUN chmod +x /usr/bin/swarm_slave.sh && mkdir -p /etc/supervisor/conf.d
COPY supervisor.conf /etc/supervisor/conf.d/supervisor.conf

  # cmake, gcc5
RUN add-apt-repository ppa:ubuntu-toolchain-r/test \
  && add-apt-repository ppa:george-edison55/cmake-3.x \
  && apt-get update \

  # Make sure the package repository is up to date.
  && echo oracle-java7-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections \
  && apt-get install -y git openssh-client zip unzip wget bzip2 file make dos2unix supervisor \
  
  # adding docker-in-docker
  && mkdir -p $DOCKER_PATH \
  && apt-get install -y iptables kmod libnfnetlink0 module-init-tools \
  && echo "#!/bin/bash\n/bin/true" > $DOCKER_PATH/docker-in-docker.sh \
  && chmod +x $DOCKER_PATH/docker-in-docker.sh \

  # Add SIM deps
  && apt-get install -y libomnithread3-dev libomniorb4-dev libcurl4-openssl-dev python-dev binutils omniidl omniorb libaio-dev flex \
  && apt-get install -y gcc-4.8 g++-4.8 gcc-5 g++-5 cmake rsync curl openjdk-7-jre openjdk-7-jdk bison libbison-dev \
  && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 60 --slave /usr/bin/g++ g++ /usr/bin/g++-5 \

  # clean all cache to clean space
  && rm -rf /var/lib/apt/lists/* \
  && apt-get clean \
  && apt-get -y autoremove
  
ENTRYPOINT [ "/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf" ]
