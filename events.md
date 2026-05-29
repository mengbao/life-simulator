# 模拟人生 - 事件数据库

> 本文件为 SKILL.md 的事件详细数据补充，包含所有事件的触发条件、前置条件、描述文案和选项效果。
> **所有事件必须严格遵守 SKILL.md 中「逻辑自洽与边界控制」章节的规则。**

---

## 事件结构

每个事件包含以下字段：

```
id:            事件唯一标识
stage:         所属阶段（baby/child/teen/young/middle/old）
trigger:       触发条件（age/属性阈值/随机概率/连锁）
type:          事件类型（age/attribute/random/chain）
rarity:        稀有度（common/uncommon/rare/legendary）
prerequisites: 前置条件（必须全部满足，含flags/互斥/冷却期）
description:   事件描述（200字以内，幽默风格）
narrative_ref: 叙事回溯引用点（需引用的历史选择）
choices:       选项列表（2-4个）
flags_set:     本事件设置的标记
flags_block:   本事件会阻止的后续事件
chain:         可能触发的连锁事件ID
cooldown:      触发后冷却年数
```

### 前置条件编写规范

```
prerequisites:
  - age: [最小, 最大]           // 年龄范围
  - stage: [阶段]              // 当前阶段
  - stats: {属性: 阈值}        // 属性要求
  - flags_required: [标记列表] // 必须存在的标记
  - flags_blocked: [标记列表]  // 不能存在的标记（互斥）
  - not_triggered: [事件ID列表] // 本事件不能已触发过
  - cooldown: [事件类别, 年数]  // 冷却期检查
```

---

## 一、婴儿期事件（0-3岁）

### EVT_B001 - 出生天赋
- **触发**：出生时必定
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [0, 0]
  - stage: baby
  - flags_blocked: 无（首个事件）
- **描述**：护士把你抱给爸妈看，你居然对着他们笑了！护士惊讶地说"这孩子真机灵"
- **叙事引用**：无（首个事件，无历史）
- **选项**：
  - A. 爱笑宝宝 → charm+5
  - B. 安静观察 → iq+5
  - C. 哭闹不止 → health-3，但父母更多关注（后期wealth+1000）
- **flags_set**：birth_talent = smile/observe/cry
- **cooldown**：无

### EVT_B002 - 抓周仪式
- **触发**：1岁，必定
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [1, 1]
  - stage: baby
  - not_triggered: [EVT_B002]
- **描述**：家人摆了一桌子东西让你抓，你爬过去...
- **叙事引用**：引用EVT_B001（"还记得你出生时[哭闹/微笑/安静]的样子吗？抓周时你完全不一样..."）
- **选项**：
  - A. 抓住计算器 → iq+5
  - B. 抓住口红 → charm+5
  - C. 抓住钞票 → wealth+2000
  - D. 抓住鸡腿 → health+5
- **flags_set**：无
- **cooldown**：无

### EVT_B003 - 第一次生病
- **触发**：2岁，health<60时
- **类型**：attribute
- **稀有度**：common
- **前置条件**：
  - age: [2, 2]
  - stage: baby
  - stats: {health: <60}
  - not_triggered: [EVT_B003]
- **描述**：你发高烧到39度，爸妈急得团团转...
- **叙事引用**：引用birth_talent（"平时[活泼/安静/爱哭]的你，今天蔫了..."）
- **选项**：
  - A. 乖乖吃药 → health+3, iq+2
  - B. 死活不吃 → health-5, luck+3
- **flags_set**：无
- **cooldown**：无

### EVT_B004 - 家庭环境
- **触发**：0岁，必定
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [0, 0]
  - stage: baby
  - not_triggered: [EVT_B004]
- **描述**：你出生在一个[家庭背景]家庭
- **叙事引用**：无
- **选项**：无（被动事件）
- **效果**：
  - 富裕家庭（20%概率）：wealth+10000, iq+3
  - 普通家庭（60%概率）：wealth+3000
  - 贫困家庭（20%概率）：wealth+0, health-3
- **flags_set**：family_background = rich/normal/poor
- **cooldown**：无

---

## 二、童年事件（4-12岁）

### EVT_C001 - 兴趣班选择
- **触发**：6岁，必定
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [6, 6]
  - stage: child
  - not_triggered: [EVT_C001]
- **描述**：妈妈给你报了兴趣班，说是要"赢在起跑线上"。但你想选的和妈妈想让你选的不一样...
- **叙事引用**：引用birth_talent和抓周结果（"你从小就[活泼/安静]，抓周时[选了XX]，妈妈觉得你应该..."）
- **选项**：
  - A. 奥数班 → iq+8, health-2
  - B. 舞蹈班 → charm+8, wealth-500
  - C. 足球班 → health+8, iq-2
  - D. 什么都不学 → eq+5
- **flags_set**：hobby = math/dance/sport/none
- **flags_block**：无，但影响后续职业路径
- **cooldown**：无

### EVT_C002 - 考试作弊风波
- **触发**：8-10岁，随机（30%）
- **类型**：random
- **稀有度**：common
- **前置条件**：
  - age: [8, 10]
  - stage: child
  - not_triggered: [EVT_C002]
  - cooldown: [negative_event, 3]
