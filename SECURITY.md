# 🔒 安全策略 | Security Policy

## 报告漏洞 | Reporting a Vulnerability

发现安全漏洞（凭据泄漏、脚本注入、恶意依赖等）请**不要公开提交 issue**，直接：

- 私信 GitHub：[Security Advisories](https://github.com/yhbcode000/paper-share-skills/security/advisories/new)（推荐，仅维护者可见）
- 或邮件：yhbcode000@foxmail.com

请附上：复现步骤、受影响版本（commit / tag）、影响范围描述。

## 处理承诺 | What to Expect

- 24 小时内确认收悉；
- 确认有效后优先修复，修复完成前不公开披露；
- 修复后发布 release 并在 Security Advisory 中致谢报告者（如同意署名）。

## 安全注意 | Operational Notes

- 本套件涉及 B 站登录凭据（`~/.bilibili/cookies.json`）与网络下载，请勿将凭据文件提交到任何仓库；
- 使用第三方技能 / 脚本前请审阅其来源；fork 后修改需自行审查。
