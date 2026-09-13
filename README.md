# Quantitative Research Learning Portfolio

经典券商金融工程研报的复现与扩展，覆盖市场择时、微观交易结构因子和多因子指数增强。项目使用 Python 与 Jupyter Notebook 实现，展示研究逻辑拆解、数据处理、因子构建、回测评价和组合优化流程。

> 本仓库用于个人学习与研究交流，不构成投资建议。历史回测结果不代表未来表现。

## Projects

| 项目 | 研究主题 | 核心实现 |
| --- | --- | --- |
| [RSRS 择时](./rsrs-timing/RSRS_2017.ipynb) | 沪深 300 市场择时 | 滚动 OLS、标准分、拟合优度修正、右偏标准分、交易成本与风险指标 |
| [聪明钱因子](./smart-money-factor/因子计算以及回测.ipynb) | 分钟行情中的价格发现能力 | Q 因子、参数网格、Rank IC、分组收益与多空组合 |
| [多因子指数增强](./multi-factor-enhancement/华泰多因子指数增强-2018.ipynb) | A 股多因子选股与组合构建 | 因子清洗、中性化、正交化、IC/IR 加权和约束优化 |

## RSRS 择时

在滚动窗口内建立最高价与最低价的线性模型：

$$
High_t = \alpha + \beta Low_t + \varepsilon_t
$$

使用斜率衡量阻力与支撑的相对强度，并进一步构造标准分、拟合优度修正标准分和右偏标准分。代码将交易信号滞后一期执行，对比均线、布林带与 RSRS 策略，并计算年化收益、累计收益、最大回撤和夏普比率。

## 聪明钱因子

使用分钟收益与成交量构造交易的“聪明程度”：

$$
S_t = \frac{|R_t|}{Volume_t^{\beta}}
$$

按 $S_t$ 排序后，选取累计成交量靠前的高信息交易，通过其成交量加权平均价与全时段成交量加权平均价之比构造 Q 因子。项目实现了月末计算、次月首个交易日调仓的时间对齐，并使用 Rank IC、分组收益及多空组合评价因子。

代码入口：

- [`数据获取.ipynb`](./smart-money-factor/数据获取.ipynb)：在聚宽研究环境获取并导出数据；
- [`因子计算以及回测.ipynb`](./smart-money-factor/因子计算以及回测.ipynb)：本地因子计算与回测。

## 多因子指数增强

构建估值、成长、盈利质量、动量、波动率和流动性因子，包括 EP、SP、BP、SUE、SUR、ROE/ROA 变化、ROC、VOL、ATR 和 ILLIQ。项目实现截面清洗、行业和市值中性化、因子正交化、Rank IC 与滚动 IR 加权，并在行业暴露、全投资和个股权重上限约束下求解组合权重。

组合优化采用 HiGHS 线性规划求解器，删除与全投资约束线性相关的冗余行业约束。每期求解后重新验证权重和、行业暴露及上下界；求解失败或约束误差超限时回退至基准指数权重。

## Data and reproducibility

本仓库不公开分发完整商业行情。数据分为两种运行模式：

- **Demo mode**：使用 `data/sample/` 中许可公开或合成的小型样例数据，验证完整代码链路；
- **Full mode**：用户使用自己的 Tushare 或聚宽权限获取完整数据，本地原始数据不提交至 Git。

数据目录和授权注意事项见 [`data/README.md`](./data/README.md)。API Token 必须通过环境变量或本地配置文件管理，不得写入代码。

## Tech stack

Python、pandas、NumPy、SciPy、statsmodels、Matplotlib、Seaborn、empyrical、Tushare、聚宽 `jqdata/jqfactor`、Jupyter Notebook。

## Current limitations

- 当前以研究型 Notebook 为主，尚未完全模块化；
- 完整回测依赖用户自己的行情数据权限；
- Demo 数据集、统一依赖文件和自动化测试仍待补充；
- 回测结果仍需进一步进行样本外检验、参数敏感性分析和交易成本压力测试。

## References

- 光大证券 RSRS 择时系列研究；
- 开源证券聪明钱因子相关研究；
- 华泰证券多因子指数增强相关研究。

本仓库为独立学习复现，原始研报版权归相应机构及作者所有。