- **描述**：数学考试你完全不会，同桌给你传了小抄...
- **叙事引用**：引用hobby（"你在[奥数班/舞蹈班/足球班]的表现[还不错/一般]，但这次考试..."）
- **选项**：
  - A. 果断拒绝 → iq+3, eq-2
  - B. 偷偷看一眼 → luck测试：成功标记cheated，失败eq-5
  - C. 主动举报 → eq-5, 标记teacher_favor
- **flags_set**：cheated(条件性) / teacher_favor(条件性)
- **cooldown**：3年

### EVT_C003 - 校园霸凌
- **触发**：9岁，charm<50或eq<50时
- **类型**：attribute
- **稀有度**：uncommon
- **前置条件**：
  - age: [9, 9]
  - stage: child
  - stats: {charm: <50} 或 {eq: <50}（满足其一即可）
  - not_triggered: [EVT_C003]
- **描述**：班里有几个同学总是欺负你，抢你的零食，还给你起外号...
- **叙事引用**：引用hobby（"你在[兴趣班]交到的朋友不多/不少，但班里有几个混蛋..."）
- **选项**：
  - A. 告诉老师和家长 → eq+3
  - B. 打回去 → health-5, charm+5
  - C. 花钱收买他们 → wealth-1000, 标记has_followers（需wealth≥1000）
  - D. 默默忍受，化悲愤为学习动力 → eq-3, iq+5
- **flags_set**：has_followers(条件性) / personality倾向更新
- **cooldown**：无（一次性事件）

### EVT_C004 - 天赋发现
- **触发**：10岁，iq>70时
- **类型**：attribute
- **稀有度**：rare
- **前置条件**：
  - age: [10, 10]
  - stage: child
  - stats: {iq: >70}
  - not_triggered: [EVT_C004]
  - flags_blocked: [prodigy]（已有天赋标记则不再触发）
- **描述**：你在课堂上随口说出的答案，居然比老师的方法还简便！全班都安静了...
- **叙事引用**：引用hobby（"你在[奥数班]的刻苦训练终于有了回报..."）
- **效果**：iq+5, 标记prodigy
- **flags_set**：prodigy = true
- **chain**：可触发 EVT_T003（天才培养）
- **cooldown**：无

### EVT_C005 - 初恋萌芽
- **触发**：12岁，charm>60时
- **类型**：attribute
- **稀有度**：uncommon
- **前置条件**：
  - age: [11, 12]
  - stage: child
  - stats: {charm: >60}
  - not_triggered: [EVT_C005]
- **描述**：你发现自己总是忍不住看班里的那个TA...上课走神，下课偷看
- **叙事引用**：引用hobby（"你在[舞蹈班]练舞时，TA总是在窗外偷偷看你..."）
- **选项**：
  - A. 写情书表白 → luck测试：成功eq+10，失败eq-5
  - B. 默默喜欢 → eq+3
  - C. 告诉闺蜜/兄弟 → 消息传开charm-3, eq+2
- **flags_set**：romance_count+1(条件性)
- **cooldown**：无

### EVT_C006 - 家庭变故
- **触发**：随机，luck<40时（20%）
- **类型**：random
- **稀有度**：uncommon
- **前置条件**：
  - age: [7, 12]
  - stage: child
  - stats: {luck: <40}
  - not_triggered: [EVT_C006]
  - flags_blocked: [family_saved]（如果之前已挽救家庭，不再触发）
  - cooldown: [negative_event, 3]
- **描述**：爸妈吵架了，摔了碗，说要离婚...你躲在房间里不敢出声
- **叙事引用**：引用family_background（"你[富裕/普通/贫困]的家庭也逃不过这个问题..."）
- **选项**：
  - A. 哭着求他们不要离 → eq+5, health-3
  - B. 假装不知道 → iq+3, eq-5
  - C. 找长辈帮忙调解 → eq+8, 标记family_saved
- **flags_set**：family_saved(条件性)
- **cooldown**：5年

### EVT_C007 - 童年意外
- **触发**：随机，luck<30时（15%）
- **类型**：random
- **稀有度**：uncommon
- **前置条件**：
  - age: [7, 12]
  - stage: child
  - stats: {luck: <30}
  - not_triggered: [EVT_C007]
  - cooldown: [health_event, 5]
- **描述**：你从树上摔下来了！膝盖流了好多血...
- **叙事引用**：引用hobby（"你在[足球班]练完球后爬树..."）
- **选项**：
  - A. 去医院缝针 → wealth-2000, health恢复
  - B. 自己用创可贴糊弄 → health-5, charm+2
- **flags_set**：无
- **cooldown**：5年

---

## 三、青少年事件（13-18岁）

### EVT_T001 - 文理分科
- **触发**：15岁，必定
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [15, 15]
  - stage: teen
  - not_triggered: [EVT_T001]
