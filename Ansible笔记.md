Encrypted String: 0702120e195a000109005604001c5252125e4a5556170c4f0d5205521f02000400015f16570450081714

# Ansible基础

## 一、前期准备

<b>安装ansible</b>

```c
yum install -y epel-release ansible libselinux-python
```

Tip:安装命令重复两次，第一次未能安装ansible，前面安装的是ansible的配置文件

## 二、免密登录：

1、<b>主机目录配置文件</b>

```c
vi /etc//ansible/hosts
    //进入编辑页面后
[lyh]
192.168.xxx.xxx
192.168.xxx.xxx    
```

![1a1c17c0952233a3531bc490b4000db](D:\音乐\system files\ansible笔记\1a1c17c0952233a3531bc490b4000db.jpg)

其他格式：

```c
[web:vars]
ansible_port=22 / ansible password='__'
[web]
192.168.192.[__:__] //多台虚拟机免密登录
```

<b>ansible指纹链接确认的问题</b>

在Linux中，点开头的是隐藏文件

1）在root/ssh/known_hosts

2）首次远程连接，需要确认指纹（也可以忽略）

3）一般用法

​		ansible测试连接的标准

​				1.指纹确认

​				2.密码认证或公钥认证

4）关于ansible如何初始化的使用，有三个方案

​	①一键分发公钥，实施批量登录再使用ansible免密远程执行命令（基于ssh完成指纹认证）

​	②手动ssh连接，确认指纹后再进行免密远程执行命令（ssh root@xxx）

​	③直接忽略指纹确认，asnible配置文件：/etc/ansible/ansible.cfg #host_key_checking=False //71行左右

5）常见错误

端口、密码、用户

如果出错

①找/etc/ansible/hots文件里面的语法是否出错

②看目标机器提供了什么样的ssh连接形式（sshd_config）

<font color=red>注意:ansible是一个命令行工具，不是一个服务</font>

<b>使用ssh免密登录</b>

```c
ssh-keygan
ssh-copy-id root@主机地址
//例如：
//ssh-copy-id root@192.168.86.128
//ssh-copy-id root@192.168.86.130
```

<b>测试执行</b>

```c
ansible lyh -m ping
```

![b16ad73cf77c692efe1ca940463a0a9](D:\音乐\system files\ansible笔记\b16ad73cf77c692efe1ca940463a0a9.jpg)

<font size=5><b>Ansible命令语法</b></font>
ansible的命令行模式，也叫纯命令模式，又叫ad-hoc模式，下一个叫playbook模式，也叫剧本模式
ansible chaoge -m command -a "hostname 
chaoge 是主机组的名字
-m：指定模块
command：模块
-a “hostname”：执行的操作

<font size=5><b>ansible命令执行的的结果（颜色状态）</b></font>
前两个表示命令执行成功了
绿色：命令已经执行了，但是状态没发生改变
黄色：命令已经执行了，并且状态了发生改变
红色：命令错误，执行失败
紫色：警告信息，说明ansible提示你使用更适合的用法
蓝色：详细的执行过程
运维命令执行的方式一半有两个：
1、shell脚本远程执行
	shell脚本不够智能，不会记录上一次的执行状态，因此导致傻瓜式的重复执行，效率低
2、ansible模块远程执行

## <font size=6><b>模块</b></font>

### <font size=5><b>shell模块</b></font>

```c
ansible web -m shell -a “free -m”//查看内存信息
```

### <font size=5><b>command模块</b></font>

帮助信息

ansible -doc -s command

<font color=red><b>注意:使用command简单命令模块不能使用变量，也不能使用特殊符号(< > | : & - .....)</b></font>

-m后面接调用模块的名字

-a后面接调用模块的参数

command模块是ansible的默认模块，也就是默认指定了-m command，他只支持简单命令

如果要执行复杂命令则用shell

例如：

查看主机名

```c
ansible 名字 -a 'hostname'
```

远程获取机器负载

```c
ansible 名字 -a 'uptime'
```

远程创建gwng用户

```c
ansible 名字 -a "useradd gwng"
```

使用command提供的专用命令

| 条件参数        | 选项说明                                                 |
| --------------- | -------------------------------------------------------- |
| chdir           | 在执行命令时是通过cd命令进入                             |
| creates         | 定义一个文件是否存在，如果不存在则执行相应命令，否则跳过 |
| free_from(必须) | 参数命令可以使用任何系统参数，达到远程管理目的           |
| removes         | 定义一个文件是否存在，如果不存在则执行相应命令，否则跳过 |

