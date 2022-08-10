# openEuler RISC-V 发行版操作系统自动化测试工具

openEuler（以下简称oE） RISC-V 发行版操作系统使用基于oE上游社区的mugen测试框架根据oE RISC-V 发行版操作系统自定义的镜像预装包和基础支持包二次开发mugen-riscv测试脚本和测试用例。mugen-riscv测试用例移植的第一步是对mugen原生的测试用例进行筛选，之后才能开始对有问题的测试用例进行修复。mugen-riscv测试用例筛选本质上就是将通过运行测试查看结果，将mugen X86/AArch64中可直接用于RISC-V oE测试的测试用例筛选出来。筛选的对象包括所有mugen中的测试用例（前提是其测的是目前的RISC-V oE有的功能/软件），系统的基本功能优先。

## 1. 运行步骤

1. 运行测试用例，首先去掉结果failed的测试用例（也需结合日志文件判断，排除例如超时的简单问题）
2. 运行单个测试套或测试例可直接用mugen.sh，使用方法参考mugen官方使用方法，如下。

### 1.1 安装依赖软件

`bash dep_install.sh`

### 1.2 配置测试套环境变量

- 命令执行：`bash mugen.sh -c --ip $ip --password $passwd --user $user --port $port`
- 参数说明：
  - ip：测试机的ip地址
  - user：测试机的登录用户，默认为root
  - password: 测试机的登录密码
  - port：测试机ssh登陆端口，默认为22
- 环境变量文件：./conf/env.json

```python
{
    "NODE": [
        {
            "ID": 1,
            "LOCALTION": "local",
            "MACHINE": "physical",
            "FRAME": "aarch64",
            "NIC": "eth0",
            "MAC": "55:54:00:c8:a9:21",
            "IPV4": "192.168.0.10",
            "USER": "root",
            "PASSWORD": "openEuler12#$",
            "SSH_PORT": 22,
            "BMC_IP": "",
            "BMC_USER": "",
            "BMC_PASSWORD": ""
        },
        {
            "ID": 2,
            "LOCALTION": "remote",
            "MACHINE": "kvm",
            "FRAME": "aarch64",
            "NIC": "eth0",
            "MAC": "55:54:00:c8:a9:22",
            "IPV4": "192.168.0.11",
            "USER": "root",
            "PASSWORD": "openEuler12#$",
            "SSH_PORT": 22,
            "HOST_IP": "",
            "HOST_USER": "",
            "HOST_PASSWORD": ""
            "HOST_SSH_PORT": ""
        }
    ]
}
```

- 在用例中的使用：NODE${id}_${LOCALTION}

### 1.3 用例执行

- 执行所有用例
`bash mugen.sh -a`
- 执行指定测试套
`bash mugen.sh -f testsuite`
- 执行单条用例
`bash mugen.sh -f testsuite -r testcase`
- 日志输出shell脚本的执行过程

```bash
bash mugen.sh -a -x 
bash mugen.sh -f testsuite -x
bash mugen.sh -f testsuite -r testcase -x
```

### 1.4 用例添加

- 根据模板编写用例脚本(推荐)

```bash
source ${OET_PATH}/libs/locallibs/common_lib.sh

# 需要预加载的数据、参数配置

```
function config_params() {
    LOG_INFO "Start to config params of the case."

    LOG_INFO "No params need to config."

    LOG_INFO "End to config params of the case."
}
```

### 1.5 测试对象、测试需要的工具等安装准备

```
function pre_test() {
    LOG_INFO "Start to prepare the test environment."

    LOG_INFO "No pkgs need to install."

    LOG_INFO "End to prepare the test environment."
}
```

### 1.6 测试点的执行

```
function run_test() {
    LOG_INFO "Start to run test."

    # 测试命令：ls
    ls -CZl -all
    CHECK_RESULT 0

    # 测试/目录下是否存在proc|usr|roor|var|sys|etc|boot|dev目录
    CHECK_RESULT "$(ls / | grep -cE 'proc|usr|roor|var|sys|etc|boot|dev')" 7

    LOG_INFO "End to run test."
}
```

### 1.7 后置处理，恢复测试环境

```bash
function post_test() {
    LOG_INFO "Start to restore the test environment."

    LOG_INFO "Nothing to do."

    LOG_INFO "End to restore the test environment."
}

