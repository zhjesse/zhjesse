# 依赖
sudo apt install kustomize

# 部署
## 在项目根目录执行
kustomize build wazuh-deployment/wazuh-kubernetes/envs/eks/ | python3 -c "
import sys, re
vars = {}
with open('.env') as f:
    for line in f:
        line = line.strip()
        if not line or line.startswith('#'):
            continue
        k, v = line.split('=', 1)
        vars[k.strip()] = v
data = sys.stdin.read()
for k, v in vars.items():
    data = data.replace('\${%s}' % k, v)
print(data, end='')
" | kubectl apply -f -

## 重启pod（按需）
kubectl rollout restart sts wazuh-manager-master -n wazuh
kubectl rollout restart sts wazuh-manager-worker -n wazuh