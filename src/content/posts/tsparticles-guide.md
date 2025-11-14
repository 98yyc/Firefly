---
title: tsParticles - 打造炫酷的网页粒子动画效果
published: 2024-01-17
description: 详细介绍 tsParticles 粒子动画库的使用方法，从基础配置到高级特效，帮助你轻松创建各种炫酷的粒子动画效果。
tags: [tsParticles, JavaScript, 动画, 前端, 特效]
category: 前端开发
draft: false
---

# tsParticles - 打造炫酷的网页粒子动画效果

tsParticles 是一个轻量级的 TypeScript 库，用于创建高度可定制的粒子动画。它是 particles.js 的升级版本，提供了更多功能和更好的性能。

::github{repo="tsparticles/tsparticles"}

## 一、项目简介

### 1.1 主要特性

- 🎨 **高度可定制** - 支持数百种配置选项
- 🚀 **性能优异** - 使用 Canvas 和 WebGL 渲染
- 📦 **轻量级** - 核心库仅 2KB（gzip）
- 🎭 **丰富的预设** - 内置多种动画效果
- 🔌 **插件系统** - 支持自定义扩展
- 📱 **响应式** - 自动适配不同屏幕尺寸
- 🎯 **交互性** - 支持鼠标悬停、点击等交互
- 🌈 **多框架支持** - React、Vue、Angular、Svelte 等

### 1.2 与 particles.js 的区别

| 特性 | particles.js | tsParticles |
|------|--------------|-------------|
| 开发语言 | JavaScript | TypeScript |
| 包大小 | ~10KB | ~2KB (核心) |
| 活跃维护 | ❌ 已停止 | ✅ 持续更新 |
| 插件系统 | ❌ | ✅ |
| 预设效果 | 有限 | 丰富 |
| 性能 | 一般 | 优秀 |

## 二、快速开始

### 2.1 安装

#### 使用 npm

```bash
npm install @tsparticles/engine @tsparticles/slim
```

#### 使用 CDN

```html
<!-- 完整版 -->
<script src="https://cdn.jsdelivr.net/npm/@tsparticles/engine@3/tsparticles.engine.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@tsparticles/slim@3/tsparticles.slim.min.js"></script>

<!-- 或使用 unpkg -->
<script src="https://unpkg.com/@tsparticles/engine@3"></script>
<script src="https://unpkg.com/@tsparticles/slim@3"></script>
```

### 2.2 基础使用

#### HTML 结构

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>tsParticles Demo</title>
    <style>
        #tsparticles {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: -1;
        }
    </style>
</head>
<body>
    <div id="tsparticles"></div>
    
    <script src="https://cdn.jsdelivr.net/npm/@tsparticles/engine@3"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tsparticles/slim@3"></script>
    <script src="app.js"></script>
</body>
</html>
```

#### JavaScript 配置

```javascript
// app.js
(async () => {
    await loadSlim(tsParticles);

    await tsParticles.load({
        id: "tsparticles",
        options: {
            background: {
                color: {
                    value: "#0d47a1",
                },
            },
            fpsLimit: 120,
            particles: {
                color: {
                    value: "#ffffff",
                },
                links: {
                    color: "#ffffff",
                    distance: 150,
                    enable: true,
                    opacity: 0.5,
                    width: 1,
                },
                move: {
                    direction: "none",
                    enable: true,
                    outModes: {
                        default: "bounce",
                    },
                    random: false,
                    speed: 2,
                    straight: false,
                },
                number: {
                    density: {
                        enable: true,
                    },
                    value: 80,
                },
                opacity: {
                    value: 0.5,
                },
                shape: {
                    type: "circle",
                },
                size: {
                    value: { min: 1, max: 5 },
                },
            },
            detectRetina: true,
        },
    });
})();
```

## 三、框架集成

### 3.1 React 集成

#### 安装

```bash
npm install @tsparticles/react @tsparticles/slim
```

#### 使用

```jsx
import { useCallback } from "react";
import Particles from "@tsparticles/react";
import { loadSlim } from "@tsparticles/slim";

