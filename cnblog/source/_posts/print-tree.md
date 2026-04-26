---
title: 打印项目树的打印脚本
date: 2025-08-15 15:09:28
tags:
categories:
    - Python
    - 服务器
permalink: /posts/print-tree.html
---


# Python 项目目录树打印脚本（可复用）

本文档包含：
- 可直接运行的脚本 `print_tree.py`
- 注释模板 `annotations.json` 示例
- 使用方式与常见参数

---

## 1. 脚本代码

```python
# print_tree.py
import os
import json
import argparse
from pathlib import Path

DEFAULT_IGNORES = {".git", "__pycache__", ".venv", "venv", "node_modules", ".idea", ".vscode", "build", "dist"}

def natural_key(s: str):
    # 简单自然排序：数字会按大小而不是字典序排列
    import re
    return [int(t) if t.isdigit() else t.lower() for t in re.split(r"(\d+)", s)]

def load_annotations(path: str | None) -> dict[str, str]:
    if not path:
        return {}
    p = Path(path)
    if not p.exists():
        raise FileNotFoundError(f"annotations file not found: {p}")
    with p.open("r", encoding="utf-8") as f:
        return json.load(f)

def should_skip(entry: Path, ignores: set[str], show_hidden: bool) -> bool:
    name = entry.name
    if not show_hidden and name.startswith(".") and name not in {".env"}:
        return True
    return name in ignores

def format_line(prefix: str, name: str, is_dir: bool, comment: str | None) -> str:
    base = f"{prefix}{name}/" if is_dir else f"{prefix}{name}"
    if comment:
        pad = " " if base.endswith("/") else " "
        return f"{base: <60}# {comment}"
    return base

def print_tree(root: Path, max_depth: int, ignores: set[str], show_hidden: bool, annotations: dict[str, str]):
    def recurse(cur: Path, prefix_parts: list[bool], depth: int):
        if depth < 0:
            return
        entries = [e for e in cur.iterdir() if not should_skip(e, ignores, show_hidden)]
        entries.sort(key=lambda p: (p.is_file(), natural_key(p.name)))  # 目录在前，文件在后
        total = len(entries)
        for i, e in enumerate(entries):
            is_last = (i == total - 1)
            branch = "└─ " if is_last else "├─ "
            pipe_prefix = "".join("   " if last else "│  " for last in prefix_parts)
            line_prefix = pipe_prefix + branch
            rel = e.relative_to(root).as_posix()
            comment = annotations.get(rel) or annotations.get(e.name)

            print(format_line(line_prefix, e.name, e.is_dir(), comment))

            if e.is_dir():
                recurse(e, prefix_parts + [is_last], depth - 1)

    root_name = root.name or str(root)
    root_comment = annotations.get(".") or annotations.get(root_name)
    print(format_line("", root_name, True, root_comment))
    recurse(root, [], max_depth)

def main():
    parser = argparse.ArgumentParser(description="Print project directory tree with optional annotations.")
    parser.add_argument("--root", default=".", help="项目根目录（默认当前目录）")
    parser.add_argument("--max-depth", type=int, default=10, help="最大遍历深度（根为第0层）")
    parser.add_argument("--ignore", nargs="*", default=[], help="额外忽略的目录/文件名（空格分隔）")
    parser.add_argument("--show-hidden", action="store_true", help="是否显示隐藏文件/目录（.开头）")
    parser.add_argument("--annotations", help="JSON 文件，键为相对路径或文件名，值为注释")
    parser.add_argument("--output", help="将结果输出到文件（默认打印到控制台）")
    args = parser.parse_args()

    root = Path(args.root).resolve()
    if not root.exists():
        raise SystemExit(f"Root not found: {root}")

    annotations = load_annotations(args.annotations)
    ignores = DEFAULT_IGNORES | set(args.ignore)

    if args.output:
        from io import StringIO
        import sys
        buf = StringIO()
        _stdout = sys.stdout
        sys.stdout = buf
        try:
            print_tree(root, args.max_depth, ignores, args.show_hidden, annotations)
        finally:
            sys.stdout = _stdout
        Path(args.output).write_text(buf.getvalue(), encoding="utf-8")
        print(f"📄 已写入: {args.output}")
    else:
        print_tree(root, args.max_depth, ignores, args.show_hidden, annotations)

if __name__ == "__main__":
    main()

```

---

## 2. 注释模板（可选）

创建 `annotations.json`：

```json
{
  ".": "项目根目录",
  "app": "应用代码",
  "app/main.py": "入口(create_app) & app实例",
  "app/api/v1/routes.py": "路由聚合",
  "app/core/config.py": "配置(读取 .env)",
  "app/core/middleware.py": "中间件安装函数",
  "app/core/logging.py": "日志(可选)",
  "schemas": "Pydantic 模型",
  "models": "ORM 模型",
  "services": "业务逻辑",
  "deps": "依赖注入",
  "tests": "测试",
  ".env": "环境变量",
  "README.md": "项目说明"
}

```

---

## 3. 使用方法

### 3.1 最简单
```bash
python print_tree.py --root .
```

### 3.2 使用注释
```bash
python print_tree.py --root . --annotations annotations.json
```

### 3.3 忽略目录、限制深度、显示隐藏文件
```bash
python print_tree.py --root . --ignore .cache coverage htmlcov --max-depth 6 --show-hidden
```

### 3.4 导出到文件
```bash
python print_tree.py --root . --annotations annotations.json --output tree.txt
```

---

## 4. 小贴士
- 默认忽略：`.git`、`__pycache__`、`venv/.venv`、`node_modules`、`.idea/.vscode`、`build/dist`
- 支持中文注释输出（终端建议 UTF-8 编码）
- 需要自然排序（数字 1,2,10 按数值排序）已内置
- 可自行扩展 `--auto-notes` 功能以自动生成注释模板
