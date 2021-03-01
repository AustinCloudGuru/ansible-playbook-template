# ansible-playbook-template
This Ansible Template can be used as a starting point for creating and installing applications. You will need to update it for your own roles.



## user_data
The easies way to get the ansible to run is to add the following script to the user_data of the instance.  Make sure you update it with the correct repo information

```shell
#!/bin/bash

## Prepare the node
yum install -y python3 python3-pip python3-libselinux python3-devel git jq

## Get Vault Token
VAULT_ADDR="https://vault.example.com"
VAULT_TOKEN=$(curl -X POST "$VAULT_ADDR/v1/auth/aws/login" -d '{"role":"default-instance-role","pkcs7":"'$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/pkcs7 | tr -d '\n')'"}'|jq -r .auth.client_token)

## Setup Github Credentials
GIT_TOKEN=$(curl -s --header "X-Vault-Token: $VAULT_TOKEN" $VAULT_ADDR/v1/acg/globals/data/ghe|jq -r .data.data.api_key)
echo "machine github.com login vaultbot password $GIT_TOKEN" > ~/.netrc
chmod 600 ~/.netrc

## Get Datadog Credentials
DATADOG_TOKEN=$(curl -s --header "X-Vault-Token: $VAULT_TOKEN" $VAULT_ADDR/v1/acg/globals/data/datadog|jq -r .data.data.api_key)

## Run Ansible
/usr/bin/python3 -m venv ansible
. ./ansible/bin/activate
pip install ansible
git clone https://github.com/austincloudguru/ansible-playbook-template
cd ansible-playbook-template
ansible-galaxy install -r requirements.yml
ansible-playbook -i inventory -e datadog_api_key=$DATADOG_TOKEN playbook.yml

## Cleanup
rm -rf ~/.netrc
unset VAULT_ADDR
unset VAULT_TOKEN
unset GIT_TOKEN
unset DATADOG_TOKEN
```
