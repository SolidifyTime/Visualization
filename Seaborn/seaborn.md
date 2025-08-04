# Seaborn学习笔记

## 1. Figure-level vs. Axes-level 函数

在 `seaborn` 中，绘图函数大致可以分为两类：**Axes-level（轴级）** 函数和 **Figure-level（图级）** 函数。理解这两者的区别是掌握 `seaborn` 的关键。

**一句话总结**: Figure-level 函数适合快速创建标准化的多图组合（像套餐）；Axes-level 函数提供了在单个图表上精细操作的更高灵活性（像单点）。

### 1.1 Axes-level 函数

**Axes-level** 函数是更基础的绘图单元。它们的作用是在一个已经定义好的 `matplotlib` 图表（即一个 `Axes` 对象）上绘制图形。你可以把 `Axes` 对象想象成一块画布，`Axes-level` 函数就是在这块画布上作画的工具。

**特点:**
* **灵活性高**: 你可以自由地控制图表的布局，将多个 `Axes-level` 函数绘制的图表组合在一个 `Figure`（图形窗口）中。
* **与 Matplotlib 深度整合**: 它们可以和 `matplotlib` 的各种函数无缝衔接，让你对图表的细节有完全的控制权（例如，添加标题、坐标轴标签、图例等）。
* **返回值**: 通常返回一个 `Axes` 对象，允许你继续在该对象上进行操作。

**在你提供的 Jupyter 文件中，属于 Axes-level 的函数有:**
* `sns.scatterplot()`: 散点图
* `sns.lineplot()`: 折线图
* `sns.histplot()`: 直方图
* `sns.boxplot()`: 箱线图
* `sns.violinplot()`: 小提琴图
* `sns.barplot()`: 条形图
* `sns.heatmap()`: 热力图

### 1.2 Figure-level 函数

**Figure-level** 函数则是一种更高级的绘图方式。它们会自动创建一个 `Figure` 对象，并可能包含一个或多个子图（`Axes`）。这类函数通常用于创建特定类型的、结构化的多图表组合。

**特点:**
* **高度集成**: 一个函数调用就可以生成一个完整的、美观的图表，包括图例、标题等。
* **便捷性**: 对于一些标准化的多图组合（例如，按类别分面的图表），使用 `Figure-level` 函数会非常方便快捷。
* **通过 `kind` 参数控制**: 很多 `Figure-level` 函数内部集成了多个 `Axes-level` 函数的功能，通过 `kind` 参数来选择具体的图表类型。例如，`relplot()` 可以通过 `kind='scatter'` 或 `kind='line'` 来创建散点图或折线图。
* **返回值**: 通常返回一个 `FacetGrid` 对象，这是一个用于管理多个子图的特殊对象。

**在你提供的 Jupyter 文件中，属于 Figure-level 的函数有:**
* `sns.relplot()`: 关系图 (可以生成 `scatterplot` 和 `lineplot`)
* `sns.displot()`: 分布图 (可以生成 `histplot`, `kdeplot`, `ecdfplot`)
* `sns.catplot()`: 分类图 (可以生成 `stripplot`, `swarmplot`, `boxplot`, `violinplot`, `barplot` 等)
* `sns.jointplot()`: 联合分布图
* `sns.pairplot()`: 变量关系组图

## 2. 坐标轴美化与精细控制

Seaborn 允许我们对图表的细节进行深度定制，尤其是坐标轴的样式。这些操作可以分为几类：对 `Figure-level` 函数返回对象的操作，以及通用的美化函数。

### 2.1 移除坐标轴边框 (Spines)

默认情况下，图表有四条边框（上下左右）。通常为了美观，我们会移除顶部和右侧的边框。

#### `sns.despine()`

这是一个独立的函数，用于移除边框。

* **默认行为**: `sns.despine()` 会移除当前图表的顶部和右侧边框。
* **高级用法**: 可以通过参数进行更精细的控制。

```python
# 绘制一个基础图表
fig, ax = plt.subplots()
sns.scatterplot(x="total_bill", y="tip", data=tips, ax=ax)

# despine 的高级用法
sns.despine(offset=10,  # 让坐标轴从数据点偏移 10个像素
            trim=True,    # 裁剪坐标轴的范围，使其长度与刻度范围匹配
            left=True)    # 额外移除左侧的边框（不常用）

```

