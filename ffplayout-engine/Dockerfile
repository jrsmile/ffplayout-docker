FROM python:3-alpine

RUN pip install supervisor

# get ffplayout engine and install all dependencies
RUN cd /opt && \
    wget "https://github.com/ffplayout/ffplayout-engine/archive/master.zip" && \
    unzip 'master.zip' && \
    mv 'ffplayout-engine-master' 'ffplayout-engine' && \
    rm master.zip && \
    cd ffplayout-engine && \
    pip install --no-cache-dir -r requirements-base.txt && \
    mkdir /etc/ffplayout

# install build tools, ffmpeg dependencies and compile ffmpeg
RUN apk update && \
    apk add --no-cache build-base gcc libtool gperf nasm yasm linux-headers \
        expat-dev zlib-dev openssl-dev fontconfig-dev libxml2-dev lame-dev \
        gnu-libiconv libpng-dev fribidi-dev zeromq-dev freetype-dev opus-dev \
        fdk-aac-dev libvpx-dev libass-dev x264-dev x265-dev && \
    # psutil needs gcc
    pip install --no-cache-dir psutil && \
    cd /opt && \
    wget "https://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2" && \
    tar xfvj ffmpeg-snapshot.tar.bz2 && \
    rm ffmpeg-snapshot.tar.bz2 && \
    cd ffmpeg && \
    ./configure --prefix='/usr/local/' --disable-debug --disable-doc --enable-gpl \
    --enable-version3 --enable-nonfree --enable-runtime-cpudetect --enable-fontconfig \
    --enable-openssl --enable-libass --enable-libfdk-aac --enable-libfreetype \
    --enable-libfribidi --enable-libmp3lame --enable-libopus --enable-libvpx \
    --enable-libx264 --enable-libx265 --enable-libzmq && \
    make -j $(nproc | awk '{print $1 / 2}') && \
    make install && \
    cd .. && \
    rm -rf ffmpeg && \
    # cleanup
    apk del build-base gcc libtool gperf nasm yasm

COPY assets/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY assets/socket-server.py /usr/local/bin/socket-server.py
COPY assets/engine-listener.py /usr/local/bin/engine-listener.py

WORKDIR /opt/ffplayout-engine
EXPOSE 5555 64233

CMD ["/usr/local/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
