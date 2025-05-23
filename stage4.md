### 2025.05.20 领取四阶段小任务
-  改进axvisor tutorial book(writing)文档 https://arceos-hypervisor.github.io/doc/
- TODO: 看现有文档 -> 复现 -> 启动arceos -> boot，完善文档。
### 2025.05.21 准备4阶段环境
```
heirish@heirish-VirtualBox:~/myproj/OS/qinghua/stage4$ tree -L 1
.
├── arceos
├── axvisor
├── axvisor-doc
└── scripts -> axvisor-doc/scripts/
heirish@heirish-VirtualBox:~/myproj/OS/qinghua/stage4/axvisor-doc$ git remote -v
origin  git@github.com:heirish/axvisor-doc.git (fetch)
origin  git@github.com:heirish/axvisor-doc.git (push)
heirish@heirish-VirtualBox:~/myproj/OS/qinghua/stage4/axvisor-doc$ git branch
* heirish-dev
  main
```
### 2025.05.22~05.23 复现quickstart, axvisor运行arceos
- 从文件系统加载guest os,成功启动。但是客户机打印的内容为乱码。axvisor#134己记录
  - 尝试用最新的opensbi v1.6，也是乱码；用rcore里的rustsbi, axvisor启动不了，直接卡死。
- 从mem加载guest os, 成功启动。可以在运行axvisor的命令里加上BLK=n, 就不用给一个空的DISK_IMG了, 命令如下
  ```
  make ACCCEL=n ARCH=riscv64 LOG=info \
    VM_CONFIGS=$SRC_CONFIG BLK=n \
    MEM=1G FEATURES=page-alloc-64g run
  ```
