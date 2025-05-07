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
### 2-25.05.07 unikernel基础与框架
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