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
curl -k -v -XPATCH \
  -H "Accept: application/json" \
  -H "Content-Type: application/json-patch+json" \
  https://<YOUR KUBE ADDRESS>/api/v1/nodes/<YOUR NODE NAME>/status \
  --data '[{"op":"add","path":"/status/addresses/-", "value": {"type": "ExternalIP", "address": "<ADDRESS HERE>"} }]'
```


### Cluster scaling

See kubespray documentation for scaling the cluster [here](https://kubespray.io/#/docs/getting-started?id=adding-nodes).

The command:
```
ansible-playbook -i inventory/ovh/hosts.yaml scale.yml -b -v --private-key=~/.ssh/key --user ubuntu
```