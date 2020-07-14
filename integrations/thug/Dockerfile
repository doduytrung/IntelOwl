FROM ubuntu:18.04

# Step 1: Build and install thug

USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    sudo \
    python \
    python3 \
    python3-dev \
    python3-setuptools \
    python3-wheel \ 
    libboost-dev \
    libboost-python-dev \
    libboost-system-dev \
    python3-pip \
    libxml2-dev \
    libxslt-dev \
    tesseract-ocr \
    git \
    libtool \
    graphviz-dev \
    automake \
    libffi-dev \
    graphviz \
    libfuzzy-dev \
    libfuzzy2 \
    libjpeg-dev \
    libffi-dev \
    pkg-config \
    autoconf && \
  rm -rf /var/lib/apt/lists/*

RUN pip3 install --upgrade pip
RUN pip3 install --upgrade setuptools
RUN pip3 install --upgrade pytesseract
WORKDIR /home/thug

RUN pip3 install https://github.com/area1/stpyv8/releases/download/v8.3.110.9/stpyv8-8.3.110.9-cp36-cp36m-linux_x86_64.whl

RUN git clone https://github.com/buffer/libemu.git && \
  cd libemu && \
  autoreconf -v -i && \
  ./configure && \
  make install && \
  cd .. && \
  rm -rf libemu

RUN ldconfig
RUN pip3 install --no-cache-dir thug

RUN git clone https://github.com/buffer/thug.git && \
  mkdir /etc/thug && \
  mkdir /etc/thug/rules && \
  mkdir /etc/thug/personalities && \
  mkdir /etc/thug/scripts && \
  mkdir /etc/thug/plugins && \
  mkdir /etc/thug/hooks && \
  cp -R thug/thug/Classifier/rules/* /etc/thug/rules/ && \
  cp -R thug/thug/DOM/personalities/* /etc/thug/personalities && \
  cp thug/thug/DOM/thug.js /etc/thug/scripts && \
  cp thug/thug/DOM/storage.js /etc/thug/scripts && \
  cp thug/thug/DOM/date.js /etc/thug/scripts && \
  cp thug/thug/DOM/eval.js /etc/thug/scripts && \
  cp thug/thug/DOM/write.js /etc/thug/scripts && \
  cp thug/conf/thug.conf /etc/thug && \
  rm -rf thug

RUN groupadd -r thug && \
  useradd -r -g thug -d /home/thug -s /sbin/nologin -c "Thug User" thug && \
  chown -R thug:thug /home/thug


# Step 2: Build Flask REST API

ENV PROJECT_PATH /opt/deploy/thug-flask
ENV LOG_PATH /var/log/intel_owl/thug

# Create log files
RUN mkdir -p ${LOG_PATH}
WORKDIR ${LOG_PATH}
VOLUME ${LOG_PATH}
RUN touch thug.log thug_errors.log gunicorn_access.log gunicorn_errors.log \
    && chown -R thug:thug  ./

# install flask dependencies
WORKDIR ${PROJECT_PATH}
COPY app.py requirements.txt ./
RUN pip3 install -r requirements.txt --no-cache-dir

# Step 3: Run the gunicorn server to serve the flask application

EXPOSE 4001

USER thug
ENTRYPOINT gunicorn 'app:app' \
            --bind '0.0.0.0:4001' \
            --user thug \
            --log-level ${LOG_LEVEL} \
            --access-logfile ${LOG_PATH}/gunicorn_access.log \
            --error-logfile ${LOG_PATH}/gunicorn_errors.log