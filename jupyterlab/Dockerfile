FROM python:3.7-alpine

RUN ["mkdir", "/home/test"]
WORKDIR /home/test

RUN apk add --no-cache --upgrade bash

RUN echo "http://mirror.leaseweb.com/alpine/edge/community" >> /etc/apk/repositories
RUN echo "http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories
RUN apk add --virtual .build-deps \
        --repository http://dl-cdn.alpinelinux.org/alpine/edge/community \
        --repository http://dl-cdn.alpinelinux.org/alpine/edge/main \
        gcc libc-dev geos-dev geos && \
    runDeps="$(scanelf --needed --nobanner --recursive /usr/local \
    | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
    | xargs -r apk info --installed \
    | sort -u)" && \
    apk add --virtual .rundeps $runDeps
RUN ["pip", "install", "shapely"]

RUN apk add libffi-dev

RUN apk add --update alpine-sdk nodejs ca-certificates musl-dev gcc make cmake g++ gfortran libpng-dev freetype-dev libxml2-dev libxslt-dev
RUN apk add --update git

# Install Jupyter
RUN pip install jupyter
RUN pip install ipywidgets
RUN jupyter nbextension enable --py widgetsnbextension

RUN pip install jupyterlab && jupyter serverextension enable --py jupyterlab

ENV LANG=C.UTF-8

# Install Python Packages & Requirements (Done near end to avoid invalidating cache)
# COPY requirements.txt requirements.txt
# RUN pip install -r requirements.txt
RUN pip install cython numpy pandas
# RUN pip install matplotlib
RUN pip install lxml
RUN pip install folium
RUN pip install nltk

RUN mkdir gdal && cd gdal && \
    wget -O gdal.tar.gz "https://download.osgeo.org/gdal/2.4.2/gdal-2.4.2.tar.gz" && \
    tar --extract --file gdal.tar.gz --strip-components 1 && \
    ./configure && make && make install

RUN cd ..

RUN pip install rasterio

RUN apk add unzip tiff-dev curl-dev 

ARG PROJ_DATUMGRID_LATEST_LAST_MODIFIED
RUN \
    mkdir -p /build_projgrids/usr/share/proj \
    && curl -LOs http://download.osgeo.org/proj/proj-datumgrid-latest.zip \
    && unzip -q -j -u -o proj-datumgrid-latest.zip  -d /build_projgrids/usr/share/proj \
    && rm -f *.zip

RUN apk add sqlite sqlite-dev

ARG PROJ_VERSION=master
RUN mkdir proj \
    && apk add --no-cache cmake \
    && wget -q https://github.com/OSGeo/PROJ/archive/${PROJ_VERSION}.tar.gz -O - \
        | tar xz -C proj --strip-components=1 \
    && cd proj \
    && cmake . \
        -DBUILD_SHARED_LIBS=ON \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DENABLE_IPO=ON \
        -DBUILD_TESTING=OFF \
    && make install \
    && make install DESTDIR="/build_proj" \
    && cd .. \
    && rm -rf proj \
    && for i in /build_proj/usr/lib/*; do strip -s $i 2>/dev/null || /bin/true; done \
    && for i in /build_proj/usr/bin/*; do strip -s $i 2>/dev/null || /bin/true; done \
    && apk del cmake

RUN pip install geopandas
RUN pip install matplotlib


RUN apk add cmake
RUN wget http://download.osgeo.org/geos/geos-3.10.2.tar.bz2 \
    && tar xvfj geos-3.10.2.tar.bz2 \
    && cd geos-3.10.2 \
    && cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    && make \
    && make install

RUN pip install geopy contextily

RUN pip install pygeos ipyleaflet faker

RUN jupyter nbextension enable --py --sys-prefix ipyleaflet

RUN apk add gfortran openblas-dev lapack-dev --repository http://dl-3.alpinelinux.org/alpine/edge/testing/ --allow-untrusted

RUN pip install scipy

# RUN sklearn

RUN pip install phonenumbers==8.12.5

RUN pip install twilio==6.42.0

# Expose Jupyter port & cmd
EXPOSE 8888
RUN mkdir -p /opt/app/data
CMD jupyter lab --ip=* --port=8888 --no-browser --notebook-dir=/opt/app/data --allow-root
