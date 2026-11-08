# PROJECT · CypherJam

## 一、这个项目是什么

**CypherJam**：一个让「一个人」也能在派对上撑起整场的浏览器 DJ 工具。第一版只做单机、不做多人、不做账号、不做联机；使用者戴有线耳机、在桌面浏览器里用键盘表演。

它不是一个「按下去有声音」的节奏玩具，目标是**一个人不冷场地撑二十分钟**：四轨十六步网格、四组 pattern、实时表演控制、build / slam 过渡、可被记住的钩子、可导出的 WAV，以及把现实声音（地铁报站、开罐、脚步、市场环境声）导入进节拍的能力。

名字由 **Cypher**（街舞圈围圈轮替）+ **Jam**（无谱合奏）组合而成——原本指向多人同频轮替；当前版本先做单人，机制留给以后的联机形态。

- `app/index.html`：唯一入口，打开即用；`app/vendor/Tone.js` 为本地化的 Tone.js 15.1.22（MIT）。
- `docs/01-参考资料与技术调研.md`：Strudel vs Tone.js、多人同步、竞品、iOS 风险、域名查询。
- `docs/02-v1-实现说明.md`：v1 的架构、侧链做法、低频处理、导入流程、录音导出、实测与未测项、素材授权清单。
- `docs/03-v2-教程与风格包.md`：v2 的可发现性诊断与修复、新手教程设计、「带 A 的方块」结论、三套风格包与每组 pattern 的鼓组/和声设计、swing 与锁调做法、实测记录。
- `docs/04-v3-风格扩充契约与预研.md`：v3 的十组 pattern 分组、风格扩充契约（必填/可选/锁调性校验）、swing 导出修复的成因与实测、Hip Hop / Trap 预研结论与障碍、v4 的 glide/tie/roll/half-time 引擎、TRANSITION 转场与逐 step 导出实测、网格左侧缺口修复记录。
- `ref/`：外部参考资料，目前为空。

## 二、现在到什么程度

**v4 进行中：引擎扩展、TRANSITION 转场、Trap 风格包与 Humanize 已通过实测（2026-11-07）**，代码在 `app/index.html`。

已完成：

