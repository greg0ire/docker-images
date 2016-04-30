FROM alpine:3.3

MAINTAINER Arnaud Chevillard <achevillard@universcine.com>

ENV GPG_KEYS FCF986EA15E6E293A5644F10B4322F04D67658D8

ENV FFMPEG_VERSION=3.0.2 \
    FFMPEG_FILENAME=ffmpeg-3.0.2.tar.gz \

    FDKAAC_VERSION=0.1.4 \ 
    FDKAAC_FILENAME=fdk-aac-0.1.4.tar.gz \

    X265_VERSION=1.9 \
    X265_FILENAME=x265_1.9.tar.gz \

    PKG_CONFIG_PATH=/usr/local/lib/pkgconfig \
    SRC=/usr/local

WORKDIR /tmp

RUN set -xe \
        && apk add --no-cache --virtual .build-deps \
            alpine-sdk \
            autoconf \
            automake \
            cmake \
            faac-dev \
            freetype-dev \
            gnupg \
            gsm-dev \
            lame-dev \
            libass-dev \
            libbluray-dev \
            libcdio-dev \
            libdc1394-dev \
            libogg-dev \
            libtheora-dev \
            libtool \
            libvdpau-dev \
            libvorbis-dev \
            libvpx-dev \
            libwebp-dev \
            nasm \
            opus-dev \
            openssl-dev \
            rtmpdump-dev \
            speex-dev \
            wavpack-dev \
            x264-dev \
            xvidcore-dev \
            yasm \

        && readonly NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) \
        && export MAKEFLAGS="-j$[$(NPROC) + 1]" \

        # X265
        && DIR=$(mktemp -d) && cd ${DIR} \
        && curl -fSL "http://ftp.videolan.org/pub/videolan/x265/$X265_FILENAME" -o "$X265_FILENAME" \
        && tar zxf "$X265_FILENAME" \
        && cd x265_${X265_VERSION}/build/linux \
        && cmake -G "Unix Makefiles" ../../source \
        && cmake ../../source \
        && make \
        && make install \
        && rm -rf ${DIR} \

        # FDK AAC
        && DIR=$(mktemp -d) && cd ${DIR} \
        && curl -fSL "https://codeload.github.com/mstorsjo/fdk-aac/tar.gz/v$FDKAAC_VERSION" -o "$FDKAAC_FILENAME" \
        && tar zxf "$FDKAAC_FILENAME" \
        && cd fdk-aac-${FDKAAC_VERSION} \
        && autoreconf -fiv \
        && ./configure \
            --prefix="${SRC}" \
            --disable-shared \
            --datadir="${DIR}" \
        && make \
        && make install \
        && make distclean \
        && rm -rf ${DIR} \

        # FFMPEG
        && DIR=$(mktemp -d) && cd ${DIR} \
        && curl -fSL "http://ffmpeg.org/releases/$FFMPEG_FILENAME" -o "$FFMPEG_FILENAME" \
        && curl -fSL "http://ffmpeg.org/releases/$FFMPEG_FILENAME.asc" -o "$FFMPEG_FILENAME.asc" \
        && export GNUPGHOME="$(mktemp -d)" \
        && for key in $GPG_KEYS; do \
            gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
        done \
        && gpg --batch --verify "$FFMPEG_FILENAME.asc" "$FFMPEG_FILENAME" \
        && tar zxf "$FFMPEG_FILENAME" \
        && cd ffmpeg-${FFMPEG_VERSION} \
        && ./configure \
            --prefix="${SRC}" \
            --bindir="${SRC}/bin" \
            --extra-cflags="-I${SRC}/include" \
            --extra-ldflags="-L${SRC}/lib" \
            --extra-libs=-ldl \
            --disable-debug \
            --disable-doc \
            --disable-ffserver \
            --disable-ffplay \
            --enable-avresample \
            --enable-gpl \
            --enable-hardcoded-tables \
            --enable-libbluray \
            --enable-libdc1394 \
            --enable-libfaac \
            --enable-libfdk-aac \
            --enable-libgsm \
            --enable-libmp3lame \
            --enable-libopus \
            --enable-librtmp \
            --enable-libspeex \
            --enable-libtheora \
            --enable-libvorbis \
            --enable-libvpx \
            --enable-libwavpack \
            --enable-libwebp \
            --enable-libx264 \
            --enable-libx265 \
            --enable-libxvid \
            --enable-nonfree \
            --enable-openssl \
            --enable-postproc \
            --enable-pthreads \
            --enable-runtime-cpudetect \
            --enable-small \
            --enable-vdpau \
            --enable-version3 \
        && make \
        && make install \
        && make clean \
        && make distclean \
        && rm -rf "$GNUPGHOME" ${DIR} \
        && runDeps="$( \
            scanelf --needed --nobanner --recursive /usr/local \
                | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
                | sort -u \
                | xargs -r apk info --installed \
                | sort -u \
        )" \
        && apk add --virtual .php-rundeps $runDeps \
        && apk del .build-deps \
        && rm -rf /var/cache/apk/* /tmp/* \
        && ffmpeg -buildconf

CMD ["--help"]

ENTRYPOINT ["ffmpeg"]


