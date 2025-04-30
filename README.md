# clusters-with-kind

## CREATING CLUSTERS WITH USING KIND

1. ### Setup using ubuntu 22.04LTS

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

2. ### Install KIND
    ```
        curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
        chmod +x ./kind
        sudo mv ./kind /usr/local/bin/kind
    ```

3. ### Verify Installation
    ```
        kind version
    ```

4. ### Create a KIND Cluster
    ```
        kind create cluster --name kind-cluster
    ```

5. ### Check Cluster Status
    ```
        sudo apt update
        sudo apt install -y kubectl
    ```