- **描述**：高一结束，你要选择文科还是理科了...这个选择可能影响你的一生
- **叙事引用**：引用hobby和prodigy（"你在[奥数班/舞蹈班]的经历让你对[理科/艺术]更有信心..."）
- **选项**：
  - A. 理科（需iq≥60）→ iq+5, 标记education_track=science
  - B. 文科（需eq≥60）→ eq+5, 标记education_track=arts
  - C. 艺术班（需charm≥70）→ charm+8, wealth-3000, 标记education_track=art
  - D. 体育班（需health≥70）→ health+8, iq-3, 标记education_track=sport
- **flags_set**：education_track = science/arts/art/sport
- **flags_block**：其他education_track值
- **cooldown**：无（一次性，终身锁定）

### EVT_T002 - 网瘾危机
- **触发**：16岁，随机（25%）
- **类型**：random
- **稀有度**：common
- **前置条件**：
  - age: [15, 17]
  - stage: teen
  - not_triggered: [EVT_T002]
  - flags_blocked: [prodigy]（天才儿童不太可能沉迷游戏）
  - cooldown: [negative_event, 3]
- **描述**：你迷上了一款网游，每天偷偷玩到凌晨3点...成绩直线下降
- **叙事引用**：引用education_track（"你在[理科/文科/艺术班]的压力下，找到了逃避的方式..."）
- **选项**：
  - A. 继续沉迷 → health-10, iq-5, 标记gaming_addict
  - B. 悬崖勒马 → iq+5, health+5
  - C. 边玩边学 → luck测试
- **flags_set**：gaming_addict(条件性)
- **chain**：gaming_addict → CHAIN_001（电竞路线）
- **cooldown**：3年

### EVT_T003 - 天才培养
- **触发**：16岁，有prodigy标记时
- **类型**：chain
- **稀有度**：rare
- **前置条件**：
  - age: [15, 17]
  - stage: teen
  - flags_required: [prodigy]
  - flags_required: [education_track=science]（只有理科天才才会被发掘）
  - not_triggered: [EVT_T003]
- **描述**：省里的奥数教练注意到了你，邀请你加入集训队...
- **叙事引用**：引用EVT_C004天赋发现（"还记得10岁那次惊艳全班吗？你的才华终于被更高层级的人注意到了..."）
- **选项**：
  - A. 加入集训队 → iq+10, health-5, wealth-5000
  - B. 婉拒，保持正常生活 → eq+3
- **flags_set**：无（prodigy标记已存在）
- **chain**：→ CHAIN_003（学术大牛路线）
- **cooldown**：无

### EVT_T004 - 初恋正式版
- **触发**：16-17岁，charm>50且eq>50时
- **类型**：attribute
- **稀有度**：common
- **前置条件**：
  - age: [16, 17]
  - stage: teen
  - stats: {charm: >50, eq: >50}
  - not_triggered: [EVT_T004]
  - cooldown: [romance_event, 5]
- **描述**：隔壁班的TA向你表白了！心跳加速，手心出汗...
- **叙事引用**：引用EVT_C005（如果触发过）（"还记得小学时那个偷偷喜欢的人吗？这次是真的..."）
- **选项**：
  - A. 接受，认真恋爱 → eq+10, iq-3, romance_count+1
  - B. 接受，但保持理性 → eq+5, iq+2, romance_count+1
  - C. 拒绝，专注学习 → iq+8, eq-3
  - D. 拒绝，因为心有所属 → charm+5
- **flags_set**：romance_count+1(条件性)
- **cooldown**：5年

### EVT_T005 - 高考压力
- **触发**：18岁，必定
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [18, 18]
  - stage: teen
  - not_triggered: [EVT_T005]
  - flags_required: [education_track]（必须已选科）
- **描述**：高考在即，你每天只睡5个小时，咖啡当水喝...
- **叙事引用**：引用education_track（"你在[理科/文科/艺术班]奋斗了三年，终于到了决战时刻..."）
- **选项**：
  - A. 拼命学习 → iq+8, health-8
  - B. 劳逸结合 → iq+5, health+2
  - C. 已经放弃 → iq-5, health+3, luck+5
  - D. 作弊准备 → luck测试
- **flags_set**：exam_cheated(条件性)
- **cooldown**：无

### EVT_T006 - 高考结果
- **触发**：18岁结束，必定
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [18, 18]
  - stage: teen
  - flags_required: [education_track]
  - not_triggered: [EVT_T006]
- **描述**：成绩出来了！你紧张地打开查分网站...
- **叙事引用**：引用EVT_T005选择和education_track（"你[拼命/佛系/作弊]了三年[理科/文科/艺术]，结果..."）
- **计算**：基础分 = iq × 0.8 + 努力加成 + luck随机(-10~+10)
  - exam_cheated标记：+20分（但有道德污点）
  - gaming_addict标记：-15分
  - prodigy标记：+10分
- **结果**：
  - ≥700：edu_top → iq+10, wealth+5000
  - 600-700：edu_good → iq+5
  - 500-600：edu_normal → 无变化
  - 400-500：edu_college → iq-2, wealth+3000
  - <400：edu_failed → iq-5
