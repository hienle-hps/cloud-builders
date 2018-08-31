FROM gcr.io/cloud-builders/javac:8

ARG GRADLE_VERSION=4.0
ARG USER_HOME_DIR="/root"
ARG SHA=56bd2dde29ba2a93903c557da1745cafd72cdd8b6b0b83c05a40ed7896b79dfe
ARG BASE_URL=https://services.gradle.org/distributions

ENV GRADLE_HOME "/usr/share/gradle-${GRADLE_VERSION}"
ENV GRADLE_USER_HOME "${USER_HOME_DIR}/.gradle/"

RUN apt-get -y update && \
    # JRE is required for cloud-datastore-emulator
    apt-get -y install gcc python2.7 python-dev python-setuptools wget ca-certificates \
       # These are necessary for add-apt-respository
       software-properties-common && \

    # Install Git >2.0.1
    apt-get -y update && \
    apt-get -y install git && \

    # Setup Google Cloud SDK (latest)
    mkdir -p /builder && \
    wget -qO- https://dl.google.com/dl/cloudsdk/release/google-cloud-sdk.tar.gz | tar zxv -C /builder && \
    CLOUDSDK_PYTHON="python2.7" /builder/google-cloud-sdk/install.sh --usage-reporting=false \
        --bash-completion=false \
        --disable-installation-options && \

    # install crcmod: https://cloud.google.com/storage/docs/gsutil/addlhelp/CRC32CandInstallingcrcmod
    easy_install -U pip && \
    pip install -U crcmod && \

    # Clean up
    apt-get -y remove gcc python-dev python-setuptools wget && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf ~/.config/gcloud && \

    # Install all available components
    /builder/google-cloud-sdk/bin/gcloud -q components install \
        alpha beta \
        app-engine-go \
        app-engine-java \
        app-engine-php \
        app-engine-python \
        app-engine-python-extras \
        bigtable \
        cbt \
        cloud-datastore-emulator \
        cloud-build-local \
        container-builder-local \
        datalab \
        docker-credential-gcr \
        emulator-reverse-proxy \
        kubectl \
        pubsub-emulator \
        && \

    /builder/google-cloud-sdk/bin/gcloud -q components update && \
    /builder/google-cloud-sdk/bin/gcloud components list \


    && apt-get update -qqy && apt-get install -qqy curl \
    && mkdir -p /usr/share "${GRADLE_USER_HOME}" \
    && curl -fsSL -o "gradle-${GRADLE_VERSION}-bin.zip" "${BASE_URL}/gradle-${GRADLE_VERSION}-bin.zip" \
    && echo "${SHA}  gradle-${GRADLE_VERSION}-bin.zip" | sha256sum -c - \
    && unzip -qq "gradle-${GRADLE_VERSION}-bin.zip" \
    && rm -f "gradle-${GRADLE_VERSION}-bin.zip" \
    && mv "gradle-${GRADLE_VERSION}" /usr/share \
    && ln -s "${GRADLE_HOME}/bin/gradle" /usr/bin/gradle \
    && apt-get remove -qqy --purge curl \
    && rm /var/lib/apt/lists/*_*

ADD gradle.properties "${GRADLE_USER_HOME}"
ENV PATH=/builder/google-cloud-sdk/bin/:$PATH
ENTRYPOINT ["/usr/bin/gradle"]