### <font size=5><b>command简单命令模块练习</b></font>

备份/var/log日志目录

正常情况下：cd / && tar -zcvf /root/log.tar.gz /var/log

```c
ansible lyh -m command -a "tar -zvf /root/log.tar.gz var/log chdir=1"
```

在/root下创建s666.log

```c
ansible lyh -a "touch s666.log chdir=/root"
```

备份/etc所有配置文件到/backup_config/etc.tar.gz

```c
ansible lyh -m shell -a "mkdir /backup_config ; tar -zcf /backup_config/etc.tar.gz /etc"
```

### <font size=5><b>Copy模块</b></font>

```c
ansible 主机名 -m copy -a "xxxx"
```

简单发送文件

```c
ansible lyh -m copy -a "src=/root/flag.txt dest=/root/lyh-flag.txt"
```

发送文件且指定文件属性

```c
权限修改为600，修改为www用户（目标记起要求存在www用户
    
    //创建www用户
    ansible lyh -m shell -a "useradd www"
    //拷贝文件且修改为600（ansible-doc -s copy）
    ansible lyh -m copy -a "src=/root/flag.txt dest=/root/lyh-flag.txt owner=www group=www mode=600"
```

发送文件切线做好准备

使用backup参数，防止覆盖远程文件而丢失备份，提前备份该目标机器的数据

<b>1、检查目标机器文件</b>

```c
ansible lyh -m shell -a "ls -l /root | grep 'flag'"
ansible lyh -m shell -a "cat /root/lyh-flag.txt"
```

<b>2、远程拷贝文件，且做好备份</b>

```c
ansible lyh -m shell -a "src=/root/flag.txt dest=/root/lyh-flag.txt backup=yes"
```

<b>3、指定数据写入到远程文件中</b>

```c
ansible lyh -m copy -a "content='牛' dest=/root/lyh-flag2.txt"
```

<b>4、复制文件夹（<font color=red>注意结尾斜杠</font>)</b>

这里有个坑！！！

前期准备

```c
 s #!/bin/bash

#创建三个普通文件
for i in {1..3}
do
  touch "falg${i}.txt"
  done
  
  //创建三个文件夹，并将flag1~3.txt分别复制到文件夹内
  
  for i in {1..3}
    do mkdir ”flag${i}"
    cp "flag${i}.txt" "flag_d${i}/"
  done
```

远程拷贝/root/flag_dir下的所有内容到目标机器

```c
ansible lyh -m copy -a "src=/root/flag_d/ dest=/root/"
```

### <font size=5><b>File模块</b></font>

要和copy区分开

file模块的作用是创建。以及设置文件目录属性

copy模块：src（管理机），dest（目标机器）

file是专门同于在远程机器上关于文件的所有操作

<font size=4><b>远程创建文件</b></font>

在远程web服务器组中穿件一个文本hello_lyh.log

```c
ansible lyh -m file -a "path=/root/hello_lyh.log state=touch"
```

<b>远程创建文件夹</b>

```c
ansible lyh -m file -a "path=/root/xxx state=directory"
```

<b>创建文件夹并设定权限</b>

```c
ansible lyh -m file -a "path=/root/helloxxx.log state=touch owner=www group=www"

ansible lyh -m file -a "path=/root/helloxxx2.log state=touch owner=www
group=www mode=777"
```

<b>创建软连接文件</b>

```c
ansible lyh -m shell -a "src=/etc/passwdback dest=/root/passwd state=link"

 /*强制创建软连接*/
ansible lyh -m shell -a "src=/etc/passwdback dest=/root/passwd state=link force=yes"
```

<b>修改已存在文件/文件夹属性</b>

```c
ansible lyh -m file -a "path=/root/helloxxx.log  owner=www group=www mode=777"
```

### <font size=5><b>script模块</b></font>

```c
ansible lyh -m script -a "/root/systeminfo.sh"
```

### <font size=5><b>Cron定时任务模块</b></font>

<font size=4><b>语法格式</b></font>

```c
* * * * *要执行的命令
```

| 定时任务            | 分     | 时   | 日   | 月    | 周      |
| ------------------- | ------ | ---- | ---- | ----- | ------- |
|                     | *      | *    | *    | *     | *       |
| ansible定时任务模块 | minute | hour | day  | month | weekday |

```c
*/2 * * * * xxxxx

ansible lyh -m cron -a "name='xxxxx' job='xxxxx' minute=/*2"
```

ansible lyh -m cron -a "name='牛逼' job='牛逼' minute=*/2"

