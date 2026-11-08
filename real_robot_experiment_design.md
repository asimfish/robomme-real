（进度勾选：⬜未开始 🟨进行中 ✅完成）

# 真机 setting（Franka 单臂 | Astra 零样本，无训练数据）

> 目标会议：CVPR 2027。论文注册截止 11-10，正文截止 11-16，补充材料截止 11-23（均为 AoE）
> 仿真对照：RoboMME / RoboMME-Plus（https://robomme.github.io/plus.html）

## 1. 实验对比 Baseline

- Baseline-1：Astra direct（直接控制）。配置与仿真 RoboMME-Plus 榜单上的 Embodied-Astra 一致：文本历史 + 每个相机最近 2 帧 RGB
- Baseline-2：Astra + 长上下文。均匀抽取 K 帧历史图像放进上下文，token 预算与 Ours 相当；回答"是不是只是多给了图"（优先级：中，时间不够放补充材料）
- Ours：Astra + Harness
- 参考上限（可选）：Astra + Oracle 记忆。把真值记忆状态以文本形式给 Astra（如"目标在 2 号杯下""已完成 2/3"），测执行上限，对应 RoboMME 的 GroundSG+Oracle

各组共用：同一 Astra 版本和采样参数、同一 prompt 和任务指令、同一 move_eef 接口、同一相机、同一安全层、同一轮数上限。唯一差别是记忆部分。Harness 只能处理同一路观测流，不能用额外传感器或真值信息。

## 目标

- Harness 是否让 Astra 在四类记忆上"记对"：数对次数（计数）、找对被盖住的目标（物体永久性）、认对同一个物体（指代）、复现对动作（模仿）。
- 收益是否来自记忆而不是执行：记忆决策正确率提升，而在记忆决策正确的试验里，两组的执行成功率接近。
- 记忆负载下降曲线（drop curve）：随交换次数、计数 N、目标数增加，Baseline-1 成功率快速下降，Ours 下降更慢。
- 仿真与真机一致：真机上的提升与 RoboMME / RoboMME-Plus 仿真上的提升方向一致。

## 统一评估指标

- 成功率 SR：每条件 N=10，与 RoboMME 真机协议一致（时间够可加到 20）。初始摆放、目标、交换序列由 episode 卡片随机化
- 记忆决策正确率：每个任务单独定义，并标出随机猜的概率
- 平均完成时间 Time：总耗时 + 模型调用轮数；Ours 另报 harness 额外调用次数和 token
- 安全性：拒绝目标次数、越界、人工急停 / 人工干预次数

---

# A. 核心任务进度（MDT1 / MDT2 / MDT3 / MDT4）

> MDT = Memory-Dependent Task。四个任务与 RoboMME 论文的 4 个真机任务同名、同映射；每个任务分 Base（对应 RoboMME）和 Plus（对应 RoboMME-Plus）两档。
> 仿真参考数字均为 RoboMME-Plus 零样本榜单上 Embodied-Astra 的成功率。

## MDT1 计数：PutFruits（Counting，时间记忆；对应仿真 BinFill）

任务名称 / Prompt ⬜
- T1：Put two apples and one lemon into the bin, then press the button to stop.（数量和种类由卡片决定）

实验 Setting ⬜
- 场景：桌面 + 果篮（2–3 种塑料水果）+ 深的不透明桶（相机看不到桶内数量）+ USB 按钮（按下时间戳自动记日志）
- 相机：第三视角 + 腕部相机（待确认）
- 随机化：要求的数量和种类、水果在篮中的位置
- 难度：
  - Base：要求 2–3 个，1–2 种，物体静止
  - Plus：要求 4–6 个，2–3 种；中途人按脚本往篮里补水果（对应传送带陆续到达）；第 k 次放入后人从桶里取走 1 个（与 RoboMME 真机的人为干预一致）
- 重复次数：N=10 / 条件 / 难度
- 仿真参考：BinFill 30%
- 图片：

准备进度 ⬜
- 道具：
- episode 卡片：Base 10 张 + Plus 10 张 + dev 3 张
- 干预脚本：

系统配置进度 ⬜
- Baseline-1 配置与仿真对齐：
- Ours 配置冻结（版本号）：

实验测试效果 ⬜
- SR Base：Baseline-1 __/10 ｜ Ours __/10
- SR Plus：Baseline-1 __/10 ｜ Ours __/10
- 记忆决策（按按钮时各类水果的计数误差之和）：Baseline-1 __ ｜ Ours __
- 失败模式：提前按 / 多放 / 漏放 / 没按 / 抓取失败 / 安全停机 / 超时
- 实验视频：________

