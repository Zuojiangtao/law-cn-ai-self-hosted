# AI 法律助手 - docker私有化部署版

项目来源：https://github.com/lvwzhen/law-cn-ai

vercel部署教程： https://eibot3u32o.feishu.cn/docx/L46Pdp3fjouPUvxaNzPckKctno3

通过教程部署好后，领导不满意。他想要私有化部署，经过摸索成功实现，记录下过程。

> 我暂时使用supabase云服务，后续把supabase部署到本地再出一份文档。

## 1. 项目下载

克隆代码到本地： 

原项目：`git clone --depth=1 https://github.com/lvwzhen/law-cn-ai.git [project-name]`

改造后的项目： `git clone --depth=1 https://github.com/Zuojiangtao/law-cn-ai-self-hosted.git [project-name]`

## 2. 准备

1. 进入supabase进入项目，左侧菜单找到 `project setting` 然后找到 `API`;
2. 分别将 `Project URL` 和 `Project API keys` 下的 `anon pulic` 、 `service_role`拷贝到项目 `.env` 文件内 `NEXT_PUBLIC_SUPABASE_URL` 、`NEXT_PUBLIC_SUPABASE_ANON_KEY` 和 `SUPABASE_SERVICE_ROLE_KEY` 字段；
3. 将自己的 openai key 配置到 `.env` 的 `OPENAI_KEY` 字段;

按照上面部署教程将数据embeddings到supabase服务。这是为了使用supabase的服务。

## 3. 改造项目

1. `package.json` 构建脚本命令更改: `"build": "next build",`;
2. `next.config.js` 添加 `output: 'standalone'`;
3. Dockerfile脚本编写;

> 改造好的项目: https://github.com/Zuojiangtao/law-cn-ai-self-hosted

## 4. 本地运行验证

本地运行命令 `npm run dev`, 如果按上面的步骤来，应该可以再本地使用，问了法律相关问题会有答案返回。

## 5. 镜像发布

我将代码除去 `supabase` 的文件夹得内容上传并打包镜像。将docker镜像发布到服务集群。

## 6. Q&A

- 部署后后台报500或返回服务器繁忙？

    确定supabase是否有 `nods_page` 和 `nods_page_section` 这2张表。如果没有请到 `supabase -> migrations` 下复制sql到 `supabase -> peoject -> SQL Editor` 选择 `create table` 然后点击右下角 `run`。

    另外vercel中执行可能比较慢，可以手动再执行一次。到vercel项目点击deployment，然后再点击三个点执行 `redeploy` 可以重新部署。

    注意：要确保supabase有数据再改造 `package.json` 的build命令。

- 本地运行可以，但是docker镜像部署后仍然报错500？

  请求后端仍然报500。经过分析发现是我们的服务不能访问openapi接口导致的，给服务集群锁节点并配置透明代理就好了。
