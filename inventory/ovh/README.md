# Usage

Install a virtualenv and install the requirements:
```bash
virtualenv -p python3 .venv
source .venv/bin/activate
pip install -r requirements.txt
```


To run with the following command from the root of the kubespray directory:
`ansible-playbook -i inventory/ovh/hosts.yaml cluster.yml -b -v --private-key=~/.ssh/key --user ubuntu`

## Add ExternalIP to the node
This is necessary because the CL/EL helm charts require the ExternalIP to be set on the node. This is not done automatically by kubespray.
```bash
export KUBECONFIG=<path-to-your-kubeconfig>

kubectl config view --raw -o=jsonpath='{.users[0].user.client-certificate-data}' | base64 -d > client.crt

kubectl config view --raw -o=jsonpath='{.users[0].user.client-key-data}' | base64 -d > client.key

kubectl config view --raw -o=jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 -d > ca.crt

export KUBE_API_SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')

# Replace <NODE NAME> and <NODE EXTERNAL IP> with the correct values
curl -k -v -XPATCH \
  -H "Accept: application/json" \
  -H "Content-Type: application/json-patch+json" \
  --cert client.crt --key client.key --cacert ca.crt \
  "$KUBE_API_SERVER/api/v1/nodes/<NODE NAME>/status" \
  --data '[{"op":"add","path":"/status/addresses/-", "value": {"type": "ExternalIP", "address": "<NODE EXTERNAL IP>"} }]'
```

```bash
kubectl taint node <NODE NAME> node.cloudprovider.kubernetes.io/uninitialized-
```

### Cluster scaling

See kubespray documentation for scaling the cluster [here](https://kubespray.io/#/docs/getting-started?id=adding-nodes).

The command:
```
ansible-playbook -i inventory/ovh/hosts.yaml scale.yml -b -v --private-key=~/.ssh/key --user ubuntu
```