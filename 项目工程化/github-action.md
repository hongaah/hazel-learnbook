## 使用 Github Actions 自动部署 VitePress 到 Github Pages

### 1. 准备工作

#### 1.1 获取 GitHub Personal Access Token (PAT)
1. 访问 [GitHub Token 设置页面](https://github.com/settings/tokens)
2. 点击 "Generate new token" → "Generate new token (classic)"
3. 设置权限范围：
   - 必须勾选 `repo` 权限（包含所有仓库权限）
   - 如需访问私有仓库，还需勾选 `workflow` 权限
4. 生成后务必立即复制保存（页面关闭后将无法再次查看）

#### 1.2 添加 Token 到仓库 Secrets
1. 进入你的 GitHub 仓库 → Settings → Secrets and variables → Actions
2. 点击 "New repository secret"
3. 名称填写 `ACCESS_TOKEN`，值粘贴刚才复制的 token

### 2. 项目配置

#### 2.1 VitePress 基础配置
确保 `docs/.vitepress/config.js` 中已设置正确的 `base` 路径：
```javascript
export default {
  base: '/your-repo-name/', // 如果是用户/组织页面则用 '/'
  // 其他配置...
}
```

#### 2.2 使用 pnpm 的注意事项
1. 在项目根目录添加 `pnpm-lock.yaml` 文件（如果不存在）
2. 创建 `.npmrc` 文件并添加：
   ```
   shamefully-hoist=true
   ```
3. 推荐在 `package.json` 中添加部署脚本：
   ```json
   "scripts": {
     "docs:build": "vitepress build docs",
     "docs:preview": "vitepress preview docs"
   }
   ```

### 3. GitHub Actions 配置

#### 3.1 创建工作流文件
在 `.github/workflows` 目录下创建 `deploy.yml` 文件：

```yaml
name: Deploy VitePress to GitHub Pages

on:
  push:
    branches: [main] # 或 master，根据你的主分支名称

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write # 允许工作流写入仓库
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # 获取完整历史记录

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20 # 推荐使用 LTS 版本
          cache: 'pnpm' # 启用 pnpm 缓存

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8 # 指定 pnpm 版本

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build VitePress
        run: pnpm docs:build

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.ACCESS_TOKEN }}
          publish_dir: docs/.vitepress/dist
          keep_files: false # 清除旧文件
          force_orphan: true # 强制使用单次提交
```

#### 3.2 工作流配置说明
1. **缓存优化**：`setup-node` 的 `cache: 'pnpm'` 可以显著加速后续构建
2. **权限控制**：`contents: write` 是部署到 gh-pages 分支的必要权限
3. **部署选项**：
   - `keep_files: false` 确保每次部署都是全新状态
   - `force_orphan: true` 创建干净的提交历史

### 4. 常见问题排查

#### 4.1 部署失败常见原因
1. **Token 权限不足**：确保 token 有 `repo` 权限
2. **base 路径错误**：用户页面应为 `/`，项目页面应为 `/repo-name/`
3. **分支保护**：检查仓库设置中是否限制了 gh-pages 分支的写入

#### 4.2 缓存优化技巧
1. 可以添加额外缓存步骤加速构建：
   ```yaml
   - name: Cache VitePress build
     uses: actions/cache@v3
     with:
       path: |
         docs/.vitepress/cache
         node_modules
   ```

### 5. 进阶配置

#### 5.1 自定义域名
1. 在 `docs/.vitepress/config.js` 中设置 `base: '/'`
2. 在仓库 Settings → Pages 中设置 Custom domain
3. 在项目根目录添加 `CNAME` 文件（内容为你的域名）

#### 5.2 多环境部署
可以扩展工作流实现开发/生产环境分离：
```yaml
jobs:
  deploy-dev:
    if: github.ref == 'refs/heads/dev'
    # 开发环境配置...
  
  deploy-prod:
    if: github.ref == 'refs/heads/main'
    # 生产环境配置...
```

### 6. 参考资源
1. [VitePress 官方部署指南](https://vitepress.vuejs.org/guide/deploying#github-pages)
2. [GitHub Actions 官方文档](https://docs.github.com/en/actions)
3. [pnpm 与 GitHub Actions 集成](https://pnpm.io/continuous-integration)
4. 修改包管理器为 pnpm：[setup-node](https://github.com/actions/setup-node/blob/main/docs/advanced-usage.md#caching-packages-data)
