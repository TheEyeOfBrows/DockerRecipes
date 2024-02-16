# This image requires the NET_ADMIN capability, and net.ipv4.ip_forward=1 to be set on the host
# ex.
#   cap_add:
#       - NET_ADMIN
#   sysctls:
#       - net.ipv4.ip_forward=1

# Use a base image with iptables
FROM alpine:latest

# Update and install iptables
RUN apk update && apk add iptables

# Set the RULES environment variable
# RULES is a comma-separated list of triplets, each of which is a colon-separated list of a source port, a destination host, and a destination port
ENV RULES="\
1666    :   10.0.0.1   :  1666,\
1667    :   10.0.0.1   :  1666,\
1668    :   10.0.0.1   :  1666"

# Create the script to set up iptables
RUN echo -e "#!/bin/sh\n\
# Get the non-loopback IP address of the container\n\
IP=\$(ip -o -4 addr list eth0 | awk '{print \$4}' | cut -d/ -f1)\n\
# Flush iptables\n\
iptables -F\n\
# Cleanup rules whitespace\n\
LRULES=\"\$(echo \"\$RULES\" | tr -d ' ')\"\n\
# Set up NAT for each triplet in the RULES environment variable\n\
for RULE in \$(echo \"\$LRULES\" | tr \",\" \"\\\\n\"); do\n\
  SOURCE_PORT=\$(echo \$RULE | cut -d: -f1)\n\
  DEST_HOST=\$(echo \$RULE | cut -d: -f2)\n\
  DEST_PORT=\$(echo \$RULE | cut -d: -f3)\n\
  echo \"Setting up iptables for \$SOURCE_PORT -> \$DEST_HOST:\$DEST_PORT\"\n\
  iptables -t nat -A PREROUTING -p tcp --dport \$SOURCE_PORT -j DNAT --to-destination \$DEST_HOST:\$DEST_PORT\n\
  iptables -t nat -A POSTROUTING -p tcp --dport \$SOURCE_PORT -j SNAT --to-source \$IP\n\
done\n\
# Keep the container running\n\
echo 'Iptables set up. Keeping the container running...'\n\
tail -f /dev/null" > /setup-iptables.sh

# Make the script executable
RUN chmod +x /setup-iptables.sh

# Run the script when the container starts
CMD ["/setup-iptables.sh"]