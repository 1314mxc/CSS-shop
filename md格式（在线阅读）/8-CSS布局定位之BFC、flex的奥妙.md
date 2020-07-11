# 霸下负天：CSS布局定位之BFC、flex的奥妙

相信所有前端开发者的入门课，都是从div和css开始的，CSS作为基础中的基础，除了让页面因为样式而变得丰富美观外，更是决定了页面元素的排列布局，然而很多时候，当我们对CSS一知半解，我们总会在开发中遇到元素不能按照我们所想的那样呈现出来的问题，所以本文就来系统的讲讲CSS的布局定位。
## 盒模型
这个在前面已经说过，这里来回顾一下。在CSS中，我们的每个元素本质上都是一个盒子，盒模型决定了一个元素的大小和他所占大小，盒模型由以下几部分组成：
- Margin(外边距) - 清除边框外的区域，外边距是透明的。
- Border(边框) - 围绕在内边距和内容外的边框。
- Padding(内边距) - 清除内容周围的区域，内边距是透明的。
- Content(内容) - 盒子的内容，显示文本和图像。

那么元素的大小到底取决于什么呢？目前有两种盒模型：W3C盒模型和IE盒模型，两种盒模型的计算方法不同，在CSS中，我们可以用box-sizing指定使用哪种方式计算：
- content-box（W3C盒模型）：盒子宽高 = 内容宽高（content）
- border-box（IE盒模型）：盒子宽高 = 内容宽高（content） + 内边距 （padding）+ 边框（border）

看个例子：
```
// W3C盒模型
<style>
  .box {
    width: 200px;
    height: 200px;
    padding: 10px;
    margin: 10px;
    border: 5px solid red;
    background-color: yellowgreen;
  }
</style>

<div class="box"></div>
```
```
// IE盒模型
<style>
  .box {
    width: 200px;
    height: 200px;
    padding: 10px;
    margin: 10px;
    border: 5px solid red;
    background-color: yellowgreen;
    box-sizing: border-box;       /** !!! **/
  }
</style>

<div class="box"></div>
```
所以我们发现，不管是哪一种盒模型，margin的长度都不会被包含在元素的宽高里，但是magin将决定元素在其父元素中的位置。
### display 
display用于定义建立布局时元素生成的显示框类型，会影响到兄弟元素之间的布局，他的取值有以下几种：
- none：此元素不会被显示。
- block： 此元素将显示为块级元素，此元素前后会带有换行符。
- inline：默认值。此元素会被显示为内联元素，元素前后没有换行符。
- inline-block：行内块元素。
- list-item：此元素会作为列表显示。
- table相关：此元素会被显示为表格或表格元素。
- flex：弹性盒子。
- run-in：此元素会根据上下文作为块级元素或内联元素显示。

下面我们将对inline-block，flex这几个属性做详细讲解。
## inline-block
inline-block和float，flex并称实现元素行内显示的“三巨头”，相比较flex的丰富属性，inline-block就略显朴素，使用也十分简单，看起来似乎完全不值得对它进行详细介绍，但不知道你们在使用的时候是否也踩过那些inline-block的坑，所以这里我们来讲讲inline-block的必坑指南
坑一：行内元素间隙
我们先来看一个例子：
```
<div class="content">
  <div class="cell"></div>
  <div class="cell"></div>
  <div class="cell"></div>
</div>

.content {
  background-color: yellow;
  width: 350px;
  height: 100px;
}
.cell {
  display: inline-block;
  width: 100px;
  height: 50px;
  background-color: red;
}
```

我们会发现，几个行内元素之间出现了的空隙，这个空隙实际上是因为我们的HTML书写导致的，我们编写代码时输入空格、换行都会产生空白符。而浏览器是不会忽略空白符的，对于多个连续的空白符浏览器会自动将其合并成一个，所以就产生了所谓的间隙。 想要解决这个间隙，我们可以给父元素设置font-size:0。
```
<div class="content">
  <div class="cell"></div>
  <div class="cell"></div>
  <div class="cell"></div>
</div>

.content {
  background-color: yellow;
  font-size: 0;
  width: 350px;
  height: 100px;
}
.cell {
  display: inline-block;
  width: 100px;
  height: 50px;
  background-color: red;
}
```