- 四轨网格（drums / bass / stab / atmos）+ 额外 SAMPLE 轨，16 步，反拍 hi-hat。
- **每风格 10 组 pattern（A–J），键盘 `1–9` / `0` 即时切换**：`1–4` 为 CORE 主段落，`5–0` 为 TOOLS 工具与变体；UI 用绿/蓝两排 + 组标题区分，运行中切换落在小节线，播放不断。
- 默认 pattern 故意留白：A 只有 kick + 反拍 hat，其余轨道由用户自己加。
- PERFORM：全局 filter、reverb send、delay send、sidechain、每轨 M/S，全部实时可改；`R` 一键复位。
- 过渡：BUILD / RISER（2 小节）、SLAM（静默一个八分音符后重击）。
- **TRANSITION 风格转场（`T`）**：预设 **AUTO ON**，播放中点风格键立即开始转场（默认 **4 小节**，127 BPM 约 7.5 秒）：降编 → 逐 step 推速 → 小节线换风格 + 落点重击 → 回满编；品牌栏即时显示 `→ 目标风格 · 剩余小节`，转场进度条持续前进，目标风格键全程琥珀高亮。AUTO OFF 时保留原两段式（staged + `T` 确认）。转场中 `T`／`Esc` 可在换风格前取消，按其它风格键替换目标、不叠加；非播放时仍硬切。AUTO 开关与 2 / 4 / 8 小節长度写入 `localStorage`。
- **转场可录可导出**：日志逐步记录 `stepSecs / styleIds / swings / swingSubs` 与 `transitions[]`；离线渲染按风格分段、按每步绝对时间重放，现场 tick 间隔与日志误差 ≤ 0.1 ms。
- 钩子：FMSynth 合成的短促打击型音色（非商业人声采样），可用导入采样顶替。
- 锁 A 小调：7 个 live pads + 导入采样的自动测音/移调。
- 音频导入：WAV / MP3 拖入或选择，自动裁起音、归一化、测基频、尽力移调到 A 小调；测不到会明确标「未自動調音」。
- 录音导出：记录现场日志后用 `Tone.Offline` 离线渲染 16-bit WAV，失败退回 MediaRecorder。
- **新手教程：顶栏 TUTORIAL 入口；首次只教 4 步（声音 / 音符 / CORE-TOOLS pattern / 切換風格），其余 15 步从菜单自选；每步检测真实操作、可跳過、繁体中文、不吃键盘。**
- **可发现性修复：空格显示 `+`、每格有 hover 提示、bass 显示完整音高 `A2`、新增 NOTE 音高选择、轨道副标题。**
- **五套风格包：TECH HOUSE（127，10 组）/ DEEP HOUSE（122，10 组）/ JAZZ HOUSE（124 + 16 分 swing 0.35，10 组，七/九和弦全在 A 小调内）/ TRAP · MEMPHIS（132，half-time，品牌栏显示 `FEEL 66`，10 组：失真 808、FM 牛铃、lo-fi 底噪、稀疏切分 hat；`bpmAlt:140` 可切）/ DRILL（142，half-time，`FEEL 71`，10 组：滑音 808（`glideMax 0.09 s`）、snare 固定第三拍、切分 kick、冷色 Am/Em/Dm loop、hat 只点缀）。**
- **风格包数据化**：一种风格 = BPM / swing / 鼓组与合成参数 / pattern / 和声规则 / 显示名称；注册时跑契约校验，调外和弦或改调性会被直接拒绝，新增曲风不用改 UI 与键盘代码。
- **Jazz swing 已写进导出 WAV**：日志保存 swing 与 styleId，离线重放复现 Tone 的 swing 偏移；转场期间逐步的 swing 也被记录。
- **Humanize（人性化时序）**：每颗音符带力度微变与前后微偏移，三档 `OFF / 緊(±3ms) / 自然(±8ms) / 鬆(±15ms)`；偏移由绝对步序号的哈希决定（不用随机数），现场与导出走同一算法；日志逐步记录 `hums[]`，WAV 离线渲染逐 step 重现；pad/air 只改力度、SAMPLE 轨不动；风格包可选默认档位（当前五风格全 `off`）。
- 录音导出可选 WAV 48 kHz / 44.1 kHz / 跟随浏览器；30 分钟立体声 16-bit 约 345 MB（48 kHz）／317 MB（44.1 kHz），已在 REC 面板与 `docs/04 §6.4` 写明。
- **多軌音軌（2026-11-07）**：4 Loop Clip（進格子、跟 transport、自動測長取最近小節、±3% 速率對拍、IN/OUT、淡入淡出、音量、lowpass、16 格起始步）+ 4 Ambient Bed（不佔格子、不跟 BPM、獨立 play/stop、淡入淡出、音量、mute）；合計上限 8。頁面內 MediaRecorder 錄音（單次 60 s、只存記憶體、重新載入消失；四條 1 分鐘立體聲約 92 MB）。播放層接 `masterGain`、不進 mixBus，會被 SLAM/drop 切斷但不受風格 lo-fi 影響。新 mute 鍵 `M` + `1–8`，不動既有鍵位。日誌記 `trackSteps[]` + `trackSnap`，導出重現 IN/OUT、淡化、循環與起始步。
- 键盘表演：Space / 1–9 / 0 / **T** / B / S / Z X C V N / **M + 1–8** / R，`Esc` 取消待切或转场；AUTO ON 时 `T` 兼作转场中途取消。
- 声音引擎：Tone.js 15.1.22 本地加载；Tech House BPM 127（参考曲公开资料核实值）。

已在无头 Chromium 实测：加载无报错、播放步进、小节线切 pattern、mute、build/slam、离线渲染、导入裁切与调音、端到端录音下载、复位、键盘焦点不误触；v2 另测教程三步检测/跳過/菜单、教程打开时键盘仍有效、三风格切换与 BPM/swing、jazz 和弦锁调校验、三风格离线渲染；v3 另测三风格各 10 组、`9/0` 键位、pattern 编辑跨风格不丢、实时 swing 时间点与导出 onset 对齐、三风格离线渲染；v4 另测三 House 回归（时长/包络 lag 0）、glide/tie/roll/half-time、TRANSITION 全阶段、转场录制与离线重放、现场 tick 间隔与日志误差 ≤ 0.1 ms、网格点亮格左缘修复（3 风格 × 3 视口 × DPR 1/2）、Trap 10 组与离线渲染（`chainBass` / `rollSteps` 均走到）、`1–0` 键位、三 House 回归、390/360 移动端风格行不挤、Humanize 三档偏移范围与同参数可复现（64/64）、纯 kick 导出起音位移 ≤ 1 采样误差、录制日志 `hums` 与步数一致、非法档位被契约拒绝、无新增键位、移动端 HUMANIZE 分段不溢出；Memphis 重做另测 132 BPM 步长/小节/kick/hat 间隔（`0.113636 / 1.818182 / 0.909091 / 0.028409 s`）、808 失真谐波 `h3/h1≈0.199`、牛铃 880 Hz 主导与 2300 Hz 泛音、lo-fi 底噪 `-47.8 dBFS` 与频宽收窄、六组 pattern 峰值 `≤0.9072` 无削波、`file://` 无报错；转场另测 127→132 ramp 逐级推速、132→122 drop 静空 `0.45454 s` 且导出可重放；Drill 另测 142 BPM 步长/小节（`0.105634 / 1.690141 s`）、`glideMax 0.09 s` 的滑音时线与目标音、808 谐波 `h3/h1≈0.097` 与约 0.8 s 收敛、十组 pattern 峰值 `≤0.8383` 无削波、132→142 drop 静空 `0.45454 s`、五风格回归与 `file://` 无报错；多轨另测 4 Loop + 4 Bed 载入/静音/录音、IN/OUT 与起始步进导出（IN 50% 使 rms `0.169→0.079`、startStep 8 使 `0.193→0.144`）、五风格回归不变、390/360 `scrollWidth` 等于视口、本机 HTTPS 录音 2.58 s 并可导出、`file://` 无报错；AUTO TRANSITION 另测默认 ON/4 小節、点风格即时转场（品牌 `→ DEEP HOUSE · 4 BARS`、进度条前进、目标键高亮）、`T` 中途取消、OFF 两段式与 `T` 确认、偏好重载持久化、390/360 点风格可转场且 `scrollWidth` 等于视口、教程第 4 步依模式显示文案、转场录制日志含 `style`/`impact` 且离线重放成功（92 steps / 13.08 s）。**听感、真机、20 分钟长跑未验证。**

