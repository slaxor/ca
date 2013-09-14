#!/bin/bash

BASEDIR=$(dirname $(readlink -nf $BASH_SOURCE))/..
CACERT=${BASEDIR}/certs/cacert.pem
CAKEY=${BASEDIR}/keys/cakey.pem
OPENSSL_CONF=${BASEDIR}/openssl.conf

mkca() {
  cd ${BASEDIR}
  openssl req -config ${OPENSSL_CONF} -x509 -newkey rsa:2048 -out ${CACERT} -outform PEM -keyout ${CAKEY} -keyform PEM -nodes
  chmod 600 ${CAKEY}
  cd -
}

mkcrl() {
  cd ${BASEDIR}
  openssl ca -config ${OPENSSL_CONF}  -gencrl -out crl/ca.crl
  cd -
}

mkcert() {
  set -x
  cd ${BASEDIR}
  REQ=${BASEDIR}/newcerts/${1}_certreq.pem
  KEY=${BASEDIR}/keys/${1}_key.pem
  CERT=${BASEDIR}/certs/${1}_cert.pem
  openssl req -config ${OPENSSL_CONF} -newkey rsa:1024 -nodes -keyout ${KEY}  -keyform PEM -out ${REQ} -outform PEM
  chmod 600 ${KEY}
  openssl ca -config ${OPENSSL_CONF} -cert ${CACERT} -key ${CAKEY} -in ${REQ} -out ${CERT}
  # cp ${BASEDIR}/newcerts/00.pem ${CERT}
  openssl verify -config ${OPENSSL_CONF} -CAfile ${CACERT} ${CERT}
  openssl verify -config ${OPENSSL_CONF} -CAfile ${CACERT} -verbose ${CERT}
  openssl verify -config ${OPENSSL_CONF} -CAfile ${CACERT} -issuer_checks ${CERT}
  openssl verify -config ${OPENSSL_CONF} -CAfile ${CACERT} -purpose sslserver ${CERT}
  openssl verify -config ${OPENSSL_CONF} -CAfile ${CACERT} -purpose sslserver -verbose ${CERT}
  dd if=/dev/random count=2 | openssl dhparam -rand - 512 >> ${CERT}
  cd -
  set +x
}

# generate_openssl_cnf() {
# 
# }
# 
revokecert() {
  cd ${BASEDIR}
  CERT=${BASEDIR}/certs/${1}_cert.pem
  openssl ca -config ${OPENSSL_CONF} -revoke ${CERT}
  cd -
}

ResetCA() { #capitalized because its dangerous
  cd ${BASEDIR}
  rm -f index*
  rm -f serial*
  rm -f crlnumber*
  rm -f ${BASEDIR}/certs/*
  rm -f ${BASEDIR}/keys/*
  rm -f ${BASEDIR}/newcerts/*
  rm -f ${BASEDIR}/crl/*
  echo 00 >serial
  echo 00 >crlnumber
  touch index.txt
  cd -
}

setup() {
  ln -nsf $(readlink -nf $BASH_SOURCE) ${BASEDIR}/bin/mkca
  ln -nsf $(readlink -nf $BASH_SOURCE) ${BASEDIR}/bin/mkcrl
  ln -nsf $(readlink -nf $BASH_SOURCE) ${BASEDIR}/bin/mkcert
  ln -nsf $(readlink -nf $BASH_SOURCE) ${BASEDIR}/bin/revokecert
  ln -nsf $(readlink -nf $BASH_SOURCE) ${BASEDIR}/bin/ResetCA
}

if [ $(basename ${0}) == "functions.sh" ] ; then
  setup
else
  $(basename $0) $*
fi