坑二：高度塌陷，对齐问题
这个问题，是在开发的时候偶然遇见的，话不多说，先看看这两个例子：
```
<div class="content">
  <div class="cell">有内容</div>
  <div class="cell"></div>
  <div class="cell">有内容</div>
</div>

.content {
  background-color: #eee;
  font-size: 0;
  width: 350px;
  height: 100px;
}
.cell {
  display: inline-block;
  width: 100px;
  height: 50px;
  font-size: 16px;
  background-color: #666;
  color: #fff;
}
```
在这个例子里，我们发现有字的行内块元素和没有字的行内块元素显示不在同一高度。然后我们再看另一个例子：
```
<div class="content">
  <div class="cell">有内容</div>
  <div class="cell" id="spec">有内容</div>
  <div class="cell">有内容</div>
</div>

.content {
  background-color: #eee;
  font-size: 0;
  width: 350px;
  height: 100px;
}
.cell {
  display: inline-block;
  box-sizing: border-box;
  width: 100px;
  height: 50px;
  line-height: 50px;
  border-radius: 50px;
  margin: 5px;
  font-size: 16px;
  text-align: center;
  background-color: #fff;
  color: #333;
}
.active {
  border: 2px solid #E20000;
  color: #E20000;
}
let spec = document.getElementById("spec");let flag = false;
spec.addEventListener("click", () => {
  flag = !flag;
  if (flag) {
    spec.classList.add('active')
  } else {
    spec.classList.remove('active')
  }
});
```

我们会发现，当中间的元素添加了边框后，三个行内元素都发生了上下抖动，那是否是因为他们的高度发生了变化呢？那么我们就来测量一下：

测量结果显示，这三个元素的高度一样，并没有发生改变，那么究竟是什么原因导致了行内元素高度塌陷、上下抖动的呢？
其实造成这一结果的是vertical-align属性。inline-block作为行内块元素，也遵循行内元素的vertical-align垂直对齐方式，vertical-align的取值有以下几种：
- baseline：将元素的基线与父元素的基线对齐。
- sub：将元素的基线与其父元素的下标基线对齐。
- super：将元素的基线与其父代的上标 - 基线对齐。
- text-top：将元素的顶部与父元素的字体顶部对齐。
- text-bottom：将元素的底部与父元素的字体的底部对齐。
- middle：将元素的中间与基线对齐加上父元素的x-height的一半。
- top：将元素的顶部和其后代与整行的顶部对齐。
- bottom：将元素的底部和其后代与整行的底部对齐。
- ```<length>```：将元素的基线对准给定长度高于其父元素的基线。
- ```<percentage>```：像<长度>值，百分比是line-height属性的百分比。

在默认情况下，会根据baseline进行定位，文字的定位不同，最终导致了整个块元素的高度塌陷。想要解决这一问题，我们只需将vertical-align设置为top，如下： 情况一：
```
<div class="content">
  <div class="cell">有内容</div>
  <div class="cell"></div>
  <div class="cell">有内容</div>
</div>

.content {
  background-color: #eee;
  font-size: 0;
  width: 350px;
  height: 100px;
}
.cell {
  display: inline-block;
  vertical-align: top;
  width: 100px;
  height: 50px;
  font-size: 16px;
  background-color: #666;
  color: #fff;
}
```
情况二：
```
<div class="content">
  <div class="cell">有内容</div>
  <div class="cell" id="spec">有内容</div>
  <div class="cell">有内容</div>
</div>

.content {
  background-color: #eee;
  font-size: 0;
  width: 350px;
  height: 100px;
}
.cell {
  display: inline-block;
  vertical-align: top;
  box-sizing: border-box;
  width: 100px;
  height: 50px;
  line-height: 50px;
  border-radius: 50px;
  margin: 5px;
  font-size: 16px;
  text-align: center;
  background-color: #fff;
  color: #333;
}
.active {
  border: 2px solid #E20000;
  color: #E20000;
}
let spec = document.getElementById("spec");let flag = false;
spec.addEventListener("click", () => {
  flag = !flag;
  if (flag) {
    spec.classList.add('active')
  } else {
    spec.classList.remove('active')
  }
});
```