1、每个小时的第五分钟记录当前的日期和时间，并将这些信息追加到/root/croncheck.log中

```
正常命令： 
5 * * * * date >> /root/croncheck.log 
Ansible命令： ansible lyh -m cron -a "name='logdate' job='date >> /root/croncheck.log' minute=5" -b  
```

-b：告诉ansible以become模式运行，意味着使用sudo权限（如果是使用普通用户的话）。因为管理Cron作业通常需要最高系统权限

2、每2分钟记录一次当前用户的ID信息，存入到/root/userid.log中

```
正常命令：
*/2 * * * * id >> /root/userid.log
ansible命令：
ansible lyh cron -a "name='useridlog' job='id >> /root/userid.log' minute='*/2'"

```

3、远程查看定时任务

ansible lyh -m shell -a "crontab -l"

4、远程删除定时任务

ansible lyh -m cron -a "name='logdate' state=absent" 

5、修改指定名称的定时任务

在原代码的基础上修改需要修改的地方



### **group模块**

1、创建gtan组，gid=1234

````
ansible lyh -m group -a "name=gtan gid=1234"
````

2、删除用户组

```
ansible lyh -m group -a "name=gtan gid=1234 state=absent"
```



### **user用户模块**

uid

用户名

用户主组

用户附加组

创建

删除

修改

用户公私钥

用户过期时间

用户密码过期时间

**常用参数模块**

| create_home | 创建家目录         |
| ----------- | ------------------ |
| gropu       | 创建用户组         |
| name        | 用户的名字（必填） |
| password    | 密码               |
| uid         |                    |

**1、创建user1，uid=4321**

 ```
 ansible lyh -m user -a "name=user1 uid=4321"
 ```

2、创建user2，uid和gid都为5678.没有家目录，不允许登录

```
首先保证5678这个组必须存在
ansible lyh -m group -a "name=gp2 gid=5678"

ansible lyh -m user -a "name=user2 uid=5678  group=5678 create_home=no shell=/sbin/nologin"
```



### **yum安装软件**

安装apache最新版本

```
ansible lyh -m yum -a "name="
```

卸载apache

```
ansible lyh -m yum -a "name=httpd state=absent"
```

service/systemd模块

- 针对yum包管理
- service只适用于cenOS6前的
- systemd用于cenOS7

| 主要参数      | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| daemon_reload | 在执行任何其他操作之前运行守护进程重新加载以确保systemd已经读取其他更改 |
| enabled       | 服务是否开启启动（yes/no）——enable和state至少有一个被定义    |
| masked        | 是否将服务设置为masked状态，被mask的服务是无法启动的         |
| name          | 必须项（服务的名称）                                         |
| state         | 当前服务的执行。启动（started）停止（stopped）重启（restarted）重新加载（reloaded） |

**安装和启动apache服务**

- 安装服务

 ```
 ansible lyh -m yum -a "name=httpd state=installed"
 ```

- 启动服务

 ```
 ansible lyh -m systemd -a "name=httpd state=started"
 ```

- 查看服务状态

 ```
 ansible lyh -a "systemctl status httpd"
 ```

- 停止服务

```
ansioble lyh -m systemd -a "name=httpd state=stopped"
```

- 设置开机自启

```
ansible lyh -m systemd -a "name=httpd enabled=yes state=started"
```



### **archive压缩模块**

- 支持类型

bz2、gz（default）、tar、xz、zip

**压缩/etc配置文件到指定路径**

```
ansible lyh -m archive -a "path=/etc dest=/root/etc.zip"
```



### **unarchive解压压缩模块**

