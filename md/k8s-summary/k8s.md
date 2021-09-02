- 常用命令

  - 查看pods
    kubectl get po -n -o wide

  - 查看pod日志 
    kubectl logs slcbridgebase-5dcc6bd786-ltrn8 -n slc-ns -c
  -  进入pod
    kubectl 