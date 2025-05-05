# clusters-with-kind

https://istio.io/latest/docs/tasks/security/cert-management/plugin-ca-cert/

### CREATING CLUSTERS WITH USING KIND

1. #### Setup using ubuntu 22.04LTS

- #### update packages and install certificates
```
sudo apt update
```
```
    sudo apt install -y ca-certificates curl gnupg lsb-release
```

- #### Add Docker’s official GPG key:
```
    sudo mkdir -p /etc/apt/keyrings
```
```
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

- #### Add Docker repository:
```
    echo \
    "deb [arch=$(dpkg --print-architecture) \
    signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

-  #### Install Docker Engine:
```
    sudo apt update
    sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


- #### Enable Docker without sudo (optional):
```
    sudo usermod -aG docker $USER
    newgrp docker
```

2. #### Install KIND
```
    curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
    chmod +x ./kind
    sudo mv ./kind /usr/local/bin/kind
```

3. #### Verify Installation
```
    kind version
```

4. #### Create a KIND Cluster
```
    kind create cluster --name kind-cluster
```

5. #### Check Cluster Status
```
    sudo apt update
    sudo apt install -y kubectl
```

6. #### Installation of metallb
a. install metal mb
    ```
        kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
    ```

b.  Check the docker network
    ```
        docker network inspect -f '{{.IPAM.Config}}' kind
    ```

c.  create namespaces
    * create istio-system namespace on cluster1
    * create istio-system namespace on cluster2

d.  install the certificates.
    * you will need to download istio directory from
        https://istio.io/latest/docs/setup/getting-started/
    
    ```
    curl -L https://istio.io/downloadIstio | sh -
    ```
e. install the certificates <br>
    [<kbd>https://istio.io/latest/docs/tasks/security/cert-management/plugin-ca-cert/ </kbd>](https://istio.io/latest/docs/tasks/security/cert-management/plugin-ca-cert/)

- navigate to your istio root directory
- create a new directory and push the certs to it.
    ```
        mkdir -p certs
        pushd certs
    ```
- Generate root certificates and key.
    ```
            make -f ../tools/certs/Makefile.selfsigned.mk root-ca
    ```
> [!NOTE] 
> <b>Sample output generated.</b><br>
> sample output generating root-key.pem <br> generating root-cert.csr <br> generating root-cert.pem <br> 
> Certificat request self-signature ok <br> subject=O = Istio, CN = Root CA

- Generate an intermediate certificate for each cluster
    ```
    make -f ../tools/certs/Makefile.selfsigned.mk cluster1-cacerts
    make -f ../tools/certs/Makefile.selfsigned.mk cluster2-cacerts
    ```
- In each cluster, create a secret cacerts including all the input files ca-cert.pem, ca-key.pem,
  root-cert.pem and cert-chain.pem.    For example, for cluster1

  ```
    kubectl create namespace istio-system
    kubectl create secret generic cacerts -n istio-system \
      --from-file=cluster1/ca-cert.pem \
      --from-file=cluster1/ca-key.pem \
      --from-file=cluster1/root-cert.pem \
      --from-file=cluster1/cert-chain.pem
  ```
- if using kind, use the command below
    ```
    kubectl create secret generic cacerts -n istio-system \
      --from-file=cluster1/ca-cert.pem \
      --from-file=cluster1/ca-key.pem \
      --from-file=cluster1/root-cert.pem \
      --from-file=cluster1/cert-chain.pem --context=kind-cluster1

      kubectl create secret generic cacerts -n istio-system \
      --from-file=cluster2/ca-cert.pem \
      --from-file=cluster2/ca-key.pem \
      --from-file=cluster2/root-cert.pem \
      --from-file=cluster2/cert-chain.pem --context=kind-cluster2
    ```
- install istio ctl
  ```
    istioctl install -n istio-system --context=kind-cluster1

    istioctl install -n istio-system --context=kind-cluster2
  ```
