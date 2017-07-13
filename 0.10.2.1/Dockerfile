# Kafka and Zookeeper
FROM java:openjdk-8-jre

ENV JMX_PORT 9999
ENV DEBIAN_FRONTEND noninteractive
ENV SCALA_VERSION 2.12
ENV KAFKA_VERSION 0.10.2.1
ENV KAFKA_HOME /opt/kafka
# Python
ENV PYTHON_VERSION="2.7.9-1"
ENV PYTHON_PIP_VERSION="8.1.2"

# Install Kafka, Zookeeper and other needed things
RUN apt-get update && \
    apt-get install -y apt-transport-https curl wget netcat python=${PYTHON_VERSION} net-tools zookeeper supervisor dnsutils \
    && echo "===> Installing python packages ..."  \
    && curl -fSL "https://bootstrap.pypa.io/get-pip.py" | python \
    && pip install --no-cache-dir --upgrade pip==${PYTHON_PIP_VERSION} \
    && pip install --no-cache-dir jinja2 \
                                  requests \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean \
    && wget -q http://apache.mirrors.spacedump.net/kafka/"$KAFKA_VERSION"/kafka_"$SCALA_VERSION"-"$KAFKA_VERSION".tgz -O /tmp/kafka_"$SCALA_VERSION"-"$KAFKA_VERSION".tgz \
    && tar xfz /tmp/kafka_"$SCALA_VERSION"-"$KAFKA_VERSION".tgz -C /opt \
    && mv /opt/kafka_"$SCALA_VERSION"-"$KAFKA_VERSION" /opt/kafka \
    && rm /tmp/kafka_"$SCALA_VERSION"-"$KAFKA_VERSION".tgz

# add things to container
COPY scripts/dub /usr/local/bin/dub
COPY scripts/templates $KAFKA_HOME/templates
ADD scripts/start-kafka.sh /usr/bin/start-kafka.sh
ADD scripts/create-topics.sh /usr/bin/create-topics.sh

# fix for the missing rmi port mapping
RUN sed -i -e 's/-Dcom.sun.management.jmxremote.port=$JMX_PORT/-Dcom.sun.management.jmxremote.rmi.port=$JMX_PORT -Dcom.sun.management.jmxremote.port=$JMX_PORT/g' "$KAFKA_HOME"/bin/kafka-run-class.sh

# Supervisor config
ADD supervisor/kafka.conf supervisor/zookeeper.conf /etc/supervisor/conf.d/

# 2181 is zookeeper, 9092 is kafka
EXPOSE 2181 9092 9999

CMD ["supervisord", "-n"]
