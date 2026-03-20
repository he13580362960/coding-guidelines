# 前端项目规范
## 项目基础信息规范
1. 技术栈与框架：前端采用Vue 3 + TypeScript + Vite，UI组件为Element Plus，状态管理为Pinia，路由为Vue Router，HTTP为Axios。
2. 运行环境：Node 16+
3. 部署环境：静态资源构建后部署到Nginx，网关代理路径以README示例与环境变量为准。
4. 配置规范：默认使用.env.*管理不同环境。

## 技术架构规范
### 系统分层架构规范
1. 路由层：src/router定义模块路由并分模块维护（finance/logistics/warehouse等）。
2. 视图层：src/views按业务模块拆分页面，组件复用放置于src/components。
3. 状态层：Pinia集中在src/store/modules，用户、权限、配置、标签页等模块独立维护。
4. 服务层：src/api按业务域拆分，接口文件以业务域名文件夹维护。
5. 工具层：src/utils提供请求封装、存储、通用方法与格式处理。

### 3.2 接口设计规范
1. 请求封装：统一使用src/utils/request.ts，默认Content-Type为application/json，特殊场景（如登录）可切换为multipart/form-data。
2. 接口路径：统一以业务域前缀分组，如/ipd、/system、/oms、/wms等，具体前缀由VITE_APP_BASE_API拼接。
3. 返回格式：响应包含code、data、msg，列表可包含rows字段，成功码为'200'，未登录为'401'。code与msg在服务层统一处理，并只返回data数据到业务逻辑中。
4. 错误处理：默认弹出错误提示，若接口需静默处理需显式设置noErrorMessage。

### 3.3 依赖与配置规范
1. 依赖管理：使用npm统一管理依赖与版本，新增依赖需在package.json中明确记录。
2. 代码格式化：统一使用Prettier（.prettierrc.js）与ESLint（.eslintrc.js）。
3. 环境配置：.env.dev/.env.production等为唯一环境配置来源，禁止硬编码地址。

### 3.4 模块目录结构规范
1. 业务模块目录：src/views/{module}/{page}，保持与src/router/{module}一一对应。
2. API目录：src/api/{domain}，每个业务域有index.ts汇总导出。
3. 通用组件目录：src/components/{ComponentName}，包含index.vue与必要的types.ts。
4. 类型定义：页面类型优先放置在同级types.ts，公共类型放置在src/api或src/types。

## 代码规范
### 4.1 命名规范
1. 文件命名：页面与组件目录使用小写加中划线（如new-product），组件名使用PascalCase。
2. 接口命名：API方法使用动词+业务名的camelCase风格（如orgLevelPage、developDesignAudit）。
3. 类型命名：类型/接口使用PascalCase（如LoginForm、OperationListType），枚举使用PascalCase并在内部使用驼峰成员。

### 4.2 注释规范
1. 模块与接口注释：API接口需有简要说明，参数与返回值以块注释或JSDoc结构标明。
2. 业务规则注释：涉及状态码、流程分支、权限控制的关键逻辑需用中文注释说明业务含义。

### 4.3 代码格式规范
1. 缩进与字符集：统一使用2空格缩进、UTF-8编码、LF换行。
2. 语句格式：使用单引号、末尾分号、箭头函数参数省略括号（单参数）。

### 4.4 异常处理规范
1. 请求异常：统一在request拦截器处理，业务代码仅处理成功分支与必要的提示。
2. 登录失效：code为401或A0230时执行统一重定向与提示策略。

### 4.5 代码复用规范
1. 通用逻辑：公共逻辑下沉至src/utils或src/components，禁止页面内复制粘贴逻辑。
2. 枚举与字典：枚举与字典优先集中维护在filters.ts或后端数据字典接口中。