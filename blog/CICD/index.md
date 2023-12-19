# CI/CD
## 概念
  CI/CD 是一种持续的软件开发方法，您可以在其中持续构建、测试、部署和监视迭代代码更改。

  此迭代过程有助于减少您基于有错误或者失败的先前版本开发新代码的机会。 GitLab CI/CD 可以在开发周期的早期捕获错误，并帮助确保部署到生产的所有代码都符合您既定的代码标准。

## 基本结构
- ```.gitlab-ci.yml```文件: 包含了 CI/CD 管道的配置。
- Runners: Runners 是运行脚本的代理，这个代理可以运行在物理机器或虚拟实例上。
- Pipelines: 管道由 Jobs 和 stages 组成。Jobs 定义了你想做什么。Jobs 被分为多个 stages，每个 stage 至少包含一个Job
  1. 每个 Job 都包含一个 script 并属于一个阶段
  2. stage 描述了 Job 的顺序执行。如果有可用的运行程序，则单个阶段中的作业会合并运行。
  3. 使用关键字```needs```可以不按照 stage 顺序运行作业。这会创建一个有向无环图。 
- CI/CD 变量：帮助您实现自定义 Jobs
- CI/CD 组件：CI/CD 组件是可重用的单管道配置单元。可以使用它们来组成一个完整的管道配置或者较大管道的一小部分。
  
## 关键字
  [完整关键字](https://docs.gitlab.com/ee/ci/yaml/)

  1. stages, stage: 把 Job 进行分组。同一个阶段的 Job 可以并行运行，后面阶段的 Job 需要等待前面阶段的作业完成。如果作业失败，则认为整个阶段失败，后面阶段的 Job 不会开始运行。
  7. pages: 托管静态网站，需要使用 gitlab pages
  1. needs: 可以不按照 stage 顺序运行 Job
  2. rules: 指定何时运行或跳过Job
  3. cache，artifacts: 将跨作业和阶段信息保存在管道中。这些关键字是存储依赖项和作业输出的方法，即使为每个作业使用临时运行程序也是如此。**Job是独立的，彼此之间资源不共享。如果希望在一个 Job 中生成的文件在另一个 Job 中使用，需要将他们使用 artifacts 等存储。**
  4. default: 指定用于所有 Job 的配置项。此关键字通常用来定义在每个作业运行的 ```before_script``` 和 ```after_script``` 部分。
  5. image: 告诉 runner 去使用哪个 docker 容器去运行 Job
  8. allow_failure: 间歇性失败或者预计会失败的作业会降低工作效率或难以排除故障。 使用 ```allow_failure```让作业失败而不停止管道执行。
  7. dependencies: 控制各个作业中的 artifact 下载。
  10. include: 从其他 yaml 文件导入配置
  11. variables: 为管道中的所有作业定义 CI/CD 变量。

## stage
  默认有三个阶段：
  1. build
  2. test
  3. deploy
  

