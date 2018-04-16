# phaser文档
## 简介

phaser是一款专门用来做2d游戏的框架，底层渲染机制基于PIXI.js，在同类游戏引擎里面性能最好（参考[HTML5游戏引擎深度测评](https://juejin.im/entry/5710a800ebcb7d00559448a5 "HTML5游戏引擎深度测评")）

---------

### phaser架构
![](https://vipweb.bs2cdn.yy.com/vipinter_ce94c9c4db2f4a49af2fbd6ce2b3ad9e.jpg)

!> 出自[phaser小站](https://www.phaser-china.com/)


## 代码组织
网上示例多以es5为主，以下代码均采用es6语法，可以参考[level up your phaser games width es6](https://www.joshmorony.com/level-up-your-phaser-games-with-es6/ "level up your phaser games width es6")

> 状态（state）是Phaser官方建议的游戏代码组织方式，开发者可以把游戏逻辑封装到一个个state里面，然后提交给框架去执行

可以将`state`理解游戏的每一个场景(每一屏)，也可以将`state`理解为游戏的关卡，不同的`state`做不同的事情

可以将代码结构初步规划如下：
    + state
        Preload.js
        Play.js
        ...
    index.js

`index.js`作为程序的入口，`state`里面包含每一个场景，顾名思义，`Preload.js`用于程序资源的预加载，显示loading图片与加载进度，`Play.js`用于编写游戏逻辑，显示游戏界面

index.js代码如下：
```javascript
import Preload from './states/Preload'
import Play from './states/Play'

class Game extends Phaser.Game {
  constructor() {
    super(302, 505, Phaser.CANVAS, 'game')

    this.state.add('Preload', Preload)
    this.state.add('Play', Play)

    this.state.start('Preload')
  }
 }

 new Game()
```
super中的参数含义分别是，游戏的宽，高，渲染机制与装载在到哪个元素上，宽高请参考设计图，这个涉及到适配，适配问题比较麻烦，需根据具体游戏具体分析；由于我们游戏需求多数都放在移动端，所以第三个参数务必填写Phaser.CANVAS；第四个参数指明了我们在模板里面应该要有`<div id='game'></game>`（注意：这个标签元素的padding样式必须为0）

紧接着代码添加Preload与Play状态，并启动了Preload状态

状态的代码参照如下：
```javascript
export default class StateName extends Phaser.State {
    preload() {
    console.log('preload resource....')
    }
    create() {
    console.log('game obj create...')
    }
    update() {
    console.log('view update...')
    }
    render() {
    console.log('view render...')
    }
}
```
Phaser为我们提供了很多的钩子，以上钩子会依次执行：

- preload：资源加载接口，在这个接口实现中加载游戏资源，触发1次
- create：游戏对象创建接口，在这个接口实现中初始化游戏数据，触发1次
- update：游戏逻辑更新接口，在这个接口实现中编写游戏逻辑，不断触发，称之为游戏循环（game loop）
- render：附加渲染接口，在这个接口实现中编写附加的渲染逻辑，不断触发，如debug调试之类的东西可以丢在这，正式环境就不要执行了(vipm中通过LOCAL_ROOT可以区分)

合法的state只需要包含以上钩子中的任意一个即可，此外还有很多别的钩子，如：init(初始化的时候调用，比如屏幕适配的东西可以丢在这), paused(程序切到后台模式的时候), shutdown(切换到下一个state之前触发), resize, loadUpdate等等，更多详情请参考官方文档


## Arcade物理系统
> Arcade是Phaser默认的物理系统，启动游戏之时，会自动开启Arcade物理系统

> Arcade系统的一个局限性在于它只支持矩形或元素的物理物体，当使用的游戏对象是不规则形状的时候，Arcade就无能为力了

phaser还支持其他功能丰富的物理系统，如p2，详情看官网
#### 基本使用
```javascript
启动精灵的物理特性
组对象，设置该属性后所以成员都将拥有物理属性
groupObj.enableBody = true
// or 可以设置组对象与显示对象
this.physics.enable(sprite)  第一个参数也可以是数组，可以批量处理, 第二个参数是要启动的物理引擎，默认使用aracde
```
### 物理实体中的运动学参数
```javascript
velocity   速度
acceleration  加速度
drag  阻力，或者说是摩擦系数
gravity  重力
```
以上值的设置有两种方式

**全局设置**(同样有两种设置方式)
```javascript
this.physics.arcade.gravity.x = 100
this.physics.arcade.gravity.y = 100
// or
this.physics.arcade.gravity.setTo(100, 100)

```
**在精灵对象上单独设置**(两种方式)
```javascript
sprite.body.velocity.x = 100
sprite.body.velocity.x = 100
// or
sprite.body.velocity.setTo(100, 100)
```

<script async src="//jsfiddle.net/xiaodianxie/n3bzfejz/17/embed/"></script>

### 物理实体中的动力学参数
```javascript
mass：质量，默认是1，不同物体的重量是不一样，不同重量的物体相撞也是会产生不一样的反冲力，比如质量较大的物体反冲运动不会特别大
bounce: 弹性系数
immovable：不可移动的物体，碰撞不会影响其运动
friction：摩擦系数，仅限于immovable实体使用。类型：Phaser.Point， 默认值：(1,0)
```
<script async src="//jsfiddle.net/xiaodianxie/n3bzfejz/18/embed/"></script>
