## to build own puppet server with Akurath CloudCode space IDE   
## with reference of codespace.io pupppet server 
## 


FROM ubuntu:latest
RUN rm /bin/sh && ln -s /bin/bash /bin/sh
#########################   Prerequired installation and configuration 
RUN apt-get update

RUN apt-get install -y openssh-server net-tools dnsutils vim sudo curl build-essential libssl-dev git python man wget
RUN mkdir /var/run/sshd
RUN systemctl enable ssh
RUN service ssh start

RUN echo 'root:ubuntu' |chpasswd
RUN useradd -m -G sys,sudo,systemd-journal -s /bin/bash devops
RUN echo "devops:devops"|chpasswd

RUN sed -ri 's/^#?PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config

# nvm environment variables
ENV NVM_DIR /usr/local/nvm
ENV NODE_VERSION v0.10.40
ENV NVM_VERSION v0.16.1

# install nvm
# https://github.com/creationix/nvm#install-script
RUN curl --silent -o- https://raw.githubusercontent.com/creationix/nvm/$NVM_VERSION/install.sh | bash

# install node and npm
RUN source $NVM_DIR/nvm.sh \
    && nvm install $NODE_VERSION \
    && nvm alias default $NODE_VERSION \
    && nvm use default

# add node and npm to path so the commands are available
ENV NODE_PATH $NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH $NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH

# install and Configure akurath
WORKDIR	/opt
RUN  git clone https://github.com/codespaces-io/akurath.git
RUN source $NVM_DIR/nvm.sh && npm install grunt -g
WORKDIR /opt/akurath
RUN source $NVM_DIR/nvm.sh && npm install grunt-exec grunt-contrib-compress grunt-contrib-compress grunt-contrib-compress; exit 0
COPY akurath /etc/init.d/akurath
COPY akurath.service /lib/systemd/system/akurath.service
#RUN systemctl daemon-reload
#RUN systemctl enable akurath
RUN mkdir /myworkspace
RUN ln -s /lib/systemd/system/akurath.service /etc/systemd/system/akurath.service
RUN chmod +x /etc/init.d/akurath
RUN /etc/init.d/akurath start

######################### install puppet-master 

WORKDIR /tmp

RUN wget https://apt.puppetlabs.com/puppet-release-bionic.deb && dpkg -i puppet-release-bionic.deb
RUN apt-get install puppetmaster && /lib/systemd/systemd-sysv-install enable puppet-master 
RUN lib/systemd/systemd-sysv-install start puppet-master
RUN ufw allow 8140/tcp



#########################   cleaning part to reduce the bill size

##  to clean up packages cache
RUN apt-get clean && \
rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

EXPOSE 22 8000

CMD ["/usr/sbin/sshd", "-D"]
