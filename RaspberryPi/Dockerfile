# argus 3.0.8
#
# VERSION               1
# NOTES:
#
#FROM      opennsm/debian
FROM resin/rpi-raspbian
MAINTAINER Sebastian Garcia <sebastian.garcia@agents.fel.cvut.cz>

# Metadata
LABEL organization=StratosphereIPS
LABEL program=argus

# Specify container username e.g. training, demo
ENV VIRTUSER root
# Specify program
ENV PROG argus
# Specify source extension
ENV EXT tar.gz
# Specify argus server and client version to download and install
ENV VERS 3.0.8.1
ENV CVERS 3.0.8
# Install directory
ENV PREFIX /opt
# Path should include prefix
ENV PATH /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:$PREFIX/sbin:$PREFIX/bin

# Install dependencies
RUN apt-get update -qq
RUN apt-get install -yq gcc make bison flex libpcap-dev libsasl2-dev libwrap0-dev libgeoip-dev libpcre3-dev wget --no-install-recommends

# Compile and install argus
USER $VIRTUSER
WORKDIR /$VIRTUSER
RUN wget --no-check-certificate http://qosient.com/argus/src/$PROG-$VERS.$EXT
RUN tar -zxf $PROG-$VERS.$EXT
WORKDIR /$VIRTUSER/$PROG-$VERS
RUN ./configure --with-sasl --prefix=/opt && make
USER root
RUN make install
RUN chmod u+s $PREFIX/sbin/argus

# Compile and install argus client
USER $VIRTUSER
WORKDIR /$VIRTUSER
RUN wget --no-check-certificate http://qosient.com/argus/src/$PROG-clients-$CVERS.$EXT
RUN tar -zxf $PROG-clients-$CVERS.$EXT
WORKDIR /$VIRTUSER/$PROG-clients-$CVERS
RUN ./configure --with-sasl --with-GeoIP --with-libpcre --prefix=/opt && make
USER root
RUN make install

# Cleanup
RUN rm -rf /$VIRTUSER/$PROG-$VERS
RUN rm -rf /$VIRTUSER/$PROG-clients-$CVERS

# Install pipework
USER $VIRTUSER
WORKDIR /$VIRTUSER
RUN wget --no-check-certificate https://raw.githubusercontent.com/jpetazzo/pipework/master/pipework

# Create the argus configuration for Stratosphere
USER $VIRTUSER
WORKDIR /$VIRTUSER
RUN echo "ARGUS_FLOW_TYPE=\"Bidirectional\"" > argus.conf
RUN echo "ARGUS_FLOW_KEY=\"CLASSIC_5_TUPLE\"" >> argus.conf
RUN echo "ARGUS_DAEMON=no" >> argus.conf
RUN echo "ARGUS_ACCESS_PORT=562" >> argus.conf
# eth1 is created by pipework. This is for Raspberry pi with docker mostly
RUN echo "ARGUS_INTERFACE=eth1" >> argus.conf
RUN echo "ARGUS_CAPTURE_FULL_CONTROL_DATA=yes" >> argus.conf
# The important line to send the flows to Stratosphere SDA
RUN echo "ARGUS_OUTPUT_STREAM=argus-udp://147.32.80.119:10500" >> argus.conf
RUN echo "ARGUS_SET_PID=yes" >> argus.conf
RUN echo "ARGUS_PID_PATH=\"/var/run\"" >> argus.conf
RUN echo "ARGUS_FLOW_STATUS_INTERVAL=3600" >> argus.conf
RUN echo "ARGUS_MAR_STATUS_INTERVAL=60" >> argus.conf
RUN echo "ARGUS_DEBUG_LEVEL=0" >> argus.conf
RUN echo "ARGUS_GENERATE_RESPONSE_TIME_DATA=yes" >> argus.conf
RUN echo "ARGUS_GENERATE_PACKET_SIZE=yes" >> argus.conf
RUN echo "ARGUS_GENERATE_JITTER_DATA=yes" >> argus.conf
RUN echo "ARGUS_GENERATE_MAC_DATA=yes" >> argus.conf
RUN echo "ARGUS_GENERATE_APPBYTE_METRIC=yes" >> argus.conf
RUN echo "ARGUS_GENERATE_TCP_PERF_METRIC=yes" >> argus.conf
RUN echo "ARGUS_GENERATE_BIDIRECTIONAL_TIMESTAMPS=yes" >> argus.conf
RUN echo "ARGUS_KEYSTROKE_CONF=\"DC_MIN=20;DS_MIN=20\"" >> argus.conf
RUN echo "ARGUS_KEYSTROKE=\"yes\"" >> argus.conf

# Environment
WORKDIR /$VIRTUSER
USER root
