#todo 

### emr  kerberos 快速安装

1 安装体外客户端参考

https://cloud.tencent.com/developer/article/1118756

2 安装kerberos

1、执行 source /etc/profile

2、执行klist指令，

a 如果出现提示：klist: No credentials cache found (filename: /tmp/krb5cc_0)，说明存在kerberos客户端环境，可以正常认证。

b 如果没有kerberos环境，输入以下指令安装kerberos命令：

yum install -y krb5-workstation krb5-libs

3 拷贝master节点的 /etc/krb5.conf配置文件至提交机上

4、拷贝master端的emr.keytab 至本地服务器（集群端emr.keytab存放在/var/krb5kdc/emr.keytab）

5、修改权限

chmod 644 /var/krb5kdc/emr.keytab

6、认证krb

su hadoop -c "kinit -kt /var/krb5kdc/emr.keytab -p hadoop/masterIp@EMR-GMZ8TDMV"

如果新增其它用户，需要执行步骤5的操作：

su userA-c "kinit -kt /var/krb5kdc/emr.keytab -p hadoop/masterIp@EMR-GMZ8TDMV"

su userB -c "kinit -kt /var/krb5kdc/emr.keytab -p hadoop/masterIp@EMR-GMZ8TDMV"

7、查看是否认证成功，su hadoop -c "klist"