## Flex box
Flex box可能是目前用来做自适应用的最多的布局方式了，Flex使用的丰富，也为开发者提供了更便捷更可靠的布局。
容器属性
### flex-direction
决定主轴的方向（即项目的排列方向）。取值：
- row（默认值）：主轴为水平方向，起点在左端。
- row-reverse：主轴为水平方向，起点在右端。
- column：主轴为垂直方向，起点在上沿。
- column-reverse：主轴为垂直方向，起点在下沿。

### flex-wrap
决定了子元素如何换行。取值：
- nowrap（默认）：不换行。
- wrap：换行，第一行在上方。
- wrap-reverse：换行，第一行在下方。

#### flex-flow
是flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap。
#### justify-content
决定了项目在主轴上的对齐方式。取值：
- flex-start（默认值）：左对齐
- flex-end：右对齐
- center： 居中
- space-between：两端对齐，项目之间的间隔都相等。
- space-around：每个项目两侧的间隔相等。项目之间的间隔比项目与边框的间隔大一倍。

#### align-items
决定了项目在交叉轴上如何对齐。取值：
- flex-start：交叉轴的起点对齐。
- flex-end：交叉轴的终点对齐。
- center：交叉轴的中点对齐。+
- baseline: 项目的第一行文字的基线对齐。
- stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。

#### align-content
决定了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。取值：
- flex-start：与交叉轴的起点对齐。
- flex-end：与交叉轴的终点对齐。
- center：与交叉轴的中点对齐。
- space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
- space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
- stretch（默认值）：轴线占满整个交叉轴。

子元素属性
### order
决定项目的排列顺序。数值越小，排列越靠前，默认为0。
### flex-grow
决定项目的放大比例，默认为0，即如果存在剩余空间，也不放大。
### flex-shrink
决定项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。
### flex-basis
决定在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。
### flex
是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个属性可选。
align-self
允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。

> 注意点:设为 Flex 布局以后，子元素的float、clear和vertical-align属性都将失效。
## Float 浮动
在Flex box被广泛使用以前，我们经常会利用float来实现块元素行内两端显示，它的使用也十分简单，float属性有以下几种取值：
- none：默认值，没有浮动效果。
- left：元素浮动到父元素左侧。
- right：元素浮动到父元素右侧。
- inherit：规定应该从父元素继承 float 属性的值。

float本身没有什么难度，但是**设置了float会导致父元素的无法被撑开，margin失效**，所以清除float也就成了开发必备知识。
## 清除浮动
原始效果：
```
<div class="content">
  <div class="cell spe1">块1</div>
  <div class="cell spe2">块2</div>
</div>

.content {
  background-color: #eee;
  font-size: 0;
  width: 350px;
}
.cell {
  font-size: 16px;
  background-color: #666;
  color: #fff;
}
.spe1 {
  width: 80px;
  height: 20px;
  float: left;
}
.spe2 {
  width: 80px;
  height: 30px;
  float: right;
}
```

方法一：设置空元素