## MDT2 物体永久性：TrackCube（Permanence，空间记忆；对应 VideoUnmask / VideoUnmaskSwap）

任务名称 / Prompt ⬜
- T2（Base）：Watch the video carefully, then pick up the container hiding the green cube.
- T2（Plus）：Watch the video carefully, then pick up the container hiding the blue cube, followed by the one hiding the red cube, and finally the one hiding the green cube.

实验 Setting ⬜
- 场景：桌面 + 不透明塑料杯 + 红 / 绿 / 蓝泡沫方块。演示视频：人把方块盖进杯子，再交换杯子位置
- 相机：第三视角 + 腕部相机（待确认）
- 随机化：方块颜色与杯子的对应、交换序列、杯子初始位置
- 难度：
  - Base：3 个杯子，1 个目标，0–2 次交换
  - Plus：5 个杯子，3 个有序目标，3–5 次交换；交换中有一段用挡板遮住相机（对应 board occlusion）
- 演示视频每张卡片只录一次，两组看同一段视频；之后按卡片把杯子复位到录像结束时的位置
- 重复次数：N=10 / 条件 / 难度
- 仿真参考：VideoUnmask 12%，VideoUnmaskSwap 2%
- 图片：

准备进度 ⬜
- 道具：
- episode 卡片：
- 演示视频录制：__/20

系统配置进度 ⬜
- Baseline-1 配置与仿真对齐（视频帧数、抽帧方式）：
- Ours 配置冻结：

实验测试效果 ⬜
- SR Base：Baseline-1 __/10 ｜ Ours __/10
- SR Plus：Baseline-1 __/10 ｜ Ours __/10
- 记忆决策（第一个拿起的杯子是否正确；随机猜 Base 1/3，Plus 1/5）：Baseline-1 __ ｜ Ours __
- 失败模式：选错杯 / 顺序错 / 漏拿 / 抓取失败 / 碰倒杯子 / 安全停机
- 实验视频：________

## MDT3 指代：RepickBlock（Reference，物体记忆；对应 VideoRepick）

任务名称 / Prompt ⬜
- T3：Watch the video carefully, identify the block that was picked up, then pick up and place down the same block twice, finally press the button to stop.

实验 Setting ⬜
- 场景：桌面 + 外观完全相同的积木 + 按钮。演示视频：人拿起其中一块放回，然后滑动、交换积木位置
- 相机：第三视角 + 腕部相机（待确认）
- 随机化：被拿起的积木、交换序列、积木初始位置
- 难度：
  - Base：3 块，1–2 次交换
  - Plus：5–6 块，人连续滑动积木，局部遮挡
- 演示视频每张卡片只录一次，两组看同一段视频
- 重复次数：N=10 / 条件 / 难度
- 仿真参考：VideoRepick 18%
- 图片：

准备进度 ⬜
- 道具：
- episode 卡片：
- 演示视频录制：__/20

系统配置进度 ⬜

实验测试效果 ⬜
- SR Base：Baseline-1 __/10 ｜ Ours __/10
- SR Plus：Baseline-1 __/10 ｜ Ours __/10
- 记忆决策（第一次抓的是否是被演示的那块；随机猜 1/块数）：Baseline-1 __ ｜ Ours __
- 失败模式：认错积木 / 次数错 / 没按按钮 / 抓取失败 / 安全停机
- 实验视频：________

## MDT4 模仿：DrawPattern（Imitation，过程记忆；对应 PatternLock）

任务名称 / Prompt ⬜
- T4：Watch the video carefully, then use the drawing stick attached to the robot to reproduce the same pattern.

实验 Setting ⬜
- 场景：桌面 + 打印点阵板 + 夹爪夹一支泡沫头画笔。演示视频：画出一条经过若干节点的路径（人手持画笔或遥操作录制），演示后擦除
- 相机：第三视角 + 腕部相机（待确认）
- 随机化：路径节点序列、点阵板位置
- 难度：
  - Base：3×3 点阵，3–4 个节点
  - Plus：4×4 点阵，6–8 个节点；演示后平移或旋转点阵板；执行从 home 位姿开始
