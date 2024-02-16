# P4P Docker
# Can't run on Alpine, needs Debian
FROM debian:stable-slim

ARG P4P_DOWNLOAD=https://www.perforce.com/downloads/perforce/r23.2/bin.linux26x86_64/helix-core-server.tgz

ENV P4PORT=0.0.0.0:1666
ENV P4PCACHE=/p4pcache
ENV P4SSLDIR=/p4ssldir
ENV P4TARGET=0.0.0.0:1666
ENV P4PFSIZE=1000000
ENV P4DEBUG=0

# Install curl and other necessary utilities
RUN apt-get update && apt-get install -y \
    curl \
    tar \
    && rm -rf /var/lib/apt/lists/*

# Install p4p
RUN mkdir -p /opt/p4p \
    && curl -sL -o /opt/p4p/p4p.tar.gz $P4P_DOWNLOAD \
    && tar -xf /opt/p4p/p4p.tar.gz -C /opt/p4p && rm /opt/p4p/p4p.tar.gz \
    && ln -s /opt/p4p/p4p /usr/local/bin/p4p \
    && p4p -V

# Create the startup script
RUN echo '#!/bin/bash\n\
if [ -z "$(ls -A $P4SSLDIR)" ]; then\n\
   echo "The $P4SSLDIR directory is empty. Running first-time command..."\n\
   p4p -Gc\n\
fi\n\
\n\
# Start p4p\n\
p4p -S' > /start-p4p.sh && chmod +x /start-p4p.sh

CMD ["/bin/bash", "/start-p4p.sh"]