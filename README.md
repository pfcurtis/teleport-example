# teleport-example
Example for Teleport


`kubeadm` Installation

### Application Stack


oci://registry-1.docker.io/bitnamicharts/wordpress
`helm template paul oci://registry-1.docker.io/bitnamicharts/wordpress -n wp > /tmp/foo`

`awk '/^kind: /{print tolower($2)}' /tmp/foo | sort | uniq`

`kubectl apply -f 00-namespace.yaml`

`kubectl apply -f 01-namespace-admin-rbac.yaml`

`kubectl apply -f 02-namespace-runtime-rbac.yaml`

`kubectl view-serviceaccount-kubeconfig admin -n wp > kc`

`chmod 0400 kc`

`helm install wp oci://registry-1.docker.io/bitnamicharts/wordpress -n wp --kubeconfig kc`
