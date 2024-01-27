# shell: 10 个 Shell 脚本实例
**脚本 1**：该脚本用于读取`Linux`系统`/etc/passwd`文件中的所有`/bin/bash`作为登录默认`Shell`的用户列表，并提取每个用户的用户名。对于这些用户名进行检查，是否不等于`root`和`tidb`。如果用户名不等于`root`和`tidb`，则使用`usermod`命令对该用户的默认`Shell`修改为：`/sbin/nologin`，它通常用于禁止用户远程登录系统。

```bash
#!/bin/bash
for user in $(cat /etc/passwd | grep /bin/bash | cut -d ":" -f 1)
do
    if [ $user != "root" ] && [ $user != "tidb" ]; then
        usermod -s /sbin/nologin $user
    fi
done

```

**脚本 2**：该脚本首先自定义了`md5_list`和`md5_no_hash.txt`两个文件，然后循环读取`md5_list`文件中每一行的哈希值，判断是否在`md5_no_hash.txt`文件中存在。

如果检查出哈希值在`md5_no_hash.txt`中存在，则打印信息：`MD5值 xxx 在 md5_no_hash.txt 中存在`。

如果检查出哈希值在`md5_no_hash.txt`中不存在，则打印信息：`MD5值 xxx 在 md5_no_hash.txt 中不存在`。

```bash
#!/bin/bash  


md5_list_file="md5_list"
md5_no_hash_file="md5_no_hash.txt"


while IFS= read -r md5; do
    
    if grep -q "$md5" "$md5_no_hash_file"; then
        echo "MD5值 $md5 在 $md5_no_hash_file 中存在"  
    else
        echo "MD5值 $md5 在 $md5_no_hash_file 中不存在"  
    fi
done < "$md5_list_file"

```

**脚本 3**：该脚本的主要功能是：遍历读取一个包含`IP`地址的文件，如该文件名为：`mmyd`，然后对该文件里的每行`IP`地址执行`ping`操作，并检查丢包率。如果`ping`的丢包率为：100%，则输出该`IP`不可达的日志；如果`ping`不存在丢包率，则输出该`IP`可达的日志。

```bash
#!/bin/bash  


current_time=$(date +"%Y-%m-%d-%H:%M:%S")

start_time=$(date +%s)

for i in `cat mmyd`  
do
  ping=`ping -c 10 $i | grep loss | awk '{print $6}' | awk -F "%" '{print $1}'`
  
  Packet_Loss_Rate=`ping -c 10 $i | grep loss | awk '{print $6}'`

  if [ $ping -eq 100 ];then
  
    echo "$current_time 某某移动-活跃 IP 地址：$i ping 失败了，丢包率为：$Packet_Loss_Rate" >>/opt/jacktian/mmyd_ping.log
  else
    echo "$current_time 某某移动-活跃 IP 地址：$i ping 成功了！" >>/opt/jacktian/mmyd_ping.log
  fi
done


end_time=$(date +%s)


execution_time_seconds=$((end_time - start_time))


minutes=$((execution_time_seconds / 60))
seconds=$((execution_time_seconds % 60))

echo "执行完毕！该脚本执行时间共: $minutes 分钟 $seconds 秒"

```

**脚本 4**：该脚本用于在`Linux`系统中创建新用户，并为该用户设置密码、省份代码、目录权限以及`vsftpd`服务的配置。

```bash
#!/bin/sh

read -p "user:" user
read -p "passd:" pass
read -p "province:" province

useradd $user -d /bigdata/sftp/province/$province/


echo $pass | passwd --stdin $user

chown $user /bigdata/sftp/province/$province/ -R

chmod 750 /bigdata/sftp/province/$province/ -R

echo $user>>/etc/vsftpd/chroot_list

echo $user>>/etc/vsftpd/user_list

systemctl restart vsftpd.service

```

*   `read -p "user:" user`：提示用户输入一个用户名，并将输入的值存储在变量`user`中
*   `read -p "passd:" pass`：提示用户输入一个密码，并将输入的值存储在变量`pass`中。注意：在输入密码时，不会显示任何字符
*   `read -p "province:" province`：提示用户输入一个省份代码，并将输入的值存储在变量`province`中
*   `useradd $user -d /bigdata/sftp/province/$province/`：该命令将创建一个新用户，其用户名为之前输入的`user`，其家目录为：`/bigdata/sftp/province/$province/`
*   `echo $pass | passwd --stdin $user`：该命令会将之前输入的密码通过标准输入传给`passwd`命令，为新创建的用户设置密码
*   `chown $user /bigdata/sftp/province/$province/ -R`：该命令将更改新创建用户成为`/bigdata/sftp/province/$province/`目录及其子目录的所有者
*   `chmod 750 /bigdata/sftp/province/$province/ -R`：该命令会设置`/bigdata/sftp/province/$province/`目录及其子目录的权限为:`750`，也就是指：用户有读、写、执行的权限，而用户组只有读和执行的权限
*   `echo $user>>/etc/vsftpd/chroot_list`：该命令会将新创建的用户添加到`vsftpd`服务的`chroot`列表中。这表示当`vsftpd`服务在运行时，该用户会被限制在其自己的目录中，不能访问系统的其他目录
*   `echo $user>>/etc/vsftpd/user_list`：该命令会将新创建的用户添加到`vsftpd`服务的用户列表中。这表示当`vsftpd`服务在运行时，这个用户可以登录并访问系统。
*   `systemctl restart vsftpd.service`：重启`vsftpd`服务，使之前的所有配置生效

