#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
#

FROM openjdk:8

WORKDIR /root/

ARG PREBUILD=true
ARG HADOOP_VERSION=2.7.6
ARG HIVE_VERSION=2.1.1

ENV JAVA8_HOME /docker-java-home
ENV HIVEMALL_VERSION=0.5.1
ENV BASE_URL='https://www.apache.org/dyn/mirrors/mirrors.cgi?action=download&filename='
ENV HADOOP_HOME='/usr/local/hadoop'
ENV HIVE_HOME='/usr/local/hive'
ENV HIVEMALL_PATH='/opt/hivemall'
ENV HADOOP_OPTS=' \
      -Dsystem:java.io.tmpdir=/tmp \
      -Dsystem:user.name=root \
      -Dderby.stream.error.file=/root/derby.log'
ENV PATH="${HADOOP_HOME}/bin:${HIVE_HOME}/bin:${PATH}"

COPY . ${HIVEMALL_PATH}/

COPY resources/docker/tez-0.9.1.tar.gz ${HIVEMALL_PATH}/tez-0.9.1.tar.gz
ADD resources/docker/tez-site.xml ${HADOOP_HOME}/etc/hadoop/tez-site.xml
RUN mkdir -p ${HIVEMALL_PATH}/tez-0.9.1
RUN tar xvzf ${HIVEMALL_PATH}/tez-0.9.1.tar.gz -C ${HIVEMALL_PATH}/tez-0.9.1

# environment settings
ENV TEZ_DIST ${HIVEMALL_PATH}/tez-0.9.1
RUN echo 'TEZ_JARS=${TEZ_DIST}/*' >> ${HADOOP_HOME}/etc/hadoop/hadoop-env.sh
RUN echo 'TEZ_LIB=${TEZ_DIST}/lib/*' >> ${HADOOP_HOME}/etc/hadoop/hadoop-env.sh
RUN echo 'TEZ_CONF=${HADOOP_HOME}/etc/hadoop' >> ${HADOOP_HOME}/etc/hadoop/hadoop-env.sh
RUN echo 'export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$TEZ_CONF:$TEZ_JARS:$TEZ_LIB' >> ${HADOOP_HOME}/etc/hadoop/hadoop-env.sh

RUN set -eux && \
    apt update && \
    curl -sL https://deb.nodesource.com/setup_9.x | bash - && \
    apt install -y --no-install-recommends openssh-server maven g++ make ruby nodejs && \
    npm install -g gitbook-cli

# install hadoop

RUN wget ${BASE_URL}hadoop/common/hadoop-${HADOOP_VERSION}/hadoop-${HADOOP_VERSION}.tar.gz -O - \
      | tar xz && \
    mv hadoop-${HADOOP_VERSION} ${HADOOP_HOME} && \
    sed -i -e 's!${JAVA_HOME}!'"${JAVA_HOME}!" ${HADOOP_HOME}/etc/hadoop/hadoop-env.sh && \
    ssh-keygen -q -P '' -f ~/.ssh/id_rsa && \
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && \
    echo 'host *\n  StrictHostKeyChecking no' > ~/.ssh/config && \
    mv ${HIVEMALL_PATH}/resources/docker/etc/hadoop/*.xml ${HADOOP_HOME}/etc/hadoop

 RUN hdfs namenode -format && hdfs dfs -put ${TEZ_DIST} /tez/

# install hive
RUN wget ${BASE_URL}hive/hive-${HIVE_VERSION}/apache-hive-${HIVE_VERSION}-bin.tar.gz -O - \
      | tar xz && \
    mv apache-hive-${HIVE_VERSION}-bin ${HIVE_HOME} && \
    cat ${HIVE_HOME}/conf/hive-default.xml.template \
      | sed -e 's!databaseName=metastore_db!databaseName=/root/metastore_db!' \
      > ${HIVE_HOME}/conf/hive-site.xml && \
    \
    cd ${HIVEMALL_PATH} && \
    \
    (if ${PREBUILD}; then \
      cd ${HIVEMALL_PATH} && bin/build.sh; \
    fi)

ADD resources/docker/tez-site.xml ${HIVE_HOME}/conf/tez-site.xml

# ADD resources/docker/hive-hwi-1.0.jar ${HIVE_HOME}/lib/hive-hwi-1.0.jar
# ADD resources/docker/hive-hwi-1.0.war ${HIVE_HOME}/lib/hive-hwi-1.0.war

# install hivemall
RUN mkdir -p /root/bin /root/hivemall && \
    find ${HIVEMALL_PATH}/resources/docker/home/bin -mindepth 1 -maxdepth 1 \
      -exec sh -c 'f={} && ln -s $f /root/bin/${f##*/}' \; && \
    ln -s ${HIVEMALL_PATH}/resources/docker/home/.hiverc /root && \
    ln -s ${HIVEMALL_PATH}/resources/ddl/define-all.hive /root/hivemall/define-all.hive && \
    rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/* /root/.m2/* /root/.npm/*

ADD resources/docker/hivemall-all-0.5.1-incubating-SNAPSHOT.jar /root/hivemall/hivemall-all-with-dependencies.jar

VOLUME ["/opt/hivemall/", "/root/data/"]
EXPOSE 8088 19888 50070

CMD ["sh", "-c", "./bin/init.sh && bash"]
