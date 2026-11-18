# Engine skeleton（通用时间轴引擎）

按当前领域填充 `config` 与步骤工厂，勿改引擎语义（renderTo 的确定性重建、落效才写入、聚光灯与故障注入的时序）。命名保持通用：站点（station）= 流程经过的地方；单元（unit）= 流动的离散个体；落效（land）= 单元到达后对状态的改变。

```js
(function(){
'use strict';
var $ = function(id){ return document.getElementById(id) };

// ---- config：按领域填充 ----
var STATIONS = ['a','b','c'];        // 因果方向从左到右；键自定义（端点/角色/阶段/状态）
var STATE_HOST = 'c';                 // 状态落点所在的泳道（契约 3）
// feeds / lanes / heads / flightsEl / linesEl / barEl / readout / breakPanel 由 HTML id 约定取得

var activeAnims = [];
var breakMode = false, focusMode = true;

function chip(station, cls, txt){ /* 泳道内追加一条徽章并滚动到底 */ }
function addLog(kind, text){ /* 步骤日志追加一行；落效步骤才调用；kind 含独立机制类别 */ }
function fly(from, to, label, cls, dur){
  /* 生成一个飞行包元素，WAAPI 从 from 站点中心移动到 to 站点中心；
     注册进 activeAnims（暂停时 pause），结束自移除。 */
}
function flash(node){ /* 落效时的高亮闪 */ }

// ---- unit 工厂（契约 2）----
// 站点自产的结构性单元：两段（发出 → 落效）
function hop(o){ return [
  { id: o.id + '-a', ph: o.ph, dur: o.durA || 850, lab: o.labA, log: null,
    focus: o.from, state: function(){ chip(o.from, o.cls, o.fromChip); },
    live: function(){ fly(o.from, o.to, o.flight, o.fcls || 'ev', 700); } },
  { id: o.id + '-b', ph: o.ph, dur: o.durB || 800, lab: o.labB,
    log: o.log, logKind: o.logKind,
    focus: o.to, mem: true,                                   // 落效步：状态面板泛光
    state: function(){ chip(o.to, o.cls, o.toChip); o.write(); },
    live: null },
];}

// 由上游逐载荷生成的单元：三段（产生 → 移动 → 落效）
function unit(o){ return [
  { id: o.id + '-a', ph: o.ph, dur: 800, lab: '产生 ' + o.unit, log: null,
    focus: o.gen, state: function(){ chip(o.gen, 'tok', o.genChip); },
    live: function(){ fly(o.gen, o.carry, o.flight0, 'token', 620); } },
  { id: o.id + '-b', ph: o.ph, dur: 850, lab: '转交 ' + o.unit, log: null,
    focus: o.carry, state: function(){ chip(o.carry, o.cls, o.carryChip); },
    live: function(){ fly(o.carry, o.to, o.flight, o.fcls || 'ev', 680); } },
  { id: o.id + '-c', ph: o.ph, dur: 800, lab: '落效（' + o.land + '）',
    log: o.log, logKind: o.logKind,
    focus: o.to, mem: true,
    state: function(){ chip(o.to, o.cls, o.toChip); o.write(); },
    live: null },
];}

// ---- 步骤总表：组合器拼装，禁止手抄长数组 ----
var steps = [].concat(
  /* setupSteps(), ...hop(...), ...unit(...), teardownSteps() */
);
var BREAK_AT = (function(){
  for (var i = 0; i < steps.length; i++) if (steps[i].id === BREAK_ID) return i;
  return -1;
})();

// ---- 引擎（勿改语义）----
var idx = -1, playing = false, timer = null, speed = 1;

function clearDynamic(){ /* 清空各泳道 feed、日志、瞬态层、状态面板，重置步骤引用变量 */ }
function applyStep(i){ var s = steps[i]; if (s.state) s.state(); if (s.log) addLog(s.logKind || 'sys', s.log); }
function renderTo(i){                       // 契约 10：确定性重建
  clearDynamic();
  idx = Math.max(-1, Math.min(i, steps.length - 1));
  for (var k = 0; k <= idx; k++) applyStep(k);
  setFocus(idx >= 0 ? steps[idx] : null);
  updateUI();
}
function next(){                            // 契约 2：state 代表"落效"语义
  if (!playing) return;
  if (idx >= steps.length - 1) { pause(); updateUI(); return; }
  idx++;
  var s = steps[idx];
  if (s.state) s.state();
  if (s.log) addLog(s.logKind || 'sys', s.log);
  if (s.live) try { s.live(); } catch (e) {}
  setFocus(s);
  updateUI();
  timer = setTimeout(next, s.dur / speed);
}
function pause(){ playing = false; clearTimeout(timer);
  for (var k = 0; k < activeAnims.length; k++) try { activeAnims[k].pause(); } catch (e) {}
  updateUI(); }
function play(){ if (playing) return;
  if (idx >= steps.length - 1) renderTo(-1);
  playing = true;
  for (var k = 0; k < activeAnims.length; k++) try { activeAnims[k].play(); } catch (e) {}
  next(); }

function setFocus(s){                       // 契约 5：聚光灯
  STATIONS.forEach(function(k){ lanes[k].classList.remove('hot'); lanes[k].classList.remove('dim'); });
  if (!focusMode || !s || !s.focus) { stage.classList.remove('spot'); return; }
  stage.classList.add('spot');
  lanes[s.focus].classList.add('hot');
  STATIONS.forEach(function(k){ if (k !== s.focus) lanes[k].classList.add('dim'); });
  if (s.mem) memPanelGlow(900);             // 落效步：状态面板短促泛光
}

function breakDemo(){                       // 契约 9：故障注入
  pause(); breakMode = true; renderTo(BREAK_AT);
  breakPanel.classList.add('on');
  addLog('brk', '⚡ 流程中断 —— 以下单元未到达 ↓');
  for (var k = BREAK_AT + 1; k < steps.length; k++) {
    if (steps[k].log) { var d = addLog(steps[k].logKind || 'sys', steps[k].log);
      d.classList.add('lost'); d.classList.remove('cur'); }
  }
}
function exitBreak(){ breakMode = false; breakPanel.classList.remove('on'); renderTo(idx); }

// ---- UI 绑定：播放/暂停/单步/进度条（每步一段，title=步骤标签）/倍速/键盘/聚焦开关 ----
// ---- 冒烟测试钩子：window.__seek(i)、window.__idxOf(id)、window.__info() ----

renderTo(-1);
})();
```

## 验证脚本要点（第四步用）

- 断言时间点至少含：空态、首个结构单元落效前/后、某个在途单元的 b 步（**此时状态必须是旧值**）、故障断点、终态。
- 布局恒定断言：多个 seek 之间，播放按钮的页面坐标必须相等。
- 真实浏览器执行（可用系统自带内核），逐张目检截图。