function App() {
    const particlesInit = useCallback(async engine => {
        await loadSlim(engine);
    }, []);

    const particlesLoaded = useCallback(async container => {
        console.log(container);
    }, []);

    return (
        <Particles
            id="tsparticles"
            init={particlesInit}
            loaded={particlesLoaded}
            options={{
                background: {
                    color: {
                        value: "#0d47a1",
                    },
                },
                fpsLimit: 120,
                particles: {
                    color: {
                        value: "#ffffff",
                    },
                    links: {
                        color: "#ffffff",
                        distance: 150,
                        enable: true,
                        opacity: 0.5,
                        width: 1,
                    },
                    move: {
                        enable: true,
                        speed: 2,
                    },
                    number: {
                        value: 80,
                    },
                    opacity: {
                        value: 0.5,
                    },
                    size: {
                        value: { min: 1, max: 5 },
                    },
                },
                detectRetina: true,
            }}
        />
    );
}

export default App;
```

### 3.2 Vue 3 集成

#### 安装

```bash
npm install @tsparticles/vue3 @tsparticles/slim
```

#### 使用

```vue
<template>
    <Particles
        id="tsparticles"
        :options="particlesOptions"
        :particlesInit="particlesInit"
        :particlesLoaded="particlesLoaded"
    />
</template>

<script setup>
import { loadSlim } from "@tsparticles/slim";
import Particles from "@tsparticles/vue3";

const particlesOptions = {
    background: {
        color: {
            value: "#0d47a1",
        },
    },
    fpsLimit: 120,
    particles: {
        color: {
            value: "#ffffff",
        },
        links: {
            color: "#ffffff",
            distance: 150,
            enable: true,
            opacity: 0.5,
            width: 1,
        },
        move: {
            enable: true,
            speed: 2,
        },
        number: {
            value: 80,
        },
        opacity: {
            value: 0.5,
        },
        size: {
            value: { min: 1, max: 5 },
        },
    },
    detectRetina: true,
};

const particlesInit = async (engine) => {
    await loadSlim(engine);
};

const particlesLoaded = async (container) => {
    console.log("Particles loaded", container);
};
</script>
```

### 3.3 Svelte 集成

#### 安装

```bash
npm install @tsparticles/svelte @tsparticles/slim
```

#### 使用

```svelte
<script>
    import { onMount } from "svelte";
    import Particles from "@tsparticles/svelte";
    import { loadSlim } from "@tsparticles/slim";

    let particlesConfig = {
        background: {
            color: {
                value: "#0d47a1",
            },
        },
        particles: {
            color: {
                value: "#ffffff",
            },
            links: {
                enable: true,
                distance: 150,
            },
            move: {
                enable: true,
                speed: 2,
            },
            number: {
                value: 80,
            },
            size: {
                value: { min: 1, max: 5 },
            },
        },
    };

    let onParticlesLoaded = (event) => {
        const particlesContainer = event.detail.particles;
        console.log("Particles loaded", particlesContainer);
    };

    let particlesInit = async (engine) => {
        await loadSlim(engine);
    };
</script>

<Particles
    id="tsparticles"
    options={particlesConfig}
    on:particlesLoaded={onParticlesLoaded}
    {particlesInit}
/>
```

## 四、预设效果

tsParticles 提供了多种内置预设，可以快速实现常见效果。

### 4.1 使用预设

```bash
# 安装预设包
npm install @tsparticles/preset-confetti
npm install @tsparticles/preset-fireworks
npm install @tsparticles/preset-snow
npm install @tsparticles/preset-stars
```

### 4.2 彩纸效果 (Confetti)

```javascript
import { loadConfettiPreset } from "@tsparticles/preset-confetti";