- 判定：末端轨迹投影到点阵后自动判定。笔尖进入节点 2 cm 半径内且低于接触高度，算经过该节点
- 重复次数：N=10 / 条件 / 难度
- 仿真参考：PatternLock 14%
- 图片：

准备进度 ⬜
- 道具：
- episode 卡片：
- 演示视频录制：__/20

系统配置进度 ⬜

实验测试效果 ⬜
- SR Base：Baseline-1 __/10 ｜ Ours __/10
- SR Plus：Baseline-1 __/10 ｜ Ours __/10
- 记忆决策（节点序列的归一化编辑距离）：Baseline-1 __ ｜ Ours __
- 失败模式：漏节点 / 多节点 / 顺序错 / 方向镜像 / 笔尖压桌触发安全停机
- 实验视频：________

---

# 进度看板

| 日期 | 内容 | 状态 |
|---|---|---|
| 11-08（日） | 定方案；准备道具；生成 episode 卡片 | ⬜ |
| 11-09（一） | 相机标定、安全层、按钮、日志；每个任务每组跑 2 次 dev 卡片，测单次耗时；跑一次 Oracle 记忆看执行上限 | ⬜ |
| 11-10（二） | **论文注册截止**。冻结 prompt 和 harness 配置；开跑 MDT2、MDT3 的 Base 档 | ⬜ |
| 11-11（三） | MDT1、MDT4 的 Base 档 | ⬜ |
| 11-12（四） | Plus 档 | ⬜ |
| 11-13（五） | Plus 档补完；重跑设备故障的试验；标注 | ⬜ |
| 11-14（六） | 统计、出图表 | ⬜ |
| 11-15 – 11-16 | 写作。**11-16 正文截止** | ⬜ |
| 11-17 – 11-23 | Baseline-2、Oracle、消融、扩展任务、补充视频。**11-23 补充材料截止** | ⬜ |
| 2027-01-25 后 | 按审稿意见补实验（rebuttal） | ⬜ |

试验量（单次耗时按 12 分钟估算，需用 11-09 的 dev 实测替换）：

| 档 | 内容 | 次数 | 机器人时间 |
|---|---|---|---|
| 正文必做 | MDT1–4 × Base/Plus × Baseline-1/Ours × 10 | 160 | 约 32 h |
| 正文精简（时间不够时） | MDT1–4 Base 全做 + Plus 只做 MDT1、MDT2 | 120 | 约 24 h |
| 补充材料 | Baseline-2、Oracle 在 Plus 档各 40 次 + drop curve 第三档 40 次 + 消融 + 扩展任务 | 约 200 | 约 40 h |

决策规则：dev 实测单次超过 15 分钟，正文改用精简版。

---

# 详细实验结果记录 v1（跑完填）

## MDT1 PutFruits

Prompt：
episode 卡片：Base 10 张 / Plus 10 张（文件：）
实验场景及道具：
[图片]

1、Baseline-1：Astra direct
1.1 Base
  测试结果：__/10 [成功次数/实验总次数]
  失败结果记录：
    提前按按钮（计数少）-- _ 次
    多放 -- _ 次
    抓取失败 -- _ 次
    安全停机 -- _ 次
  配置：Astra 版本 __ ｜ prompt 版本 __ ｜ 日志目录 __
  任务成功视频记录：
1.2 Plus
  测试结果：__/10
  失败结果记录：
  任务成功视频记录：

2、Ours：Astra + Harness
2.1 Base
  测试结果：__/10
  失败结果记录：
  配置：Astra 版本 __ ｜ harness 版本 __ ｜ 日志目录 __
  任务成功视频记录：
2.2 Plus
  测试结果：__/10
  失败结果记录：
  任务成功视频记录：

## MDT2 TrackCube
（同 MDT1 结构；失败原因按"选错杯 / 顺序错 / 漏拿 / 抓取失败 / 碰倒杯子 / 安全停机"记录）

## MDT3 RepickBlock
（同 MDT1 结构；失败原因按"认错积木 / 次数错 / 没按按钮 / 抓取失败 / 安全停机"记录）

## MDT4 DrawPattern
（同 MDT1 结构；另记每次的节点序列编辑距离）

---

# 一、实验总体目标

