# Web Audio API 音效合成技能

## 概述
使用 Web Audio API 在浏览器中合成音效，无需外部音频文件。适用于游戏音效、UI 反馈音等场景。

## 核心概念

### AudioContext
音频操作的上下文环境，是所有音频节点的容器。
```javascript
const audioContext = new (window.AudioContext || window.webkitAudioContext)();
```

### 关键节点
- **OscillatorNode**: 声音发生器，产生不同波形的声音
- **GainNode**: 音量控制器，控制声音大小和淡入淡出
- **连接链**: oscillator → gain → destination(扬声器)

## 常见音效实现

### 1. 短促点击音（击中效果）
```javascript
function playHitSound() {
    const osc = audioContext.createOscillator();
    const gain = audioContext.createGain();
    
    osc.connect(gain);
    gain.connect(audioContext.destination);
    
    const now = audioContext.currentTime;
    
    osc.type = 'sine';
    osc.frequency.setValueAtTime(800, now);
    osc.frequency.exponentialRampToValueAtTime(400, now + 0.1);
    
    gain.gain.setValueAtTime(0.3, now);
    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.1);
    
    osc.start(now);
    osc.stop(now + 0.1);
}
```

### 2. 上升琶音（开始/成功）
```javascript
function playStartSound() {
    const osc = audioContext.createOscillator();
    const gain = audioContext.createGain();
    
    osc.connect(gain);
    gain.connect(audioContext.destination);
    
    const now = audioContext.currentTime;
    
    osc.type = 'sine';
    osc.frequency.setValueAtTime(400, now);
    osc.frequency.linearRampToValueAtTime(800, now + 0.15);
    
    gain.gain.setValueAtTime(0.2, now);
    gain.gain.linearRampToValueAtTime(0, now + 0.3);
    
    osc.start(now);
    osc.stop(now + 0.3);
}
```

### 3. 下降音调（结束/失败）
```javascript
function playEndSound() {
    const osc = audioContext.createOscillator();
    const gain = audioContext.createGain();
    
    osc.connect(gain);
    gain.connect(audioContext.destination);
    
    const now = audioContext.currentTime;
    
    osc.type = 'sine';
    osc.frequency.setValueAtTime(600, now);
    osc.frequency.exponentialRampToValueAtTime(200, now + 0.5);
    
    gain.gain.setValueAtTime(0.3, now);
    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.5);
    
    osc.start(now);
    osc.stop(now + 0.5);
}
```

### 4. 和弦音效（庆祝/成就）
```javascript
function playChord(frequencies, duration) {
    const now = audioContext.currentTime;
    const masterGain = audioContext.createGain();
    masterGain.connect(audioContext.destination);
    masterGain.gain.setValueAtTime(0.2, now);
    masterGain.gain.exponentialRampToValueAtTime(0.01, now + duration);
    
    frequencies.forEach(freq => {
        const osc = audioContext.createOscillator();
        osc.type = 'sine';
        osc.frequency.setValueAtTime(freq, now);
        osc.connect(masterGain);
        osc.start(now);
        osc.stop(now + duration);
    });
}

// C 大调和弦：C4(523.25Hz), E4(659.25Hz), G4(783.99Hz)
playChord([523.25, 659.25, 783.99], 0.4);
```

## 波形类型

| 类型 | 音色特点 | 适用场景 |
|------|---------|---------|
| `sine` | 纯净柔和 | 提示音、简单音效 |
| `square` | 尖锐复古 | 8-bit 游戏、警报 |
| `sawtooth` | 明亮刺耳 | 电子音乐、特殊效果 |
| `triangle` | 温暖圆润 | 乐器模拟、柔和提示 |

## 频率参考（常见音符）

| 音符 | 频率 (Hz) |
|------|----------|
| C4 | 523.25 |
| D4 | 587.33 |
| E4 | 659.25 |
| F4 | 698.46 |
| G4 | 783.99 |
| A4 | 880.00 |
| B4 | 987.77 |
| C5 | 1046.50 |

## 浏览器兼容性注意事项

### 自动播放策略
现代浏览器要求音频必须在用户交互后初始化：
```javascript
// 在用户点击/触摸时初始化
function initAudio() {
    if (!audioContext) {
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
    }
    if (audioContext.state === 'suspended') {
        audioContext.resume();
    }
}

// 在游戏开始按钮点击时调用
startBtn.addEventListener('click', () => {
    initAudio();
    startGame();
});
```

### 跨浏览器前缀
```javascript
const AudioContextClass = window.AudioContext || window.webkitAudioContext;
```

## 性能优化

### 1. 复用 AudioContext
不要为每个音效创建新的 AudioContext，全局复用：
```javascript
let audioContext = null;

function getAudioContext() {
    if (!audioContext) {
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
    }
    return audioContext;
}
```

### 2. 对象池模式（高频音效）
对于频繁播放的音效，可以复用 oscillator 和 gain 节点。

### 3. 音量控制
提供音效开关和音量调节：
```javascript
const GAME_CONFIG = {
    AUDIO_ENABLED: true,
    MASTER_VOLUME: 0.3
};

function playSound(type) {
    if (!GAME_CONFIG.AUDIO_ENABLED) return;
    // ...
}
```

## 完整示例结构

```javascript
// 1. 全局音频上下文
let audioContext = null;

// 2. 初始化函数
function initAudio() {
    if (!audioContext) {
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
    }
    if (audioContext.state === 'suspended') {
        audioContext.resume();
    }
}

// 3. 音效播放函数
function playSound(type) {
    if (!audioContext) return;
    
    const osc = audioContext.createOscillator();
    const gain = audioContext.createGain();
    osc.connect(gain);
    gain.connect(audioContext.destination);
    
    const now = audioContext.currentTime;
    
    switch (type) {
        case 'hit': /* ... */ break;
        case 'start': /* ... */ break;
        case 'end': /* ... */ break;
    }
}

// 4. 在用户交互时初始化
button.addEventListener('click', () => {
    initAudio();
    playSound('start');
});
```

## 应用场景

- 🎮 网页游戏音效（击中、得分、升级、失败）
- 🔔 UI 交互反馈（按钮点击、通知提示）
- ⏰ 定时器/倒计时提醒
- 🎵 简单的音乐播放

## 优势

✅ 无需外部音频文件，加载快  
✅ 完全可编程，灵活控制音高、时长、音色  
✅ 零带宽消耗  
✅ 跨浏览器支持良好  

## 局限性

❌ 复杂音乐需要大量代码  
❌ 不如采样音频真实  
❌ 需要处理浏览器自动播放限制  

---

**参考资源:**
- [Web Audio API MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [Web Audio API 入门](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Audio_API/Using_Web_Audio_API)
