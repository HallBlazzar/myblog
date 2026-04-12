FROM debian:13.2

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get install -qq -y  \
        python3.13 \
        git \
        dpkg-dev \
        curl \
        locales \
        wget

# set locale for lb and package installation
RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && \
    locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

RUN wget https://github.com/gohugoio/hugo/releases/download/v0.157.0/hugo_extended_withdeploy_0.157.0_linux-amd64.deb && \
    dpkg -i hugo_extended_withdeploy_0.157.0_linux-amd64.deb

#RUN wget https://go.dev/dl/go1.26.1.linux-amd64.tar.gz && \
#    tar -xvzf go1.26.1.linux-amd64.tar.gz -C /usr/local && \
#    export PATH=\$PATH:/usr/local/go/bin
#
#RUN wget https://github.com/sass/dart-sass/releases/download/1.98.0/dart-sass-1.98.0-linux-x64.tar.gz && \
#    tar -xvzf dart-sass-1.98.0-linux-x64.tar.gz -C /usr/local && \
#    export PATH=\$PATH:/usr/local/dart-sass
#RUN apt-get install -qq -y hugo

ENTRYPOINT ["sleep", "infinity"]
