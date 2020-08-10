---
title: Linux命令速记
---

1. 使用密码进行ssh连接

   ```powershell
   ssh root@127.0.0.1 -p 8895
   ```

   其中```root```为用户名，- p 8895 是通过8895端口连接，如果不带 -p即使用默认的22端口连接服务器

2. scp命令 Linux和Windows文件互传

   ```
   1.本地文件(夹)传到远程Linux：
   scp -rp /d/data root@127.0.0.1:/data
   
   2. 本地Windows获取远程Linux文件：
   scp -P 8868 root@127.0.0.1:/data/1.sh /d/data
   ```

   