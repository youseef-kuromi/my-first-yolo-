# my-first-yolo-
我的第一个目标检测练习 Codex
# 用 VS Code 和 GitHub 做第一个 YOLO 项目

这份教程按 Windows 电脑编写，适合刚开始学 Python 的人。做完后，你会有一个能识别图片物体的小程序，并把代码放到自己的 GitHub 账号里。

## 先分清三个东西

- **VS Code**：在电脑上打开文件、写代码、运行程序的地方。
- **Python**：真正执行 Python 代码的软件。只安装 VS Code 还不够。
- **GitHub**：网上保存和展示项目的地方。VS Code 可以把电脑上的项目上传到 GitHub。

## 第 1 步：准备 VS Code

### 安装 Python 扩展

1. 打开 VS Code，点击左边四个方块的图标（Extensions/扩展）。
2. 搜索 `Python`。
3. 安装发布者是 **Microsoft** 的 Python 扩展。

注意：这个扩展不是 Python 本身。若电脑还没安装 Python，请从 [Python 官方下载页](https://www.python.org/downloads/windows/) 安装 Python 3；安装时勾选 `Add python.exe to PATH`，完成后重启 VS Code。

### 检查 Python 和 Git

1. 在 VS Code 顶部点击 `Terminal` → `New Terminal`。
2. 在底部终端输入下面两条命令，每次输入一条并按 Enter：

```powershell
py --version
git --version
```

如果显示版本号，就可以继续。如果 `py` 找不到，说明 Python 还没装好。如果 `git` 找不到，从 [Git for Windows 官方页面](https://git-scm.com/download/win) 安装 Git，然后重启 VS Code 再试。

## 第 2 步：在 GitHub 建一个空项目

1. 在浏览器打开并登录 GitHub。
2. 点击右上角 `+` → `New repository`。
3. `Repository name` 填 `my-first-yolo`。
4. 选择 `Public`（公开），这样之后可以把作品链接给别人看。不要放私人照片、密码或其他隐私资料。
5. 勾选 `Add a README file`，然后点击 `Create repository`。

现在 GitHub 上有了一个项目。接下来把它下载到电脑里的 VS Code。

## 第 3 步：把 GitHub 项目打开到 VS Code

1. 回到 VS Code，按 `Ctrl + Shift + P`。
2. 输入并选择 `Git: Clone`。
3. 选择 `Clone from GitHub`。如果弹出登录提示，按页面提示登录并允许 VS Code 连接 GitHub。
4. 选择刚才创建的 `my-first-yolo`。
5. 选择保存位置 `D:\AI`。请选这个父文件夹，不要选里面已有的 `tensorflow-github-ecosystem` 文件夹。
6. 下载完成后，点 `Open` 打开项目。

如果列表里没有仓库，也可以在 GitHub 仓库页面点击绿色 `Code`，复制 HTTPS 地址；回到 VS Code 的 `Git: Clone`，把地址粘贴进去。

## 第 4 步：给这个项目准备 Python 环境

先让 VS Code 为项目建一个单独的小环境，这样安装的学习工具不会影响电脑上其他 Python 项目。

1. 在 VS Code 按 `Ctrl + Shift + P`。
2. 输入并选择 `Python: Create Environment`。
3. 选择 `Venv`。
4. 选择已安装的 Python 版本。
5. 环境名称保持默认的 `.venv`，按 Enter。
6. 若询问是否安装常用软件包，选择“跳过包安装”。稍后会专门安装 YOLO。
7. 等待左侧文件列表出现 `.venv` 文件夹。
8. 点击顶部 `Terminal` → `New Terminal`，输入：

```powershell
python -m pip install --timeout 300 --retries 5 ultralytics
```

安装需要一些时间，并会使用网络下载机器学习工具，其中 PyTorch 可能有一百多 MB。先用 CPU 就可以，不用配置显卡。如果下载中断，等终端重新出现 `(.venv) PS ...>` 后，再运行同一条命令重试。

## 第 5 步：保留原图并添加其他图片

保留你现在的 Kuromi 图片 `test.jpg`，不要覆盖它。再把其他 JPG 图片也复制到 VS Code 左侧项目文件列表中，和 `README.md` 放在同一层。文件名可以任意，例如你添加的人像 `cxk.jpg`，不必改名。

图片不要包含私人或敏感内容。Kuromi 图片可以保留；想看到识别框的话，再添加一张清晰的猫、狗、汽车或人物照片。

## 第 6 步：告诉 GitHub 哪些东西不要上传

`.venv` 里装着很多程序文件，不应该上传到 GitHub。我们用 `.gitignore` 告诉 Git 忽略它们。

1. 在 VS Code 左侧文件列表上，点击“新建文件”。
2. 文件名输入 `.gitignore`（文件名前面有一个点）。
3. 把下面几行复制进去并按 `Ctrl + S` 保存：

```text
.venv/
__pycache__/
*.pyc
*.pt
runs/
results/
```

这样上传代码时就不会把虚拟环境、模型权重和几十张识别结果一起传上去。以后想展示精选结果时，可以单独挑几张放进项目。

## 第 7 步：创建并运行识别程序

1. 在 VS Code 左侧文件列表上，点击“新建文件”图标。
2. 文件名输入 `detect.py`。
3. 把下面代码复制进去，按 `Ctrl + S` 保存：

```python
from pathlib import Path

from ultralytics import YOLO

# 自动读取项目文件夹中的 JPG、JPEG、PNG、BMP 和 WEBP 图片
image_extensions = {".jpg", ".jpeg", ".png", ".bmp", ".webp"}
input_folder = Path(".")
image_paths = sorted(
    (
        path
        for path in input_folder.iterdir()
        if path.is_file()
        and path.suffix.lower() in image_extensions
        and not path.stem.lower().startswith("detected")
    ),
    key=lambda path: path.name.lower(),
)

if not image_paths:
    raise FileNotFoundError("项目文件夹里没有找到支持的图片。")

model = YOLO("yolo26n.pt")
output_folder = Path("results")
output_folder.mkdir(exist_ok=True)

for index, image_path in enumerate(image_paths, start=1):
    result = model.predict(source=str(image_path), conf=0.25, verbose=False)[0]
    output_path = output_folder / (
        f"detected_{index:03d}_{image_path.stem}{image_path.suffix.lower()}"
    )
    result.save(filename=str(output_path))
    print(f"{image_path.name}：检测到 {len(result.boxes)} 个目标；结果保存在 {output_path}")

print(f"全部完成，共检查 {len(image_paths)} 张图片。")
```

4. 点击编辑区右上角的 ▶（Run Python File/运行 Python 文件）。
5. 第一次运行会下载模型。等终端显示“全部完成，共检查 N 张图片”。
6. 左侧打开 `results` 文件夹，可查看每张图片对应的识别结果。

Kuromi 图片仍可能没有识别框，因为现成模型没学过这个角色。猫、狗、汽车等常见物体更容易识别。模型没框出每个物体也正常；照片清晰度、光线和拍摄角度都会影响结果。

## 第 8 步：把成果保存到 GitHub

VS Code 左边栏点击“分叉线”样子的 **Source Control（源代码管理）** 图标，或按 `Ctrl + Shift + G`。

1. 你会看到新建或修改的文件。
2. 在上方消息框输入：`完成第一个YOLO识别练习`。
3. 点击 `Commit`（提交）。如果提示先暂存，选择 `Yes` 或 `Stage All Changes`。
4. 点击 `Sync Changes` 或 `Publish Branch`，按提示登录 GitHub。
5. 回到 GitHub 网页刷新 `my-first-yolo`，确认能看到 `detect.py` 和 `README.md`。

**保存和上传是两件事：** `Commit` 是先把改动记录在电脑上；`Sync Changes` / `Publish Branch` 才会把它传到 GitHub。

## 第 9 步：写项目说明

在 VS Code 点开 `README.md`，把内容改成下面这样并按 `Ctrl + S`：

````markdown
# 我的第一个目标检测项目

我用 YOLO 识别图片中的常见物体。

## 文件
- `detect.py`：识别图片的 Python 程序。
- 项目文件夹里的 JPG、JPEG、PNG、BMP、WEBP 图片：程序会一起检查，不要求特定文件名。
- `results` 文件夹：保存每张输入图片对应的识别结果。

## 怎么运行
在 VS Code 终端运行：

```powershell
python detect.py
```

如果还没安装 YOLO，先运行 `python -m pip install --timeout 300 --retries 5 ultralytics`。

这是一个入门练习，使用已经训练好的模型，没有用自己的图片重新训练模型。
````

保存后，回到“源代码管理”，再做一次 `Commit` 和 `Sync Changes`，README 修改也会更新到 GitHub。

## 常见问题

**点 ▶ 后提示找不到 Python？** 确认已安装 Python 和 Microsoft Python 扩展，然后重启 VS Code。

**没有处理某张图片？** 检查图片是否和 `detect.py` 放在同一层，格式是否为 JPG、JPEG、PNG、BMP 或 WEBP。

**提示找不到 `ultralytics`？** 按 `Ctrl + Shift + P`，选择 `Python: Select Interpreter`，选项目里的 `.venv`，再打开一个新终端，运行 `python -m pip install --timeout 300 --retries 5 ultralytics`。黄色波浪线是编辑器提示；最终是否能运行，以终端运行结果为准。

**PyTorch 下载到一半报错？** 通常是网络下载中断。等终端回到 `(.venv) PS ...>` 提示符后，再运行安装命令重试；如果仍反复超时，可以先改用 Google Colab 跑识别，避免在本机下载大型依赖。

**GitHub 看不到新文件？** 回到 VS Code 的源代码管理，检查是否完成 `Commit`，然后点击 `Sync Changes` 或 `Publish Branch`。

遇到其他错误时，把 VS Code 终端里的红色报错文字或截图发来，我可以根据你看到的内容继续带你排查。

## 参考

- [VS Code 官方：Python 入门](https://code.visualstudio.com/docs/python/python-tutorial)
- [VS Code 官方：连接 GitHub](https://code.visualstudio.com/docs/sourcecontrol/github)
- [Ultralytics 官方：安装和第一次预测](https://docs.ultralytics.com/quickstart)

