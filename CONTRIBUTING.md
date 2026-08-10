# 🤝 贡献指南 | Contributing

感谢你愿意为技能包贡献。本仓库是**面向公众分发的可移植技能套件**，所有提交须通过以下检查（与 `publish-skill-suite-from-omp` / `paper-share-skills-porting` 的发布规则一致）：

## 技能目录规范

- 每个技能一个目录：`SKILL.md`（YAML frontmatter：`name` 必须等于目录名 + `description`）+ 脚本/模板。
- `SKILL.md` 中所有机器相关路径必须使用环境变量占位符：
  `$PP_ROOT` / `$MINERU_PYTHON` / `PP_PYTHON` / `$LUALATEX` / `$POPPLER_DIR` / `$PRESENTER` / `$INSTITUTE` / `<SKILLS_DIR>`。
- **禁止**提交个人路径（`C:/Users/...`、`D:/Envs/...`、`texlive`）、个人身份信息、`voices/` 语音样本、cookie / 登录凭据。
- 公开版不包含主播贴纸 / 旁白角色叠加（`NARRATOR:` / `LilyWhite` / `Miku`），保留 `% NARRATION:` 音频契约。

## 提交前检查（必须全部通过）

```bash
# 1. 无个人路径 / 贴纸残留（paper-venue-discovery 中的 "discovery" 是良性误报）
grep -rIl -E 'disco|Paper_Survey_Env|一点一刻|texlive|Haobo|杨昊波|NARRATOR|LilyWhite|sticker' --exclude-dir=.git --exclude=README.md .

# 2. 每个 .py 可通过语法编译
for f in $(git ls-files '*.py'); do py -3 -m py_compile "$f" || exit 1; done
rm -rf __pycache__

# 3. 无 .venv / pycache / voices / cookie 混入
git status --short
```

## 提交流程

1. Fork 本仓库，新建分支 `feat/<your-change>`。
2. 按上述规范修改，跑完检查。
3. 提交并开 Pull Request，描述改动动机与验证结果。

## 补充许可说明

论文分享 / 组会汇报等**非分发**场合（内部汇报、课堂展示、个人学习）使用本套件，免于保留许可文本与署名义务 —— 见 [LICENSE](LICENSE) 补充许可条款。