- **flags_set**：education = edu_top/edu_good/edu_normal/edu_college/edu_failed
- **flags_block**：其他education值（终身锁定）
- **cooldown**：无

### EVT_T007 - 校园暴力事件
- **触发**：随机，luck<35时（15%）
- **类型**：random
- **稀有度**：uncommon
- **前置条件**：
  - age: [14, 18]
  - stage: teen
  - stats: {luck: <35}
  - not_triggered: [EVT_T007, EVT_C003]（童年霸凌已触发则不再）
  - cooldown: [negative_event, 3]
- **描述**：放学路上，几个社会青年拦住你要钱...
- **叙事引用**：引用EVT_C003（如果触发过）（"上次的霸凌你[打回去/告诉老师]了，这次换成了校外的..."）
- **选项**：
  - A. 乖乖给钱 → wealth-500, health-2
  - B. 奋力反抗 → luck测试
  - C. 跑路 → health+3, eq+3
- **flags_set**：无
- **cooldown**：3年

---

## 四、青年事件（19-35岁）

### EVT_Y001 - 大学专业选择
- **触发**：19岁，有大学标记时
- **类型**：chain
- **稀有度**：common
- **前置条件**：
  - age: [19, 19]
  - stage: young
  - flags_required: [edu_top 或 edu_good 或 edu_normal]
  - flags_required: [education_track]
  - not_triggered: [EVT_Y001]
- **描述**：你要选专业了，这决定了你未来的职业方向...
- **叙事引用**：引用education_track和hobby（"你从[奥数班/舞蹈班]开始，经过[理科/艺术班]，现在..."）
- **选项**（受education_track约束）：
  - education_track=science：可选 计算机/金融/医学/师范
  - education_track=arts：可选 金融/师范/新闻
  - education_track=art：可选 设计/艺术/师范
  - education_track=sport：可选 体育/师范
- **flags_set**：career方向预标记
- **cooldown**：无

### EVT_Y002 - 第一份工作
- **触发**：22岁（大学毕业）或19岁（未上大学）
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [19, 25]（大学毕业22岁，未上大学可更早）
  - stage: young
  - not_triggered: [EVT_Y002]
  - flags_blocked: [career]（已有职业标记则不触发）
- **描述**：你开始找工作了，投了无数简历...
- **叙事引用**：引用education和education_track（"你带着[顶尖大学/普通本科/大专/无学历]的背景，[专业方向]的技能..."）
- **选项**（受education和属性约束，参考SKILL.md职业准入矩阵）：
  - 只显示满足准入条件的职业选项
  - 不满足条件的选项显示为灰色并说明原因
- **flags_set**：career = 具体职业名
- **flags_block**：其他career值（除非触发转行事件）
- **cooldown**：无

### EVT_Y003 - 职场第一次升职
- **触发**：25岁，有工作标记时
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [24, 28]
  - stage: young
  - flags_required: [career]（必须有职业）
  - not_triggered: [EVT_Y003]
- **描述**：主管位置空缺，你和同事竞争...
- **叙事引用**：引用career（"你在[职业名]已经干了3年了，终于等到了机会..."）
- **选项**：
  - A. 凭实力竞争 → iq测试
  - B. 拉拢关系 → eq测试
  - C. 给领导送礼 → wealth-10000, 标记unethical
  - D. 佛系不争 → health+3
- **flags_set**：unethical(条件性)
- **cooldown**：5年

### EVT_Y004 - 创业机会
- **触发**：27岁，iq≥75且wealth≥20000时
- **类型**：attribute
- **稀有度**：rare
- **前置条件**：
  - age: [25, 35]
  - stage: young
  - stats: {iq: ≥75, wealth: ≥20000}
  - flags_required: [career]（有工作经验才能创业）
  - not_triggered: [EVT_Y004]
  - flags_blocked: [entrepreneur]（已创业成功不再触发）
  - cooldown: [career_event, 5]
- **描述**：同学找你合伙创业，项目看起来很有前景...
- **叙事引用**：引用career（"你在[职业名]积累了几年经验和人脉，终于有人找你合伙..."）
- **选项**：
  - A. 全力投入 → wealth归零, health-10, luck测试
  - B. 兼职参与 → wealth-5000
  - C. 拒绝 → 错过机会
- **flags_set**：entrepreneur(条件性，luck>70时)
- **chain**：→ CHAIN_002（创业成功）
- **cooldown**：5年

### EVT_Y005 - 相亲/恋爱
- **触发**：28岁，未婚时
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [26, 35]
  - stage: young
  - flags_blocked: [married]（已婚不触发）
  - not_triggered: [EVT_Y005]
  - cooldown: [romance_event, 5]
- **描述**：家里催婚了，七大姑八大姨轮番上阵...
- **叙事引用**：引用romance_count（"你[谈过0次/几次]恋爱，但都没走到最后..."）
- **选项**：
  - A. 去相亲 → charm+eq测试
  - B. 拒绝，等待真爱 → charm+3
  - C. 随便找个人结婚 → 标记married, 婚姻质量luck决定
- **flags_set**：married(条件性)
- **flags_block**：married=true后关闭所有恋爱/相亲事件
- **cooldown**：5年