main "$@"
```

- 单纯的shell脚本或python脚本，通过脚本执行的返回码判断用例是否成功。
  - 可参考样例：testsuite测试套下用例oe_test_casename_02和oe_test_casename_03


### 1.8 suite2cases中json文件的写法

- 文件名为测试套名，文件后缀.json
- 文件内容说明：  

### 1.9 框架的公共函数

- 如何加载到用例中，以供调用

```bash
source ${OET_PATH}/libs/locallibs/common_lib.sh
```

```python
import os, sys, subprocess

LIBS_PATH = os.environ.get("OET_PATH") + "/libs/locallibs"
sys.path.append(LIBS_PATH)
import xxx
```
  
- 公共函数
  - 日志打印
  
  ```bash
  # bash 
  LOG_INFO "$message"
  LOG_WARN "$message"
  LOG_DEBUG "$message"
  LOG_ERROR "$message"
  ```
  
  ```python
  # python
  import mugen_log
  mugen_log.logging(level, message) # level:INFO,WARN,DEBUG,ERROR;message:日志输出
  ```
  - 结果检查
  
  ```bash
  # bash
  CHECK_RESULT $1 $2 $3 $4
  # $1：测试点的返回结果
  # $2：预期结果
  # $3：对比模式，0：返回结果和预期结果相对；1：返回结果和预期结果不等
  # $4：发现问题，日志输出
  ```
  - rpm包安装卸载
  ```bash
  # bash
  ## func 1
  DNF_INSTALL "vim bc" "$node_id"
  DNF_REMOVE "$node_id" "jq" "$mode"
  # mode：默认为0，会删除用例中安装的包，当为非0时，则只卸载当软件包
  ```
  ```python
  # python
  import rpm_manage
  tpmfile = rpm_manage.rpm_install(pkgs, node, tmpfile)
  rpm_manage.rpm_remove(node, pkgs, tmpfile)
  ```
  - 远程命令执行
  ```bash
  # bash
  ## func 1
  SSH_CMD "$cmd" $remote_ip $remote_password $remote_user $time_out $remote_port`
  ## func 2
  P_SSH_CMD --cmd $cmd --node $node(远端节点号)
  P_SSH_CMD --cmd $cmd --ip $remote_ip --password $remote_password --port $remote_port --user $remote_user --timeout $timeout
  # python
  conn = ssh_cmd.pssh_conn(remote_ip, remote_password,remote_port,remote_user,remote_timeout)
  exitcode, output = ssh_cmd.pssh_cmd(conn, cmd)

  # port：默认为22
  # user：默认为root
  # timeout: 默认不限制
  ```
  - 目录文件传输
  ```bash
  # bash
  ## func 1
  ### 本地文件传输到远端  
  `SSH_SCP $local_path/$file $REMOTE_USER@$REMOTE_IP:$remote_path "$REMOTE_PASSWD"`
  ### 远端文件传输到本地  
  `SSH_SCP $REMOTE_USER@$REMOTE_IP:$remote_path/$file $local_path "$REMOTE_PASSWD"`
  ## func 2
  ### 目录传输
  SFTP get/put --localdir $localdir --remotedir $remotedir
  ### 文件传输
  SFTP get/put --localdir $localdir --remotedir $remotedir --localfile/remotefile $localfile/$remotefile
  # python
  ### 目录传输
  import ssh_cmd, sftp
  conn = ssh_cmd.pssh_conn(remote_ip, remote_password,remote_port,remote_user,remote_timeout)
  psftp_get(conn,remote_dir, local_dir)
  psftp_put(local_dir=local_dir, remote_dir=remote_dir)
  ### 文件传输
  import ssh_cmd, sftp
  psftp_get(remote_dir, remote_file, local_dir)
  psftp_put(local_dir=local_dir, local_file=local_file, remote_dir=remote_dir)

  
  # get：从远端获取
  # put：传输到远端
  # localdir: 默认为当前目录
  # remotedir：默认为远端根目录
  ```
  - 获取空闲端口
  ```bash
  # bash
  GET_FREE_PORT $ip
  # python
  import free_port
  free_port.find_free_port(ip)
  ```
  - 检查端口是否被占用
  ```bash
  # bash 
  IS_FREE_PORT $port $ip
  ```
  ```python
  # python
  import free_port
  free_port.is_free_port(port, ip)
  ```
  - 获取测试网卡
  ```bash
  # bash
  TEST_NIC $node_id
  ```
  ```python
  # python
  import get_test_device
  get_test_nic(node_id)

  # node_id：默认为1号节点
  ```
  - 获取测试磁盘
  ```bash
  # bash
  TEST_DISK $node_id
  ```
  ```python
  # python
  import get_test_device
  get_test_disk(node_id)

  # node_id：默认为1号节点
  ```
  - 睡眠等待
  ```bash
  # bash
  SLEEP_WAIT $wait_time $cmd
  ```
  ```python
  # python
  import sleep_wait
  sleep_wait.sleep_wait(wait_time,cmd)
  ```
  - 远端重启等待
  ```bash
  # bash
  REMOTE_REBOOT_WAIT $node_id $wait_time
  ```

