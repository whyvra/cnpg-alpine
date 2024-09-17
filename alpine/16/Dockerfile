ARG PG_BASE_IMAGE=16.4-alpine

FROM postgres:$PG_BASE_IMAGE

ARG PG_AUDIT_VERSION=REL_16_STABLE \
    PG_FAILOVER_VERSION=v1.1.0 \
    PGVECTOR_VERSION=v0.7.4 \
    PG_BARMAN_VERSION=3.11.1

# Install build dependencies
RUN apk add -U --no-cache -t .build-deps1 \
		$DOCKER_PG_LLVM_DEPS \
		git \
		build-base \
		openssl-dev \
		krb5-dev \
	\
	# Build and install pgaudit
	&& mkdir /pgaudit && cd /pgaudit \
	&& git clone https://github.com/pgaudit/pgaudit --branch $PG_AUDIT_VERSION . \
	&& make install USE_PGXS=1 PG_CONFIG=/usr/local/bin/pg_config \
	# Build and install pg_failover_slots
	&& mkdir /pg_failover_slots && cd /pg_failover_slots \
	&& git clone https://github.com/EnterpriseDB/pg_failover_slots --branch $PG_FAILOVER_VERSION . \
	&& make install \
	# Build and install pgvector
	&& mkdir /pgvector && cd /pgvector \
	&& git clone https://github.com/pgvector/pgvector --branch $PGVECTOR_VERSION . \
	&& make install OPTFLAGS="" \
	# Install barman dependencies
	&& mkdir /barman && cd /barman \
	&& apk add -U --no-cache -t .run-deps \
		python3 \
		rsync \
		py3-argcomplete \
		py3-dateutil \
		py3-psycopg2 \
		py3-boto3 \
	&& apk add -U --no-cache -t .build-deps2 \
		py3-gpep517 \
		py3-setuptools \
		py3-wheel \
	# Configure barman user
	&& addgroup -S barman \
	&& adduser -SD -h /var/lib/barman/ -s /sbin/nologin -G barman -g barman barman \
	# Install barman
	&& git clone https://github.com/EnterpriseDB/barman --branch release/$PG_BARMAN_VERSION . \
	&& gpep517 build-wheel \
		--wheel-dir .dist \
		--output-fd 3 3>&1 >&2 \
	&& python3 -m installer -d "/" .dist/*.whl \
	&& cp doc/barman.conf /etc \
	# Remove source files and build deps
	&& cd / \
	&& rm -rf /pgaudit /pg_failover_slots /pgvector /barman \
	&& apk del .build-deps1 .build-deps2 \
	# Change the uid of postgres to 26
	&& apk add --no-cache shadow \
	&& usermod -u 26 postgres \
	&& apk del shadow

USER 26

LABEL name="PostgreSQL Container Images" \
    vendor="The CloudNativePG Contributors" \
    version="16.4" \
    summary="PostgreSQL Container images" \
    description="Container image with PostgreSQL 16.4 and Barman Cloud running on Alpine."

LABEL org.opencontainers.image.description="Container image with PostgreSQL 16.4 and Barman Cloud running on Alpine."
