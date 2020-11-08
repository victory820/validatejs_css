
#### 浏览器包含的进程

1. Browser进程。
    作用：主进程，负责浏览器的界面显示和各个页面的管理，负责其他进程的创建和销毁，它有且只有一个

2. Renderer进程。
    作用：网页渲染进程，负责页面的渲染，可以有多个。

3. 各种插件进程

4. GPU进程

#### 查看进程

cmd + 空格 =》 搜索活动监视器



#### 渲染引擎

##### 主要模块
  HTML解析器：解析html文档，主要将html解释成DOM树
  CSS解析器：为DOM的各个元素计算出样式信息，为布局提供基础设施
  JS引擎：可以修改网页内容，也能修改css信息，通过dom接口和css树接口来修改网页内容和样式
  布局layout模块：在DOM创建后，内核（webkit）将元素对象和样式结合起来，计算他们的大小位置等布局信息，形成一个能表达这所有信息的内部表示模型
  绘图模块：将布局计算后的各个网页节点绘制成图像结果。

##### 大致的渲染过程

浏览器渲染页面的整个过程：浏览器从上到下解析文档
1. 遇见HTML标记调用**HTML解析器**解析为对应token（一个token就是标签文本的序列化）构建成DOM树(就是一块内存，保存着token信息，建立它们之间关系)
2. 遇见style/link标记调用**相应解析器**处理css标记，构建css样式树
3. 遇见script标记调用js引擎，处理script标记、绑定事件、修改dom树/css树等
4. 将dom树和css树合并成一个渲染树
5. 根据渲染树渲染，计算每个节点的几何信息（需要依赖GPU）
6. 最终将各个节点绘制到屏幕上

渲染中还会依赖其他基础模块。如：网络，存储，2D/3D图像，音视频和图片解码


查看渲染

performance=>刷新页面
loading加载信息
scripting脚本
rendering渲染
painting打印

start time不会从0开始，因为渲染其他信息需要时间


#### style样式的渲染

页面内style用html解析器解析


link引入的css会阻塞渲染，需要加loading处理


#### js渲染

js阻塞页面渲染


### 阻塞渲染

#### 1. css阻塞（只有link引入的外部css才能够产生阻塞）
  1. style标签中内容

      1. 由html解析器进行解析
      2. 不阻塞浏览器渲染
      3. 不阻塞DOM解析
  2. link引入内容

      1. 由css解析器进行解析
      2. 阻塞浏览器渲染
      3. 阻塞后面js的执行
      4. 大部分浏览器中不阻塞DOM解析

  3. 优化核心：提高外部css的加载速度

      1. 使用CDN方式进行加速
      2. 对css进行压缩
      3. 减少http请求，合并css.（辩证看待）
      4. 优化样式代码

#### 2. js阻塞
  
  1. 阻塞后续dom解析。原因：js是可以操作dom的，如果不阻塞，可能做无用功
  2. 阻塞页面渲染。原因：js可以修改样式，如果不阻塞，可能做无用功
  3. 阻塞后续js的执行。原因：维护依赖关系

#### 附加

  1. css解析和js解析是互斥的。css解析时js停止执行，js执行时css停止解析
  2. 无论css阻塞还是js阻塞都不会阻塞浏览器加载外部资源(图片音频视频样式)。为了效率浏览器自己会协调返回数据的具体使用时间。
  3. webkit和firefox都行了预解析优化，在执行js脚本时，浏览器的其他线程会预解析文档的其余部分，找出并通过网络加载其他资源，但预解析器不会修改dom树


<script defer src=""></script>
defer紧随页面dom解析以后触发

### 验证js和css对页面渲染和dom解析的影响

  1. link进来的css阻塞页面渲染
  2. link进来的css不阻塞页面解析
  3. js阻塞页面解析
  4. js阻塞页面渲染

#### layout和paint

Layout:重排

paint:重绘

### 图层

浏览器在渲染一个页面时，会将页面分为很多个图层，图层有大有小上面有一个或多个节点。

