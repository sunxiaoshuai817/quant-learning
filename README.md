# Quantitative Research Portfolio

基于经典券商金融工程研究完成的三项 A 股量化策略复现与改进，覆盖市场择时、分钟级交易行为因子和多因子指数增强。仓库重点呈现从经济逻辑、数据处理、因子构建、信号检验到组合优化与回测评价的完整研究流程。

> 本仓库用于个人研究与方法展示，不构成投资建议。历史回测不代表未来表现。

## Research Highlights

- **研究链路完整**：实现数据清洗、因子构造、截面标准化、中性化、正交化、Rank IC、分组检验、组合构建及绩效评价。
- **重视回测可信度**：交易信号滞后执行，显式处理因子计算日、调仓日与下期收益的时间对齐，并在择时策略中计入交易成本。
- **在复现基础上处理实际问题**：针对严格风险中性线性规划频繁求解失败，引入“严格中性 → 暴露受限放宽 → 基准回退”的分层优化机制，并对返回权重进行事后约束校验。
- **兼顾研究与工程实现**：使用聚宽完成数据获取，使用本地 Python 环境完成因子研究；敏感凭据通过环境变量管理，商业数据不直接写入代码。

## Projects

| 项目 | 核心问题 | 方法与实现 | 研究区间 |
| --- | --- | --- | --- |
| [RSRS Market Timing](./rsrs-timing/RSRS_2017.ipynb) | 支撑位与阻力位的相对强度能否刻画市场状态 | 滚动 OLS、标准分、$R^2$ 修正、右偏标准分、交易成本和风险指标 | 2005–2017 |
| [Smart Money Factor](./smart-money-factor/因子计算以及回测.ipynb) | 分钟级成交中哪些交易具有更强的价格发现能力 | 30 分钟行情、Q 因子、参数网格、Rank IC、分组收益及多空组合 | 2013–2019 |
| [Multi Factor Index Enhancement](./multi-factor-enhancement/华泰多因子指数增强-2018.ipynb) | 如何将多因子预期收益转化为风险受控的沪深 300 增强组合 | 因子清洗与正交化、滚动 IR 加权、分层抽样、约束线性规划 | 2010–2020 |

## 1 RSRS Market Timing

### Research idea

在长度为 $N$ 的滚动窗口中，以最低价解释最高价：

$$
High_t = \alpha + \beta Low_t + \varepsilon_t
$$

斜率 $\beta$ 衡量阻力与支撑的相对变化。项目由基础斜率出发，依次构造滚动标准分、拟合优度修正标准分和右偏标准分，并检验不同指标区间与后续收益、上涨概率的关系。

### Implementation details

- 使用滚动 OLS 同时计算斜率与 $R^2$；
- 对历史斜率进行标准化，降低绝对水平随市场状态变化的影响；
- 使用 $R^2$ 修正信号质量，并加入斜率方向构造右偏标准分；
- 将当日生成的交易信号滞后一期执行，避免直接使用当日收益；
- 与均线、布林带策略进行对照；
- 比较费前与费后净值，并计算年化收益、最大回撤和夏普比率。

**Notebook:** [`rsrs-timing/RSRS_2017.ipynb`](./rsrs-timing/RSRS_2017.ipynb)

## 2 Smart Money Factor

### Research idea

以分钟收益的绝对变化和成交量构造交易的“聪明程度”：

$$
S_t = \frac{|R_t|}{Volume_t^{\beta}}
$$

按 $S_t$ 从高到低排序，选择累计成交量靠前的交易，比较其成交量加权平均价与全时段成交量加权平均价，得到 Q 因子。因子试图识别对价格变化贡献更高、但不完全依赖大额成交的交易信息。

### Implementation details

- 从聚宽批量导出月末前 10 个交易日的 30 分钟行情；
- 过滤 ST、上市不足 60 日等不符合条件的股票；
- 明确区分月末因子计算日与次月首个交易日调仓日；
- 使用下一调仓日价格计算持有期收益，减少时间错配；
- 对成交量惩罚参数 $\beta$ 进行网格检验；
- 使用 Spearman Rank IC、分组收益和多空组合评价因子有效性。

**Notebooks:**

