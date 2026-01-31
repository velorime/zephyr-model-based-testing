FROM ubuntu:22.04 AS tlatools-builder

ARG TLA_TOOLS_REPO=https://github.com/tlaplus/tlaplus.git
ARG TLA_TOOLS_COMMIT=3bc9e865ac07a495860f7cd4ac7598e5c1cdf65c

ARG TLA_COMMUNITY_MODULES_REPO=https://github.com/tlaplus/CommunityModules.git
ARG TLA_COMMUNITY_MODULES_COMMIT=d0fe42f233c3e837dca03cb969323dd67e11f1a7

RUN apt-get update && apt-get install -y \
        git \
        openjdk-17-jdk \
        ant \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt

RUN git init tla-plus \
    && cd tla-plus \
    && git remote add origin ${TLA_TOOLS_REPO} \
    && git fetch origin ${TLA_TOOLS_COMMIT} \
    && git checkout FETCH_HEAD;

WORKDIR /opt/tla-plus/tlatools/org.lamport.tlatools
RUN ant -f customBuild.xml info compile compile-test dist

RUN find . -name 'tla2tools*.jar' -exec cp {} /opt/tla2tools.jar \; || true

WORKDIR /opt

RUN git init CommunityModules \
    && cd CommunityModules \
    && git remote add origin ${TLA_COMMUNITY_MODULES_REPO} \
    && git fetch origin ${TLA_COMMUNITY_MODULES_COMMIT} \
    && git checkout FETCH_HEAD;

WORKDIR /opt/CommunityModules

RUN mkdir tlc && cp /opt/tla2tools.jar tlc/
RUN ant -Dskip.download=true -f build.xml dist
RUN cp dist/CommunityModules-deps.jar /opt/CommunityModules-deps.jar


FROM ghcr.io/zephyrproject-rtos/zephyr-build:v0.27.4

USER root

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y \
        vim \
        bash-completion \
        openjdk-17-jre  # required for TLA+

RUN pip install pre-commit

RUN mkdir -p /workdir/zephyr-model-based-testing
RUN chown user:user /workdir
RUN chsh -s /bin/bash user

# Install Community Modules and (copy) the built TLA+ tools from the builder stage
COPY --from=tlatools-builder /opt/tla2tools.jar /usr/share/java/tla2tools.jar
COPY --from=tlatools-builder /opt/CommunityModules-deps.jar /usr/share/java/CommunityModules-deps.jar

# Add TLA+ tools, community modules and our TLA+ library to the Java classpath
ENV CLASSPATH=/usr/share/java/tla2tools.jar:/usr/share/java/CommunityModules-deps.jar:/workdir/zephyr-model-based-testing/specs/lib

USER user
COPY .bash_aliases /home/user/.bash_aliases
