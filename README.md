## Teleport Example

### Cluster Creation
This is a standard `kubeadm` installation on whichever platform is requested. In testing, we used **kind** clusters. The only required output from this installation is the cluster admin `kubeconfig` which is needed for initial namespace and role creation.

In this README, the created **kind** cluster is named "kind-teleport".

### Application Stack
Some assumptions:
  1. Namespace ("wp") installation
  2. Namespace scoped service accounts and users
  3. Pods running with service accounts 

Installation using the Bitnami Wordpress chart. It is very complete and provides policy and service accounts. It is located at oci://registry-1.docker.io/bitnamicharts/wordpress

In order to get the policies required for the namespace admin user, we have to look at what the Helm chart installs.

    helm template wp oci://registry-1.docker.io/bitnamicharts/wordpress -n wp > helm-chart.txt
    awk '/^kind: /{print tolower($2)}' helm-chart.txt | sort | uniq

For this, we can create two manifests for the initialization of the namespace and the namespace admin user. These two manifests are the only portion of the installation run with the cluster admin credentials. The ***namespace*** admin user is "paul" in these manifests. Two roles are created: one for installation and one for runtime. The major difference is the runtime role is essentially "read-only."

    kubectl apply -f 00-namespace.yaml
    kubectl apply -f 01-namespace-admin-rbac.yaml

Create (or obtain) user certificate/bearer token for user "paul". These steps can be used to create a user and credentials for this example:
- Get the cluster's CA certificate and key. (depends on where your cluster is running)
- `openssl genrsa -out paul.key 2048`
- `openssl req -new -key paul.key -out paul.csr -subj "/CN=paul"`
- `openssl x509 -req -in paul.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out paul.crt -days 365`
- `kubectl config set-credentials paul --client-certificate=paul.crt --client-key=paul.key`

Now that the user is created, update the context for your cluster to use the user "paul" for interactions with the cluster using the `kubeconfig`. This is not the only way to handle this, but it is the simplest. The Helm chart install could be run with "--kube-as-user" as well.

    kubectl config set-context kind-teleport --user=paul

Now install the Helm chart. The installation user is "paul" It creates two service accounts: wp-mariadb and wp-wordpress"

    helm install wp oci://registry-1.docker.io/bitnamicharts/wordpress -n wp

Now bind those service accounts to our namespace scoped runtime role:

    kubectl apply -f 02-namespace-runtime-rbac.yaml