### EVT_Y006 - 婚姻危机
- **触发**：32岁，已婚，随机（20%）
- **类型**：random
- **稀有度**：uncommon
- **前置条件**：
  - age: [30, 40]
  - stage: young 或 middle
  - flags_required: [married]
  - not_triggered: [EVT_Y006]
  - cooldown: [romance_event, 5]
- **描述**：你发现伴侣手机里有暧昧消息...
- **叙事引用**：引用EVT_Y005的结婚方式（"你们[相亲/自由恋爱/随便]走到一起已经[X]年了..."）
- **选项**：
  - A. 摊牌沟通 → eq测试
  - B. 默默忍受 → health-10
  - C. 果断离婚 → wealth-50%, 标记divorced
  - D. 你也出轨报复 → luck测试
- **flags_set**：divorced(条件性), married=false(条件性)
- **cooldown**：5年

### EVT_Y007 - 投资陷阱
- **触发**：30岁，wealth≥50000, luck<50时
- **类型**：attribute
- **稀有度**：uncommon
- **前置条件**：
  - age: [28, 40]
  - stats: {wealth: ≥50000, luck: <50}
  - not_triggered: [EVT_Y007]
  - flags_blocked: [scammed]（已被骗过则不再触发同类）
  - cooldown: [finance_event, 3]
- **描述**：朋友推荐一个"稳赚不赔"的投资项目，年化收益30%...
- **叙事引用**：引用career和wealth（"你在[职业]攒了一些钱，有人盯上了你的钱包..."）
- **选项**：
  - A. 全仓投入 → wealth-80%
  - B. 小额试水 → wealth-10000, iq+3
  - C. 坚决不投 → iq+5
- **flags_set**：scammed(条件性), investor=true
- **cooldown**：3年

### EVT_Y008 - 意外之财
- **触发**：随机，luck>70时（10%）
- **类型**：random
- **稀有度**：rare
- **前置条件**：
  - age: [20, 60]
  - stats: {luck: >70}
  - not_triggered: [EVT_Y008]
  - cooldown: [positive_event, 5]
  - flags_blocked: [bankrupt]（刚破产不会意外之财）
- **描述**：你买的彩票中奖了！/ 老家拆迁了！/ 投资暴涨了！
- **叙事引用**：引用最近的财务状况（"你最近[手头紧/还算宽裕]，没想到..."）
- **效果**：wealth+random(50000, 200000)
- **flags_set**：无
- **cooldown**：5年

### EVT_Y009 - 职业转型
- **触发**：30岁，随机（15%）
- **类型**：random
- **稀有度**：uncommon
- **前置条件**：
  - age: [28, 40]
  - flags_required: [career]（必须有当前职业）
  - not_triggered: [EVT_Y009]
  - flags_blocked: 无（但如果career_changed=true，转行次数已用完）
  - cooldown: [career_event, 5]
- **描述**：你对现在的工作越来越厌倦，想换个方向...
- **叙事引用**：引用career（"你在[职业名]干了[X]年了，每天重复同样的事情..."）
- **选项**：
  - A. 辞职转行 → wealth-1年收入, 重新选择（受职业准入矩阵约束）
  - B. 考研提升 → wealth-30000, iq+8
  - C. 继续忍耐 → health-5
- **flags_set**：career_changed=true（如果选A）
- **cooldown**：5年

### EVT_Y010 - 健康警告
- **触发**：33岁，health<45时
- **类型**：attribute
- **稀有度**：uncommon
- **前置条件**：
  - age: [30, 40]
  - stats: {health: <45}
  - not_triggered: [EVT_Y010]
  - cooldown: [health_event, 5]
- **描述**：体检报告亮了好几个红灯，医生严肃地建议你改变生活方式...
- **叙事引用**：引用career（"你在[职业]的工作强度让你的身体亮起了红灯..."）
- **选项**：
  - A. 彻底改变 → health+15, eq-3
  - B. 吃药控制 → health+5, wealth-5000/年
  - C. 无视警告 → health-10
- **flags_set**：chronic_disease(条件性，选C时)
- **cooldown**：5年

---

## 五、中年事件（36-60岁）

### EVT_M001 - 中年危机
- **触发**：40岁，必定
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [39, 42]
  - stage: middle
  - not_triggered: [EVT_M001]
- **描述**：你突然开始怀疑人生意义...凌晨3点睡不着，盯着天花板发呆
- **叙事引用**：引用career和married（"你在[职业]干了快20年，[已婚/单身]，突然觉得..."）
- **选项**：
  - A. 买跑车找刺激 → wealth-50000, charm+10, health-5
  - B. 换年轻伴侣 → married=false, divorced=true, charm+5（需已婚）
  - C. 辞职追梦 → wealth归零, career重置
  - D. 看心理医生 → health+10, iq+3
  - E. 接受现实 → eq+10, 标记mature
- **flags_set**：mature(条件性)
- **cooldown**：无（一次性）

