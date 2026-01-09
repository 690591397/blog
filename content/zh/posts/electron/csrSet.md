---
title: macOS Developer ID Application 自分发证书申请
date: 2026-01-06
draft: true
descritpion: ''
categories:
  - Electron
series:
  - Electron
---

> 本文介绍如何申请 macOS 代码签名证书，并配置到 GitHub Actions 中实现自动化构建。

## 一、证书类型选择

### macOS 应用分发证书对比

| 证书类型 | 用途 | 分发渠道 |
|---------|------|---------|
| **Developer ID Application** | 签名 .app 文件 | Mac App Store **外部**（网站下载、直接分发） |
| **Apple Distribution** | 提交审核和发布 | Mac App Store **内部** |

**关键区别**：
- Developer ID Application：无需审核，可自由分发，通过 Gatekeeper 验证
- Apple Distribution：必须经过 Apple 审核，只能通过 App Store 分发
- 两种证书**不能混用**，选择后无法更改分发渠道

## 二、证书申请流程

### 1. 生成证书签名请求 (CSR)

在 macOS 上操作：

```bash
应用程序 > 实用工具 > 钥匙串访问.app
```

1. 菜单栏选择：`钥匙串访问 > 证书助理 > 从证书颁发机构请求证书`
2. 填写信息：
   - 用户电子邮件地址：Apple 开发者账号邮箱
   - 常用名称：你的名字或公司名称
   - CA 电子邮件地址：**留空**
   - 请求是：选择 **"存储到磁盘"**
3. （可选）勾选 "让我指定密钥对信息"：
   - 密钥大小：2048 位
   - 算法：RSA
4. 保存为：`CertificateSigningRequest.certSigningRequest`

### 2. 在 Apple Developer 创建证书

1. 登录 [Apple Developer](https://developer.apple.com/account/)
2. 进入 `Certificates, Identifiers & Profiles`
3. 点击 `+` 创建新证书
4. 选择 **Developer ID Application**
5. 选择 **G2 Sub-CA**（推荐，有效期至 2031 年）
6. 上传刚才生成的 CSR 文件
7. 下载生成的 `.cer` 证书文件

### 3. 安装证书

双击下载的 `.cer` 文件，自动添加到钥匙串。

验证：打开钥匙串访问，在 **"我的证书"** 中查看：

```
🔑 Developer ID Application: Your Name (TEAM_ID)
  └── 🔐 私钥 (private key)
```

## 三、配置 GitHub Actions

### 1. 导出证书为 .p12

在钥匙串访问中：

1. 选中证书（不是展开的私钥）
2. 右键 > **"导出"**
3. 文件格式：选择 **"个人信息交换 (.p12)"**
4. 设置密码（记住这个密码！）

### 2. 转换为 Base64

```bash
# 转换并复制到剪贴板
base64 -i /path/to/certificate.p12 | pbcopy
```

### 3. 配置 GitHub Secrets

#### Repository Secrets（推荐用于私有仓库）

仓库 `Settings` > `Secrets and variables` > `Actions` > `New repository secret`

添加两个 Secrets：

| Name | Value |
|------|-------|
| `MACOS_CERTIFICATE` | 粘贴 Base64 字符串 |
| `MACOS_CERTIFICATE_PWD` | .p12 导出时设置的密码 |

#### Organization Secrets（用于公开仓库）

组织 `Settings` > `Secrets and variables` > `Actions` > `New organization secret`

- **免费计划**：只能选择 "Public repositories"
- **付费计划**：可选 "Private repositories" 或 "Selected repositories"

### 4. GitHub Actions Workflow 配置

创建 `.github/workflows/build.yml`：

```yaml
name: Build macOS App

on:
  push:
    tags:
      - 'v*'

jobs:
  build-macos:
    runs-on: macos-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Import certificate to keychain
        env:
          CERTIFICATE_BASE64: ${{ secrets.MACOS_CERTIFICATE }}
          CERTIFICATE_PASSWORD: ${{ secrets.MACOS_CERTIFICATE_PWD }}
          KEYCHAIN_PASSWORD: temp_keychain_password
        run: |
          # Create temporary keychain
          security create-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
          security default-keychain -s build.keychain
          security unlock-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
          security set-keychain-settings -t 3600 -u build.keychain
          
          # Import certificate
          echo "$CERTIFICATE_BASE64" | base64 --decode > certificate.p12
          security import certificate.p12 -k build.keychain -P "$CERTIFICATE_PASSWORD" -T /usr/bin/codesign
          security set-key-partition-list -S apple-tool:,apple:,codesign: -s -k "$KEYCHAIN_PASSWORD" build.keychain
          
          # Clean up
          rm certificate.p12
      
      - name: Build application
        run: pnpm run build:mac
        env:
          CSC_LINK: ${{ secrets.MACOS_CERTIFICATE }}
          CSC_KEY_PASSWORD: ${{ secrets.MACOS_CERTIFICATE_PWD }}
      
      - name: Clean up keychain
        if: always()
        run: security delete-keychain build.keychain
```

## 四、重要提示

### 证书数量限制

一个 Apple Developer 账号可以创建：
- ✅ 最多 **5 个** Developer ID Application 证书
- ✅ 最多 **5 个** Developer ID Installer 证书
- 证书有效期：**5 年**

### GitHub Secrets 优先级

```
Repository Secret (最高优先级)
    ↓
Organization Secret
    ↓
Environment Secret (最低优先级)
```

仓库级别的 Secret 会**覆盖**同名的组织级别 Secret。

### 安全建议

- ❌ 不要将 `.p12` 文件提交到 Git
- ✅ 使用强密码保护 `.p12` 文件
- ✅ 定期检查证书有效期
- ✅ 如证书泄露，立即在 Apple Developer 撤销

## 五、常见问题

**Q: Developer ID 签名的应用能上传 App Store 吗？**  
A: 不能。必须使用 Apple Distribution 证书重新签名。

**Q: 证书过期后已发布的应用还能用吗？**  
A: 可以。用户仍可下载和运行已签名的应用，但无法签名新版本。

**Q: 私有仓库能使用组织的免费 Secrets 吗？**  
A: 不能。免费计划只支持公开仓库，私有仓库需使用 Repository Secrets 或升级付费计划。

---

**参考资料**：
- [Apple Developer ID 官方文档](https://developer.apple.com/support/developer-id/)
- [electron-builder 代码签名指南](https://www.electron.build/code-signing)