(async () => {
    await loadConfettiPreset(tsParticles);

    await tsParticles.load({
        id: "tsparticles",
        options: {
            preset: "confetti",
        },
    });
})();
```

### 4.3 烟花效果 (Fireworks)

```javascript
import { loadFireworksPreset } from "@tsparticles/preset-fireworks";

(async () => {
    await loadFireworksPreset(tsParticles);

    await tsParticles.load({
        id: "tsparticles",
        options: {
            preset: "fireworks",
        },
    });
})();
```

### 4.4 雪花效果 (Snow)

```javascript
import { loadSnowPreset } from "@tsparticles/preset-snow";

(async () => {
    await loadSnowPreset(tsParticles);

    await tsParticles.load({
        id: "tsparticles",
        options: {
            preset: "snow",
        },
    });
})();
```

### 4.5 星空效果 (Stars)

```javascript
import { loadStarsPreset } from "@tsparticles/preset-stars";

(async () => {
    await loadStarsPreset(tsParticles);

    await tsParticles.load({
        id: "tsparticles",
        options: {
            preset: "stars",
        },
    });
})();
```

## 五、高级配置

### 5.1 粒子形状

```javascript
{
    particles: {
        shape: {
            type: ["circle", "square", "triangle", "polygon", "star"],
            // 多边形边数
            polygon: {
                sides: 5
            },
            // 星形
            star: {
                sides: 5
            },
            // 自定义图片
            image: {
                src: "https://example.com/particle.png",
                width: 100,
                height: 100
            }
        }
    }
}
```

### 5.2 颜色渐变

```javascript
{
    particles: {
        color: {
            value: "#ff0000",
            // 渐变色
            animation: {
                h: {
                    count: 0,
                    enable: true,
                    offset: 0,
                    speed: 50,
                    sync: false
                },
                s: {
                    count: 0,
                    enable: false,
                    offset: 0,
                    speed: 1,
                    sync: true
                },
                l: {
                    count: 0,
                    enable: false,
                    offset: 0,
                    speed: 1,
                    sync: true
                }
            }
        }
    }
}
```

### 5.3 交互模式

#### 鼠标悬停效果

```javascript
{
    interactivity: {
        events: {
            onHover: {
                enable: true,
                mode: "grab" // grab, bubble, repulse, bounce
            }
        },
        modes: {
            grab: {
                distance: 140,
                links: {
                    opacity: 1
                }
            },
            bubble: {
                distance: 400,
                size: 40,
                duration: 2,
                opacity: 0.8
            },
            repulse: {
                distance: 200,
                duration: 0.4
            }
        }
    }
}
```

#### 点击效果

```javascript
{
    interactivity: {
        events: {
            onClick: {
                enable: true,
                mode: "push" // push, remove, bubble, repulse
            }
        },
        modes: {
            push: {
                quantity: 4
            },
            remove: {
                quantity: 2
            }
        }
    }
}
```

### 5.4 动画效果

#### 粒子大小动画

```javascript
{
    particles: {
        size: {
            value: { min: 1, max: 10 },
            animation: {
                enable: true,
                speed: 20,
                minimumValue: 0.1,
                sync: false,
                destroy: "none", // none, min, max
                startValue: "random" // random, min, max
            }
        }
    }
}
```

#### 透明度动画

```javascript
{
    particles: {
        opacity: {
            value: { min: 0.1, max: 0.5 },
            animation: {
                enable: true,
                speed: 1,
                minimumValue: 0.1,
                sync: false
            }
        }
    }
}
```

### 5.5 运动轨迹

```javascript
{
    particles: {
        move: {
            enable: true,
            speed: 6,
            direction: "none", // none, top, top-right, right, bottom-right, bottom, bottom-left, left, top-left
            random: false,
            straight: false,
            outModes: {
                default: "bounce", // bounce, out, destroy
                bottom: "out",
                left: "out",
                right: "out",
                top: "out"
            },
            attract: {
                enable: false,
                rotateX: 600,
                rotateY: 1200
            },
            trail: {
                enable: true,
                length: 10,
                fillColor: "#000000"
            },
            warp: true
        }
    }
}
```

## 六、实用案例

### 6.1 粒子连线网络

```javascript
const networkConfig = {
    background: {
        color: "#000000"
    },
    particles: {
        color: {
            value: "#ffffff"
        },
        links: {
            color: "#ffffff",
            distance: 150,
            enable: true,
            opacity: 0.4,
            width: 1
        },
        move: {
            enable: true,
            speed: 2
        },
        number: {
            value: 100
        },
        opacity: {
            value: 0.5
        },
        size: {
            value: 3
        }
    },
    interactivity: {
        events: {
            onHover: {
                enable: true,
                mode: "grab"
            },
            onClick: {
                enable: true,
                mode: "push"
            }
        },
        modes: {
            grab: {
                distance: 140,
                links: {
                    opacity: 1
                }
            },
            push: {
                quantity: 4
            }
        }
    }
};
```

### 6.2 泡泡效果

```javascript
const bubbleConfig = {
    background: {
        color: "#87CEEB"
    },
    particles: {
        color: {
            value: ["#ffffff", "#e0f7fa", "#b2ebf2"]
        },
        shape: {
            type: "circle"
        },
        opacity: {
            value: { min: 0.1, max: 0.5 }
        },
        size: {
            value: { min: 5, max: 30 },
            animation: {
                enable: true,
                speed: 5,
                minimumValue: 5,
                sync: false
            }
        },
        move: {
            enable: true,
            speed: { min: 1, max: 3 },
            direction: "top",
            outModes: {
                default: "out",
                top: "destroy",
                bottom: "none"
            }
        },
        number: {
            value: 50
        }
    }
};
```

### 6.3 流星效果

```javascript
const meteorConfig = {
    background: {
        color: "#000033"
    },
    particles: {
        color: {
            value: "#ffffff"
        },
        shape: {
            type: "line"
        },
        opacity: {
            value: { min: 0, max: 1 },
            animation: {
                enable: true,
                speed: 3,
                minimumValue: 0,
                sync: false,
                startValue: "max",
                destroy: "min"
            }
        },
        size: {
            value: { min: 0.1, max: 1 }
        },
        life: {
            duration: {
                value: 2
            },
            count: 1
        },
        move: {
            enable: true,
            speed: 20,
            direction: "bottom-right",
            straight: true,
            trail: {
                enable: true,
                length: 10,
                fillColor: "#000033"
            }
        },
        number: {
            value: 0
        }
    },
    emitters: {
        direction: "bottom-right",
        rate: {
            delay: 1,
            quantity: 1
        },
        size: 
{
            width: 0,
            height: 0
        },
        life: {
            count: 1
        },
        position: {
            x: { random: true },
            y: 0
        }
    }
};
```

### 6.4 DNA 螺旋效果

```javascript
const dnaConfig = {
    background: {
        color: "#000000"
    },
    particles: {
        color: {
            value: ["#00ff00", "#0000ff"]
        },
        shape: {
            type: "circle"
        },
        opacity: {
            value: 0.8
        },
        size: {
            value: 3
        },
        move: {
            enable: true,
            speed: 2,
            path: {
                enable: true,
                generator: "perlinNoise",
                options: {
                    size: 50,
                    increment: 0.01,
                    columns: 6,
                    rows: 6
                }
            }
        },
        number: {
            value: 200
        }
    }
};
```

## 七、性能优化

### 7.1 减少粒子数量

```javascript
{
    particles: {
        number: {
            value: 50, // 根据设备性能调整
            density: {
                enable: true,
                area: 800 // 每800px²的粒子数量
            }
        }
    }
}
```

### 7.2 限制帧率

```javascript
{
    fpsLimit: 60, // 限制最大帧率
    detectRetina: true // 视网膜屏幕优化
}
```

### 7.3 响应式配置

```javascript
{
    responsive: [
        {
            maxWidth: 768,
            options: {
                particles: {
                    number: {
                        value: 30 // 移动设备减少粒子
                    },
                    move: {
                        speed: 1 // 降低移动速度
                    }
                }
            }
        },
        {
            maxWidth: 1024,
            options: {
                particles: {
                    number: {
                        value: 50
                    }
                }
            }
        }
    ]
}
```

### 7.4 使用 Slim 版本

```bash
# 只安装核心功能,减小包体积
npm install @tsparticles/engine @tsparticles/slim