### EVT_M002 - 子女教育焦虑
- **触发**：42岁，有子女时
- **类型**：chain
- **稀有度**：common
- **前置条件**：
  - age: [40, 45]
  - flags_required: [has_children]
  - not_triggered: [EVT_M002]
- **描述**：孩子成绩不好，你焦虑得整宿睡不着...
- **叙事引用**：引用education（"你自己的[学历]经历让你对孩子的教育格外重视..."）
- **选项**：
  - A. 疯狂报补习班 → wealth-30000
  - B. 顺其自然 → eq+5
  - C. 亲自辅导 → health-5
- **flags_set**：无
- **cooldown**：无

### EVT_M003 - 职业天花板
- **触发**：45岁，未达高管标记时
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [43, 48]
  - flags_required: [career]
  - flags_blocked: [career_peak]（已达巅峰不触发）
  - not_triggered: [EVT_M003]
  - cooldown: [career_event, 5]
- **描述**：你意识到这辈子可能升不上去了...新来的领导比你小10岁
- **叙事引用**：引用career和EVT_Y003（"你在[职业]干了[X]年，还记得那次[升职/错过升职]吗..."）
- **选项**：
  - A. 跳槽 → luck测试
  - B. 创业 → 参考EVT_Y004（需满足条件）
  - C. 接受现状 → health+5, eq+5
  - D. 搞副业 → wealth+10000/年, health-3
- **flags_set**：无
- **cooldown**：5年

### EVT_M004 - 健康警报（中年版）
- **触发**：48岁，health<60时
- **类型**：attribute
- **稀有度**：common
- **前置条件**：
  - age: [46, 52]
  - stats: {health: <60}
  - not_triggered: [EVT_M004, EVT_Y010]（青年版已触发则变体）
  - cooldown: [health_event, 5]
- **描述**：体检报告出来了，脂肪肝、高血压、高血糖...三高齐了
- **叙事引用**：引用EVT_Y010（如果触发过）（"你[之前/一直没]忽视的健康问题，现在爆发了..."）
- **选项**：
  - A. 彻底改变 → health+15, eq-3
  - B. 吃药控制 → health-10
  - C. 无视 → health-15, 标记serious_illness
- **flags_set**：serious_illness(条件性)
- **cooldown**：5年

### EVT_M005 - 父母养老
- **触发**：50岁，必定
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [49, 52]
  - not_triggered: [EVT_M005]
- **描述**：父母年纪大了，身体越来越差...你该怎么办？
- **叙事引用**：引用family_background（"你[富裕/普通/贫困]家庭出身的父母，现在需要你了..."）
- **选项**：
  - A. 接到家里照顾 → health-5, eq+5
  - B. 送养老院 → wealth-20000/年, eq-3
  - C. 请保姆 → wealth-30000/年
- **flags_set**：无
- **cooldown**：无

### EVT_M006 - 子女独立
- **触发**：55岁，有子女时
- **类型**：chain
- **稀有度**：common
- **前置条件**：
  - age: [53, 58]
  - flags_required: [has_children]
  - not_triggered: [EVT_M006]
- **描述**：孩子考上大学/工作了，要离开家了...空荡荡的房间让你有点失落
- **叙事引用**：引用EVT_M002（"还记得你为孩子的教育[焦虑/佛系]吗？现在TA终于独立了..."）
- **效果**：wealth+（子女给生活费）, health-3
- **flags_set**：无
- **cooldown**：无

### EVT_M007 - 投资暴富
- **触发**：随机，luck>80且wealth≥100000时（5%）
- **类型**：random
- **稀有度**：legendary
- **前置条件**：
  - age: [35, 60]
  - stats: {luck: >80, wealth: ≥100000}
  - flags_required: [investor]（必须有投资经历才能暴富）
  - not_triggered: [EVT_M007]
  - cooldown: [positive_event, 5]
- **描述**：你早年投资的股票暴涨了10倍！/ 买的房子翻倍了！
- **叙事引用**：引用EVT_Y007（"还记得那次[投资/被骗]吗？这次你赌对了！"）
- **效果**：wealth+random(500000, 2000000)
- **flags_set**：无
- **cooldown**：5年

### EVT_M008 - 事业巅峰
- **触发**：50岁，iq≥80且wealth≥500000时
- **类型**：attribute
- **稀有度**：rare
- **前置条件**：
  - age: [48, 55]
  - stats: {iq: ≥80, wealth: ≥500000}
  - flags_required: [career]
  - flags_blocked: [career_peak]
  - not_triggered: [EVT_M008]
- **描述**：你获得了行业最高荣誉！颁奖典礼上，你感慨万千...
- **叙事引用**：引用career和EVT_Y003（"从[第一份工作]到[升职/跳槽]，你在[职业]走了[X]年..."）
- **效果**：iq+5, charm+5, wealth+100000
- **flags_set**：career_peak = true
- **cooldown**：无

---

## 六、老年事件（61岁+）

### EVT_O001 - 退休生活
- **触发**：60岁（男）/55岁（女），必定
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [55, 62]（根据性别）
  - stage: old
  - flags_required: [career]
  - not_triggered: [EVT_O001]