**脚本 5**：该脚本用于在`Linux`系统中创建新用户，并为该用户设置密码、省份代码、idcid、目录权限以及`vsftpd`服务的配置。跟如上脚本 4 略有差异。

```bash
#!/bin/sh

read -p "user:" user
read -p "passd:" pass
read -p "province:" province
read -p "idcid:" idcid

mkdir -p /bigdata/sftp/province/$province/$idcid

useradd $user -d /bigdata/sftp/province/$province/$idcid


echo $pass | passwd --stdin $user

chown $user /bigdata/sftp/province/$province/$idcid/ -R

chmod 750 /bigdata/sftp/province/$province/$idcid -R

echo $user>>/etc/vsftpd/chroot_list

echo $user>>/etc/vsftpd/user_list

systemctl restart vsftpd.service

```

**脚本 6**：该脚本首先自定义了省份编码列表为多个目录路径，循环遍历自定义的省份编码列表。对于每一个省份编码，脚本将其分割为三个部分：省份编码、运营商和数据上报类型。然后进入对应的省份目录。

执行`du -sh 2023-10-*` 命令，查询所有以`2023-10-`开头目录的大小，并将结果输出到一个名为`$province_code_file_size.txt`的文件中。

执行`for`循环，循环遍历所有子目录并查询每个子目录中的文件数量，然后将结果输出到一个名为`$province_code_file_count.txt`的文件中。

最后，脚本会返回到上级目录，以便对下一个省份编码下的文件大小及文件数量进行查询。所有的查询结果将保存在`/opt/`目录下的以省份编码命名的文件中。

```bash
#!/bin/bash  



  

provinces=("110000/dianxin/1024" "120000/liantong/1024" "130000/yidong/1024")  
  

for province in "${provinces[@]}"; do  
  
    
    province_code=$(echo $province | cut -d'/' -f1)  
    operator=$(echo $province | cut -d'/' -f2)  
    category=$(echo $province | cut -d'/' -f3)  
  
    
    cd /bigdata/sftp/province/$province_code/$operator/$category  
  
    
    du -sh 2023-10-* >> /opt/"$province_code"_file_size.txt  
  
    
    for date in $(ls -d */ | cut -d'/' -f1);  
        do  
            echo $date $(ls -1 $date | wc -l) >> /opt/"$province_code"_file_count.txt  
        done  
  
    
    cd ..  
  
done

```

**脚本 7**：该脚本用于循环查询特定省份、特定数据上报类型的文件大小及文件个数。跟如上脚本 6 略有差异。

```bash
#!/bin/bash



  

provinces=("110000" "120000" "130000")  
  

for province in "${provinces[@]}"; do  
  
    
    cd /bigdata/sftp/province/$province/1024
  
    
    du -sh 2023-10-* >> /opt/"$province"_file_size.txt  
  
    
    for date in $(ls -d */ | cut -d'/' -f1);   
        do   
            echo $date $(ls -1 $date | wc -l) >> /opt/"$province"_file_count.txt  
        done  
  
    
    cd ..  

done

```

**脚本 8**：该脚本主要用于为某些特定的 XML 文件在特定时间段内的修改情况，并把结果保存在日志中。

```bash
#!/bin/bash  
  

current_date=$(date +%Y-%m-%d)  
  

directory_path="/bigdata/sftp/province/110000/yidong/1024/$current_date"  
output_file="/opt/log_110000_yidong_$current_date.txt"  
cd "$directory_path" && ls -l *.xml | awk '{print $8,$9}' | grep -v '^$' | awk -F '[/:]' '{hour=substr($1,1,2); if ((hour >= "00" && hour < "08") || (hour >= "10" && hour < "12") || (hour >= "14" && hour <= "24")) print}' >> "$output_file" 
  

directory_path="/bigdata/sftp/province/120000/dianxin/1024/$current_date"  
output_file="/opt/log_120000_dianxin_$current_date.txt"  
cd "$directory_path" && ls -l *.xml | awk '{print $8,$9}' | grep -v '^$' | awk -F '[/:]' '{hour=substr($1,1,2); if ((hour >= "00" && hour < "08") || (hour >= "10" && hour < "12") || (hour >= "14" && hour <= "24")) print}' >> "$output_file"  
  

directory_path="/bigdata/sftp/province/130000/liantong/1024/$current_date"  
output_file="/opt/log_130000_liantong_$current_date.txt"  
cd "$directory_path" && ls -l *.xml | awk '{print $8,$9}' | grep -v '^$' | awk -F '[/:]' '{hour=substr($1,1,2); if ((hour >= "00" && hour < "08") || (hour >= "10" && hour < "12") || (hour >= "14" && hour <= "24")) print}' >> "$output_file"
  

exit

```

