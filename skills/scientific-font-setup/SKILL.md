---
name: scientific-font-setup
description: Configure standard scientific fonts for Matplotlib with Chinese-English-digit support, including detection, fallback chains, and install guidance on macOS/Linux. Use when user asks about 科研字体, 图表字体, 中文乱码, glyph missing, Times New Roman, Arial, Noto CJK, or publication-ready plotting style.
---

# Scientific Font Setup (Matplotlib)

## 适用场景

当用户提到以下需求时应用本技能：
- 科研绘图字体规范、投稿字体、期刊图表字体
- 中文乱码、`Glyph missing`、负号显示异常
- 中英数字混排字体统一
- 希望自动检测本机字体并在缺失时安装

## 默认原则

1. 图表模式优先无衬线：`Noto Sans CJK SC + Arial/Helvetica`  
2. 论文模式优先衬线：`Noto Serif CJK SC + Times New Roman`  
3. 必须先检测再配置，缺失再安装，不猜测字体可用性。  
4. 修改后必须运行脚本验证，确认无 `Glyph missing`。  

## 标准工作流

复制以下清单并执行：

```text
Task Progress:
- [ ] 1. 读取目标绘图脚本（如 plot_strain.py）
- [ ] 2. 检测 Matplotlib 可见字体
- [ ] 3. 确定双模式回退链（图表/论文）
- [ ] 4. 若缺失则安装字体（macOS 或 Linux）
- [ ] 5. 清理 Matplotlib 字体缓存并重建
- [ ] 6. 修改 rcParams（含 pdf/ps 嵌入）
- [ ] 7. 运行脚本验证并检查告警
```

## 字体检测命令

```bash
python3 - <<'PY'
import matplotlib.font_manager as fm
names = sorted({f.name for f in fm.fontManager.ttflist})
for k in [
    "Noto Sans CJK SC", "Noto Serif CJK SC",
    "Arial", "Helvetica", "Times New Roman",
    "Songti SC", "PingFang SC"
]:
    print(f"{k}:", "YES" if k in names else "NO")
PY
```

## 安装命令

### macOS（Homebrew）

```bash
brew install --cask font-noto-sans-cjk-sc font-noto-serif-cjk-sc
```

### Linux（Debian/Ubuntu）

```bash
sudo apt-get update
sudo apt-get install -y fonts-noto-cjk
fc-cache -fv
```

### Linux（RHEL/CentOS/Fedora）

```bash
sudo dnf install -y google-noto-sans-cjk-fonts google-noto-serif-cjk-fonts || \
sudo yum install -y google-noto-sans-cjk-fonts google-noto-serif-cjk-fonts
fc-cache -fv
```

## Matplotlib 推荐配置模板（双模式）

```python
import matplotlib.font_manager as fm
from matplotlib import rcParams

available_fonts = {f.name for f in fm.fontManager.ttflist}

figure_stack = ["Noto Sans CJK SC", "Arial", "Helvetica", "PingFang SC", "Songti SC"]
paper_stack = ["Noto Serif CJK SC", "Times New Roman", "Songti SC"]

selected_figure = [f for f in figure_stack if f in available_fonts]
selected_paper = [f for f in paper_stack if f in available_fonts]

rcParams["font.family"] = "sans-serif"
rcParams["font.sans-serif"] = selected_figure or ["DejaVu Sans"]
rcParams["axes.unicode_minus"] = False
rcParams["pdf.fonttype"] = 42
rcParams["ps.fonttype"] = 42

SCIENTIFIC_MANUSCRIPT_FONTS = selected_paper
```

## 缓存刷新（必要时）

若安装后仍识别不到字体，执行：

```bash
python3 - <<'PY'
import os, matplotlib
cache_dir = matplotlib.get_cachedir()
for fn in os.listdir(cache_dir):
    if fn.startswith("fontlist-") and fn.endswith(".json"):
        os.remove(os.path.join(cache_dir, fn))
print("font cache cleared:", cache_dir)
PY
```

## 验证标准

1. 运行绘图脚本成功生成图片。  
2. 控制台无 `Glyph missing`。  
3. `print(rcParams["font.sans-serif"])` 显示期望回退链。  
4. 中文、英文、数字、负号均正常显示。  

## 输出要求

向用户汇报时必须包含：
- 已检测到的关键字体（存在/缺失）
- 是否执行了安装及安装结果
- 修改了哪些文件
- 实际运行验证结果（是否还有告警）