### 2025.05.23 开始小任务升级rust工具链
- 升级rust工具链2025-0520：组件化[axvisor hypervisor](https://github.com/arceos-hypervisor/axvisor)的底层[arceos](https://github.com/arceos-hypervisor/arceos)，可学习参考[arceos-org的arceos的commit](https://github.com/arceos-org/arceos/commit/f4428b49970c9b4d871d4dd37cf3e0f1cc200bb1)
  - fork https://github.com/arceos-org/arceos
  - clone到本地，在arceos目录下执行"make run"，如果能看到最后一行的“Hello, world!”，表明arceos可以用rust 2025-05-20-nightly正常编译并运行
  - 分析 git commit f4428b49970c9b4d871d4dd37cf3e0f1cc200bb1的具体内容, 即学习 arceos如何修改，修改了哪些文件
  - clone https://github.com/arceos-hypervisor/arceos 到本地，根据3的学习，在 vmm branch上，完成类似的修改，通过“make run”。
  
- 用nightly-2025-05-20 toolchain编译并运行最新arceos-org/arceos
  ```
  root@f48b7fd9c82d:/mnt/arceos# rustup toolchain list
  stable-x86_64-unknown-linux-gnu
  nightly-x86_64-unknown-linux-gnu
  nightly-2025-05-20-x86_64-unknown-linux-gnu (active, default)
  
  root@f48b7fd9c82d:/mnt/arceos# git remote -v
  origin  https://github.com/arceos-org/arceos.git (fetch)
  origin  https://github.com/arceos-org/arceos.git (push)
  root@f48b7fd9c82d:/mnt/arceos# git log
  commit f4428b49970c9b4d871d4dd37cf3e0f1cc200bb1 (HEAD -> main, origin/main, origin/HEAD)
  Merge: 29bdc926 56b75167
  Author: Yuekai Jia <equation618@gmail.com>
  Date:   Fri May 23 12:57:16 2025 +0800
  
      Merge pull request #228 from arceos-org/upgrade-toolchain
      
      Upgrade toolchain to nightly-2025-05-20
  

  root@f48b7fd9c82d:/mnt/arceos# make run ARCH=riscv64
  axconfig-gen configs/defconfig.toml configs/platforms/riscv64-qemu-virt.toml  -w smp=1 -w arch=riscv64 -w platform=riscv64-qemu-virt -o "/mnt/arceos/.axconfig.toml" -c "/mnt/arceos/.axconfig.toml"
      Building App: helloworld, Arch: riscv64, Platform: riscv64-qemu-virt, App type: rust
  cargo -C examples/helloworld build -Z unstable-options --target riscv64gc-unknown-none-elf --target-dir /mnt/arceos/target --release  --features "axstd/log-level-warn"
      Finished `release` profile [optimized] target(s) in 0.39s
  rust-objcopy --binary-architecture=riscv64 examples/helloworld/helloworld_riscv64-qemu-virt.elf --strip-all -O binary examples/helloworld/helloworld_riscv64-qemu-virt.bin
      Running on qemu...
  qemu-system-riscv64 -m 128M -smp 1 -machine virt -bios default -kernel examples/helloworld/helloworld_riscv64-qemu-virt.bin -nographic
  
  OpenSBI v1.3
     ____                    _____ ____ _____
    / __ \                  / ____|  _ \_   _|
   | |  | |_ __   ___ _ __ | (___ | |_) || |
   | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
   | |__| | |_) |  __/ | | |____) | |_) || |_
    \____/| .__/ \___|_| |_|_____/|___/_____|
          | |
          |_|
  
  Platform Name             : riscv-virtio,qemu
  Platform Features         : medeleg
  Platform HART Count       : 1
  Platform IPI Device       : aclint-mswi
  Platform Timer Device     : aclint-mtimer @ 10000000Hz
  Platform Console Device   : uart8250
  Platform HSM Device       : ---
  Platform PMU Device       : ---
  Platform Reboot Device    : sifive_test
  Platform Shutdown Device  : sifive_test
  Platform Suspend Device   : ---
  Platform CPPC Device      : ---
  Firmware Base             : 0x80000000
  Firmware Size             : 322 KB
  Firmware RW Offset        : 0x40000
  Firmware RW Size          : 66 KB
  Firmware Heap Offset      : 0x48000
  Firmware Heap Size        : 34 KB (total), 2 KB (reserved), 9 KB (used), 22 KB (free)
  Firmware Scratch Size     : 4096 B (total), 760 B (used), 3336 B (free)
  Runtime SBI Version       : 1.0
  
  Domain0 Name              : root
  Domain0 Boot HART         : 0
  Domain0 HARTs             : 0*
  Domain0 Region00          : 0x0000000002000000-0x000000000200ffff M: (I,R,W) S/U: ()
  Domain0 Region01          : 0x0000000080040000-0x000000008005ffff M: (R,W) S/U: ()
  Domain0 Region02          : 0x0000000080000000-0x000000008003ffff M: (R,X) S/U: ()
  Domain0 Region03          : 0x0000000000000000-0xffffffffffffffff M: (R,W,X) S/U: (R,W,X)
  Domain0 Next Address      : 0x0000000080200000
  Domain0 Next Arg1         : 0x0000000087e00000
  Domain0 Next Mode         : S-mode
  Domain0 SysReset          : yes
  Domain0 SysSuspend        : yes
  
  Boot HART ID              : 0
  Boot HART Domain          : root
  Boot HART Priv Version    : v1.12
  Boot HART Base ISA        : rv64imafdch
  Boot HART ISA Extensions  : time,sstc
  Boot HART PMP Count       : 16
  Boot HART PMP Granularity : 4
  Boot HART PMP Address Bits: 54
  Boot HART MHPM Count      : 16
  Boot HART MIDELEG         : 0x0000000000001666
  Boot HART MEDELEG         : 0x0000000000f0b509
  
         d8888                            .d88888b.   .d8888b.
        d88888                           d88P" "Y88b d88P  Y88b
       d88P888                           888     888 Y88b.
      d88P 888 888d888  .d8888b  .d88b.  888     888  "Y888b.
     d88P  888 888P"   d88P"    d8P  Y8b 888     888     "Y88b.
    d88P   888 888     888      88888888 888     888       "888
   d8888888888 888     Y88b.    Y8b.     Y88b. .d88P Y88b  d88P
  d88P     888 888      "Y8888P  "Y8888   "Y88888P"   "Y8888P"
  
  arch = riscv64
  platform = riscv64-qemu-virt
  target = riscv64gc-unknown-none-elf
  build_mode = release
  log_level = warn
  smp = 1
  
  Hello, world!
  root@f48b7fd9c82d:/mnt/arceos# 
  ```