### docker相关
```
#重命令docker image
docker image tag <old_name>:<old_tag> <new_name>:<new_tag>

#attach到正在运行的container
docker attach <container-id>

#进入正在运行的docker并新建一个bash 进程
docker exec -it <container-it> /bin/bash
```
### rust相关
```
###查看rust支持的所有target
rustup target list

#查看己安装的target
rustup target list --installed
```
### 其它
```
#查看gcc默认的链接脚本
ld -verbose

#预处理阶段）
gcc -E hello_world.c -o hello_world.i
#编译阶段）
gcc -S hello_world.i -o hello_world.s
#汇编阶段）
gcc -c hello_world.s -o hello_world.o
#链接阶段）
#这里面使用-lc链接C库
gcc hello_world.o -lc -o  hello_world

#读取elf文件信息
readelf -a target/riscv64gc-unknown-none-elf/debug/os | less

rust-objdump -S target/riscv64gc-unknown-none-elf/debug/os
```