```
<div class="content">
  <div class="cell spe1">块1</div>
  <div class="cell spe2">块2</div>
  <div class="clr"></div>
</div>

.clr {
  clear: both;
}
```
方法二：after伪元素
```
.content:after {
  content: '';
  display: block;
  visibility: hidden;
  clear: both;
}
```
该方法也是利用clear:both来清除浮动，与方法一的区别在于，方法一添加了一个clear元素，而这一个是在元素内部增加一个类似于clear的效果，其中有几个注意点：
- content：可以为空，也可以取任意值。
- display：必须设置为block，否则在FF/chrome/opera/IE8下不能正常起作用。
- visibility：设置为hidden是为了允许浏览器渲染它，但是不显示出来。

方法三：利用overflow:hidden属性
```
.content {
  overflow: hidden;
}
```
通过设置overflow达到清除浮动效果，其原理是创建了一个BFC，有关BFC的知识，我们会在后面章节里详细介绍。
### position 定位
position属性决定了某个元素的定位方式，它的取值有以下几种：
- static：该关键字指定元素使用正常的布局行为，即元素在文档常规流中当前的布局位置。(**常规流值**)
- relative：该关键字下，元素先放置在未添加定位时的位置，再在不改变页面布局的前提下调整元素位置（因此会在此元素未添加定位时所在位置留下空白）。(**常规流值**)
- absolute：元素会被移出正常文档流，并不为元素预留空间，通过指定元素相对于最近的非 static 定位祖先元素的偏移，来确定元素位置。绝对定位的元素可以设置外边距（margins），且不会与其他边距合并。
- fixed：元素会被移出正常文档流，并不为元素预留空间，而是通过指定元素相对于屏幕视口（viewport）的位置来指定元素位置。元素的位置在屏幕滚动时不会改变。打印时，元素会出现在的每页的固定位置。
- sticky：元素根据正常文档流进行定位，然后相对它的最近滚动祖先（nearest scrolling ancestor）和 containing block (最近块级祖先 nearest block-level ancestor)，包括table-related元素，基于top, right, bottom, 和 left的值进行偏移。偏移值不会影响任何其他元素的位置。

在这些属性里，其中relative我们称之为相对定位，absolute为绝对定位，fixed为固定定位，sticky为粘性定位，而这几个定位又可以区分为Normal flow（常规流）和Unnormal Flow（非常规流）。
Normal flow（常规流）
常规流具备以下特性：
- 盒一个接着一个排列;
- 在块级格式化上下文里面， 元素竖着排列；
- 在行内格式化上下文里面， 元素横着排列;
- 对于静态定位(static positioning)，position: static，盒的位置是常规流布局里的位置；
- 对于相对定位(relative positioning)，position: relative，盒偏移位置由这些属性定义top，bottom，leftandright。即使有偏移，仍然保留原有的位置，其它常规流不能占用这个位置。

常规流里的元素属于格式上下文（formatting context），块元素和行内元素表现不同会产生快格式上下文和行内格式上下文，也就是我们常说的BFC和IFC。
## BFC (Block Formatting Context)
BFC的创建方法:
- 根元素或其它包含它的元素。
- 浮动 (元素的float不为none)。
- 绝对定位元素 (元素的position为absolute或fixed)。
- 行内块inline-blocks(元素的 display: inline-block)。
- 表格单元格(元素的display: table-cell，HTML表格单元格默认属性)。
- overflow的值不为visible的元素。
- 弹性盒flex boxes(元素的display: flex或inline-flex)。

**BFC的本质就是创建一个隔离的空间**，他的具体效果如下：
1. 内部的盒会在垂直方向一个接一个排列（可以看作BFC中有一个的常规流）。
2. 处于同一个BFC中的元素相互影响，可能会发生margin collapse。
3. 每个元素的margin box的左边，与容器块border box的左边相接触(对于从左往右的格式化，否则相反)，即使存在浮动也是如此。
4. BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素，反之亦然。
5. 计算BFC的高度时，考虑BFC所包含的所有元素，连浮动元素也参与计算。
6. 浮动盒区域不叠加到BFC上。

