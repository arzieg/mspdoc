# Install

## kubectl on WSL

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

curl -kLO "https://dl.k8s.io/release/$(curl -Lk -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -kLO "https://dl.k8s.io/release/$(curl -Lk -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
kubectl version --client
