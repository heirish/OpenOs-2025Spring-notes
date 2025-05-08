### 2025.05.06 unikernel基础与框架 
- 学习unikernel基础与框架录频
- 准备docker环境:
  ```
  git clone git@github.com:LearningOS/2025s-arceos-heirish.git
  cd 2025s-arceos-heirish
  cp ../2025s-rcore-heirish/Makefile ./
  将docker image改名为arceos-exercise
  运行make build_docker生成docker image
  ```
  - Makefile内容如下
  ```
    DOCKER_NAME ?= arceos-exercise
    .PHONY: docker build_docker
    
    docker:
            docker run --rm -it -v ${PWD}:/mnt -w /mnt ${DOCKER_NAME} bash
    
    build_docker: 
            docker build -t ${DOCKER_NAME} .
    
    #fmt:
    #       cd easy-fs; cargo fmt; cd ../easy-fs-fuse cargo fmt; cd ../os ; cargo fmt; cd ../user; cargo fmt; cd ..
  ```
- Notes
  - 主流内核模式
    ![](images/kernel-types.png)
  - unikernel实验路线
    ![](images/unikernel-exercise-roadmap.png)
  - 实验一框架组件构成与协作流程:
    ![](images/component-archi.png)
  - 小结
    ![](images/summary1.png)
- exercise 1: 通过在axhal/src/lib.rs的write_bytes里hardcoding，让终端打印字符变成绿色(在输出字会前后加ANSI转义格式)
  ```
  pub fn write_bytes(bytes: &[u8]) {
        let prefix_bytes = "\u{1B}[32m".as_bytes();
        let suffix_bytes = "\u{1B}[m".as_bytes();
        for c in prefix_bytes {
            putchar(*c);
        }
        for c in bytes {
            putchar(*c);
        }
        for c in suffix_bytes {
            putchar(*c);
        }
    }
  ```
  - 这个会导致github ci测试通不过，因为测试shell是通过`tail -n1 output.txt | grep keyword`来判断的，如果按上述的在write_bytes里加前后缀，换行符在输出实际bytes内容时就写了，这样后面的suffix就会跳到下一行，因此实际最后测试找的keyword在倒数第二行，如果在write_bytes里写，需要处理最后一个字节是"\n"的情况，后面改成在axstd的macro.rs中实现
  ```
  #[macro_export]
  macro_rules! print {
      ($($arg:tt)*) => {
          $crate::io::__print_impl(format_args("\u{1B}[32m{}\u{1B}[m", format_args!($($arg)*)));
      }
  }
  
  /// Prints to the standard output, with a newline.
  #[macro_export]
  macro_rules! println {
      () => {
          $crate::io::__print_impl("\n");
      };
      ($($arg:tt)*) => {
          $crate::io::__print_impl(format_args!("\u{1B}[32m{}\u{1B}[m\n", format_args!($($arg)*)));
      }
  }
  ```
### 2025.05.07 unikernel基础与框架
- 完成exercise 2
  - 主要参考std中的map实现，也使用了hashbrown库
  - vec是在alloc crate里提供了，而hashmap需要自己实现.
  - 为了不改动exercise中的代码，hashmap在axstd::collections里，在axstd/src/lib.rs中将alloc的collections进行了改名，然后声明另一个collections将alloc::collections和axstd::axcollections包含进去
    ```
    pub use alloc::{boxed, collections as alloc_collections, format, string, vec};
    mod axcollections;
    pub mod collections {
      pub use super::alloc_collections::*;
      pub use super::axcollections::HashMap;
    }
    ```
  - globalallocator struct己定义，要使vec,string等可用，globalallocator必须实现trait GlobalAlloc并使用`#[cfg_attr(all(target_os = "none", not(test)), global_allocator)]`定义collections默认使用的global_allocator,refer:https://docs.rust-embedded.org/book/collections/
