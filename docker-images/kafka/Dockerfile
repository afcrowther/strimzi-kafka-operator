FROM strimzi/base:latest

ARG KAFKA_DIST_DIR
ARG KAFKA_VERSION
ARG THIRD_PARTY_LIBS
ARG strimzi_version
#####
# Add Kafka Exporter
#####
ARG KAFKA_EXPORTER_ARCH=amd64
ENV KAFKA_EXPORTER_VERSION=1.2.0
ENV KAFKA_EXPORTER_HOME=/opt/kafka-exporter

RUN yum -y install gettext nmap-ncat stunnel net-tools bind-utils && yum clean all -y

# Add kafka user with UID 1001
# The user is in the group 0 to have access to the mounted volumes and storage
RUN useradd -r -m -u 1001 -g 0 kafka

#####
# Add Kafka
#####
ENV KAFKA_HOME=/opt/kafka
ENV KAFKA_VERSION=${KAFKA_VERSION}
ENV STRIMZI_VERSION=${strimzi_version}

COPY $KAFKA_DIST_DIR $KAFKA_HOME
COPY ./scripts/ $KAFKA_HOME

RUN curl -LO https://github.com/danielqsj/kafka_exporter/releases/download/v${KAFKA_EXPORTER_VERSION}/kafka_exporter-${KAFKA_EXPORTER_VERSION}.linux-${KAFKA_EXPORTER_ARCH}.tar.gz \
    && mkdir $KAFKA_EXPORTER_HOME \
    && tar xvfz kafka_exporter-${KAFKA_EXPORTER_VERSION}.linux-${KAFKA_EXPORTER_ARCH}.tar.gz -C $KAFKA_EXPORTER_HOME --strip-components=1 \
    && rm -f kafka_exporter-${KAFKA_EXPORTER_VERSION}.linux-${KAFKA_EXPORTER_ARCH}.tar.gz*

COPY ./exporter-scripts $KAFKA_EXPORTER_HOME

#####
# Add Strimzi agents
#####
COPY ./tmp/kafka-agent.jar ${KAFKA_HOME}/libs/
COPY ./tmp/mirror-maker-agent.jar ${KAFKA_HOME}/libs/
COPY ./tmp/tracing-agent.jar ${KAFKA_HOME}/libs/

#####
# Add 3rd party libs
#####
COPY kafka-thirdparty-libs/${THIRD_PARTY_LIBS}/target/dependency/ ${KAFKA_HOME}/libs/

#####
# Add Cruise Control
#####
ENV CRUISE_CONTROL_HOME=/opt/cruise-control
RUN mkdir $CRUISE_CONTROL_HOME
COPY kafka-thirdparty-libs/cc/target/dependency/ ${CRUISE_CONTROL_HOME}/libs/
COPY ./cruise-control-scripts $CRUISE_CONTROL_HOME

#####
# Add Stunnel
#####
ENV STUNNEL_HOME=/opt/stunnel
RUN mkdir $STUNNEL_HOME && mkdir -p -m g+rw /usr/local/var/run/
COPY ./stunnel-scripts/ $STUNNEL_HOME

#####
# Add Kafka Connect S2I feature
#####
ENV S2I_HOME=/opt/kafka/s2i
COPY ./s2i-scripts $S2I_HOME

WORKDIR $KAFKA_HOME

USER 1001
