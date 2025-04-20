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
### 2025.04.17 ~ 2025.04.20 ch2
- 学习第二节课录频及rcore-tutorial-guide-2025s第二章(录频音画不同步基本听不太懂，主要自己看slides和tutorial)
  - 批处理系统 (Batch System):出现于计算资源匮乏的年代，其核心思想是： 将多个程序打包到一起输入计算机；当一个程序运行结束后，计算机会 自动 执行下一个程序。
  - 应用程序难免会出错，如果一个程序的错误导致整个操作系统都无法运行，那就太糟糕了。 保护 操作系统不受出错程序破坏的机制被称为 特权级 (Privilege) 机制， 它实现了用户态和内核态的隔离。
  - 代码仓库
    - make run执行过程: os/Makefile -> to user , user/Makefile -> python3 user/build.py -> objcopy -> back to os, cargo run build.rs -> cargo run build -> objcopy -> start quemu to run os.bin
    - user/src目录下是user_lib的代码，user/src/bin目录下是所有ch的测试bin代码。本章ch2总共有7个(4个异常退出的，3个正常退出的), 都需要链接user_lib库，但是在编译流程中未看到有编译这个库?(cargo rustc --bin会触发)
      ```
      使用 cargo rustc --bin 时：
        - Cargo 会先处理整个依赖图（包括 user_lib）
        - 然后专门编译指定的 binary
        - 等同于 cargo build --bin 但允许传递额外 rustc 参数
        - user_lib的编译并未生成最终目标库，而是作为app的依赖进行编译生成中间库，存放在target/riscv64gc-unknown-none-elf/release/deps目录下。可以通过在cargo rustc编译中添加-v查看编译详细信息。
      ```
    - 第二章的批处理系统主要代码实现都在os/batch.rs和os/trap中。 
    - 所有测试app执行流程如下:要么是通过trap_handler运行下一个app, 要么是通过sys_exit系统调用运行下一个app. ![](images/ch2-batch-flow.png)
- Notes:
  - 编译相关
    - build.rs 是一个 Rust 程序，会在正式编译你的 crate 之前被编译和运行。主要用于需要在编译时生成代码、检测系统环境、链接外部库等场景。
    - cmake参数-Clink-args=-Ttext=0x80400000, 告诉链接器将代码段(text section)的加载地址设置为指定值, 这里为0x80400000。编译完成后通过readelf -a <bin>就可以看到Entry point address为0x80400000.
    - GNU汇编指令
      ```
      # 定义一个或多个 64 位（8 字节）的数值
      .quad <expression> [,expression]... # 每个expression会被计算并存储为一个64bit的值。

      # 在汇编过程中直接包含二进制文件的内容到目标文件中
      .incbin <file-name>[,<offset>[,<length>]]
      ```
    - 让cargo build打印详细信息:`cargo build -v`
      - 完整的编译器调用命令
      - 每个编译步骤的详细参数
      - 依赖项的编译过程
    - RISCV 特权级切换
      ![](images/model-switch.png)
      当 CPU 执行完一条指令并准备从用户特权级 陷入（ Trap ）到 S 特权级的时候，硬件会自动完成如下这些事情：
      - sstatus 的 SPP 字段会被修改为 CPU 当前的特权级（U/S）。
      - sepc 会被修改为 Trap 处理完成后默认会执行的下一条指令的地址。
      - scause/stval 分别会被修改成这次 Trap 的原因以及相关的附加信息。
      - CPU 会跳转到 stvec 所设置的 Trap 处理入口地址，并将当前特权级设置为 S ，然后从Trap 处理入口地址处开始执行。

      而当 CPU 完成 Trap 处理准备返回的时候，需要通过一条 S 特权级的特权指令 sret 来完成，这一条指令具体完成以下功能：
      - CPU 会将当前的特权级按照 sstatus 的 SPP 字段设置为 U 或者 S ；
      - CPU 会跳转到 sepc 寄存器指向的那条指令，然后继续执行。
- 问题
  - 由于是在docker中跑的实验， 并且项目的Cargo.toml中有指定target, 在host pc上打开项目后rust-analyzer报错。只能在虚拟机中按实验仓库的脚本配置rust环境, 然后vscode通过remote ssh连接到虚拟机(好像可以直接vscode连接到virtualbox中的docker?后面再研究)
  - 编译时报错" linking with `rust-lld` failed: exit status: 1", 并且git branch时会报错"fatal:detected dubious ownership in reprository...", 按提示执行`git config --global --add safe.directory <dir>`, 然后再重新编译即可
### 2025.04.20 ch3
- 学习第二节课录频及rcore-tutorial-guide-2025s第二章(录频音画不同步基本听不太懂，主要自己看slides和tutorial)
  - 代码仓库:
    - 相比于ch2, ch3的user/app的entry point address有变化(build.py)，由固定的0x80400000变成了0x80400000+0x2000*i(i为app index), 也就是说一个app最大size为0x20000
    - 相比于ch2一次加载一个app到内存，ch3在启动时一次性将所有app加载到内存(0x80400000+0x20000*i), 通过src/loader.rs实现
    - 与ch2相比，ch3多了app之间的同级(User态)切换，在task模块中实现
    - 与ch2相比，ch3多了task调度系统，在timer.rs和trap_handler的Trap::Interrupt(Interrupt::SupervisorTimer) 分支实现 
- Notes
  - 分时多任务系统，它能并发地执行多个用户程序，并调度这些程序。为此需要实现
    - 一次性加载所有用户程序，减少任务切换开销；
    - 支持任务切换机制，保存切换前后程序上下文；
    - 支持程序主动放弃处理器，实现 yield 系统调用；
    - 以时间片轮转算法调度用户程序，实现资源的时分复用。
- 遇到的問題
  - 每次新進docker后執行cargo命令時都會同步"info: syncing channel updates for", 并且下載時很慢。進docker之后執行下面兩句設置環境變量. TODO:在啟動docker時自動設置？
    ```
    export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
    export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
    ```
  - 代碼修改后，`make run BASE=0`和`make run BASE=1`都能通過。但是執行`make run BASE=2`通不過，在get_time時返回的值始終為0. 并且在ch3_trace.rs中的第一個測試count_syscall(SYSCALL_GETTIMEOFDAY)返回的值很大。
    - 剛開始懷穎是實現的sys_trace系統調用有問題，檢查幾遍后也沒找到異常
    - 休息一會后，想了下出問題的現象，可能是棧出問題了。回想在os/entry.asm中有設置內核棧大小。而我實現的task_syscalls會導致struct TaskManager增大64k(MAX_APP_NUM * MAX_SYSCALL_ID * sizeof(usize) = 16 * 512 * 8byte = 16 * 4k). 看了下原來的棧大小設置成了64k, 也就是只是我添加的這部分就把目前的棧空間占完了。增加kernel stack大小后test通過。修改如下
    ```
    boot_stack_lower_bound:
        # .space 4096 * 16 
        .space 4096 * 32 # after added task_syscalls to tasksControlBlock, TaskManager will increase MAX_APP_NUM * MAX_SYSCALL_ID * sizeof(usize) = 16 * 512 * 8byte = 16 * 4k = 64k
        .globl boot_stack_top
    boot_stack_top:
    ```  
    - 沒有按目前只有的5個syscall去設計task_syscalls大小，而是按更大的值512(>最大的syscall_id:410)設計數組長度, 只是為了簡化代碼。