**解压etc.zip到指定目录(远程解压）**

```
ansible lyh -m unarchive -a "src=/root/etc.zip dest=/root remote_src=yes"
```

**将管理机的压缩包解压到远程机器上**

```
ansible lyh -m unarchive -a "src=/root/test.zip dest=/root"
```



# playbook

- 任务
- 角色
- 变量
- 模板
- 处理程序
- 条件
- 循环
- 错误处理
- 标签 

Tip:每行都要空两格



ansible lyh -m ping

```yaml
- name: ping lyh Servers      # playbook 的名称，描述playbook的作用
  hosts: lyh                 # 指定这个playbook要运行的主机组
  tasks:
      - name: ping host      # 任务名称
        ping:                 # 调用的模块
        
```



ansible lyh -m shell -a "ls /"

```yaml
- name： lslyh
  hosts:lyh
  tasks:
    - name: is
      shell: is /
      register:ls_output
      
    - name:xxxx
      debug:
        msg:"{{shuchu.stdout_lines}}"
```



ansible lyh -m shell -a "free -m"

```yaml
- name: Check memory  usage on lyh group
  hosts: lyh
  tasks:
    - name: Execute free command to check memory usage
      shell: free -m
      register: ls_output
      
    - name: print memory
      debug: 
        msg: "{{ ls_output.stdout_lines }}"
```



ansible lyh -m command -a "touch /root/2024.log"

ansible lyh -m command -a "cat /root/2024"

将上面两个命令整合成一个yml文件

```yaml
- name： command
  hosts: lyh
  tasks:
    - name: touch file 
      command: touch /root/2024.log
      
      -name: w2024
       shell: ls -al > /root/2024.log
      
    - name: cat file
      command: cat /root/2024
      register: cat2024log
      
    - name: print 2024
      debug: 
        msg: "{{cat2024log.stdout}}"
```



ansible lyh -a "uptime"

```yaml
- name: uptime
  hosts: lyh
  tasks:
    - name: p1
      shell: uptime
      register: uptime
    
    - name: uptime
      debug:
        msg: "{{uptime.stdout}}"
    
```



ansible lyh -m command -a "tar -zcf /root/log.tar.gz /etc chdir=/"

```yaml
- name: tar
  hosts: lyh
  tasks:
    - name: tar
      command: tar -zcf /root/log.tar.gz /etc
      args:
       chdir: /


    - name: test
      shell: ls -al | grep log
      register: lsal

    - name: printlsal
      debug:
        msg: "{{lsal.stdout_lines}}"

```



ansible lyh -m shell -a "ps -ef | grep ssh"

```yaml
- name: ps
  hosts: lyh
  tasks:
    - name: ps
      shell: ps -ef | grep ssh
      register: ps

    - name: show
      debug:
        msg: "{{ps.stdout_lines}}"

```



ansible lyh -m copy -a "src=/root/flag.txt dest=/root/web-flag.txt"

```yaml
- name: copy
  hosts: lyh
  tasks:
    - name: copy
      copy: src=/root/flag.txt dest=/root/web-flag.txt
 //先提前在/root下创建flag.txt
 //或者用playbook创建
```



ansible lyh -m shell -a "useradd www"

ansible lyh copy -a "src=/root/flag.txt dest=/root/web-flag.txt group=www owner=www mode=666"

ansible lyh -m shell -a "ls -l /root/ | grep 'flag'"

```yaml
- name: useradd
  hosts: lyh
  tasks:
    - name: useradd
      shell: useradd www
      
    - name: copy
      copy: 
      src: /root/flag.txt 
      dest: /root/web-flag.txt 
      group: www 
      owner: www 
      mode: 666
    
    - name: ls
      shell: ls -l /root/ | grep 'flag'
      register: lsl
      
    - name: show
      debug:
        msg: "{{lsl.stdout_lines}}"
      
```



ansible lyh -m shell "rm -rf /root/"

ansible lyh -m file "path=/root/hello_ansible.log state=touch"

ansible lyh -m file "path=/root/hiansible state=directory"

ansible lyh -m shell "ls -al /root/"

```yaml
- name: touch a file
  hosts: lyh
  tasks:
    - name: rmroot
      shell: rm -rf /root/*
      
    - name: touch a new file
      file: 
        path: /root/hello_ansible.log
        state: touch
        
    - name: build a file
      file: 
        path: /root/hiansible
        state: directory
        
    - name: lsal
      shell: ls -al /root/
      register: lsal
    
    - name: show
      debug:
        msg: "{{lsal。stdout_lines}}"
```



ansible lyh -m cron -a "name='Log date to concheck every hour' minute=5 job='date >> /root/croncheck.log'" -b

```yaml
- name: log date to croncheck every hour
  hosts: lyh
  tasks: 
    - name: p1
      cron:
        name: Log date to croncheck every hour
        minute: 5
        job: 'date >> /root/croncheck.log'
```



ansible lyh -m group -a "name=gtan gid=1234"

```yaml
- name: name a group
  hosts: lyh
  tasks: 
    - name: group
      group: 
        name: gtan
        gid: 1234
    
    - name: check group
      shell: tail -2 /etc/group
      
```



ansible lyh -m user -a "name=fengge uid=4567 group=1234 create_home=no shell=/sbin/nologin"

```yaml
- name: fengge
  hosts: lyh
  tasks: 
    - name: useradd
      user:
        name: fengge
        uid: 4567
        group: gtan
        crate_home: no
        shell: /sbin/nologin
```