- 按记忆类型划分任务子集：四类各至少一个任务（计数 / 物体永久性 / 指代 / 模仿），与 RoboMME 的四个 suite 对应。
- 按难度分层：Base（对应 RoboMME）→ Plus（对应 RoboMME-Plus）。
- 记忆必要性：只看最后一帧必须无法完成任务。手段包括不透明杯、深桶、外观相同的积木、演示后擦除。
- 道具只用塑料 / 泡沫，不用玻璃仪器。原因：Astra 评测论文（arXiv 2609.24170 §5）报告真机上反复出现不安全动作并损坏设备；玻璃也易碎。
- 没有训练 / 测试划分：Astra 零样本，没有训练集。调试只用 dev 卡片，测试卡片只用于评测，测试前冻结 prompt 和 harness 配置（对应 RoboMME-Plus 的"只用训练 seed 适配"规则）。

区分几种记忆难度条件（对应原模板的"泛化条件"）：

1. 记忆负载
   1.1 单目标 / 计数 ≤ 3
   1.2 多目标 / 计数 4–6
   1.3 多目标且有顺序 / 计数 6–8
2. 历史干扰
   2.1 无交换
   2.2 1–2 次交换
   2.3 3–5 次交换或连续滑动
3. 观测缺失
   3.1 演示全程可见
   3.2 演示中有一段被挡板遮挡
4. 人为干预（计数任务）
   4.1 无干预
   4.2 按脚本在第 k 次后取走或补充物体
5. 布局变换（模仿任务）
   5.1 演示与执行布局一致
   5.2 演示后平移 / 旋转

底线是 Base 档全部完成，Plus 档更好。drop curve 在 MDT1（N = 2 / 4 / 6）和 MDT2（交换 0 / 2 / 4 次）上做。

# 二、真机任务集合

## 1. 计数（时间记忆）
- PutFruits（优先级：高）= MDT1
  条件：1.1、1.2（√），4.2（√），1.3（试试）
- PickXtimes（优先级：中）
  Prompt：Pick up the cube and place it on the target, repeating this pick-and-place action N times, then press the button to stop.
  每轮场景看起来一样，是纯计数记忆。N = 3 / 5 / 8。对应仿真 PickXtimes（56%）
- StopCube：不做。需要亚秒级时机，而 Astra 每轮要等一次模型调用，无法公平测试

## 2. 物体永久性（空间记忆）
- TrackCube（优先级：高）= MDT2
  条件：2.1、2.2（√），1.2、1.3（√），3.2（试试）
- ButtonUnmask（优先级：中）
  Prompt：First press the button, then pick up the container hiding the green cube, finally pick up another container hiding the red cube.
  开局方块可见，机器人按按钮后由人盖杯。信息只在 episode 早期可见，且由机器人自己的动作触发。对应仿真 ButtonUnmask（34%）

## 3. 指代（物体记忆）
- RepickBlock（优先级：高）= MDT3
  条件：2.2（√），2.3（试试），3.2（试试）
- PlaceOrder（优先级：中）
  Prompt：Watch the video carefully, then place the red cube on the third target it was previously placed on.
  演示中人依次把方块放到几个杯垫上；Plus 档在演示中交换杯垫。对应仿真 VideoPlaceOrder / VideoPlaceButton（均为 20%）

## 4. 模仿（过程记忆）
- DrawPattern（优先级：高）= MDT4
  条件：1.1、1.2（√），5.2（√）
- MoveCube（优先级：低）
  演示推、抓放、用工具拨三种方式之一，机器人需用同一种方式把方块移到目标。对应仿真 MoveCube（22%）
- InsertPeg、RouteStick：不做。精度要求高，而 Astra 精细操作弱（RoboDojo Precision 轴成功率 4%）

## 5. 对照（无需记忆，优先级：中）
- 所有信息当前都可见的单次抓放，用于确认 harness 不损害普通执行

# 三、实验流程（落到操作层面）

- 步骤 1：搭场景和安全层（两组共用，写进论文附录）

  | 约束 | 建议值（用 dev 卡片标定） | 超限处理 |
  |---|---|---|
  | 工作空间 | 桌面上方长方体；z 下限 = 桌面 + 1.5 cm（画笔任务按笔尖偏移另设） | 拒绝，把错误返回给模型 |
  | 单步位移 / 转角 | ≤ 20 cm / ≤ 45° | 拒绝，把错误返回给模型 |
  | 笛卡尔速度 | ≤ 0.15 m/s | 规划器限速 |
  | 碰撞阈值 | 调低 libfranka collision behavior 阈值，用阻抗控制 | 触发反射后停止 |
  | 夹爪力 | ≤ 20 N | 限幅 |
  | 人工急停 | 操作员全程手持急停 | 计为安全停机，该次失败 |
  | 轮数 / 时间上限 | 如 60 轮或 20 分钟，按 dev 校准 | 超时失败 |

