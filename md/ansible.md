## ansible

- 安装
  - centos yum安装

    ```
    yum install epel-release -y 
    yum install ansible –y
    ```

  - 安装目录如下(yum安装)：

    ```
    配置文件目录：/etc/ansible/
    执行文件目录：/usr/bin/
    Lib库依赖目录：/usr/lib/pythonX.X/site-packages/ansible/
    Help文档目录：/usr/share/doc/ansible-X.X.X/
    Man文档目录：/usr/share/man/man1/
    ```

- ansible配置

  - 配置文件

  ```
  # 配置文件查找顺序
  检查环境变量ANSIBLE_CONFIG指向的路径文件(export ANSIBLE_CONFIG=/etc/ansible.cfg)；
  ~/.ansible.cfg，检查当前目录下的ansible.cfg配置文件；
  /etc/ansible.cfg 检查etc目录的配置文件。
  
  # 配置文件中常见变量含义
  inventory = /etc/ansible/hosts		#这个参数表示资源清单inventory文件的位置
  library = /usr/share/ansible		#指向存放Ansible模块的目录，支持多个目录方式，只要用冒号（：）隔开就可以
  forks = 5		#并发连接数，默认为5
  sudo_user = root		#设置默认执行命令的用户
  remote_port = 22		#指定连接被管节点的管理端口，默认为22端口，建议修改，能够更加安全
  host_key_checking = False		#设置是否检查SSH主机的密钥，值为True/False。关闭后第一次连接不会提示配置实例
  timeout = 60		#设置SSH连接的超时时间，单位为秒
  log_path = /var/log/ansible.log		#指定一个存储ansible日志的文件（默认不记录日志）
  ```

  - 资源清单(主机清单) 定义方式：

    - 直接指明主机地址或主机名：

    ```
    ## green.example.com#
    # blue.example.com#
    # 192.168.100.1
    # 192.168.100.10
    ```
    - 定义一个主机组[组名]把地址或主机名加进去
    
    ```
    [mysql_test]
    192.168.253.159
    192.168.253.160
    192.168.253.153
    ```

- ansible 命令集

```
/usr/bin/ansible　　Ansibe AD-Hoc 临时命令执行工具，常用于临时命令的执行
/usr/bin/ansible-doc 　　Ansible 模块功能查看工具
/usr/bin/ansible-galaxy　　下载/上传优秀代码或Roles模块 的官网平台，基于网络的
/usr/bin/ansible-playbook　　Ansible 定制自动化的任务集编排工具
/usr/bin/ansible-pull　　Ansible远程执行命令的工具，拉取配置而非推送配置（使用较少，海量机器时使用，对运维的架构能力要求较高）
/usr/bin/ansible-vault　　Ansible 文件加密工具
/usr/bin/ansible-console　　Ansible基于Linux Consoble界面可与用户交互的命令执行工具
```

- 

  