#### `FacetGrid.despine()`

对于 Figure-level 函数（如 relplot, displot）返回的 FacetGrid 对象（通常命名为 g），可以直接调用其 despine 方法。

```python
g = sns.displot(data=penguins, x="flipper_length_mm", hue="species", col="sex")

# 移除所有子图的左侧边框
g.despine(left=True)
```

### 2.2 设置坐标轴标签和图例

#### FacetGrid 对象的方法

对于 Figure-level 函数返回的 g 对象，不能使用 plt.xlabel()，而应使用其专属方法来设置整个图形的坐标轴标签。

```python
g = sns.displot(data=penguins, x="flipper_length_mm", hue="species", col="sex")

# 设置整个 Figure 的 X 和 Y 轴标签
g.set_axis_labels("Flipper Length (mm)", "Count")

# 单独设置每个子图的标题
g.set_titles("Sex: {col_name}")

# 设置图例的标题
g.legend.set_title("Species")
```
注意: 对于 Axes-level 函数返回的 ax 对象，我们仍然使用 Matplotlib 的标准方法，如 ax.set_xlabel(), ax.set_ylabel(), ax.set_title()。

### 2.3 控制子图的坐标轴共享

#### facet_kws

在使用 Figure-level 函数（如 displot, relplot, catplot）创建分面（facet）图时，默认情况下所有子图会共享 X 轴和 Y 轴的范围和刻度，以方便比较。我们可以通过 facet_kws 参数来修改这个行为。

facet_kws 接受一个字典，其中的键值对会传递给 FacetGrid 对象，从而控制分面网格的属性。
```python
# 使用 facet_kws 来让每个子图拥有独立的 Y 轴
g = sns.relplot(
    data=penguins,
    x="flipper_length_mm", y="bill_length_mm", col="sex",
    facet_kws=dict(sharey=False)  # 关键参数：Y 轴不共享
)
```
同样，你也可以设置 sharex=False 来让 X 轴不共享。

### 2.4 全局主题与风格设置

#### sns.set_theme()

一个功能强大的函数，可以一次性设置主题、风格、调色板等。

```python
# 在你的 notebook 开头进行全局设置
sns.set_theme(context='paper',       # 设置上下文 (paper, notebook, talk, poster)
              style='whitegrid',     # 设置网格风格
              palette='colorblind',  # 设置调色板
              font_scale=1.1)        # 字体缩放比例
```

#### plt.rcParams

对于一些更底层的设置，比如中文字体支持、分辨率等，我们通常使用 Matplotlib 的 rcParams

```python
# Windows/Mac 设置中文字体，解决负号显示问题
# plt.rcParams['font.sans-serif'] = ['SimHei'] # Windows
plt.rcParams['font.sans-serif'] = ['STHeiti'] # Mac
plt.rcParams['axes.unicode_minus'] = False

# 设置图形分辨率
plt.rcParams['figure.dpi'] = 300
```

## 3. Figure-level 函数详解

Figure-level 函数是更高层次的封装，它们会自动创建并管理一个 Matplotlib Figure，通常包含多个子图（Axes）。这些函数通过 `kind` 参数来调用底层的 Axes-level 函数，并使用 `col`, `row`, `hue` 等参数轻松实现数据分面。

### 3.1 `sns.relplot()`: 关系图

`relplot` (relational plot) 用于探索两个变量之间的关系。它通过 `kind` 参数可以生成散点图或折线图。

**核心参数:**
* `data`: DataFrame 数据。
* `x`, `y`: 指定 x 轴和 y 轴对应的列名。
* `kind`: 图的类型。
    * `'scatter'` (默认): 绘制散点图，底层调用 `sns.scatterplot()`。
    * `'line'`: 绘制折线图，底层调用 `sns.lineplot()`。
* `hue`: 指定一个分类变量，用颜色区分数据点。
* `col`, `row`: 指定分类变量，用于在列方向或行方向上创建子图（分面）。
* `size`: 指定一个数值变量，用点的大小来表示。
* `style`: 指定一个分类变量，用不同的标记样式来区分。

