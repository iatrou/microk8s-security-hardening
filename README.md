# Microk8s Security Hardened Host

### Open bugs
* [Microk8s: Microk8s being tested with Sonobuoy fails #3096](https://github.com/canonical/microk8s/issues/3096)
* [USG: Failing disa_stig tailored file audit and fix #7](https://github.com/canonical/ubuntu-security-guide/issues/7)

### Install Microk8s
Follow [official guide](https://microk8s.io/docs/getting-started)

Enable DNS and Storage plugins
```bash
microk8s enable dns storage
```

## Security Hardening
[Ubuntu security certs docs](https://ubuntu.com/security/certifications/docs)

Ubuntu Advantange (UA) Suscription

```bash
sudo apt install ubuntu-advantage-tools
sudo ua attach $TOKEN
```

> Get your token in https://ubuntu.com/advantage


Ubuntu Security Guide (USG) install
```bash
sudo ua enable usg
sudo apt install usg
```

### For CIS compliance
* [Canonical blog on CIS compliance](https://ubuntu.com/blog/cis-security-compliance-usg)
* [Canonical tutorial on CIS or DISA STIG compliance](https://ubuntu.com/tutorials/comply-with-cis-or-disa-stig-on-ubuntu#1-overview)
* [Audit docs](https://ubuntu.com/security/certifications/docs/usg/cis/audit)
* [Compliance docs](https://ubuntu.com/security/certifications/docs/usg/cis/compliance)

### For DISA STIG compliance
* [Audit docs](https://ubuntu.com/security/certifications/docs/disa-stig/audit)
* [Compliance docs](https://ubuntu.com/security/certifications/docs/disa-stig/compliance)

Check FIPS binaries after restarting host
```bash
cat /proc/sys/crypto/fips_enabled # 1
sysctl crypto.fips_enabled # 1
openssl md5 /dev/null # Disabled by fips
dpkg-query --show openssl # fips included in version name
```

## Testing Microk8s with Sonobouy
* [Sonobouy official docs](https://github.com/vmware-tanzu/sonobuoy)

Install Sonobouy
```bash
URL=https://github.com/vmware-tanzu/sonobuoy/releases/download/v0.56.4/sonobuoy_0.56.4_linux_amd64.tar.gz
curl -LJ $URL -o sonobuoy.tar.gz 
tar -xvf sonobuoy.tar.gz
rm LICENSE

sudo mv sonobuoy /usr/local/sbin
```

Run Sonobouy tests
```bash
byobu

# Set kubectl config
microk8s config > ~/.kube/config
export KUBECONFIG=~/.kube/config

# Full test takes around one hour
sonobuoy run --wait # [--mode quick]
RES=$(sonobuoy retrieve) && sonobuoy results $RES
sonobuoy delete --wait
```

## Adding FIPS enabled workloads
-> [K8s FIPS enabled container workload](./k8s-fips-workload/README.md)


