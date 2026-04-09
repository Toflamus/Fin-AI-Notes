# Presentations — 汇报与演示文稿

集中管理所有 Beamer / LaTeX 演示文稿。每份汇报单独放在一个子文件夹中,便于管理 LaTeX 编译产生的中间文件。

---

## 目录结构

```
Presentations/
├── README.md                                      ← 本文件
├── _templates/                                    ← 可复用的模板
│   └── beamer_metropolis_zh/                      ← 中文 Metropolis 模板
└── YYYY-MM-DD_TopicName/                          ← 单次汇报
    ├── main.tex                                   ← 主文件
    ├── Makefile                                   ← 编译脚本
    ├── .gitignore                                 ← 排除中间文件
    ├── figures/                                   ← 图片资源
    └── README.md                                  ← 这次汇报的说明
```

## 命名规范

子文件夹用 `YYYY-MM-DD_主题` 命名,例如:
- `2026-04-09_OrderFlow_Microstructure/`
- `2026-05-15_FactorEvaluation/`

便于按时间排序、易于检索。

## 编译方式

每个子文件夹内含 `Makefile`,使用 `xelatex` 编译(支持中文):

```bash
cd 2026-04-09_OrderFlow_Microstructure
make            # 编译 PDF
make clean      # 清理中间文件
make watch      # 持续监视改动并自动重编(需 latexmk)
```

## 中间文件管理

LaTeX 编译会产生大量中间文件(`.aux`、`.log`、`.toc`、`.snm`、`.nav` 等)。每个汇报文件夹内的 `.gitignore` 会自动排除这些文件,只保留源代码和最终 PDF。

## 模板与主题

默认主题:**Warsaw**（经典学术研究汇报主题）
- 蓝色色调,结构清晰,适合研究报告汇报
- 内置 section/subsection 导航栏,方便听众跟踪进度
- 可根据场景切换为 `Madrid` / `Frankfurt` / `default` 等内置主题

## 已有汇报清单

| 日期 | 主题 | 文件夹 |
|------|------|--------|
| 2026-04-09 | 订单流微观分类与 LOB 动力学(实习汇报) | [2026-04-09_OrderFlow_Microstructure](2026-04-09_OrderFlow_Microstructure/) |
