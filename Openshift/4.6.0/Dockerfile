#FROM centos:6
# This is a RHEL 7 image from Redhat
FROM registry.access.redhat.com/rhel7

MAINTAINER Couchbase Docker Team <docker@couchbase.com>

COPY help.md /tmp/

# Install yum dependencies
RUN REPOLIST=rhel-7-server-rpms,rhel-7-server-optional-rpms \
    INSTALL_PKGS="golang-github-cpuguy83-go-md2man openssl lsof lshw \
      net-tools numactl sysstat wget screen psmisc zip unzip" && \
    yum -y update-minimal --disablerepo "*" --enablerepo rhel-7-server-rpms --setopt=tsflags=nodocs \
      --security --sec-severity=Important --sec-severity=Critical && \
    yum -y install --disablerepo "*" --enablerepo ${REPOLIST} --setopt=tsflags=nodocs ${INSTALL_PKGS} && \
    go-md2man -in /tmp/help.md -out /help.1 && rm -f /tmp/help.md && \
#clean the cache
    yum clean all

RUN curl https://bootstrap.pypa.io/get-pip.py | python - && \
    pip install --no-cache-dir httplib2

COPY functions /etc/init.d/
COPY licenses /licenses

# Create Couchbase user with UID 1000 (necessary to match default
# boot2docker UID)
RUN useradd couchbase -g 0 -u1000 -m -s /bin/bash && \
    echo 'couchbase:couchbase' | chpasswd

ENV CB_VERSION=4.6.0 \
    CB_RELEASE_URL=http://packages.couchbase.com/releases \
    CB_PACKAGE=couchbase-server-enterprise-4.6.0-centos7.x86_64.rpm \
    PATH=$PATH:/opt/couchbase/bin:/opt/couchbase/bin/tools:/opt/couchbase/bin/install

# Install couchbase
RUN rpm --install $CB_RELEASE_URL/$CB_VERSION/$CB_PACKAGE && \
    chown -R couchbase:0 /opt/couchbase && \
    chmod -R g=u /opt/couchbase

COPY scripts/couchbase-start /usr/local/bin/

LABEL name=rhel7/couchbase-server
LABEL release=Latest 
LABEL vendor=Couchbase 
LABEL version=${CB_VERSION} 
LABEL architecture="x86_64"
LABEL summary="Couchbase"
LABEL description="Couchbase will ....."
LABEL run="docker run -d --rm --privileged -p 8091:8091 --restart always --name NAME IMAGE \
            -v /opt/couchbase/var:/opt/couchbase/var \
            -v /opt/couchbase/var/lib/moxi:/opt/couchbase/var/lib/moxi \
            -v /opt/couchbase/var/lib/stats:/opt/couchbase/var/lib/stats "

USER 1000

ENTRYPOINT ["couchbase-start"]
CMD ["couchbase-server", "--", "-noinput"]
# pass -noinput so it doesn't drop us in the erlang shell

EXPOSE 8091 8092 8093 11207 11210 11211 18091 18092
VOLUME /opt/couchbase/var
