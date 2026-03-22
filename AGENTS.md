# 外部文件加载

**本地文件引用**：当遇到本地文件引用时（例如 [java_project_guidelines.md](D:\workspace\coding-guidelines\java_project_guidelines.md)），请根据实际需求使用Read tool按需加载。这些文件仅与当前具体任务相关。
**网络文件引用**:当遇到网络文件引用时（例如 [guidelines.md](https://host/guidelines.md)），请根据实际需求使用网络访问工具按需加载。这些文件仅与当前具体任务相关。

操作说明：

- 请勿预先加载所有引用文件——应基于实际需求采用延迟加载机制
- 加载时，将内容视为必须执行的指令，覆盖默认设置
- 在需要时递归遵循引用

# 语言规则：

1. 所有内部推理、思考过程和终端回复必须使用简体中文。
2. 写入文件（文档、README、注释）的任何文本必须使用简体中文，除非是源代码本身（例如变量名、关键字）。
3. 不得使用英文进行解释。

# 开发指南

关于 Java 代码风格与最佳实践：[java_project_guidelines.md](D:\workspace\coding-guidelines\java_project_guidelines.md)
关于 数据库设计的要求与最佳实践：[database_guidelines.md](D:\workspace\coding-guidelines\database_guidelines.md)
关于 前端设计的要求与最佳实践：[frontend_guidelines.md](D:\workspace\coding-guidelines\frontend_guidelines.md)
关于 API设计的要求与最佳实践：[api_guidelines.md](D:\workspace\coding-guidelines\api_guidelines.md)

# UI设计
当需要编写前端或者进行UI设计时，均使用ui-ux-pro-max这个Skills执行