### 1.9 用例如何调试
- 设置OET_PATH(mugen框架路径环境变量)之后，就可以在用例所在目录直接执行用例脚本

### 1.10 用例超时说明
- 默认超时30m
- 如果某条用例执行超过30m，在用例对TIMEOUT重新赋值

### 1.11 常见问题 
- 远程后台执行命令时，用例执行卡住，导致用例超时失败
  - 原因：ssh远程执行命令不会自动退出，会一直等待命令的控制台标准输出，直至命令运行信号结束
  - 解决方案：可以将标准输出与标准错误输出重定向到/dev/null，如此ssh就不会一直等待`cmd > /dev/nul 2>&1 &`

## 2. oE RISC-V 自动化测试工具的使用

运行多个测试套可以利用RISC-V oE自动化测试脚本，即runtest.py，使用方法如下
 
- RISC-V oE自动化测试脚本  
    - openEuler的mugen项目目前用于openEuler x86/AArch64的测试，并不方便直接用于当前openEuler RISC-V的测试  
        - 测试项目（测试套）并不能很好地匹配  
        - 目前的openEuler RISC-V测试依靠QEMU虚拟机
    - mugen中一个测试套对应一个软件或服务，为了更方便地执行测试，可以再抽象一层  
    - 辅助测试脚本匹配测试列表和mugen中的测试套，并反馈缺失的测试和执行可用测试  

### 2.1 使用方法  

- 依赖安装  
    - 依赖 ```python3```  
- 使用  
    ```shell  
    usage: runtest.py [-h] -l list_file [-m]
    optional arguments:
    -h, --help    show this help message and exit
    -l list_file  Specify the test targets list
    -m, --mugen   Run native mugen test suites
    ```  
    例如
    ```shell
    python3 runtest.py -l lists/list_riscv
    ```
    - list_file为运行的测试套列表  
    - 对于有riscv版本的测试套，测试套列表中可用原不带riscv后缀的测试套名称，脚本会自动优先匹配有后缀的版本，若想测试原测试套，可在运行runtest.py时加上```-m```参数  
    - 测试套列表文件格式可参考已有的列表文件 

## 其他说明

1. 现阶段使用的用户模式网络配置的QEMU虚拟机不支持多节点测试和多硬盘，故目前需要排除用到多个节点和多硬盘的测试例（测试套文件中有"add disk"和"machine num"字段的，或可以结合测试用例代码判断）
2. 将一个测试套中可用的测试用例整合，形成一个新的测试套描述文件，命名为原文件名+后缀"-riscv"，之后使用自动化测试脚本运行时可仍可使用原测试套名，脚本会自动有限匹配带有riscv后缀的测试套，若想测试原测试套，可在运行runtest.py时加上-m参数
