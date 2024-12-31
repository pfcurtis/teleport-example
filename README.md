# teleport-example
Example for Teleport


`kubeadm` Installation

### Application Stack


oci://registry-1.docker.io/bitnamicharts/wordpress
`helm template paul oci://registry-1.docker.io/bitnamicharts/wordpress -n wp > /tmp/foo`

`awk '/^kind: /{print tolower($2)}' /tmp/foo | sort | uniq`

`kubectl apply -f 00-namespace.yaml`

`kubectl apply -f 01-namespace-admin-rbac.yaml`

Create (or obtain) user certificate/bearer token for user "paul"
To create a user and credentials:
Get the cluster's CA certificate and key.
openssl genrsa -out paul.key 2048
openssl req -new -key paul.key -out paul.csr -subj "/CN=paul"
openssl x509 -req -in paul.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out paul.crt -days 365
kubectl config set-credentials paul --client-certificate=paul.crt --client-key=paul.key
kubectl config set-context kind-teleport --user=paul

`helm install wp oci://registry-1.docker.io/bitnamicharts/wordpress -n wp --kube-as-user 'paul'`

`kubectl apply -f 02-namespace-runtime-rbac.yaml`

