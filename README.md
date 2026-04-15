## Aether K8s

This repository provisions a multi-node Kubernetes cluster with RKE2 and installs Helm by using Ansible playbooks.

## Ubuntu 25.10 從 0 到可用部署流程

> 以下流程假設你在控制端主機操作，且專案路徑為  
> `/home/runner/work/aether-k8s/aether-k8s`

### 1) 準備控制端（執行 Ansible 的機器）

```bash
sudo apt update
sudo apt install -y git make ansible sshpass python3-pip
ansible-galaxy collection install ansible.posix
```

### 2) 取得專案

```bash
cd /home/runner/work/aether-k8s
git clone <your-repo-url> aether-k8s
cd /home/runner/work/aether-k8s/aether-k8s
```

如果目錄已存在可略過 clone。

### 3) 準備目標節點（master/worker 都要）

- 節點作業系統：Ubuntu 25.10
- 可由控制端 SSH 連線
- `ansible_user` 具 sudo 權限
- 建議關閉 swap（RKE2 常見需求）

```bash
sudo swapoff -a
sudo sed -i.bak '/\sswap\s/s/^/#/' /etc/fstab
```

### 4) 設定 inventory

編輯：

`/home/runner/work/aether-k8s/aether-k8s/hosts.ini`

依實際環境填入 master/worker 節點，例如：

```ini
[all]
node1 ansible_host=172.16.220.36 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether
node2 ansible_host=172.16.108.89 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether
node3 ansible_host=172.16.232.55 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether

[master_nodes]
node1

[worker_nodes]
node2
node3
```

### 5) 準備 RKE2 參數檔

從模板複製：

```bash
cp /home/runner/work/aether-k8s/aether-k8s/roles/rke2/templates/master-config.yaml /home/runner/work/aether-k8s/aether-k8s/master-config.yaml
cp /home/runner/work/aether-k8s/aether-k8s/roles/rke2/templates/worker-config.yaml /home/runner/work/aether-k8s/aether-k8s/worker-config.yaml
```

### 6) 設定版本與參數路徑

編輯：

`/home/runner/work/aether-k8s/aether-k8s/vars/main.yml`

至少確認：

- `k8s.rke2.version`
- `k8s.helm.version`
- `k8s.rke2.config.params_file.master: "master-config.yaml"`
- `k8s.rke2.config.params_file.worker: "worker-config.yaml"`

### 7) 連線與環境檢查（建議）

```bash
cd /home/runner/work/aether-k8s/aether-k8s
HOSTS_INI_FILE=/home/runner/work/aether-k8s/aether-k8s/hosts.ini make k8s-debug
```

### 8) 開始部署（RKE2 + Helm）

```bash
cd /home/runner/work/aether-k8s/aether-k8s
HOSTS_INI_FILE=/home/runner/work/aether-k8s/aether-k8s/hosts.ini make k8s-install
```

### 9) 驗證叢集

在 master 節點執行：

```bash
kubectl get nodes
```

預期可看到 master/worker 節點狀態為 `Ready`。

### 10) 需要重建時

```bash
cd /home/runner/work/aether-k8s/aether-k8s
HOSTS_INI_FILE=/home/runner/work/aether-k8s/aether-k8s/hosts.ini make k8s-uninstall
```
