FROM debian:jessie

RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak \
        && echo "deb http://kambing.ui.ac.id/debian/ jessie main contrib non-free" > /etc/apt/sources.list \
        && echo "deb http://kambing.ui.ac.id/debian/ jessie-updates main contrib non-free" >> /etc/apt/sources.list \
        && echo "deb http://kambing.ui.ac.id/debian-security/ jessie/updates main contrib non-free" >> /etc/apt/sources.list

RUN apt-get update && apt-get install -y beanstalkd --no-install-recommends && rm -rf /var/lib/apt/lists/*

RUN echo "START=yes" >> /etc/default/beanstalkd

EXPOSE 11300

CMD ["beanstalkd"]
