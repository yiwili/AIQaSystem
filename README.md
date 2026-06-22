# AIQaSystem (企业级RAG智能知识库)

面向企业私有数据的高性能RAG(Retrieval-Augmented Generation)智能知识库与流式问答引擎。本项目采用前后端分离架构，重点解决高维向量数据权限隔离、长文档上下文切分、高并发Embedding事务阻塞及SSE流式对话延迟等核心痛点，确保系统数据安全性、检索准确率与高可用交互体验。

## 🎥 系统演示

**文档解析与删除操作演示**

<video src="https://github.com/user-attachments/assets/bca0e3cf-78dc-4578-bb5f-183abde8618b" width="300" controls="controls"></video>

**RAG智能问答演示**

https://github.com/user-attachments/assets/8c41f025-49c5-45df-8eb5-c7e958afb289


## 🛠 技术栈

* **后端框架**：Java17、SpringBoot3、SpringSecurity、JPA
* **AI与向量引擎**：SpringAI、Milvus、DJL(Qwen2.5离线分词模型)


* **文档解析**：ApacheTika
* **安全与认证**：JWT(JSON Web Token)


* **前端框架**：React、ReactRouter、AntDesign
* **部署运维**：Docker、DockerCompose



## ✨ 核心特性

* **RAG流式架构与防腐层设计**：通过SpringAI编排RAG业务闭环，采用SSE协议实现低延迟流式响应。搭建严密的DTO/VO防腐层并实施全局异常处理(GlobalExceptionHandler)，隔离底层Entity，避免JPA级联查询引发的OOM风险。


* **物理级数据隔离与零信任安全**：将RBAC权限体系下沉至向量数据库。在Milvus写入时注入JSON角色标签，检索阶段强制附加多租户校验条件，实现高维索引级别的物理隔离，从底层阻断RAG场景的水平越权(IDOR)漏洞。
* **低内存解析与上下文感知切分**：采用ApacheTika的SAX事件流替代传统DOM解析，构建低内存提取管道，精准提取PDF物理页码以支持引用溯源。结合滑动窗口分块算法，实现整句级Overlap与块首元数据注入，降低大模型语境丢失与幻觉率。
* **高并发调度与Token精准管控**：集成离线分词器(如Qwen2.5)精确计算Token消耗，通过倒序滑动窗口截断历史会话防止Prompt超限。针对知识库构建场景进行长事务解耦，利用自定义IO密集型线程池配置(ThreadPoolConfig)异步编排Embedding与Milvus批量写入，提升系统吞吐量。



## 📂 项目结构

```text
AIQaSystem/
├── src/main/java/com/yiwilee/aiqasystem/
│   ├── common/         # 全局异常拦截、分页与通用响应结构[cite: 1]
│   ├── config/         # Milvus配置、SpringAI配置、Security及线程池配置[cite: 1]
│   ├── controller/     # 认证(Auth)、文档、角色、权限及问答(RagChat)API入口[cite: 1]
│   ├── model/          # DTO(数据传输对象)、VO(视图对象)与Entity(持久化实体)[cite: 1]
│   ├── repository/     # 数据库访问层(JPA Repository)[cite: 1]
│   ├── security/       # JWT过滤器(JwtFilter)与权限认证逻辑[cite: 1]
│   └── service/        # 核心业务逻辑(Chat、Document、Vector、Rag等实现)[cite: 1]
├── src/main/resources/
│   ├── sql/            # 数据库初始化SQL脚本(如role.sql)[cite: 1]
│   ├── tokenizers/     # 本地分词器模型配置文件(qwen-2.5-tokenizer.json等)[cite: 1]
│   └── application.yml # 核心环境配置文件[cite: 1]
├── Dockerfile          # 项目容器化构建脚本[cite: 1]
├── docker-compose.yml  # 基础设施(数据库、中间件)一键编排配置[cite: 1]
└── pom.xml             # Maven依赖管理[cite: 1]

```

## 🚀 快速开始

### 1. 环境准备

确保您的本地环境已安装以下基础组件：

* JDK17及以上版本
* Docker及DockerCompose


* Maven(项目内附带了mvnw包装器)


* Milvus向量数据库实例

### 2. 基础设施启动

通过项目中提供的Docker配置一键启动所需依赖服务：

```bash
docker-compose up -d

```

### 3. 构建与运行

克隆仓库后，使用内置的MavenWrapper执行项目构建：

```bash
# 构建项目
./mvnw clean install -DskipTests

# 运行Spring Boot服务
./mvnw spring-boot:run

```

### 4. 接口文档说明

项目集成了Swagger规范，服务启动后可通过浏览器访问API文档以进行接口测试与联调：

* 文档地址: `http://localhost:8080/swagger-ui/index.html` (参考SwaggerConfig配置)



## 🛡 开发与安全规范

* 提交代码前请确保所有单元测试(AiQaSystemApplicationTests)通过。


* API交互统一使用`Result`通用结构返回，切勿直接暴露数据库`Entity`结构至外部请求。


* 新增API请严格遵守`SysPermission`与`SysRole`定义的RBAC权限管控注解。