# 而不是完整版
# npm install @tsparticles/engine @tsparticles/all
```

## 八、常见问题

### 8.1 粒子不显示

**问题**: 页面上看不到粒子效果

**解决方案**:

```javascript
// 1. 确保容器有高度
#tsparticles {
    width: 100%;
    height: 100vh; /* 或指定具体高度 */
}

// 2. 检查 z-index
#tsparticles {
    position: fixed;
    z-index: -1; /* 或适当的层级 */
}

// 3. 确保正确加载
await loadSlim(tsParticles); // 先加载引擎
await tsParticles.load({ ... }); // 再初始化
```

### 8.2 性能问题

**问题**: 页面卡顿,CPU 占用高

**解决方案**:

```javascript
{
    fpsLimit: 30, // 降低帧率
    particles: {
        number: {
            value: 30 // 减少粒子数量
        },
        move: {
            speed: 1 // 降低速度
        }
    },
    interactivity: {
        events: {
            onHover: {
                enable: false // 禁用交互
            }
        }
    }
}
```

### 8.3 响应式适配

**问题**: 在不同设备上效果不一致

**解决方案**:

```javascript
{
    detectRetina: true,
    responsive: [
        {
            maxWidth: 768,
            options: {
                particles: {
                    number: { value: 30 }
                }
            }
        }
    ]
}
```

### 8.4 与其他库冲突

**问题**: 和其他 Canvas 库冲突

**解决方案**:

```javascript
// 使用独立的容器
<div id="tsparticles" style="position: absolute; z-index: -1;"></div>
<div id="content" style="position: relative; z-index: 1;">
    <!-- 你的内容 -->
