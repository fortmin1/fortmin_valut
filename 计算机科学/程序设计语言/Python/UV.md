
## 1. 查看帮助和版本

```bash
uv --version
uv --help
uv help
uv help add
```

## 2. 创建项目

```bash
uv init
```

在当前目录初始化一个 Python 项目。

```bash
uv init my-project
cd my-project
```

创建新项目目录。

常见效果是生成：

```text
pyproject.toml
README.md
src/ 或 main.py
```

## 3. 创建虚拟环境

```bash
uv venv
```

默认创建 `.venv`。

```bash
uv venv --python 3.12
```

指定 Python 版本创建虚拟环境。

激活环境：

```bash
# macOS / Linux
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1
```

不过很多 uv 命令不需要你手动激活环境。

## 4. 安装依赖

如果是 uv 项目，推荐用：

```bash
uv add requests
```

添加依赖到 `pyproject.toml`，并同步环境。

安装开发依赖：

```bash
uv add --dev pytest ruff
```

指定版本：

```bash
uv add "django>=5.0"
uv add "numpy==2.1.0"
```

删除依赖：

```bash
uv remove requests
```

## 5. 同步项目环境

```bash
uv sync
```

根据 `pyproject.toml` 和 `uv.lock` 安装/同步依赖。

常用于别人拉下项目后：

```bash
git clone ...
cd project
uv sync
```

只装生产依赖，不装开发依赖：

```bash
uv sync --no-dev
```

## 6. 运行命令或脚本

```bash
uv run python main.py
```

在项目环境里运行 Python 文件。

```bash
uv run pytest
uv run ruff check .
uv run python
```

这点很常用：不需要手动 `source .venv/bin/activate`。

## 7. 锁定依赖

```bash
uv lock
```

生成或更新 `uv.lock`。

```bash
uv lock --upgrade
```

升级锁文件里的依赖版本。

```bash
uv sync --locked
```

严格按已有 lock 文件同步，CI 里常用。

## 8. pip 兼容命令

uv 也提供类似 pip 的接口：

```bash
uv pip install requests
uv pip uninstall requests
uv pip list
uv pip freeze
```

配合虚拟环境使用：

```bash
uv venv
uv pip install -r requirements.txt
```

从当前环境生成 requirements：

```bash
uv pip freeze > requirements.txt
```

把 requirements 安装进环境：

```bash
uv pip install -r requirements.txt
```

## 9. Python 版本管理

查看可用/已安装 Python：

```bash
uv python list
```

安装 Python：

```bash
uv python install 3.12
uv python install 3.13
```

指定 Python 运行：

```bash
uv run --python 3.12 python --version
```

创建环境时指定版本：

```bash
uv venv --python 3.12
```

## 10. 运行一次性工具：uvx

`uvx` 类似 `pipx run`，适合临时运行命令行工具。官方文档也说明 `uvx` 是 `uv tool run` 的别名。([Astral Docs](https://docs.astral.sh/uv/?utm_source=chatgpt.com "uv"))

```bash
uvx ruff check .
uvx black .
uvx pycowsay "hello"
```

等价于：

```bash
uv tool run ruff check .
```

## 11. 安装全局工具

安装命令行工具：

```bash
uv tool install ruff
uv tool install black
uv tool install poetry
```

查看已安装工具：

```bash
uv tool list
```

升级工具：

```bash
uv tool upgrade ruff
```

卸载工具：

```bash
uv tool uninstall ruff
```

## 12. 构建和发布包

构建：

```bash
uv build
```

发布到 PyPI：

```bash
uv publish
```

一般用于 Python 包项目。

## 13. 常见工作流

新项目：

```bash
uv init my-app
cd my-app
uv add requests fastapi
uv add --dev pytest ruff
uv run python main.py
```

已有项目：

```bash
git clone <repo>
cd <repo>
uv sync
uv run pytest
```

传统 requirements 项目：

```bash
uv venv
uv pip install -r requirements.txt
uv run python app.py
```

临时跑工具：

```bash
uvx ruff check .
```

## 14. 和 pip/venv 的简单对照

|传统命令|uv 命令|
|---|---|
|`python -m venv .venv`|`uv venv`|
|`pip install requests`|`uv add requests` 或 `uv pip install requests`|
|`pip install -r requirements.txt`|`uv pip install -r requirements.txt`|
|`python main.py`|`uv run python main.py`|
|`pipx run ruff`|`uvx ruff`|
|`pipx install ruff`|`uv tool install ruff`|

最常用的几个记住就够了：

```bash
uv init
uv add
uv remove
uv sync
uv run
uv venv
uv pip install
uvx
uv python install
```