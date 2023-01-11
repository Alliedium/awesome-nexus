# ansible-playbook-nexus


## Prerequisites

Target machine for Nexus mentioned as `@hostname` in [hosts.yaml](./inventory/hosts.yaml) file should have:

* Compatible OS. This role is tested through molecule on travis CI for CentOS 8, Ubuntu Bionic (18.04), and Debian buster. Other molecule scenarios can be played locally for CentOS 7, Ubuntu Xenial (16.04), and Debian stretch

* Rsync has to be installed on the target machine (it is not needed on the host running ansible if different)

* Java 8 (mandatory).
  
  ```
  apt install openjdk-8-jdk
  apt install rsync
  ```

## Install Nexus via [playbook](./playbooks/install_nexus.yaml)

* Install [ansible galaxy role](https://github.com/ansible-ThoTeam/nexus3-oss)

```
ansible-galaxy install ansible-thoteam.nexus3-oss
```

* Ensure that role does exist in the list

```
ansible-galaxy role list
```

* Change [hosts.yaml](./inventory/hosts.yaml) file: 

```
sed -i 's/@hostname/172.17.0.3/g' ./inventory/hosts.yaml &&
sed -i 's/@username/root/g' ./inventory/hosts.yaml &&
sed -i 's/@sshprivatekey/$HOME\/.ssh\/id_rsa_test/g' ./inventory/hosts.yaml
```

Above example is for host machine `172.17.0.3`, connection via `ssh` for user `root`, with ssh key file located at `$HOME/.ssh/id_rsa_test` path

* Run [playbook](./playbooks/install_nexus.yaml)

```
ansible-playbook -v ./playbooks/install-nexus.yaml -i ./inventory
```

## Notes

This [playbook](./playbooks/install_nexus.yaml) is using [ansible-thoteam/nexus3-oss](https://galaxy.ansible.com/ansible-thoteam/nexus3-oss) role and does the following

* Install Nexus OSS - the [latest](https://help.sonatype.com/repomanager3/product-information/download#Download-DownloadtheLatestVersion) version
* Create Users and Roles for pushing to Docker-hosted and Helm-hosted repositories. Details can be found in `nexus_roles` and `nexus_local_users` variables in [playbook](./playbooks/install_nexus.yaml) 
* Create cleanup policies as described in `nexus_repos_cleanup_policies` variable of [playbook](./playbooks/install_nexus.yaml). But these policies are not applied to any repository, they should be applied manually (!)
* Setup scheduled tasks as described in `nexus_scheduled_tasks` variable of [playbook](./playbooks/install_nexus.yaml)
* Configure the following repositories:

<details>
<summary>List of repositories</summary>

## 

Conda:

```html
anaconda - proxy for https://conda.anaconda.org/anaconda/
conda-forge - proxy for https://conda.anaconda.org/conda-forge/
```

Maven:

```html
maven-central - proxy for https://repo1.maven.org/maven2/
maven-snapshots - hosted repository for custom dependencies storage
maven-releases - hosted repository for custom dependencies storage
maven-public - group repository, includes all three above repos
```

npm:
```html
npm - proxy for https://registry.npmjs.org/
npm-hosted - hosted repository for custom npm artifats storage
npm-group - group repository, includes both repos listed above
```

pip:
```html
pypi.org - proxy for https://pypi.org/
pypi-hosted - hosted repository for custom pypi artifacts
pypi-group - group repository, includes all listed above pypi repos
```

helm:
```html
oxyno-zeta.github.io_helm-charts-v2 - proxy for https://oxyno-zeta.github.io/helm-charts-v2/
argoproj.github.io_argo-helm - proxy for https://argoproj.github.io/argo-helm/
charts.bitnami.com_bitnami - proxy for https://charts.bitnami.com/bitnami
aws.github.io_eks-charts - proxy for https://aws.github.io/eks-charts
charts.crossplane.io_stable - proxy for https://charts.crossplane.io/stable
charts.bitnami.com_bitnami - proxy for https://charts.bitnami.com/bitnami
dapr.github.io_helm-charts - proxy for https://dapr.github.io/helm-charts
helm-hosted - hosted repository for custom charts
```

Docker:
```html
registry-1.docker.io - proxy for https://registry-1.docker.io which is docker hub. Http connector opened at port 8181
gcr.io - proxy for https://gcr.io.
quay.io - proxy for https://quay.io. 
ghcr.io - proxy for https://ghcr.io.
docker-hosted - hosted repository for custom images. Http connector opened at port 8182
docker-group - group repository, includes all listed above docker repos. Http connector opened at port 8183
```

Http connectors for Docker repositories can be configured by changing `nexus_docker_hosted_port`, `nexus_docker_proxy_port` and `nexus_docker_group_port` variables in [playbook](./playbooks/install_nexus.yaml)
</details>