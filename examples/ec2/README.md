# Example auto deployment on Amazon EC2

To allow automatic provision using Ansible, you have to meet the following requirement on the Ansible master host:
- Boto is installed
- Ansible is installed and has access to your Secret and Access key (via EC2
role or environment variable)
- ec2.py and ec2.ini inventory scripts are downloaded and configured
- ANSIBLE_HOSTS environment variable set
- ansible.cfg exists
- SSH agent is running (You can check with “ssh-add -L”)

### 1) Have boto installed.

```bash
pip install boto
```

### 2) Configure AWS secret and access key.

Get the AWS Secret and Access Key from Amazon EC2 Identity and Access Management (IAM) and configure them as environment variables:

```bash
export AWS_ACCESS_KEY_ID='YOUR_AWS_API_KEY'
export AWS_SECRET_ACCESS_KEY='YOUR_AWS_API_SECRET_KEY'
```

### 3) Download ec2.py and ec2.ini into /etc/ansible.

```bash
cd /etc/ansible
wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py
wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.ini
chmod 755 /etc/ansible/ec2.py
```

### 4) Configure environment variables for ec2.py and ec2.ini.

```bash
export ANSIBLE_HOSTS=/etc/ansible/ec2.py
export EC2_INI_PATH=/etc/ansible/ec2.ini
```

### 5) Configure AWS keypair.

Ensure the keypair is exist on the node. For example, if the keypair is located under ``/root/mykeypair.pem``, use following command to add it into SSH agent:

```bash
ssh-agent bash
ssh-add /root/mykeypair.pem
```

### 6) Verify

If there are running EC2 instances, you should get a list of them in JSON by running this command:

```bash
/etc/ansible/ec2.py --list
```

***Take note that you can put step 2 and 4 inside ``.bashrc`` or ``.bash_profile`` file to auto load the environment variables.***

You should be good to go.

# Deployment Steps

Configure ``ec2-instance.yml`` and ``deploy-everything.yml`` accordingly to suit your needs. Then, run the playbook in the following order:


0) Ensure the ClusterControl role is there:
```bash
ansible-galaxy install severalnines.clustercontrol
```

1) Create the EC2 instances:
```bash
ansible-playbook -i /etc/ansible/ec2.py ec2-instances.yml
```

2) Refresh the EC2 cache so the Ansible inventory is updated (important):
```bash
/etc/ansible/ec2.py --refresh-cache
```

3) Install ClusterControl and deploy the database cluster:
```bash
ansible-playbook -i /etc/ansible/ec2.py deploy-everything.yml
```
