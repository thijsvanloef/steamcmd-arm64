############################################################
# Dockerfile that contains SteamCMD and Box86/64
############################################################

FROM arm64v8/debian:bookworm-slim as build_stage

LABEL maintainer="github@snry.me"
ARG PUID=1000

ENV USER steam
ENV HOMEDIR "/home/${USER}"
ENV STEAMCMDDIR "${HOMEDIR}/steamcmd"

ENV DEBUGGER "/usr/local/bin/box86"

# Set SHELL option explicitly
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# hadolint ignore=DL3008
RUN set -x \
	# Install, update & upgrade packages
	&& dpkg --add-architecture armhf \
	&& apt-get update \
 	&& apt-get install -y --no-install-recommends --no-install-suggests \
		libc6:armhf=2.36-9+deb12u4 \
		libstdc++6:armhf=12.2.0-14 \
		ca-certificates=20230311 \
		nano=7.2-1 \
		curl=7.88.1-10+deb12u5 \
		locales=2.36-9+deb12u4 \
  		wget=1.21.3-1+b1 \
		gnupg=2.2.40-1.1 \
	&& wget --progress=dot:giga https://ryanfortner.github.io/box64-debs/box64.list -O /etc/apt/sources.list.d/box64.list \
	&& (wget -qO- https://ryanfortner.github.io/box64-debs/KEY.gpg | gpg --dearmor -o /etc/apt/trusted.gpg.d/box64-debs-archive-keyring.gpg) \
	&& wget --progress=dot:giga https://itai-nelken.github.io/weekly-box86-debs/debian/box86.list -O /etc/apt/sources.list.d/box86.list \
	&& (wget -qO- https://itai-nelken.github.io/weekly-box86-debs/debian/KEY.gpg | gpg --dearmor -o /etc/apt/trusted.gpg.d/box86-debs-archive-keyring.gpg) \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
		box64-arm64 \
		box86 \
	&& sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
	&& dpkg-reconfigure --frontend=noninteractive locales \
	# Create unprivileged user
	&& useradd -l -u "${PUID}" -m "${USER}" \
        # Download SteamCMD, execute as user
        && su "${USER}" -c \
		"mkdir -p \"${STEAMCMDDIR}\" \
                && curl -fsSL 'https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz' | tar xvzf - -C \"${STEAMCMDDIR}\" \
                && ${STEAMCMDDIR}/steamcmd.sh +quit \
                && ln -s \"${STEAMCMDDIR}/linux32/steamclient.so\" \"${STEAMCMDDIR}/steamservice.so\" \
                && mkdir -p \"${HOMEDIR}/.steam/sdk32\" \
                && ln -s \"${STEAMCMDDIR}/linux32/steamclient.so\" \"${HOMEDIR}/.steam/sdk32/steamclient.so\" \
                && ln -s \"${STEAMCMDDIR}/linux32/steamcmd\" \"${STEAMCMDDIR}/linux32/steam\" \
                && mkdir -p \"${HOMEDIR}/.steam/sdk64\" \
                && ln -s \"${STEAMCMDDIR}/linux64/steamclient.so\" \"${HOMEDIR}/.steam/sdk64/steamclient.so\" \
                && ln -s \"${STEAMCMDDIR}/linux64/steamcmd\" \"${STEAMCMDDIR}/linux64/steam\" \
                && ln -s \"${STEAMCMDDIR}/steamcmd.sh\" \"${STEAMCMDDIR}/steam.sh\"" \
	# Symlink steamclient.so; So misconfigured dedicated servers can find it
 	&& ln -s "${STEAMCMDDIR}/linux64/steamclient.so" "/usr/lib/x86_64-linux-gnu/steamclient.so" \
	&& rm -rf /var/lib/apt/lists/*

FROM build_stage AS bookworm-root
WORKDIR ${STEAMCMDDIR}

FROM bookworm-root AS bookworm
# Switch to user
USER ${USER}