- **描述**：你退休了...第一天不知道该干什么，坐在沙发上发呆了一整天
- **叙事引用**：引用career（"你在[职业]干了[X]年，突然闲下来..."）
- **选项**：
  - A. 带孙子 → health-3, eq+5（需has_children）
  - B. 发展兴趣爱好 → health+5, charm+3
  - C. 返聘做顾问 → wealth+20000/年, health-5
  - D. 环游世界 → wealth-50000, health+3, charm+5
- **flags_set**：无
- **cooldown**：无

### EVT_O002 - 老年疾病
- **触发**：65岁+, health<50时
- **类型**：attribute
- **稀有度**：common
- **前置条件**：
  - age: [63, 100]
  - stats: {health: <50}
  - not_triggered: [EVT_O002]
  - cooldown: [health_event, 5]
- **描述**：你被诊断出慢性病...每天要吃一大把药
- **叙事引用**：引用chronic_disease或serious_illness（"你[年轻时/中年]就[忽视过/注意过]健康问题..."）
- **选项**：
  - A. 积极治疗 → wealth-100000, health+10
  - B. 保守治疗 → wealth-30000, health+3
  - C. 放弃治疗 → health-20
- **flags_set**：chronic_disease = true
- **cooldown**：5年

### EVT_O003 - 老友离世
- **触发**：70岁+, 随机（20%）
- **类型**：random
- **稀有度**：common
- **前置条件**：
  - age: [68, 100]
  - not_triggered: [EVT_O003]
  - cooldown: [negative_event, 5]
- **描述**：你的老朋友走了...你参加了葬礼，看着遗照上的笑脸，泪流满面
- **叙事引用**：引用青少年时期的选择（"你想起[高中/大学]时的日子，那时候你们还年轻..."）
- **效果**：health-5
- **flags_set**：无
- **cooldown**：5年

### EVT_O004 - 遗产规划
- **触发**：75岁，必定
- **类型**：age
- **稀有度**：common
- **前置条件**：
  - age: [74, 77]
  - not_triggered: [EVT_O004]
- **描述**：你开始考虑身后事了...这辈子攒下的东西，该怎么安排？
- **叙事引用**：引用has_children和married（"你[有/没有]子女，[有/没有]伴侣，这辈子..."）
- **选项**：
  - A. 留给子女 → 需has_children
  - B. 捐给慈善 → 标记philanthropist
  - C. 挥霍享受 → health+5, charm+5
- **flags_set**：philanthropist(条件性)
- **cooldown**：无

### EVT_O005 - 天伦之乐
- **触发**：70岁+, 有子女且关系好时
- **类型**：chain
- **稀有度**：uncommon
- **前置条件**：
  - age: [68, 100]
  - flags_required: [has_children]
  - not_triggered: [EVT_O005]
  - cooldown: [positive_event, 5]
- **描述**：孙子/孙女跑过来抱住你叫"爷爷奶奶"...
- **叙事引用**：引用EVT_M002和EVT_M006（"从[教育焦虑]到[子女独立]，现在你终于享受天伦之乐..."）
- **效果**：health+5, eq+5
- **flags_set**：无
- **cooldown**：5年

### EVT_O006 - 孤独终老
- **触发**：70岁+, 未婚且无子女时
- **类型**：attribute
- **稀有度**：uncommon
- **前置条件**：
  - age: [68, 100]
  - flags_blocked: [married]（未婚）
  - flags_blocked: [has_children]（无子女）
  - not_triggered: [EVT_O006]
- **描述**：过年了，别人家热热闹闹，你一个人对着电视吃年夜饭...
- **叙事引用**：引用romance_count（"你这一生[谈过X次恋爱]，但最终..."）
- **选项**：
  - A. 去养老院 → wealth-20000/年, health+3
  - B. 领养宠物 → health+5, eq+3
  - C. 一个人也挺好 → health-5
- **flags_set**：无
- **cooldown**：无

### EVT_O007 - 百岁老人
- **触发**：100岁, health>30时
- **类型**：age
- **稀有度**：legendary
- **前置条件**：
  - age: [100, 100]
  - stats: {health: >30}
  - not_triggered: [EVT_O007]
- **描述**：你100岁了！电视台来采访你，问你长寿秘诀...
- **叙事引用**：引用整个人生轨迹（"从[家庭背景]到[职业]到[退休生活]，你活了整整一个世纪..."）
- **效果**：charm+10
- **flags_set**：无
- **cooldown**：无

---

## 七、特殊连锁事件

### CHAIN_001 - 电竞选手路线
- **触发链**：EVT_T002选A → gaming_addict标记 → 19岁时
- **前置条件**：
  - age: [18, 20]
  - flags_required: [gaming_addict]
  - flags_blocked: [prodigy]（天才不会去打电竞）
  - not_triggered: [CHAIN_001]
- **描述**：你的游戏段位打到了全国前100，有职业战队找你试训！
- **叙事引用**：引用EVT_T002（"还记得高中时你沉迷的那款游戏吗？现在它成了你的职业..."）
- **选项**：
  - A. 成为职业选手 → career=esports, wealth暴增但health持续-3/年
  - B. 放弃，回归正轨 → iq+5, health恢复