- [`smart-money-factor/数据获取.ipynb`](./smart-money-factor/数据获取.ipynb)：聚宽数据获取与导出；
- [`smart-money-factor/因子计算以及回测.ipynb`](./smart-money-factor/因子计算以及回测.ipynb)：本地因子计算与检验。

## 3 Multi Factor Index Enhancement

### Research idea

构建覆盖估值、成长、盈利质量、动量、波动率和流动性的候选因子，包括 EP、SP、BP、SUE、SUR、ROE/ROA 变化、ROC、VOL、ATR 和 ILLIQ。截面因子经过去极值、缺失值处理、标准化、行业与市值中性化及对称正交化后，以滚动 Rank IC 信息比率构造复合得分。

### Portfolio construction

仓库实现两种组合构建方法：

1. **分层抽样**：在行业与市值分组内选择综合得分较高的股票，并继承对应组的基准权重；
2. **线性规划**：最大化组合预期 Alpha，同时控制风险因子和行业暴露、全投资约束及个股权重上限。

线性规划模型为：

$$
\max_w \quad w^T f_{\mathrm{alpha}}
$$

$$
\text{s.t.}\quad w^T X_f = w_{\mathrm{index}}^T X_f,\qquad
w^T\mathbf{1}=1,\qquad
0\leq w_i\leq 2w_{\mathrm{index},i}
$$

### Optimization improvement

原始严格中性模型在部分月份无法获得有效最优解。为避免直接使用未通过约束检验的求解器输出，项目增加分层处理：

1. 首先求解风险因子与行业暴露严格中性的组合；
2. 严格求解失败时，将各项标准化风险暴露及行业权重相对基准的允许偏离放宽至 $\pm0.05$；
3. 对权重和、风险暴露及个股上下界进行事后校验；
4. 两阶段均失败时才回退至基准指数权重。

2010–2020 年的 129 个月度截面中：

- 6 期获得严格中性解；
- 123 期获得暴露受限的放宽解；
- 0 期需要回退至指数权重。

该结果表明，严格中性约束显著压缩了因子得分的优化空间；在明确控制最大风险暴露偏离后，放宽模型能够稳定产生可行组合。代码同时输出每期求解层级、相对基准的因子得分提升及最大暴露偏离，便于诊断约束是否持续触及边界。

### Backtest snapshot

分层抽样方案在 Notebook 保存的聚宽回测中取得以下结果。该指标属于分层抽样方案，不与后续线性规划结果混用：

| Metric | Value |
| --- | ---: |
| Annualized portfolio return | 16.28% |
| Annualized benchmark return | 2.41% |
| Alpha | 13.87% |
| Information ratio | 2.68 |
| Excess return Sharpe | 1.83 |
| Portfolio max drawdown | 40.75% |

**Notebook:** [`multi-factor-enhancement/华泰多因子指数增强-2018.ipynb`](./multi-factor-enhancement/华泰多因子指数增强-2018.ipynb)

## Reproducibility

项目运行依赖聚宽和 Tushare 数据权限。仓库不直接分发完整商业行情数据；用户可使用自己的数据权限运行数据获取 Notebook。数据目录约定和授权注意事项见 [`data/README.md`](./data/README.md)。

当前仓库保留了主要研究输出，便于在无法访问数据服务时查看研究过程。后续将补充许可公开或合成的最小样例数据、统一依赖说明及结果图，以支持轻量级端到端演示。

## Tech Stack

Python · pandas · NumPy · SciPy · statsmodels · Matplotlib · Seaborn · empyrical · Tushare · JoinQuant · Jupyter Notebook

## Limitations

- 研究结果基于历史样本，尚未完成完整的滚动样本外检验；
- 线性规划中的 $\pm0.05$ 风险暴露容忍度属于研究设定，仍需进行敏感性分析；
- 当前未在目标函数中显式惩罚换手率，组合收益可能对交易成本敏感；
- 不同数据源的复权、停牌和涨跌停处理可能造成复现差异；
- Notebook 尚未完全模块化，暂不定位为生产交易系统。

## References

- 光大证券 RSRS 择时系列研究；
- 开源证券聪明钱因子相关研究；
- 华泰证券多因子指数增强相关研究。

本仓库为个人独立学习复现，原始研究版权归相应机构及作者所有。