未做：多人/帳號/雲同步、持久化、真機與 iOS 適配；Hip Hop（Metro 式）按決策暫緩。已知限制：純合成鼓做不出 boom bap 的採樣質感（材質問題，見 `docs/04`）；clap/air 為噪聲合成，離線渲染的噪聲相位每次不同，音訊無法逐樣本完全一致（與 Humanize 無關）；多軌只存記憶體，重新載入消失；8 條同時響時浮點峰值可到 1.43，WAV 匯出靠歸一化到 0.92 避免削波。

## 三、怎么启动 / 打开

```bash
open ~/Projects/cypherjam/app/index.html
```

- 用桌面 Chrome，推荐有线耳机。
- 页面打开后点 **ENTER · START**（浏览器要求一次点击才允许出声）。
- `Space` 播放/停止，`1`–`9` / `0` 切 pattern，`T` 触发风格转场（AUTO ON 时点风格键即可、`T` 取消），`Esc` 取消待切/转场，`B` build，`S` slam，`Z/X/C/V/N` 分轨 mute，`R` 复位。
- 顶栏 **TUTORIAL · 教學** 随时重开课程菜单（第一次进入只自动教 4 步）。
- 顶栏 `TECH / DEEP / JAZZ HOUSE / TRAP · MEMPHIS / DRILL` 切风格包（AUTO ON 时一点即转场，BPM 与 pattern 数随包变化）。
- 右侧 PERFORM 的 **HUMANIZE** 点 `OFF / 緊 / 自然 / 鬆` 切人性化程度，无键盘快捷键。
- Tone.js 已放在 `app/vendor/Tone.js`，断网也能用。

## 四、下一步做什么

1. **真机试听与 20 分钟长跑**：音色平衡、侧链呼吸感、build/slam 的冲击、连续演奏是否漂移或爆音；判断 10 组 tech house pattern 的对比是否够、deep house pad 是否过厚、jazz 的 swing 0.35 是否到位、Trap 的 half-time 与 808 是否成立、Humanize 三档是否真的可感知。这一步只能由使用者在耳机上判断。
2. **分享链接（已完成 2026-11-07）**：`https://life-os-d3gh449gn80d019b7-1253754162.tcloudbaseapp.com/cypherjam/`。首次访问会先看到「风险提醒／页面访问提示」，按钮「确定访问」有 1–30 s 倒计时，点过才进入；只上传了 `cypherjam/index.html` 与 `cypherjam/vendor/Tone.js`，LifeOS 根目录 `index.html` 哈希未变。
3. **真机试听多轨**：8 条同时响的实际听感与音量平衡；目前只测过合成信号与本机/真实 HTTPS 的假麦克风，真手机麦克风未验。
4. **Hip Hop 风格包 + 采样鼓**：85–95 BPM、swing、稀疏留白与 ghost note；采样必须记录名称/来源/授权/日期，只用 CC0 或明确商用，CC-BY-NC 禁用；找不到足够 CC0 时做小规模可替换方案并如实说明，不得用合成鼓假装采样鼓。
5. **持久化**：pattern、参数、采样存到本地，刷新不丢；再多几个风格预设。
6. **导出 Strudel 代码文本**（日后）：仅作为目标格式，第一版不做。
7. **手机纯触控版**：等桌面版验证过再谈。
8. **Jazz / Swing 写歌产品**：**暂停**，等派对实测后按 `docs/05` 的触发信号再决定，不要动。