### 2025.05.08 unikernel基础与框架 内存映射
![](images/arcos-mem0.png)
![](images/arcos-mem1.png)
![](images/arcos-mem.png)
![](images/arcos-mem2.png)
- 第一阶段:
    - arceos/modules/axhal/src/platform/riscv64_qemu_virt/boot.rs 
    - 映射到的物理地址为0x8000_0000~0xC000_0000, 物理页page size为4K(2^12), 因此这个物理地址对应PPN为0x80000 ~ 0xC0000
    - 所有的常量定义都在arceos/platforms/riscv64-qemu-virt.toml中，但是怎么最终在编译时用到的?
      > 在arceos/modules/axconfig/build.rs中读取上面这个toml文件，并生成config.rs的，生成的文件在target/riscv64gc-unknown-none-elf/release/build/axconfig-ae9a66e9535d7184/out/config.rs
      ```
      /// Base virtual address of the kernel image.
      pub const KERNEL_BASE_VADDR: usize = 0xffff_ffc0_8020_0000;
      /// Linear mapping offset, for quick conversions between physical and virtual
      /// addresses.
      pub const PHYS_VIRT_OFFSET: usize = 0xffff_ffc0_0000_0000;
      ```
    - boot时，只有一级页表BOOT_PT_SV39, 512个PTE， 每个PTE直接映射到1G的物理block
      ```
      #[link_section = ".data.boot_page_table"]
      static mut BOOT_PT_SV39: [u64; 512] = [0; 512];
      
      unsafe fn init_boot_page_table() {
          // 0x8000_0000..0xc000_0000, VRWX_GAD, 1G block
          BOOT_PT_SV39[2] = (0x80000 << 10) | 0xef;
          // 0xffff_ffc0_8000_0000..0xffff_ffc0_c000_0000, VRWX_GAD, 1G block
          BOOT_PT_SV39[0x102] = (0x80000 << 10) | 0xef;
      }
      
      unsafe fn init_mmu() {
          let page_table_root = BOOT_PT_SV39.as_ptr() as usize;
          satp::set(satp::Mode::Sv39, 0, page_table_root >> 12);
          riscv::asm::sfence_vma_all();
      }
      ```
    - pte_index = vaddr/1G后保留最后9位,即取30~38位, 2^9 = 512
    - pte_index for vaddr 0x8000_0000: 第30~38位结果为0x2 
    - pte_index for vaddr 0xffff_ffc0_8000_0000: 第30~38位为0x102(100_0000_10)
    - **问题：只看到这里做了boot_page_table的初始化,什么时候用的呢?怎么通过虑拟地址访问到物理地址的?**
          > 只在arceos/modules/axhal/src/arch/riscv/mod.rs看到有读satp中的PPN，但是唯一用到它的地方是用来重新写PPN到satp,只是判断当前的PPN与将要写入的是否是一样的地址。 
- 第二阶段:arceos/modules/axruntime/src/lib.rs -> arceos/modules/axmm/src/lib.rs
- 练习arceos/tour/u_3_0/src/main.rs里只是通过phys_to_virt将pflash的物理地址通过加一个offset转成了虚地址（0x2200_0000 + 0xffff_ffc0_0000_0000 = 0xFFFFFFC022000000).
- tour/u_3_0的cargo.toml中不加paging feature,会报错
  ```
  Try to access dev region [0xFFFFFFC022000000], got [  0.134963 0 axhal::arch::riscv::trap:24] No registered handler for trap PAGE_FAULT
  [  0.141599 0 axruntime::lang_items:5] panicked at modules/axhal/src/arch/riscv/trap.rs:25:9:
  Unhandled Supervisor Page Fault @ 0xffffffc080203f60, fault_vaddr=VA:0xffffffc022000000 (READ):
  TrapFrame {
  ```
  加了paging后正确的输出为
  ```
  ry to access dev region [0xFFFFFFC022000000], got 0x646C6670
  Got pflash magic: pfld
  ```
  是因为virtaddr 0xFFFFFFC022000000并没有做映射
- 还是要仔细读一下https://oslearning365.github.io/arceos-tutorial-book, 比课程视频讲得更详细