##############################
# CentOS Base Image Dockerfile
##############################

FROM centos

RUN sed -i '$ a\proxy=http://172.17.42.1:3128' /etc/yum.conf && \
    yum update -y; yum clean all