#### 在渲染DOM的时候，浏览器的实际工作是：
1. 获取DOM后分割为多个图层
2. 对每个图层的节点计算样式结果(recalculate style样式重计算)
3. 为每个节点生成图形和位置(layout重排/回流)
4. 将每个节点绘制填充到图层位图中(paint重绘)
5. 图层作为纹理上传给GPU
6. 组合多个图层到页面上生成最终屏幕图形(composite layers图层重组)

#### 图层创建条件（以chrome为例,还有其他暂时只提以下同时也依赖电脑配置和浏览器设置）
1. 拥有具有3D变换的css属性
2. 使用加速视频解码的video节点
3. canvas节点
<!-- 4. css3动画的节点 -->
5. 拥有css加速属性的元素(will-change)


### 重绘repaint

是一个元素外观的改变所触发的浏览器行为。重绘不会带来重新布局，所以并不一定伴随重排。

重绘是以图层为单位的，图层中某个元素需要重绘，整个图层都会重绘

为了提高性能应该让变化的东西拥有一个自己的图层

### 重排reflow（回流）

渲染对象在创建完成并添加到渲染树时，并不包含位置和大小信息，计算这些值的过程称为重排

重排大多数情况下会导致重绘。比如改变元素的位置，就会触发重绘和重排


#### 触发重绘的属性

| - | - | - |
| :-----: | :----: | :----: |
| color | background | outline-color |
| border-style | background-image | outline |
| border-raius | background-position | outline-style |
| visibility | background-repeat | outline-width |
| text-decoration | background-size | box-shadow |

#### 触发重排的属性

| - | - | - |
| :-----: | :----: | :----: |
| width | top | text-align |
| height | bottom | overflow-y |
| padding | left | font-weight | 
| margin | right | overflow |
| display | position | font-family | 
| border-width | float | line-height |
| border | clear | vertival-align |
| min-height | white-space |

#### 常见触发重排的操作

reflow的成本比repaint高，一个节点的reflow很有可能导致子节点甚至父节点及同级节点的reflow

下面动作很大可能是成本高的
1. 增加、删除、修改DOM节点时，导致reflow，repaint
2. 移动dom位置
3. 修改css样式
4. resize窗口时
5. 修改网页的默认字体
<!-- 6. 获取某些属性（width,height）不理解 -->

display:none会触发reflow

visibility:hidden只会触发repaint

#### 优化方案

渲染页面经历的环节
1. 计算需要被加载到节点的样式结果（recalculate style）
2. 为每个节点生成图形和位置（layout）
3. 将每个节点填充到图层中（paint）
4. 组合图层到页面上（composite layers）


#### 具体方案(参照optimize文件夹)

1. 元素位置移动变换时尽量使用css3的transform来替代top/left等操作，变换(transform)和透明度(opacity)的改变仅仅影响图层的组合
2. 使用opacity来替代visibility.（新版本浏览器不适用）
    
    a. 使用visibility不触发重排，但是依然重绘
    b. 直接使用opacity既触发重绘，又触发重排（GPU设计如此）
    c. opacity配合图层使用，既不触发重绘也不触发重排 
3. 将多次改变样式属性的操作合并成一次，即不要一条一条修改dom样式，预先定义class修改classname
4. 将dom离线后再修改。先把元素display:none后在对隐藏元素进行操作不会引发其他元素重排。
5. 利用文档碎片-documentFragment.参照https://developer.mozilla.org/zh-CN/docs/Web/API/DocumentFragment
6. 不要把获取某些dom节点的属性值放到循环内当作循环变量。当向浏览器请求style信息时，就会让浏览器flush队列.如：offsetTop/Left/Width/Height;scrollTop/Left/Width/Height;clientTop/Left/Widht/Height;width/height
7. 动画实现过程中，启用GPU硬件加速transform: tranlateZ(0)
8. 为动画元素新建图层，提高动画元素的z-index
9. 编写动画时可以使用以下api
    a. requestAnimationFrame https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame


#### requestAnimationFrame
1. window.requestAnimationFram() 该方法会告诉浏览器在下一次重绘重排之前调用指定的函数。
  a. 参数：回调函数。会被自动传入一个参数DOMHighResTimeStamp标识requestAnimationFram开始触发回调函数的当前时间
  b. 返回值：一个long整数请求id唯一标识
2. window.cancleAnimationFrame(id)。取消先前的requestAnimationFram方法。