首先使用`date`命令获取当前日期，格式为：`YYYY-MM-DD`，并自定义`current_date`变量。

然后，自定义了一个目录路径`directory_path`和输出文件`output_file`。

使用`ls -l *.xml`列出所有以`.xml`结尾的文件，通过`awk '{print $8,$9}'`提取文件的修改时间和大小信息，`grep -v '^$'`过滤掉空行，`awk -F '[/:]' '{hour=substr($1,1,2); if ((hour >= "00" && hour < "08") || (hour >= "10" && hour < "12") || (hour >= "14" && hour <= "24")) print}'`筛选出每天的`00:00-07:59、10:00-11:59、14:00-23:59`时间段内修改的文件，并输出它们的修改时间和大小信息，将结果追加到指定的输出文件中。

**脚本 9**：该脚本主要用于监控系统资源使用情况，获取并记录了磁盘使用情况、CPU空闲情况、内存空闲情况和进程总数，并将这些打印信息输出到某个日志文件中。

```bash
#!/bin/bash

date=$(date +%Y-%m-%d-%H:%M:%S)


DISK_1=$(df -h | awk '{printf $NF} {printf "使用率："} {print $5} '| grep appslog | grep -v 'Filesystem')
DISK_2=$(df -h | awk '{printf $NF} {printf "使用率："} {print $5} '| grep bigdata | grep -v 'Filesystem')
DISK_3=$(df -h / | awk '{printf $NF} {printf "使用率："} {print $5} '| grep / | grep -v 'Filesystem')


CPU=$(top -n 1 | grep Cpu | awk 'BEGIN {printf"CPU 空闲使用率："} {print $8}')


MEMORY=$(free -h | awk 'BEGIN {printf"内存空闲使用率："} NR==2 {print $4}')


JINCHENG=$(ps aux | wc -l | awk 'BEGIN {printf"进程总数："} {print $1}')

echo -e "\n $date\n\n $DISK_1\n\n $DISK_3\n\n $CPU\n\n $MEMORY\n\n $JINCHENG\n" >> /opt/jacktian/inspection.log

exit

done

```

*   `date=$(date +%Y-%m-%d-%H:%M:%S)`：获取当前日期和时间，格式为：年-月-日-时:分:秒
*   `DISK_1、DISK_2、DISK_3`：该变量用于获取磁盘的使用情况。使用`df -h`命令获取磁盘信息，然后使用`awk`提取出使用率和文件系统名称。`grep`用于筛选出特定名称的磁盘（如：appslog 和 bigdata）
*   `CPU`：该变量获取`CPU`的空闲使用率。使用`top -n 1`命令获取系统状态，然后使用`grep`和`awk`提取出`CPU`的空闲使用率
*   `MEMORY`：该变量获取了内存的空闲使用率。使用`free -h`命令获取内存信息，然后使用`awk`提取出空闲内存的使用率
*   `JINCHENG`：该变量获取了系统的进程总数。使用`ps aux`命令获取进程信息，然后使用`wc -l`命令统计行数，即进程总数
*   `echo -e "\n $date\n\n $DISK_1\n\n $DISK_3\n\n $CPU\n\n $MEMORY\n\n $JINCHENG\n"`：这部分将上述所有的打印信息拼接在一起，并输出到日志文件中

**脚本 10**：该脚本主要用于定期循环连接 FTP 服务器的访问情况，当异常时通过企业微信机器人发送告警信息。

```bash
#!/bin/bash


FTP_IPS=("IP_1" "IP_2" "IP_3")


WEBHOOK_URL=https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=XXX



while :
do

       date=$(date +%Y-%m-%d-%H:%M:%S)


for ip in "${FTP_IPS[@]}"
do
  
  ftp -n $ip <<EOF

  # 退出 FTP 服务器
  exit

EOF

  
  if [ $? -ne 0 ];
  	  then
  
  
    curl --location --request POST ${WEBHOOK_URL} \
--header 'Content-Type: application/json' \
-d '{"msgtype": "markdown", "markdown": {"content": "'$date' FTP 访问异常:$ip"}}'
    echo "$date FTP 访问异常:$ip" >>/opt/ftp_check/ftp_check.log

    else
  
    echo "$date FTP 访问正常:$ip" >>/opt/ftp_check/ftp_check.log
    fi
done
exit
done

```

*   `FTP_IPS`：该变量中包含了多个`FTP`服务器的`IP`地址
*   `WEBHOOK_URL`：该变量为企业微信机器人的地址，需要将 XXX 部分替换为实际的企业微信机器人的`key`
*   使用了`while`循环来定期执行检测。在每次循环中，将获取当前日期和时间，遍历`FTP_IPS`列表。对于列表中的每个`IP`地址，脚本会尝试连接到`FTP`服务器
*   如果连接命令`ftp -n $ip`执行失败，则退出状态码不为：`0`，则表示`FTP`访问异常。在这种情况下，脚本会发送一个异常告警到企业微信机器人，并将相关信息写入日志文件
*   如果连接命令执行成功，退出状态码为：`0`，则表示`FTP`访问正常，脚本将不发送告警信息，只是在日志中记录正常访问的信息

* * *
