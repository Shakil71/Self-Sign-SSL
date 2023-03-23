# Self-Sign-SSL


echo "\n\n________________GENERATING ALL DIRECTORIES________________\n\n"
gr='\033[1;32m'
nc='\033[0m' # No Color

mkdir -p {root-ca,sub-ca,server}/{private,certs,index,serial,pem,crl,csr}
mkdir generated
touch root-ca/index/index
touch sub-ca/index/index
openssl rand -hex 16 > root-ca/serial/serial
openssl rand -hex 16 > sub-ca/serial/serial
cp root-ca.conf root-ca
cp sub-ca.conf sub-ca
echo "${gr}\n ================ FOLDERS CREATED SUCCESSFULLY ================ \n${nc}"

echo "\n\n________________GENERATING ALL THE KEYS________________\n\n"

openssl genrsa -aes256 -out root-ca/private/ca.key 4096
openssl genrsa -aes256 -out sub-ca/private/sub-ca.key 4096
openssl genrsa -out server/private/server.key 2048
echo "${gr}\n ================ KEYS CREATED SUCCESSFULLY ================ \n${nc}"

echo "\n\n________________GENERATING ROOT CERTIFICATE________________\n\n"

openssl req -config root-ca/root-ca.conf -key root-ca/private/ca.key -new -x509 -days 7305 -sha256 -extensions v3_ca -out root-ca/certs/ca.crt
echo "${gr}\n ================ ROOT CERTIFICATE CREATED SUCCESSFULLY ================ \n${nc}"

echo "\n\n________________GENERATING SUB-ROOT REQUEST________________\n\n"

openssl req -config sub-ca/sub-ca.conf -new -key sub-ca/private/sub-ca.key -sha256 -out sub-ca/csr/sub-ca.csr
echo "${gr}\n ================ SUB-ROOT REQUEST CREATED SUCCESSFULLY  ================ \n${nc}"

echo "\n\n________________GENERATING SUB-ROOT CERTIFICATE________________\n\n"

openssl ca -config root-ca/root-ca.conf -extensions v3_intermediate_ca -days 3652 -notext -in sub-ca/csr/sub-ca.csr -out sub-ca/certs/sub-ca.crt
echo "${gr}\n ================ SUB-ROOT CERTIFICATE CREATED SUCCESSFULLY ================ \n${nc}"

echo "\n\n________________GENERATING SERVER REQUEST________________\n\n"

openssl req -key server/private/server.key -new -sha256 -out server/csr/server.csr
echo "${gr}\n ================ SERVER REQUEST CREATED SUCCESSFULLY ================ \n${nc}"

echo "\n\n________________GENERATING SERVER CERTIFICATE________________\n\n"

openssl ca -config sub-ca/sub-ca.conf -extensions server_cert -days 365 -notext -in server/csr/server.csr -out server/certs/server.crt
openssl pkcs12 -inkey server/private/server.key -in server/certs/server.crt -export -out server/certs/server.pfx
echo "${gr}\n ================ SERVER CERTIFICATE CREATED SUCCESSFULLY ================ \n${nc}"

echo "\n\n________________GATHERING NECESSARY FILES________________\n\n"

cp root-ca/certs/ca.crt generated
cp sub-ca/certs/sub-ca.crt generated
cp server/certs/server.crt generated
cp server/private/server.key generated
cp server/certs/server.pfx generated
echo "${gr}\n ================ SUCCESSFULLY GATHERED ================ \n${nc}"

echo "\n\n________________CREATING HOST ENTRY________________\n\n"

echo -n "Server CommonName: "
read commonName
echo "127.0.0.1 "$commonName >> /etc/hosts

echo "${gr}\n ================ SUCCESSFULLY APPENDED HOST ================ \n${nc}"

