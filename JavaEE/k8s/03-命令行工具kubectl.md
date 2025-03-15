# 命令行工具 kubectl

## 资源类型与别名

+ pods：po
+ deployments：deploy
+ services：svc
+ namespace：ns
+ nodes：no

## 资源操作

### 创建资源

```shell
$ kubectl create -f ./my-manifest.yaml           # 创建资源
$ kubectl create -f ./my1.yaml -f ./my2.yaml     # 使用多个文件创建资源
$ kubectl create -f ./dir                        # 使用目录下的所有清单文件来创建资源
$ kubectl create -f https://git.io/vPieo         # 使用 url 来创建资源
$ kubectl run nginx --image=nginx                # 启动一个 nginx 实例
$ kubectl explain pods,svc                       # 获取 pod 和 svc 的文档
```

```shell
# 从 stdin 输入中创建多个 YAML 对象
$ cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000000"
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep-less
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000"
EOF
```

```shell
# 创建包含几个 key 的 Secret
$ cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: $(echo "s33msi4" | base64)
  username: $(echo "jane" | base64)
EOF
```

### 删除资源

```shell
# 删除 pod.json 文件中定义的类型和名称的 pod
$ kubectl delete -f ./pod.json   

# 删除名为“baz”的 pod 和名为“foo”的 service
$ kubectl delete pod,service baz foo

# 删除具有 name=myLabel 标签的 pod 和 serivce
$ kubectl delete pods,services -l name=myLabel  

# 删除具有 name=myLabel 标签的 pod 和 service，包括尚未初始化的
$ kubectl delete pods,services -l name=myLabel --include-uninitialized   

# 删除 my-ns namespace 下的所有 pod 和 serivce，包括尚未初始化的
$ kubectl -n my-ns delete po,svc --all                                      
```