对于上述效果，除了1和3，似乎都让人有些摸不着头脑，那么就让我们通过几个例子来仔细研究一下。

**案例一：margin collapse**
首先我们来了解一下什么是外边距折叠（margin collapse）。 在CSS中，两个或多个毗邻的普通流中的盒子（可能是父子元素，也可能是兄弟元素）在垂直方向上的外边距会发生折叠，这种形成的外边距称之为外边距折叠。
那么什么时候外边距会发生折叠呢？发生折叠需要满足这几种情况：
- 都属于普通流的Block box且参与到相同的BFC中
- 没有被padding、border、clear和line box分隔开 都属于垂直毗邻盒子边缘：

   1. 盒子的top margin和它第一个普通流子元素的top margin

   2. 盒子的bottom margin和它下一个普通流兄弟的top margin

   3. 盒子的bottom margin和它父元素的bottom margin

盒子的top margin和bottom margin，且没有创建一个新的块级格式上下文，且有被计算为0的min-height，被计算为0或auto的height，且没有普通流子元素

下面让我们来看看代码实例：
```
<div class="top"></div>
<div class="content">
  <div class="cell"></div>
  <div class="cell"></div>
</div>
<div class="foot">
  <div class="cell2"></div>
  <div class="spe"></div>
  <div class="cell2"></div>
</div>

.top {
  background-color: #60acfc;
  width: 100px;
  height: 50px;
}
.content {
  margin-top: 10px;
  background-color: #eee;
  width: 350px;
  margin-bottom: 10px;
}
.cell {
  margin-top: 20px;
  background-color: #5bc49f;
  height: 20px;
  margin-bottom: 30px;
}
.foot {
  background-color: #60acfc;
  width: 100px;
}
.spe {
  margin: 20px 0 30px;
}
.cell2 {
  height: 20px;
  background-color: #eee;
}
```

- .content和.cell的上边距折叠属于第一条，最终.content和.top之间的边距为20px。
- .cell之间的边距折叠属于第二条，最终.cell之间的边距为30px。
- .cell和.content的下边距折叠属于第三条，最终.content和.foot之间的边距为30px。
- .spe自己的上下边距折叠属于第四条，最终.spe占据的间隔为30px。

现在我们知道了margin collapse发生的情况，那我们盖如何避免边距折叠呢？方法有以下几种：
- 浮动元素不会与任何元素发生折叠，也包括它的子元素。
- 创建了 BFC 的元素不会和它的子元素发生外边距折叠。
- 绝对定位元素和其他任何元素之间不发生外边距折叠，也包括它的子元素。
- inline-block 元素和其他任何元素之间不发生外边距叠加，也包括它的子元素。
- 给常规流中的块级元素添加padding、border、clear属性。

**案例二：BFC计算高度包含浮动盒**
```
<div class="BFC">
  <div class="left"></div>
  <div class="right"></div>
</div>

.BFC {
  background-color: #eee;
}
.left {
  float: left;
  background-color: #60acfc;
  width: 100px;
  height:50px;
  opacity: 0.6;
}
.right {
  width: 150px;
  background-color: #5bc49f;
  height:30px;
}
```
在我们没有创建BFC的时候，父元素的高度不包含浮动元素高度。
```
.BFC {
  background-color: #eee;
  overflow: hidden;
}
```
创建BFC后，父元素就会加上浮动元素高度，这个就是我们在之前提到的清除浮动的方法之一。

**实例三：BFC内部不受浮动元素影响**
```
<div class="BFC content">
  <div class="left"></div>
  <div class="right BFC">
    <div class="cell"></div>
    <div class="cell"></div>
  </div>
</div>

.BFC {
  overflow: hidden;
}
.content {
  background-color: #eee;
}
.left {
  float: left;
  background-color: #60acfc;
  width: 100px;
  height:50px;
  opacity: 0.6;
}
.right {
  width: 150px;
  background-color: #5bc49f;
  height:30px;
}
.cell {
  width: 20px;
  margin: 5px;
  background-color: #fff;
  height:20px;
  display: inline-block;
}
```
从这个例子，我们就可以理解，BFC布局规则里的第4和第6条：
- BFC内部是一个独立区域，不会受到外部元素的影响。
- BFC外部的浮动不会和BFC发生重叠。

