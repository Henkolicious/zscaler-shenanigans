# ZScaler and Docker Shenanigans

# Notes
- Only tested for WSL Ubuntu 22.04.5 LTS
- FAQ
  - The proxy address is to some ZScaler service on your host

## TL;DR

1. Setup the http proxy environment variables in WSL `.bashrc`, lower case and capital case
1.1 Proxy: `http://127.0.0.1:9000`
1.2 `$ source .bashrc`
2. Add the Zscaler CA certificate to the cert store
3. Add proxy URL to docker desktop
4. Enable `Add the *.docker.internal names to the host's /etc/hosts file (Requires password)` checkbox in `Settings > General`
5. Disable ipv6 in docker daemon (missing by default)
6. Restart docker desktop (quit) and WSL

## Guide

1. Setup the http proxy environment variables in WSL `.bashrc`, lower case and capital case
- From your CLI, open WSL by `$ wsl`
- Edit your `.bashrc` and add these proxy settings
  - Sample `$ sudo vim .bashrc`
```
export HTTP_PROXY="http://127.0.0.1:9000"
export http_proxy="http://127.0.0.1:9000"
export HTTPS_PROXY="http://127.0.0.1:9000"
export https_proxy="http://127.0.0.1:9000"
```
- Execute reload of `.bashrc` by `$ source .bashrc`
- Verify that your environment variables are set by `$ env | grep -i proxy`
![Environment variables](/assets/envvars.png "Environment variables")

2. Add the Zscaler CA certificate to the cert store
- `$ sudo vim /usr/local/share/ca-certificates/zscalert.crt`
- Most likley this certificate, or extract your own from your PC
```
-----BEGIN CERTIFICATE-----
MIIE0zCCA7ugAwIBAgIJANu+mC2Jt3uTMA0GCSqGSIb3DQEBCwUAMIGhMQswCQYD
VQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTERMA8GA1UEBxMIU2FuIEpvc2Ux
FTATBgNVBAoTDFpzY2FsZXIgSW5jLjEVMBMGA1UECxMMWnNjYWxlciBJbmMuMRgw
FgYDVQQDEw9ac2NhbGVyIFJvb3QgQ0ExIjAgBgkqhkiG9w0BCQEWE3N1cHBvcnRA
enNjYWxlci5jb20wHhcNMTQxMjE5MDAyNzU1WhcNNDIwNTA2MDAyNzU1WjCBoTEL
MAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExETAPBgNVBAcTCFNhbiBK
b3NlMRUwEwYDVQQKEwxac2NhbGVyIEluYy4xFTATBgNVBAsTDFpzY2FsZXIgSW5j
LjEYMBYGA1UEAxMPWnNjYWxlciBSb290IENBMSIwIAYJKoZIhvcNAQkBFhNzdXBw
b3J0QHpzY2FsZXIuY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA
qT7STSxZRTgEFFf6doHajSc1vk5jmzmM6BWuOo044EsaTc9eVEV/HjH/1DWzZtcr
fTj+ni205apMTlKBW3UYR+lyLHQ9FoZiDXYXK8poKSV5+Tm0Vls/5Kb8mkhVVqv7
LgYEmvEY7HPY+i1nEGZCa46ZXCOohJ0mBEtB9JVlpDIO+nN0hUMAYYdZ1KZWCMNf
5J/aTZiShsorN2A38iSOhdd+mcRM4iNL3gsLu99XhKnRqKoHeH83lVdfu1XBeoQz
z5V6gA3kbRvhDwoIlTBeMa5l4yRdJAfdpkbFzqiwSgNdhbxTHnYYorDzKfr2rEFM
dsMU0DHdeAZf711+1CunuQIDAQABo4IBCjCCAQYwHQYDVR0OBBYEFLm33UrNww4M
hp1d3+wcBGnFTpjfMIHWBgNVHSMEgc4wgcuAFLm33UrNww4Mhp1d3+wcBGnFTpjf
oYGnpIGkMIGhMQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTERMA8G
A1UEBxMIU2FuIEpvc2UxFTATBgNVBAoTDFpzY2FsZXIgSW5jLjEVMBMGA1UECxMM
WnNjYWxlciBJbmMuMRgwFgYDVQQDEw9ac2NhbGVyIFJvb3QgQ0ExIjAgBgkqhkiG
9w0BCQEWE3N1cHBvcnRAenNjYWxlci5jb22CCQDbvpgtibd7kzAMBgNVHRMEBTAD
AQH/MA0GCSqGSIb3DQEBCwUAA4IBAQAw0NdJh8w3NsJu4KHuVZUrmZgIohnTm0j+
RTmYQ9IKA/pvxAcA6K1i/LO+Bt+tCX+C0yxqB8qzuo+4vAzoY5JEBhyhBhf1uK+P
/WVWFZN/+hTgpSbZgzUEnWQG2gOVd24msex+0Sr7hyr9vn6OueH+jj+vCMiAm5+u
kd7lLvJsBu3AO3jGWVLyPkS3i6Gf+rwAp1OsRrv3WnbkYcFf9xjuaf4z0hRCrLN2
xFNjavxrHmsH8jPHVvgc1VD0Opja0l/BRVauTrUaoW6tE+wFG5rEcPGS80jjHK4S
pB5iDj2mUZH1T8lzYtuZy0ZPirxmtsk3135+CKNa2OCAhhFjE0xd
-----END CERTIFICATE-----
```
- Update the certificate store `$ update-ca-certificates`
- Now you should have the ZScaler certificate in `/etc/ssl/certs/zscaler.pem`

3. Add proxy URL to docker desktop
- In your docker desktop application, open `settings > resources > proxies` and add the proxy URL for HTTP and HTTPS
![Proxies in docker desktop](/assets/proxies.png "Proxies in docker desktop")

4. etc host files for WSL
- Open docker desktop and navigate to `settings > general`
- Enable `Add the *.docker.internal names to the host's /etc/hosts file (Requires password)` checkbox
- Apply and restart
![Hosts](/assets/hosts.png "Hosts")

5. Disable ipv6 in docker daemon (missing by default)
- create this folder and add daemon.json
- `$ mkdir /etc/docker`
- `$ sudo vim /etc/docker/daemon.json`
- Add this
```
{
    "ipv6": false    
}
```

6. Restart docker desktop (quit) and wsl.
- Right click your docker desktop in the tray, and quit
- `$ wsl --shutdown`
- Start docker desktop again