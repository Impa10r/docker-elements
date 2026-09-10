FROM debian:trixie-slim AS builder

# VERSION of Elements Core to be download
ARG VERSION=23.3.4
ARG TARGETPLATFORM

ENV BYRON_PGP_KEY=710E44C2DAAE938F778744A1DE8F6EA20A661697
ENV PABLO_PGP_KEY=BD0F3062F87842410B06A0432F656B0610604482

RUN set -ex \
  && if [ "${TARGETPLATFORM}" = "linux/amd64" ]; then export TARGETPLATFORM=x86_64-linux-gnu; fi \
  && if [ "${TARGETPLATFORM}" = "linux/arm64" ]; then export TARGETPLATFORM=aarch64-linux-gnu; fi \
	&& apt-get update \
	&& apt-get install -qq --no-install-recommends ca-certificates wget gnupg \
	&& cd /tmp \
	&& wget -q "https://github.com/ElementsProject/elements/releases/download/elements-$VERSION/elements-$VERSION-$TARGETPLATFORM.tar.gz" \
	&& wget -qO SHA256SUMS.asc "https://github.com/ElementsProject/elements/releases/download/elements-$VERSION/SHA256SUMS.asc" \
	&& gpg --keyserver hkps://keys.openpgp.org --recv-keys "$BYRON_PGP_KEY" "$PABLO_PGP_KEY" \
	&& gpg --output SHA256SUMS --decrypt SHA256SUMS.asc \
	&& grep "elements-$VERSION-$TARGETPLATFORM.tar.gz" SHA256SUMS | sha256sum -c \
	&& mkdir bin \
	&& tar -xzvf "elements-$VERSION-$TARGETPLATFORM.tar.gz" -C /tmp/bin --strip-components=2 "elements-$VERSION/bin/elements-cli" "elements-$VERSION/bin/elementsd"

FROM debian:trixie-slim

# $USER name, and data $DIR to be used in the `final` image
ARG USER=elements
ARG DIR=/home/elements

COPY --from=builder "/tmp/bin" /usr/local/bin

# NOTE: Default GID == UID == 1000
RUN useradd -M \
            -d "$DIR/" \
            -s /usr/sbin/nologin \
            -u 1000 \
            -U \
            -c "" \
            "$USER" && \
			mkdir -p "$DIR" && \
			chown -R "$USER:$USER" "$DIR/"

USER $USER

# Prevents `VOLUME $DIR/.elements/` being created as owned by `root`
RUN mkdir -p "$DIR/.elements/"

# Expose volume containing all `elementsd` data
VOLUME $DIR/.elements/


ENTRYPOINT [ "elementsd" ]