**示例:**
```python
import seaborn as sns
import matplotlib.pyplot as plt

# 加载数据
tips = sns.load_dataset("tips")

# 创建一个按 "sex" 分列，按 "smoker" 区分颜色和样式的散点图
sns.relplot(
    data=tips,
    x="total_bill", y="tip",
    hue="smoker", style="smoker",
    col="sex"
)
plt.show()
```
### 3.2 sns.displot(): 分布图
displot (distribution plot) 用于可视化数据的分布，可以是单变量或双变量的分布。

**核心参数:**

* data, x, y: 同上。如果只提供 x，则为单变量分布；同时提供 x 和 y，则为双变量分布。

* kind: 图的类型。

    * 'hist' (默认): 直方图 (sns.histplot)。

    * 'kde': 核密度估计图 (sns.kdeplot)。

    * 'ecdf': 经验累积分布函数图 (sns.ecdfplot)。

* hue, col, row: 用于分面和颜色区分。

* rug: 是否在坐标轴边缘绘制“地毯图”（rug plot），显示每个观测值的位置。

**示例:**

```Python

penguins = sns.load_dataset("penguins")

# 创建一个按 "species" 区分颜色，按 "sex" 分列的直方图
sns.displot(
    data=penguins,
    x="flipper_length_mm",
    hue="species",
    col="sex",
    kind="hist",
    kde=True  # 可以在直方图上叠加KDE曲线
)
plt.show()
```

### 3.3 sns.catplot(): 分类图
catplot (categorical plot) 是探索分类变量和连续变量之间关系的全能工具。

**核心参数:**

* data, x, y: 同上。通常一个是分类变量，一个是连续变量。

* kind: 图的类型，分为三大家族：

    * 散点型:

        * 'strip' (默认): 简单散点图 (sns.stripplot)。

        * 'swarm': 无重叠的散点图 (sns.swarmplot)，能更好地显示分布。

    * 分布型:

        * 'box': 箱线图 (sns.boxplot)。

        * 'violin': 小提琴图 (sns.violinplot)。

        * 'boxen': 增强箱线图 (sns.boxenplot)，适合大数据集。

    * 估计型:

        * 'point': 点图 (sns.pointplot)，用点的位置和误差线表示中心趋势。

        * 'bar': 条形图 (sns.barplot)，显示中心趋势的估计（**默认为均值**）。

        * 'count': 计数条形图 (sns.countplot)，显示每个类别的数量。

* hue, col, row: 用于分面和颜色区分。

**示例:**

```Python

titanic = sns.load_dataset("titanic")

# 创建一个按 "sex" 分列的小提琴图，展示不同船舱等级的生存率
# 注意：y轴是数值型，x轴是分类型
sns.catplot(
    data=titanic,
    x="class", y="survived",
    hue="sex",
    col="pclass",
    kind="violin",
    split=True # 在小提琴图中将 hue 变量的两个类别分开展示
)
plt.show()
```

### 3.4 sns.jointplot(): 联合分布图

jointplot 用于可视化两个变量之间的联合分布和各自的边缘分布。它不像其他 Figure-level 函数那样支持 col 和 row 分面。

**核心参数:**

* data, x, y: 同上。

* kind: 图的类型。

    * 'scatter' (默认): 散点图和直方图。

    * 'reg': 散点图加回归线。

    * 'resid': 残差图。

    * 'kde': 核密度估计图。

    * 'hist': 二维直方图（色块图）。

    * 'hex': 六边形箱图。

* hue: 可以添加一个分类变量，用颜色区分。

**示例:**

```Python

penguins = sns.load_dataset("penguins")

# 创建一个 bill_length 和 bill_depth 的联合分布图
# 中心是散点图，顶部和右侧是直方图
sns.jointplot(
    data=penguins,
    x="bill_length_mm", y="bill_depth_mm",
    hue="species",
    kind="scatter"
)
plt.show()

```
### 4.5 sns.pairplot(): 变量关系组图

pairplot 会自动创建一个网格，展示数据集中多个变量两两之间的关系。

**核心参数:**

* data: DataFrame 数据。

