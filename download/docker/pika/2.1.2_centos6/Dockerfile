FROM hub.c.163.com/public/centos:6.5
ADD CentOS6-Base-aliyun.repo /etc/yum.repos.d/CentOS-Base.repo
#RUN yum makecache fast && yum -y update glic && yum install -y wget vim tar 
RUN yum -y update && yum install -y wget vim
#RUN cd /tmp && wget https://github.com/Qihoo360/pika/releases/download/v2.1.2/pika2.1.2_centos6_bin.tar.gz
ENV PIKA_HOME /opt/pika2.1.2/
RUN mkdir -p $PIKA_HOME
ADD pika2.1.2_centos6_bin.tar.gz $PIKA_HOME
#RUN cd $PIKA_HOME && ls -l
EXPOSE 9221
#CMD ["/bin/bash"]
CMD cd $PIKA_HOME/output && ./bin/pika -c conf/pika.conf
