#Docker file for setting up the acme peer
# FROM hyperledger/fabric-peer:1.4.2
FROM hyperledger/fabric-peer:2.1

LABEL  maintainer="Rajeev Sakhuja <raj@acloudfan.com>"

#1. Create a folder
RUN mkdir -p /var/hyperledger/config

#2. [PERMISSION DENIED REMEDIATION] Adding user permissions on specific directory 
# RUN chgrp -R 0 /var/hyperledger/config && \
#     chmod -R g=u /var/hyperledger/config

#3. Copy the crypto for peer crypto
COPY ./config/crypto-config/peerOrganizations/acme.com/peers/peer1.acme.com /var/hyperledger/peer1.acme.com

#4. Copy the crypto for admin crypto
COPY ./config/crypto-config/peerOrganizations/acme.com/users /var/hyperledger/users

#5. Copy the anchor peer update tx
COPY ./config/acme-peer-update.tx /var/hyperledger/config/acme-peer-update.tx

#6. Copy the channel create tx file
COPY ./config/airlinechannel.tx  /var/hyperledger/config/airlinechannel.tx

#7. Copy the core YAML
COPY ./config/acme/core.yaml /var/hyperledger/config/core.yaml

#8. Copy the bins
COPY ./bins /var/hyperledger/bins

#9. Copy the test chaincode
COPY ./nodechaincode  /var/hyperledger/nodechaincode

#10. Set the working dir
# RUN  echo "cd /var/hyperledger/bins" >> $HOME/.profile
# RUN  echo "cd /var/hyperledger/bins" >> $HOME/.bashrc
ENV  ENV="/etc/profile"
RUN  echo "cd /var/hyperledger/bins" >> /etc/profile

#11. Set the context for Admin user
# RUN echo ". /var/hyperledger/bins/set-context.sh" >> $HOME/.bashrc
RUN echo ". /var/hyperledger/bins/set-context.sh " >> /etc/profile

#12. Install the jq package - used in scripts
RUN apk update \
    && apk add jq \
    && rm -rf /var/cache/apk/*

#13. Create the package folder
RUN mkdir -p /var/hyperledger/packages

#14. Copy the gocc package tar file to the image
#COPY ./packages/gocc.1.0-1.0.tar.gz /var/hyperledger/packages/gocc.1.0-1.0.tar.gz

#15. Launch the peer
CMD peer node start
