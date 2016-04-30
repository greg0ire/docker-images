FROM alpine:3.3

MAINTAINER Arnaud Chevillard <achevillard@universcine.com>

ENV GPG_KEYS FCF986EA15E6E293A5644F10B4322F04D67658D8

ENV FFMPEG_VERSION=3.0.2 \
    FFMPEG_FILENAME=ffmpeg-3.0.2.tar.xz \
    FDKAAC_VERSION=0.1.4 \ 
    FDKAAC_FILENAME=fdk-aac-0.1.4.tar.gz \
    PKG_CONFIG_PATH=/usr/local/lib/pkgconfig \
    SRC=/usr/local

WORKDIR /tmp

RUN set -xe \
        && apk add --no-cache --virtual .build-deps \
            build-base \
            curl \
            autoconf \
            automake \
            libtool \
            make \
            openssl-dev \
            pcre-dev \
            gnupg \
            yasm-dev \
            lame-dev \
            libogg-dev \
            x264-dev \
            libvpx-dev \
            libvorbis-dev \
            x265-dev \
            freetype-dev \
            libass-dev \
            libwebp-dev \
            rtmpdump-dev \
            libtheora-dev \
            opus-dev \
            faac-dev \
            nasm \
            speex-dev \
            xvidcore-dev \

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
        && tar -Jxf "$FFMPEG_FILENAME" \
        && cd ffmpeg-${FFMPEG_VERSION} \
        && ./configure \
            --prefix="${SRC}" \
            --extra-cflags="-I${SRC}/include" \
            --extra-ldflags="-L${SRC}/lib" \
            --extra-libs=-ldl \
            --bindir="${SRC}/bin" \
            --disable-doc \
            --enable-pthreads \
            --enable-gpl \
            --enable-version3 \
            --disable-shared \
            --enable-hardcoded-tables \
            --enable-avresample \
            --enable-nonfree \
            --enable-libx264 \
            --enable-libx265 \
            --enable-libmp3lame \
            --enable-libxvid \
            --enable-libtheora \
            --enable-libvorbis \
            --enable-libfaac \
            --enable-ffplay \
            --enable-libspeex \
            --enable-libopus \
            --enable-librtmp \
            --enable-libfdk-aac \
            --enable-libvpx \
            --enable-avresample \
            --enable-libwebp \
            --enable-small \
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
        && rm -rf /var/cache/apk/* /tmp/*

CMD ["--help"]

ENTRYPOINT ["ffmpeg"]


