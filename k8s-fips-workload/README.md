# K8s FIPS enabled container workload 

### Microk8s registry
```bash
microk8s.enable registry
REGISTRY=localhost:32000
```

### Ubuntu FIPS container image
- https://github.com/canonical/ubuntu-advantage-client/blob/main/docs/tutorials/create_a_fips_docker_image.md

```bash
cp ua-attach-config.template.yaml ua-attach-config.yaml
# Add UA TOKEN to ua-attach-config.yaml

sudo DOCKER_BUILDKIT=1 docker build . --secret id=ua-attach-config,src=ua-attach-config.yaml -t ubuntu-focal-fips
sudo DOCKER_BUILDKIT=1 docker build . --secret id=ua-attach-config,src=ua-attach-config.yaml -t $REGISTRY/ubuntu-focal-fips
```

Testing FIPS binaries from Ubuntu
```bash
UFF="sudo docker container run -it ubuntu-focal-fips"
$UFF cat /proc/sys/crypto/fips_enabled # Should be 1
$UFF sysctl crypto.fips_enabled        # Should be 1 
$UFF openssl md5 /dev/null             # Should be disabled for fips
$UFF dpkg-query --show openssl         # Should include fips in version name
```

### NGINX image based on Ubuntu FIPS [local image]
```bash
cd nginx
sudo docker build . -t $REGISTRY/nginx-fips
```

List NGINX ciphers
* [FIPS mode and TLS](https://wiki.openssl.org/index.php/FIPS_mode_and_TLS)
* [Ciphers naming format mapping](https://testssl.sh/openssl-iana.mapping.html)

```bash
sudo docker run -it -d -p 8443:443 $REGISTRY/nginx-fips:latest
nmap -sV --script ssl-enum-ciphers -p 8443 localhost | grep TLS_ > /tmp/all_ciphers.out
```

### Deploy to K8s
```bash
sudo docker push $REGISTRY/ubuntu-focal-fips
sudo docker push $REGISTRY/nginx-fips
microk8s.kubectl apply -f deployment.yaml
```

Testing ciphers with NGINX
* [NGINX FIPS compliance](https://docs.nginx.com/nginx/fips-compliance-nginx-plus/)

```bash
PORT=31045
# test if the container is accepting connections
(echo "GET /" ; sleep 1) | openssl s_client -connect localhost:$PORT
# this command should work and output the default nginx homepage

# ensure RC4-MD5 is disabled
(echo "GET /" ; sleep 1) | openssl s_client -connect localhost:$PORT -cipher RC4-MD5
# expected output:
# Error with command: "-cipher RC4-MD5"
# 140202641934240:error:1410D0B9:SSL routines:SSL_CTX_set_cipher_list:no cipher match:ssl_lib.c:1383:

# ensure CAMELLIA256-SHA is disabled
(echo "GET /" ; sleep 1) | openssl s_client -connect localhost:$PORT -cipher CAMELLIA256-SHA
# expected output:
# Error with command: "-cipher CAMELLIA256-SHA"
# 140211194984352:error:1410D0B9:SSL routines:SSL_CTX_set_cipher_list:no cipher match:ssl_lib.c:1383:

# ensure AES256-SHA is enabled
(echo "GET /" ; sleep 1) | openssl s_client -connect localhost:$PORT -cipher AES256-SHA
# this command should work and output the default nginx homepage
```
