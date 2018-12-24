CSS样式的书写顺序及原理:
一.CSS书写顺序
1. 定位属性：影响文档流的属性（比如：display, visibility, float, clear, overflow, position, z-index等）
2. 自身属性：自身盒模型的属性（比如：width, height, margin, padding, border, border-radius, outline, table-layout等）
3. 文本属性：排版相关属性（比如：font, text-align, text-decoration, letter-spacing, word-spacing, white-space, vertical-align等）
4. 视觉外观：color, background, opacity, cursor, content等
5. CSS3属性：box-shadow, transition, animation等

二.CSS书写规范：
简写命名: 见名知意
缩写: padding, margin, font, 小数点前的“0”，16进制颜色代码缩写等 
css中尽量用class，而少用id（js用）
连字符： —
为选择器添加状态前缀 ”.is-”
书写时重定义的最先，伪类其次，自定义最后（其中a:link a:visited a:hover a:actived 要按照顺序写）
区域注释: /*=S 注释内容 */ 和 /*=E 注释内容 */
单行注释: /* 注释内容 */
outline: 类似border，不同的是border会影响盒模型，而outline不会
outline-width, outline-style, outline-color
outline: 1px solid red;
简写: outline-style是必须的，另外两个值则可选，默认值和border相同
font: font-size与font-family继承
font-style, font-variant, font-weight, font-size/line-height, font-family
font: italic small-caps bold 1em/1.5em sans-serif
简写： 只有在同时指定font-size和font-family属性时才起作用，如果你没有设定font-weight, font-style, font-varient, css会使用缺省值
background: 不继承，可以只声明一个，其他值默认
background-color, background-image, background-repeat, background-position, background-attachment
background: #fff url(img.png) no-repeat 0 0;
简写默认值: transparent none repeat scroll top left;

三.CSS书写目的和原理
目的：减少浏览器reflow（回流），提升浏览器渲染dom的性能
原理：浏览器的渲染流程为——
①解析html构建dom树，解析css构建css树：将html解析成树形的数据结构，将css解析成树形的数据结构
②构建render树：DOM树和CSS树合并之后形成的render树。
③布局render树：有了render树，浏览器已经知道那些网页中有哪些节点，各个节点的css定义和以及它们的从属关系，从而计算出每个节点在屏幕中的位置。
④绘制render树：按照计算出来的规则，通过显卡把内容画在屏幕上。
css样式解析到显示至浏览器屏幕上就发生在②③④步骤，可见浏览器并不是一获取到css样式就立马开始解析而是根据css样式的书写顺序将之按照dom树的结构分布render样式，完成第②步，然后开始遍历每个树结点的css样式进行解析，此时的css样式的遍历顺序完全是按照之前的书写顺序。在解析过程中，一旦浏览器发现某个元素的定位变化影响布局，则需要倒回去重新渲染正如按照这样的书写书序：
width: 100px;
height: 100px;
background-color: red ;
position: absolute;
当浏览器解析到position的时候突然发现该元素是绝对定位元素需要脱离文档流，而之前却是按照普通元素进行解析的，所以不得不重新渲染，解除该元素在文档中所占位置，然而由于该元素的占位发生变化，其他元素也可能会受到它回流的影响而重新排位。最终导致③步骤花费的时间太久而影响到④步骤的显示，影响了用户体验。
所以规范的的css书写顺序对于文档渲染来说一定是事半功倍的！
扩展：还有一个会影响浏览器渲染性能的词汇“repaint（重绘）”
repaint（重绘）：改变某个元素的背景色、文字颜色、边框颜色等等不影响它周围或内部布局的属性时，屏幕的一部分要重画，但是元素的几何尺寸没有变。
注意：
a.render树的结构不等同于DOM树的结构，一些设置display:none的节点不会被放在render树中，但会在dom树中。
b.有些情况，比如修改了元素的样式，浏览器并不会理科reflow或repaint，而是把这些操作积攒一批，然后做一次reflow，这也叫做异步reflow.但在有些情况下，比如改变窗口，改变页面默认的字体等，对于这些情况，浏览器会马上进行reflow.
c.为了更好的用户体验，渲染引擎将会尽可能早的将内容呈现到屏幕上，并不会等到所有的html都解析完成之后再去构建和布局render树。它是解析完一部分内容就显示一部分内容，同时，可能还在通过网络下载其余内容。

四.常用的CSS命名:
1)页面结构
容器: container
页头：header
内容：content
页面主体：main
页尾：footer
导航：nav
侧栏：sidebar
栏目：column
页面外围控制整体佈局宽度：wrapper
左右中：left right center
(2)导航
导航：nav
主导航：mainnav
子导航：subnav
顶导航：topnav
边导航：sidebar
左导航：leftsidebar
右导航：rightsidebar
菜单：menu
子菜单：submenu
标题: title
摘要: summary
(3)功能
标志：logo
广告：banner
登陆：login
登录条：loginbar
注册：register
搜索：search
功能区：shop
标题：title
加入：joinus
状态：status
按钮：btn
滚动：scroll
标籤页：tab
文章列表：list
提示信息：msg
当前的: current
小技巧：tips
图标: icon
注释：note
指南：guild
服务：service
热点：hot
新闻：news
下载：download
投票：vote
合作伙伴：partner
友情链接：friendlink
版权：copyright