* vars: 可选，指定要展示的列名列表。默认为所有数值列。

* kind: 非对角线上的图表类型。

    * 'scatter' (默认): 散点图。

    * 'reg': 带回归线的散点图。

* diag_kind: 对角线上的图表类型。

    * 'auto' (默认): histplot 或 kdeplot，取决于 hue 是否使用。

    * 'hist': 直方图。

    * 'kde': 核密度估计图。

* hue: 指定一个分类变量，用颜色区分。

* corner: 若为 True，则只生成下三角矩阵的图表，节省空间。

**示例:**

```Python

penguins = sns.load_dataset("penguins")

# 创建一个展示 penguins 数据集中变量两两关系的图表
# 按 "species" 区分颜色，对角线使用 KDE 图
sns.pairplot(
    data=penguins,
    hue="species",
    diag_kind="kde",
    corner=True
)
plt.show()
```
## 4. 高级应用：山脊图 (Ridge Plot)

山脊图（也称 Joyplot）是一种非常有效的可视化方法，用于展示和比较多个组的数值分布。它将每个组的密度曲线沿垂直轴堆叠，并允许部分重叠，看起来像连绵的山脉，因此得名。

这并不是一个单一的函数，而是通过创造性地组合 `sns.FacetGrid` 和 `sns.kdeplot` 实现的。

**适用场景**: 当你需要比较**大量类别**的数值分布时，山脊图比并排的小提琴图或箱线图更节省空间，且更具视觉冲击力。

### 核心思想

1.  **创建分面网格**: 使用 `sns.FacetGrid`，关键在于将**同一个分类变量**同时分配给 `row` 和 `hue` 参数。这会为每个类别创建一个水平子图，并为每个类别指定一种颜色。
2.  **映射绘图函数**: 使用 `g.map()` 方法，将 `sns.kdeplot` 应用到每个子图上。通过设置 `fill=True`，我们可以得到填充的密度图。
3.  **调整重叠与样式**: 通过调整子图间距、移除不必要的坐标轴和标签，最终形成山脊的效果。

### 完整代码示例

我们将使用 `penguins` 数据集，来观察不同企鹅种类 (`species`) 的鳍长 (`flipper_length_mm`) 分布。

```python
import seaborn as sns
import matplotlib.pyplot as plt

# 加载数据集
penguins = sns.load_dataset("penguins")

# --- 步骤 1: 创建一个 FacetGrid 对象 ---
# 为 "species" 列的每个类别创建一个行，并用颜色区分
# aspect=10, height=0.6 让每个子图变得很宽很矮，适合山脊图
g = sns.FacetGrid(penguins, row="species", hue="species", aspect=10, height=0.6)

# --- 步骤 2: 映射绘图函数，创建填充的密度图 ---
# 使用 kdeplot 绘制每个子图的核密度估计
# fill=True 将曲线下的区域进行填充
# alpha=0.6 设置填充的透明度，让重叠部分可见
g.map(sns.kdeplot, "flipper_length_mm", fill=True, alpha=0.6, linewidth=1.5)

# --- 步骤 3: 绘制轮廓线，增加层次感 ---
# 在填充图的上方再绘制一条轮廓线，让边界更清晰
g.map(sns.kdeplot, "flipper_length_mm", color="black", linewidth=1.5)

# --- 步骤 4: 精细调整与美化，这是“魔法”的关键 ---
# a. 让子图在垂直方向上重叠
# hspace 是子图之间的高度间距，设置为负值即可重叠
g.fig.subplots_adjust(hspace=-0.5)

# b. 移除每个子图的 y 轴标签和标题
# 因为每个类别的名称已经显示在行首了，所以子图的标题和y轴是多余的
g.set_titles("")
g.set(yticks=[], ylabel="")

# c. 移除左侧和底部的边框线，让图形更“干净”
g.despine(bottom=True, left=True)

# d. 为整个图形添加一个 X 轴标签和总标题
plt.xlabel("Flipper Length (mm)", fontsize=12)
g.fig.suptitle('Distribution of Flipper Lengths by Species', y=1.02, fontsize=16) # y>1 确保标题在图形上方

plt.show()
```