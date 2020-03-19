# zhang

# day27磁盘管理下部

![](https://cdn.nlark.com/yuque/0/2019/png/194754/1546516031364-8facb71c-4091-43f1-ab1c-8014e1afceea.png#align=left&display=inline&height=298&originHeight=761&originWidth=2113&status=done&width=827)

[day27磁盘管理下部.xmind](https://www.yuque.com/attachments/yuque/0/2019/xmind/194754/1554020415432-d2322ff4-99e7-45aa-a0bb-2a5562aa98e9.xmind?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2019%2Fxmind%2F194754%2F1554020415432-d2322ff4-99e7-45aa-a0bb-2a5562aa98e9.xmind%22%2C%22name%22%3A%22day27%E7%A3%81%E7%9B%98%E7%AE%A1%E7%90%86%E4%B8%8B%E9%83%A8.xmind%22%2C%22size%22%3A141682%2C%22type%22%3A%22application%2Fvnd.xmind.workbook%22%2C%22progress%22%3A%7B%22percent%22%3A0%7D%2C%22status%22%3A%22done%22%2C%22percent%22%3A0%2C%22id%22%3A%221bs5D%22%2C%22card%22%3A%22file%22%7D)

![](https://cdn.nlark.com/yuque/0/2019/png/194754/1546516138799-98763c99-e90b-4784-9a73-97a9af2c53c3.png#align=left&display=inline&height=892&originHeight=1034&originWidth=959&status=done&width=827)

![](https://cdn.nlark.com/yuque/0/2019/png/194754/1546516088591-caac8ddd-d134-4188-9387-3b0367cf22fa.png#align=left&display=inline&height=247&originHeight=2276&originWidth=7632&status=done&width=827)

1 磁盘组成<br />2 RAID 级别<br />3 如何让系统更安全
```bash
# 和md5类似 ，加密算法更复杂
[root@oldboy01 yum.repos.d]# sha
sha1sum    sha224sum  sha256sum  sha384sum  sha512sum  


http://lidao.blog.51cto.com/3388056/1910889
如何防止系统中木马
```

4 磁盘接口<br />5 如何进行计算

今天：<br />1 如何查看内存使用情况<br />2 磁盘分区<br />3 进行分区 格式化  挂载<br />4 故障案例<br />5 一大波磁盘相关命令

<a name="29f46852"></a >
# 查看内存使用情况
```bash
[root@oldboy01 yum.repos.d]# free -h
            total       used       free     shared    buffers     cached
Mem:          474M       389M        84M       232K        15M       286M
-/+ buffers/cache:        87M       387M
Swap:         767M       920K       767M
剩余内存 387M = free 84M + 15M buffers +286M cached
```
linux把你使用过的命令或文件 替你缓存（buffer cache）起来，提高下次使用速度<br />写buffer<br />读cache

<a name="725b8303"></a >
# 磁盘分区fdisk
linux启动流程：

      1. 开机自检B
