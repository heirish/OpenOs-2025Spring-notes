### 2025 04.13 准备第二阶段实验环境

- 准备第二阶段作业环境

  - 课程页面点击链接领取作业，创建仓库:https://github.com/LearningOS/2025s-rcore-heirish

  - clone到本地并配置docker环境

    ```
    ##参考:1.作业仓库README, 2.https://learningos.cn/rCore-Tutorial-Guide-2025S/0setup-devel-env.html
    git@github.com:LearningOS/2025s-rcore-heirish.git
    cd 2025s-rcore-heirish
    su root
    make docker
    ```

- 准备博客

  ```
  #fork from https://github.com/rcore-os/blog 
  git@github.com:heirish/blog.git
  cd blog
  sudo npm install hexo-cli -g 
  sudo npm install hexo --save
  hexo n OpenOs-2025Spring-heirish
  ```
  - 注意：虚拟机中不能存放到与host windows共享的目录下，要放到虚似机系统本身的目录。因为与host windows共享的目录是不能创建软链接的， 否则安装hexo时会报错

### 2025.04.15
- 学习第一节课录频
  - 应用程序执行环境:

  ![](images/app-exec-env.png)
  - toolchain tree tuple: CPU-OS-runtimelib
- 实验
  ```
  cd 2025s-rcore-heirish
  su root
  make docker
  cd os
  make run
  ```
  - 遇到的问题:发现make run执行完成之后并没有预期的rustsbi和Hello World的输出。没有任何输出，像是卡在某在地方，ctrl+c也退出不了。最后只能用docker exec -it的方式进入运行的docker, 强制kill进程。
   解决思路如下:
    - 检查了仓库里的代码与课程和tutorial里的一样，并没有需要修改的地方。并且没有rustsbi的输出，怀疑是rustsbi的问题或是docker环境的问题。
    - 对比两个仓库的rustsbi
       ```
        heirish@heirish-VirtualBox:/mnt/vmshare/OS/qinghua$ md5sum rCore-Tutorial-v3/bootloader/rustsbi-qemu.bin
        eae6f6ba9c894fb2aec444162f1efe18  rCore-Tutorial-v3/bootloader/rustsbi-qemu.bin
        heirish@heirish-VirtualBox:/mnt/vmshare/OS/qinghua$ md5sum 2025s-rcore-heirish/bootloader/rustsbi-qemu.bin
        526a492a1592503f4c959f24a52630cc  2025s-rcore-heirish/bootloader/rustsbi-qemu.bin
       ```
     - 查看了作业仓库2025s-rcore-heirish与导学阶段时的rcore仓库rCore-Tutorial-v3里的Dockerfile,发现也不一样。
     - 最后重新进入作业仓库，再重新builder docker,解决问题