FROM ubuntu:18.10 as builder

MAINTAINER "Fabio Fraga Machado"

ENV TZ=America/Sao_Paulo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update && \
apt-get install -y osmctools osm2pgsql wget curl \
build-essential && apt-get clean autoclean \
&& apt-get autoremove --yes \
&& rm -rf /var/lib/{apt,dpkg,cache,log}/

RUN mkdir /usr/src/app
WORKDIR /app

ENV GEOFBK "http://download.geofabrik.de/"
ENV MAP1 "south-america/brazil-latest.osm.pbf"
ENV MAP2 "south-america/argentina-latest.osm.pbf"
ENV MAP3 "south-america/bolivia-latest.osm.pbf"
ENV MAP4 "south-america/chile-latest.osm.pbf"
ENV MAP5 "south-america/colombia-latest.osm.pbf"
ENV MAP6 "south-america/ecuador-latest.osm.pbf"
ENV MAP7 "south-america/paraguay-latest.osm.pbf"
ENV MAP8 "south-america/peru-latest.osm.pbf"
ENV MAP9 "south-america/suriname-latest.osm.pbf"
ENV MAP10 "south-america/uruguay-latest.osm.pbf"
ENV MAP11 "south-america/venezuela-latest.osm.pbf"
ENV MAP12 "central-america/guatemala-latest.osm.pbf"
ENV MAP13 "africa/angola-latest.osm.pbf"
ENV OSM_CNV $(which osmconvert)
ENV EXT ".o5m"
ENV PBF ".pbf"
ENV CURL $(which curl)
ENV CURL_OPT "-L"
ENV DIR "/app"
ENV OSM2PG $(which osm2pgsql)
ENV USR "renderaccount"

RUN set -xe

### Stop no renderd para evitar problemas de conexoes abertas no postgres
#systemctl stop renderd

### Remocao recursiva do diretorio com os tiles ja existentes
#rm -Rfv /var/lib/mod_tile/ajt/*

### Download mapas atualizados no path correto

#cd $DIR
RUN wget $GEOFBK$MAP1
#RUN wget $GEOFBK$MAP2
#RUN wget $GEOFBK$MAP3
#RUN wget $GEOFBK$MAP4
#RUN wget $GEOFBK$MAP5
#RUN wget $GEOFBK$MAP6
#RUN wget $GEOFBK$MAP7
#RUN wget $GEOFBK$MAP8
#RUN wget $GEOFBK$MAP9
#RUN wget $GEOFBK$MAP10
#RUN wget $GEOFBK$MAP11
#RUN wget $GEOFBK$MAP12
#RUN wget $GEOFBK$MAP13
#chown $USR:$USR *.pbf

#### Merge dos mapas atualizados
RUN ls -la
# Testing only brazil
RUN $OSM_CNV brazil-latest.osm.pbf -o=brazil-latest.o5m;
#RUN $OSM_CNV argentina-latest.osm.pbf -o=argentina-latest.o5m;
#RUN $OSM_CNV bolivia-latest.osm.pbf -o=bolivia-latest.o5m;
#RUN $OSM_CNV colombia-latest.osm.pbf -o=colombia-latest.o5m;
#RUN $OSM_CNV chile-latest.osm.pbf -o=chile-latest.o5m;
#RUN $OSM_CNV ecuador-latest.osm.pbf -o=ecuador-latest.o5m;
#RUN $OSM_CNV paraguay-latest.osm.pbf -o=paraguay-latest.o5m;
#RUN $OSM_CNV peru-latest.osm.pbf -o=peru-latest.o5m;
#RUN $OSM_CNV suriname-latest.osm.pbf -o=suriname-latest.o5m;
#RUN $OSM_CNV uruguay-latest.osm.pbf -o=uruguay-latest.o5m;
#RUN $OSM_CNV venezuela-latest.osm.pbf -o=venezuela-latest.o5m;
#RUN $OSM_CNV angola-latest.osm.pbf -o=angola-latest.o5m;
#RUN $OSM_CNV guatemala-latest.osm.pbf -o=guatemala-latest.o5m;

# Testing only brazil
RUN $OSM_CNV brazil-latest.o5m -o=all_countries_iter.o5m
#RUN $OSM_CNV brazil-latest.o5m argentina-latest.o5m bolivia-latestRUN .o5m chile-latest.o5m colombia-latest.o5m ecuador-latest.o5m paraguay-latest.o5m paraguay-latest.o5m peru-latest.o5m suriname-latest.o5m uruguay-latest.o5m venezuela-latest.o5m angola-latest.o5m guatemala-latest.o5m -o=all_countries_iter.o5m

RUN $OSM_CNV all_countries_iter.o5m -o=all_countries_iter.pbf;

FROM ubuntu:xenial

COPY --from=builder /app/all_countries_iter.pbf /data/merged.osm.pbf

MAINTAINER dorukozturk <dorukozturk@kitware.com>

ENV DEBIAN_FRONTEND noninteractive
ENV LANG C.UTF-8

RUN apt-get -y update -qq && \
    apt-get -y install locales osmctools osm2pgsql wget curl build-essential && \
    locale-gen en_US.UTF-8 && \
    update-locale LANG=en_US.UTF-8 && \
    apt-get install -y build-essential cmake g++ libboost-dev libboost-system-dev \
    libboost-filesystem-dev libexpat1-dev zlib1g-dev libxml2-dev\
    libbz2-dev libpq-dev libgeos-dev libgeos++-dev libproj-dev \
    postgresql-server-dev-9.5 postgresql-9.5-postgis-2.2 postgresql-contrib-9.5 \
    apache2 php php-pgsql libapache2-mod-php php-pear php-db \
    php-intl git curl sudo \
    python-pip libboost-python-dev \
    osmosis && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/* /var/tmp/*

WORKDIR /app

# Configure postgres
RUN echo "host all  all    0.0.0.0/0  trust" >> /etc/postgresql/9.5/main/pg_hba.conf && \
    echo "listen_addresses='*'" >> /etc/postgresql/9.5/main/postgresql.conf

# Nominatim install
ENV NOMINATIM_VERSION v3.1.0
RUN git clone https://github.com/openstreetmap/Nominatim ./src
RUN cd ./src && git checkout tags/$NOMINATIM_VERSION && git submodule update --recursive --init && \
    mkdir build && cd build && cmake .. && make

# Osmium install to run continuous updates
RUN pip install osmium

# Apache configure
COPY local.php /app/src/build/settings/local.php
COPY nominatim.conf /etc/apache2/sites-enabled/000-default.conf

# Load initial data
#RUN curl http://www.nominatim.org/data/country_grid.sql.gz > /app/src/data/country_osm_grid.sql.gz

EXPOSE 5432
EXPOSE 8080

COPY start.sh /app/start.sh
COPY startapache.sh /app/startapache.sh
COPY startpostgres.sh /app/startpostgres.sh
COPY init.sh /app/init.sh


