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