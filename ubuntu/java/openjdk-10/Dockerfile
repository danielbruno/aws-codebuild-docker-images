# Copyright 2017-2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.
#
# Licensed under the Amazon Software License (the "License"). You may not use this file except in compliance with the License.
# A copy of the License is located at
#
#    http://aws.amazon.com/asl/
#
# or in the "license" file accompanying this file.
# This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, express or implied.
# See the License for the specific language governing permissions and limitations under the License.
#

FROM ubuntu:14.04.5

ENV DOCKER_BUCKET="download.docker.com" \
    DOCKER_VERSION="17.09.0-ce" \
    DOCKER_CHANNEL="stable" \
    DOCKER_SHA256="a9e90a73c3cdfbf238f148e1ec0eaff5eb181f92f35bdd938fd7dab18e1c4647" \
    DIND_COMMIT="3b5fac462d21ca164b3778647420016315289034" \
    DOCKER_COMPOSE_VERSION="1.16.1"

# Building git from source code:
#   Ubuntu's default git package is built with broken gnutls. Rebuild git with openssl.
##########################################################################
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       wget=1.15-* python=2.7.5-* python2.7-dev=2.7.6-* fakeroot=1.20-* ca-certificates \
       tar=1.27.1-* gzip=1.6-* zip=3.0-* autoconf=2.69-* automake=1:1.14.1-* \
       bzip2=1.0.6-* file=1:5.14-* g++=4:4.8.2-* gcc=4:4.8.2-* imagemagick=8:6.7.7.10-* \
       libbz2-dev=1.0.6-* libc6-dev=2.19-* libcurl4-openssl-dev=7.35.0-* libdb-dev=1:5.3.21~* \
       libevent-dev=2.0.21-stable-* libffi-dev=3.1~rc1+r3.0.13-* libgeoip-dev=1.6.0-* libglib2.0-dev=2.40.2-* \
       libjpeg-dev=8c-* libkrb5-dev=1.12+dfsg-* liblzma-dev=5.1.1alpha+20120614-* \
       libmagickcore-dev=8:6.7.7.10-* libmagickwand-dev=8:6.7.7.10-* libmysqlclient-dev=5.5.59-* \
       libncurses5-dev=5.9+20140118-* libpng12-dev=1.2.50-* libpq-dev=9.3.22-* libreadline-dev=6.3-* \
       libsqlite3-dev=3.8.2-* libssl-dev=1.0.1f-* libtool=2.4.2-* libwebp-dev=0.4.0-* \
       libxml2-dev=2.9.1+dfsg1-* libxslt1-dev=1.1.28-* libyaml-dev=0.1.4-* make=3.81-* \
       patch=2.7.1-* xz-utils=5.1.1alpha+20120614-* zlib1g-dev=1:1.2.8.dfsg-* unzip=6.0-* curl=7.35.0-* \
       e2fsprogs=1.42.9-* iptables=1.4.21-* xfsprogs=3.1.9ubuntu2 xz-utils=5.1.1alpha+20120614-* \
    && apt-get install -y -qq less=458-* groff=1.22.2-* \
    && apt-get -qy build-dep git=1:1.9.1 \
    && apt-get -qy install libcurl4-openssl-dev=7.35.0-* git-man=1:1.9.1-* liberror-perl=0.17-* \
    && mkdir -p /usr/src/git-openssl \
    && cd /usr/src/git-openssl \
    && apt-get source git=1:1.9.1 \
    && cd $(find -mindepth 1 -maxdepth 1 -type d -name "git-*") \
    && sed -i -- 's/libcurl4-gnutls-dev/libcurl4-openssl-dev/' ./debian/control \
    && sed -i -- '/TEST\s*=\s*test/d' ./debian/rules \
    && dpkg-buildpackage -rfakeroot -b \
    && find .. -type f -name "git_*ubuntu*.deb" -exec dpkg -i \{\} \; \
    && rm -rf /usr/src/git-openssl \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# Install Docker