</div>
```

## 九、实战技巧

### 9.1 动态切换配置

```javascript
// 获取容器实例
const container = await tsParticles.load({
    id: "tsparticles",
    options: config1
});

// 动态切换配置
async function switchConfig(newConfig) {
    await container.loadOptions(newConfig);
    await container.refresh();
}

// 使用
document.getElementById('btn').addEventListener('click', () => {
    switchConfig(config2);
});
```

### 9.2 控制粒子生成

```javascript
// 手动触发粒子生成
container.addParticle({
    position: {
        x: event.clientX,
        y: event.clientY
    },
    options: {
        color: { value: "#ff0000" },
        size: { value: 10 }
    }
});

// 清除所有粒子
container.particles.clear();

// 暂停/恢复动画
container.pause();
container.play();
```

### 9.3 监听事件

```javascript
await tsParticles.load({
    id: "tsparticles",
    options: config,
    // 容器加载完成
    loaded: (container) => {
        console.log("Particles loaded", container);
    }
});

// 监听粒子点击事件
container.canvas.element.addEventListener('click', (event) => {
    console.log("Canvas clicked at", event.clientX, event.clientY);
});
```

### 9.4 与动画库结合

```javascript
// 配合 GSAP 使用
import { gsap } from "gsap";

const container = await tsParticles.load({
    id: "tsparticles",
    options: config
});

// 动画粒子数量
gsap.to(container.options.particles.number, {
    value: 200,
    duration: 2,
    onUpdate: () => {
        container.refresh();
    }
});
```

## 十、最佳实践

### 10.1 代码组织

```javascript
// configs/particles.config.js
export const defaultConfig = {
    // 默认配置
};

