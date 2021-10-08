# set the base image to CentOS
# https://hub.docker.com/_/centos/
FROM centos:latest

# to clean repos
RUN yum clean all && rm -rvf /var/cache/yum

# to install epel and initscripts
RUN yum install -y epel-release # EPEL for extra packages
RUN yum install -y initscripts  # for old "service"

ENV container docker

# replace shell with bash so we can source files
RUN rm /bin/sh && ln -s /bin/bash /bin/sh

# update the repository sources list
# and install dependencies
RUN yum update -y\
    && yum install -y curl man git openssh-server openssh-clients openssh net-tools vim sudo curl bind-utils  gcc gcc-c++ make man


RUN ln -s /usr/lib/systemd/system/sshd.service /etc/systemd/system/default.target.wants/sshd.service 
#RUN systemctl start sshd
RUN echo 'root:redhat' |chpasswd
RUN useradd -m -G sys,wheel,systemd-journal -s /bin/bash devops
RUN echo "devops:devops"|chpasswd


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

# to set local time zone

RUN unlink /etc/localtime && ln -s /usr/share/zoneinfo/Asia/Kolkata /etc/localtime

EXPOSE 22 8000

VOLUME [ "/tmp", "/run", "/sys/fs/cgroup" ]
CMD ["/usr/sbin/init"]