RUN set -x \
    && curl -fSL "https://${DOCKER_BUCKET}/linux/static/${DOCKER_CHANNEL}/x86_64/docker-${DOCKER_VERSION}.tgz" -o docker.tgz \
    && echo "${DOCKER_SHA256} *docker.tgz" | sha256sum -c - \
    && tar --extract --file docker.tgz --strip-components 1  --directory /usr/local/bin/ \
    && rm docker.tgz \
    && docker -v \
# set up subuid/subgid so that "--userns-remap=default" works out-of-the-box
    && addgroup dockremap \
    && useradd -g dockremap dockremap \
    && echo 'dockremap:165536:65536' >> /etc/subuid \
    && echo 'dockremap:165536:65536' >> /etc/subgid \
    && wget "https://raw.githubusercontent.com/docker/docker/${DIND_COMMIT}/hack/dind" -O /usr/local/bin/dind \
    && curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-Linux-x86_64 > /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/dind /usr/local/bin/docker-compose \
# Ensure docker-compose works
    && docker-compose version

# Install dependencies by all python images equivalent to buildpack-deps:jessie
# on the public repos.

RUN wget "https://bootstrap.pypa.io/get-pip.py" -O /tmp/get-pip.py \
    && python /tmp/get-pip.py \
    && pip install awscli \
    && rm -fr /var/lib/apt/lists/* /tmp/* /var/tmp/*

VOLUME /var/lib/docker

COPY dockerd-entrypoint.sh /usr/local/bin/

ENV JAVA_VERSION=9 \
    JAVA_HOME="/opt/jvm/openjdk-10" \
    JDK_HOME="/opt/jvm/openjdk-10" \
    JRE_HOME="/opt/jvm/openjdk-10" \
    ANT_VERSION=1.10.3 \
    MAVEN_HOME="/opt/maven" \
    MAVEN_VERSION=3.5.2 \
    MAVEN_CONFIG="/root/.m2" \
    GRADLE_VERSION=4.2.1 \
    SBT_VERSION=1.0.3 \
    PROPERTIES_COMMON_VERSION=0.92.37.8 \
    PYTHON_TOOL_VERSION="3.3-*" \
    JDK_VERSION=10.0.1 \
    JDK_DOWNLOAD_SHA256="0b14aaecd5323457bd15dc7798d08181ad04bad4156e55387ed714190912a9ce" \
    ANT_DOWNLOAD_SHA512="73f2193700b1d1e32eedf25fab1009e2a98fb2f6425413f5c9fa1b0f2f9f49f59cb8ed3f04931c808ae022a64ecfa2619e5fb77643fea6dbc29721e489eb3a07" \
    MAVEN_DOWNLOAD_SHA1="190dcebb8a080f983af4420cac4f3ece7a47dd64" \
    GRADLE_DOWNLOAD_SHA256="b551cc04f2ca51c78dd14edb060621f0e5439bdfafa6fd167032a09ac708fbc0"

ENV JDK_DOWNLOAD_TAR="openjdk-${JDK_VERSION}_linux-x64_bin.tar.gz"

RUN apt-get update \
    && apt-get install -y software-properties-common=$PROPERTIES_COMMON_VERSION \
    && apt-get -y install python-setuptools=$PYTHON_TOOL_VERSION \

    # Install OpenJDK 10
    # Note: Installing ca-certificates-java installs JDK7 because it's a depedency.
    # We will use update-alternatives to make sure JDK10 has higher priority for all
    # the tools
    && apt-get install -y ca-certificates-java \

    && mkdir -p $JAVA_HOME \
    && curl -LSso /var/tmp/$JDK_DOWNLOAD_TAR https://download.java.net/java/GA/jdk10/10.0.1/fb4372174a714e6b8c52526dc134031e/10/openjdk-10.0.1_linux-x64_bin.tar.gz \
    && echo "$JDK_DOWNLOAD_SHA256 /var/tmp/$JDK_DOWNLOAD_TAR" | tee foo.txt | sha256sum -c - \
    && tar xzvf /var/tmp/$JDK_DOWNLOAD_TAR -C $JAVA_HOME --strip-components=1 \
    && for tool_path in $JAVA_HOME/bin/*; do \
          tool=`basename $tool_path`; \
          update-alternatives --install /usr/bin/$tool $tool $tool_path 10000; \
          update-alternatives --set $tool $tool_path; \
        done \
    && rm $JAVA_HOME/lib/security/cacerts && ln -s /etc/ssl/certs/java/cacerts $JAVA_HOME/lib/security/cacerts \

    # Install Ant
	&& curl -LSso /var/tmp/apache-ant-$ANT_VERSION-bin.tar.gz https://www.apache.org/dist/ant/binaries/apache-ant-$ANT_VERSION-bin.tar.gz  \
	&& tar -xzf /var/tmp/apache-ant-$ANT_VERSION-bin.tar.gz -C /opt \
	&& update-alternatives --install /usr/bin/ant ant /opt/apache-ant-$ANT_VERSION/bin/ant 10000 \

    # Install Maven
    && mkdir -p $MAVEN_HOME \
    && curl -LSso /var/tmp/apache-maven-$MAVEN_VERSION-bin.tar.gz https://apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz \
    && echo "$MAVEN_DOWNLOAD_SHA1 /var/tmp/apache-maven-$MAVEN_VERSION-bin.tar.gz" | sha1sum -c - \
    && tar xzvf /var/tmp/apache-maven-$MAVEN_VERSION-bin.tar.gz -C $MAVEN_HOME --strip-components=1 \
    && update-alternatives --install /usr/bin/mvn mvn /opt/maven/bin/mvn 10000 \
    && mkdir -p $MAVEN_CONFIG \

    # Install Gradle
    && curl -LSso /var/tmp/gradle-$GRADLE_VERSION-bin.zip https://services.gradle.org/distributions/gradle-$GRADLE_VERSION-bin.zip \
    && echo "$GRADLE_DOWNLOAD_SHA256 /var/tmp/gradle-$GRADLE_VERSION-bin.zip" | sha256sum -c - \
    && unzip /var/tmp/gradle-$GRADLE_VERSION-bin.zip -d /opt \
    && update-alternatives --install /usr/local/bin/gradle gradle /opt/gradle-$GRADLE_VERSION/bin/gradle 10000 \

    # Install SBT
    && curl -LSso /var/tmp/sbt-$SBT_VERSION.tgz https://github.com/sbt/sbt/releases/download/v$SBT_VERSION/sbt-$SBT_VERSION.tgz \
    && curl -LSso /var/tmp/sbt-$SBT_VERSION.tgz.asc https://github.com/sbt/sbt/releases/download/v$SBT_VERSION/sbt-$SBT_VERSION.tgz.asc \
    && curl -LSso /var/tmp/sbt.key.asc https://bintray.com/user/downloadSubjectPublicKey?username=sbt \

    && gpg --no-default-keyring --keyring /var/tmp/tmp-keyring.gpg --import /var/tmp/sbt.key.asc \
    && gpg --no-default-keyring --keyring /var/tmp/tmp-keyring.gpg --verify /var/tmp/sbt-$SBT_VERSION.tgz.asc /var/tmp/sbt-$SBT_VERSION.tgz \

    && tar xzvf /var/tmp/sbt-$SBT_VERSION.tgz -C /opt && update-alternatives --install /usr/local/bin/sbt sbt /opt/sbt/bin/sbt 10000 \

    # Cleanup
    && rm -fr /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && apt-get clean

COPY m2-settings.xml $MAVEN_CONFIG/settings.xml

#ENTRYPOINT ["dockerd-entrypoint.sh"]