- 步骤 2：生成 episode 卡片。用固定随机种子，每个任务每个难度 10 张测试卡片 + 3 张 dev 卡片。卡片写明物体网格坐标、颜色 / 种类、要求数量、交换序列、目标身份、点阵路径、干预脚本。桌面铺带网格坐标的打印垫，按坐标摆放。
- 步骤 3：dev 试跑。每个任务每组跑 2 次，测单次耗时；再跑一次 Oracle 记忆看执行上限。如果 Base 档的 Oracle 成功率低于 30%，先降低执行难度（放大物体、加大间距、放宽容差），再冻结配置，调整过程写进附录。
- 步骤 4：正式评测
  - 视频类（MDT2 / 3 / 4）：每张卡片用机器人相机录一次演示，两组喂同一段帧序列；物理场景按卡片复位到录像结束时的摆放
  - 计数类（MDT1）：历史由机器人自己的动作产生，无法回放；按卡片配对初始摆放和干预脚本
  - 同一张卡片的两组同一天连续跑；卡片之间按 ABBA 交替先后，抵消光照、设备、API 随时间的漂移
  - 重跑规则：设备、网络、API 报错可以重跑同一张卡片并记录次数；策略导致的安全停机、超时、give_up 一律计失败，不重跑
  - 每次试验一个 run ID，记录：全部相机流、每轮模型输入输出、harness 记忆状态、工具调用及接受 / 拒绝、末端轨迹、按钮时间戳、安全事件、耗时、轮数、token、操作员备注
- 步骤 5：判定。能自动判的自动判（按钮日志、末端轨迹）；其余由 2 人看录像判定，录像不显示条件标签，有分歧时讨论决定。
- 步骤 6：做对比和消融（见四）。

# 四、与系统结构对齐的对比实验设计

- Ours vs Baseline-1：全部任务，论文主表
- Ours vs Baseline-2（长上下文）：Plus 档，看收益是不是只来自多给的图像
- Ours vs 去掉 [模块 A]：重点看 __ 任务
- Ours vs 去掉 [模块 B]：重点看 __ 任务
- Ours vs Oracle 记忆：执行上限，剩下的差距归因于执行
- 离线探针（不占机器人时间）：
  - 无历史探针：只给 Astra 最后一帧 + 指令，让它回答"选哪个杯子 / 哪块积木 / 已放几个"，正确率应接近随机猜，以此证明任务需要记忆
  - 记忆问答：在录好的历史上让 Baseline-1 和 Ours 回答记忆问题，不执行动作。形式与 RoboMME 的人类 VideoQA baseline 相同，可以用几十到上百个样本补充真机每格 10 次的统计量

统一指标：成功率、记忆决策正确率、平均完成时间、人工干预次数、是否在安全范围内动作。
统计：每格报 x/10 和 Wilson 95% 置信区间；配对样本合并后做精确 McNemar 检验。不对单个任务的 10 次结果下显著性结论。

论文呈现：
- 图：硬件布置 + 4 个任务，标出每个任务需要记住的信息
- 表：真机成功率，行 = 任务 × 难度，列 = Baseline-1 / Ours，旁边放对应仿真任务的数字
- 图：仿真提升 vs 真机提升散点图
- 图：失败分类堆叠柱状图
- 图：一次 rollout 的时间轴，对比 harness 记忆状态、真值和 Baseline-1 的出错点
- 补充材料表：Baseline-2、Oracle、消融、开销（轮数、耗时、token）

背景引用（不放进同一张表比较）：RoboMME 论文的真机结果为 π0.5 4/40、GroundSG+QwenVL 19/40、FrameSamp+Modul 25/40。那些是训练过的 VLA，硬件和道具也不同。

# 目标任务示例

[图片]

---

# 待确认

1. Harness 有哪些模块（用于四里的消融命名）？Harness 会不会额外调用模型？
2. 仿真里的 Astra 直接控制配置（视频怎么输入、保留几帧、轮数上限）是否就是 RoboMME-Plus 榜单上的 Embodied-Astra 条目？
3. Astra 单轮时延和一次 episode 的平均轮数是多少？这决定正文能否做满 160 次。
4. Franka 现有的相机布置（是否有双肩 + 腕部）？
5. 可同时到场的操作员和标注者有几人？