## IFC (Inline Formatting Context)
IFC由inline元素构成，它的布局规则是这样的：
- 高度由其包含行内元素中最高的实际高度计算而来。
- IFC中的line box一般左右都贴紧整个IFC，但是会因为float元素而扰乱。float元素会位于IFC与与line box之间，使得line box宽度缩短。
- 同个ifc下的多个line box高度会不同。
- 水平的margin、padding、border有效，垂直无效。不能指定宽高。
- 行框的高度由行高来决定。

那么IFC一般有什么用呢？
- 水平居中：当一个块要在环境中水平居中时，设置其为inline-block则会在外层产生IFC，通过text-align则可以使其水平居中。
- 垂直居中：创建一个IFC，用其中一个元素撑开父元素的高度，然后设置其vertical-align:middle，其他行内元素则可以在此父元素下垂直居中。

### Unnormal Flow（非常规流）
属于非常规流的取值：
- absolute
- fixed
- sticky

对于非常规流，因为他们脱离了正常的文档流，所以他们不会对正常流的元素造成影响，很多时候我们都会配合z-index使用，前两种相信大家都不陌生，所以就不再多做介绍啦，下面我们来讲讲sticky。
### 粘性定位（sticky)
粘性定位（sticky)可以被认为是相对定位和固定定位的混合。元素在跨越特定阈值前为相对定位，之后为固定定位。
```
<div class="content">
  <h1 class="title">我是一块内容</h1>
  <div class="sticky"></div>
  <div class="main"></div>
</div>

.content {
  background-color: #eee;
}
.title {
  color: #333;
}
.sticky {
  position: sticky;
  top: 0px;
  background-color: #60acfc;
  width: 100px;
  height:50px;
}
.main {
  width:150px;
  height: 900px;
}
```
使用sticky布局的时候有几个注意点：
- 父级元素不能有任何overflow:visible以外的overflow设置。
- 父级元素设置和粘性定位元素等高的固定的height高度值，或者高度计算值和粘性定位元素高度一样，没有粘滞效果。
- 同一个父容器中的sticky元素，如果定位值相等，则会重叠；如果属于不同父元素，且这些父元素正好紧密相连，则会鸠占鹊巢，挤开原来的元素，形成依次占位的效果。
- sticky定位，不仅可以设置top，基于滚动容器上边缘定位；还可以设置bottom，也就是相对底部粘滞。如果是水平滚动，也可以设置left和right值。
- z-index无效。

## VFC
VFC，也就是Visual formatting context，虚拟格式上下文，他有BFC，IFC，GFC和FFC这几种类型，在前文中，我们已经介绍了BFC，IFC，这里再简单介绍下GFC和FFC。
## GFC
GFC(GridLayout Formatting Contexts)，"网格布局格式化上下文"，当为一个元素设置display值为grid的时候，此元素将会获得一个独立的渲染区域。
我们可以通过在网格容器（grid container）上定义网格定义行（grid definition rows）和网格定义列（grid definition columns）属性各在网格项目（grid item）上定义网格行（grid row）和网格列（grid columns）为每一个网格项目（grid item）定义位置和空间。
那么GFC有什么用呢，和table又有什么区别呢？首先同样是一个二维的表格，但GridLayout会有更加丰富的属性来控制行列，控制对齐以及更为精细的渲染语义和控制。
## FFC
FFC(Flex Formatting Contexts)，"自适应格式化上下文"，display值为flex或者inline-flex的元素将会生成自适应容器（flex container）。 所以当我们使用flex布局的时候，我们就创建了一个FFC。