- **flags_set**：career = esports(条件性)
- **互斥**：与CHAIN_003（学术路线）互斥

### CHAIN_002 - 创业成功
- **触发链**：EVT_Y004全力投入 + luck>70 → 30岁时
- **前置条件**：
  - age: [28, 35]
  - flags_required: [career]（已创业）
  - stats: {luck: >70}
  - not_triggered: [CHAIN_002]
- **描述**：你的公司上市了！敲钟的那一刻，你热泪盈眶...
- **叙事引用**：引用EVT_Y004（"还记得当年你[全力投入/兼职参与]创业吗？现在..."）
- **效果**：wealth+1000000
- **flags_set**：entrepreneur = true
- **互斥**：与稳定职业路线互斥

### CHAIN_003 - 学术大牛
- **触发链**：prodigy + edu_top + iq持续>85 → 40岁时
- **前置条件**：
  - age: [38, 45]
  - flags_required: [prodigy, edu_top]
  - stats: {iq: >85}
  - flags_required: [education_track=science]
  - not_triggered: [CHAIN_003]
- **描述**：你的论文登上了Nature/Science封面！学术界震动...
- **叙事引用**：引用EVT_C004+EVT_T003+EVT_T006（"从10岁的天赋发现，到16岁的集训队，到高考[顶尖大学]..."）
- **效果**：iq+10, wealth+200000
- **flags_set**：scholar = true
- **互斥**：与CHAIN_001（电竞路线）互斥

### CHAIN_004 - 情场浪子
- **触发链**：romance_count≥5 → 35岁时
- **前置条件**：
  - age: [33, 38]
  - romance_count: ≥5
  - flags_blocked: [married]（已婚不触发）
  - not_triggered: [CHAIN_004]
- **描述**：你突然发现，身边没有一个真心人...都是过客
- **叙事引用**：引用所有恋爱事件（"从[初恋]到[相亲]到[第N次恋爱]，你终于..."）
- **效果**：eq-10, health-5
- **flags_set**：playboy = true
- **互斥**：与稳定婚姻路线互斥

### CHAIN_005 - 逆袭人生
- **触发链**：edu_failed + 30岁时wealth≥100000
- **前置条件**：
  - age: [28, 35]
  - flags_required: [edu_failed]
  - stats: {wealth: ≥100000}
  - not_triggered: [CHAIN_005]
- **描述**：你从一个高考落榜生，逆袭成为了百万富翁！
- **叙事引用**：引用EVT_T006（"还记得高考落榜时的绝望吗？现在..."）
- **效果**：iq+5, charm+10
- **flags_set**：underdog = true

### CHAIN_006 - 网红暴富
- **触发链**：career方向为艺术 + charm>85 + luck>70
- **前置条件**：
  - age: [25, 32]
  - stats: {charm: >85, luck: >70}
  - flags_required: [education_track=art] 或 [hobby=dance]
  - not_triggered: [CHAIN_006]
- **描述**：你的一个视频爆了！一夜之间涨粉百万...
- **叙事引用**：引用hobby和education_track（"从[舞蹈班]到[艺术班]，你的积累终于爆发了..."）
- **效果**：wealth+500000, charm+5
- **flags_set**：无

---

## 八、意外事件（全阶段可触发）

### EVT_X001 - 中彩票
- **触发**：全阶段, luck>85时（3%）
- **稀有度**：legendary
- **前置条件**：
  - stats: {luck: >85}
  - not_triggered: [EVT_X001]
  - cooldown: [positive_event, 5]
- **效果**：wealth+random(100000, 1000000)
- **cooldown**：5年

### EVT_X002 - 车祸
- **触发**：全阶段, luck<20时（5%）
- **稀有度**：rare
- **前置条件**：
  - age: [16, 80]（太老或太小不触发）
  - stats: {luck: <20}
  - not_triggered: [EVT_X002]
  - cooldown: [negative_event, 5]
- **选项**：
  - A. 轻伤 → health-15, wealth-5000
  - B. 重伤 → health-30, wealth-50000
  - C. luck测试：luck>10则轻伤, luck≤10则死亡
- **flags_set**：serious_illness(条件性)
- **cooldown**：5年

### EVT_X003 - 天降横财
- **触发**：全阶段, luck>75时（5%）
- **稀有度**：rare
- **前置条件**：
  - stats: {luck: >75}
  - not_triggered: [EVT_X003]
  - cooldown: [positive_event, 5]
- **效果**：wealth+random(20000, 100000)
- **cooldown**：5年

### EVT_X004 - 被骗
- **触发**：全阶段, luck<40且wealth>10000时（10%）
- **稀有度**：uncommon
- **前置条件**：
  - stats: {luck: <40, wealth: >10000}
  - not_triggered: [EVT_X004]
  - flags_blocked: [scammed]（已被骗过不再触发）
  - cooldown: [negative_event, 3]
- **效果**：wealth-random(5000, wealth×30%)
- **flags_set**：scammed = true
- **cooldown**：3年