export const snowConfig = {
    // 雪花效果配置
};

export const fireConfig = {
    // 烟花效果配置
};

// 使用
import { defaultConfig } from './configs/particles.config.js';

await tsParticles.load({
    id: "tsparticles",
    options: defaultConfig
});
```

### 10.2 类型安全 (TypeScript)

```typescript
import type { ISourceOptions } from "@tsparticles/engine";

const particleConfig: ISourceOptions = {
    background: {
        color: {
            value: "#0d47a1",
        },
    },
    particles: {
        color: {
            value: "#ffffff",
        },
        // TypeScript 会提供类型提示和检查
    },
};
```

### 10.3 懒加载

```javascript
// 只在需要时加载
async function initParticles() {
    const { tsParticles } = await import("@tsparticles/engine");
    const { loadSlim } = await import("@tsparticles/slim");
    
    await loadSlim(tsParticles);
    await tsParticles.load({
        id: "tsparticles",
        options: config
    });
}

// 页面加载完成后再初始化
window.addEventListener('load', initParticles);
```

### 10.4 可访问性

```html
<!-- 为视觉障碍用户提供替代方案 -->
<div id="tsparticles" aria-hidden="true"></div>

<!-- 提供禁用动画的选项 -->
<button id="toggle-particles">切换粒子效果</button>

<script>
document.getElementById('toggle-particles').addEventListener('click', () => {
    const container = tsParticles.domItem(0);
    if (container.paused) {
        container.play();
    } else {
        container.pause();
    }
});

// 尊重用户的动画偏好
if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
    // 禁用或简化动画
    config.particles.move.enable = false;
}
</script>
```

## 十一、资源链接

### 官方资源

- [官方网站](https://particles.js.org/)
- [GitHub 仓库](https://github.com/tsparticles/tsparticles)
- [在线编辑器](https://particles.js.org/editor/)
- [完整文档](https://particles.js.org/docs/)
- [API 文档](https://particles.js.org/docs/modules/_tsparticles_engine.html)

### 示例和模板

- [CodePen 示例](https://codepen.io/collection/DPOage)
- [预设效果展示](https://particles.js.org/samples/)
- [React 模板](https://github.com/tsparticles/react)
- [Vue 模板](https://github.com/tsparticles/vue3)

### 社区资源

- [Discord 社区](https://discord.gg/hACwv45Hme)
- [Stack Overflow 标签](https://stackoverflow.com/questions/tagged/tsparticles)
- [npm 包](https://www.npmjs.com/package/@tsparticles/engine)

## 十二、总结

tsParticles 是一个功能强大且灵活的粒子动画库,主要优势包括:

### ✅ 核心优势

1. **高性能** - 优化的渲染引擎,支持大量粒子
2. **易于使用** - 简单的 API,丰富的预设效果
3. **高度可定制** - 数百个配置选项
4. **跨框架** - 支持 React、Vue、Angular 等
5. **轻量级** - 核心库仅 2KB
6. **活跃维护** - 持续更新和支持

### 🎯 使用场景

- 网站背景动画
- 节日特效(雪花、烟花、彩纸)
- 数据可视化
- 游戏效果
- 品牌展示
- 创意设计

### 💡 选择建议

| 场景 | 推荐版本 |
|------|----------|
| 简单背景效果 | @tsparticles/slim |
| 完整功能 | @tsparticles/all |
| 特定预设 | @tsparticles/preset-* |
| React 项目 | @tsparticles/react |
| Vue 项目 | @tsparticles/vue3 |

通过本教程,你应该已经掌握了 tsParticles 的基本使用和高级技巧。立即开始为你的网站添加炫酷的粒子动画效果吧！🎉

---

**提示**: 使用[在线编辑器](https://particles.js.org/editor/)可以实时预览和调试配置,非常方便!