title: 手把手和你用原生JS写一个循环播放图片轮播
tags:
  - JavaScript
id: 331
categories:
  - JavaScript-demo
date: 2017-06-1 20:18:00
---
前段时间学习了淘宝首页的静态页面，其中收获较大的的就是这个**循环**播放的图片轮播组件，本文就将相关制作经验分享给大家。

先看看在线DEMO：[原生JS循环播放图片轮播组件](http://juniortour.net/demo/standard-js-carousel/standard-js-carousel.html) （支持IE8+，**本文中的在线demo均未经过压缩，可以直接在浏览器中调试**）

以及GitHub仓库地址及完整代码：[JuniorTour/simple-standard-js-carousel](https://github.com/JuniorTour/simple-standard-js-carousel)

<!--more-->

### 一、思路讲解:

#### 1.先说基本的非循环无过渡图片轮播：

这个思路还是很简单的，通过观察一些图片轮播就可以发现，图片轮播一般是以一个尺寸较小的父元素作为窗口，包裹住一组较长的长条状的项目（item）子元素，再利用` overflow: hidden; `，将父元素作为“窗口”，只显示出的项目子元素的一部分，并通过改变项目子元素的定位或translate3d属性，实现多张图片项目动态播放。

**基本原理可以参考这个demo：[图片轮播基本原理演示](http://juniortour.net/demo/standard-js-carousel/basic-theory-demonstration.html)**

#### 2.比较有意思的其实是**循环**的功能：

但是这样简单的轮播是不会循环播放的，也就是说当一轮图片项目（item）播放到结尾；或者当在第一张图（第一个项目）继续向前时，就会超出内容子元素，出现空白部分，这一般不是我们想要的结果。

有多种思路可以实现循环播放，我观察到淘宝网首页的图片轮播是这样的思路：

>复制开头和结尾的项目，并分别放在开头和结尾，当播放到开头或结尾的项目，继续播放，需要循环时，临时取消transition属性，并立即用定位跳转至相应的真正的开头或结尾之后，再恢复原来的transition，继续正常滚动播放，从而利用视觉上的“欺骗”，实现带有过渡效果的循环播放。

**相应的原理可以参考这个demo：[图片轮播循环原理演示](http://juniortour.net/demo/standard-js-carousel/loop-theory-demonstration.html)**

### 二、HTML标记部分

核心理念是简洁、语义化。这部分因为我学过bootstrap框架所以借鉴了[bootstrap的HTML标记结构](http://v3.bootcss.com/javascript/#carousel)。

整体结构为：

外层的`.carousel-wrapper`包裹着轮播的三个主要部分，分别是：

`.carousel-item-wrapper`：项目内容部分（作为演示，本文中的demo使用a标签代替了图片，大家可以自行尝试替换为图片；同时添加了文字序号标记，以便于观察理解，**尤其要注意两个复制的开头和结尾项目copy-1和copy-5）**。

`.carousel-control-wrapper`：控制按钮部分，即两个用于控制左右移动的按钮。

`.carousel-index-wrapper`：索引按钮部分，即图片轮播中的那一排“小圆点”。为了便于用JS操控，我添加了id作为“钩子”。而bootstrap在这里用的是自定义的data属性。

``` html
<div class="carousel-wrapper">
    <div class="carousel-item-wrapper" style="left: -520px;">
        <div class="carousel-item">
            <a href="#">
                <!--作为演示，用a标签代替了图片。-->
                <!--<img src="img/carousel-img-5" alt="">-->
            </a>
            <div class="carousel-index-mark">
                copy-5
            </div>
        </div>
        <div class="carousel-item">
            <a href="#">
                <!--<img src="img/carousel-img-1" alt="">-->
            </a>
            <div class="carousel-index-mark">
                1
            </div>
        </div>
        <div class="carousel-item">
            <a href="#">
                <!--<img src="img/carousel-img-2" alt="">-->
            </a>
            <div class="carousel-index-mark">
                2
            </div>
        </div>
        <div class="carousel-item">
            <a href="#">
                <!--<img src="img/carousel-img-3" alt="">-->
            </a>
            <div class="carousel-index-mark">
                3
            </div>
        </div>
        <div class="carousel-item">
            <a href="#">
                <!--<img src="img/carousel-img-4" alt="">-->
            </a>
            <div class="carousel-index-mark">
                4
            </div>
        </div>
        <div class="carousel-item">
            <a href="#">
                <!--<img src="img/carousel-img-5" alt="">-->
            </a>
            <div class="carousel-index-mark">
                5
            </div>
        </div>
        <div class="carousel-item">
            <a href="#">
                <!--<img src="img/carousel-img-1" alt="">-->
            </a>
            <div class="carousel-index-mark">
                copy-1
            </div>
        </div>
    </div>
    <div class="carousel-control-wrapper">
        <button id="prev">
            <!--prev-->
            <i>&lt;</i>
        </button>
        <button id="next">
            <!--next-->
            <i>&gt;</i>
        </button>
    </div>
    <div class="carousel-index-wrapper">
        <ul>
            <li class="carousel-index-btn active-carousel-index-btn" id="carousel-to-1">carousel-index-1</li>
            <li class="carousel-index-btn" id="carousel-to-2">carousel-index-2</li>
            <li class="carousel-index-btn" id="carousel-to-3">carousel-index-3</li>
            <li class="carousel-index-btn" id="carousel-to-4">carousel-index-4</li>
            <li class="carousel-index-btn" id="carousel-to-5">carousel-index-5</li>
        </ul>
    </div>
</div>
```

### 三、CSS样式部分
总的来说比较简单，重要的地方我加上了注释，有存疑的地方，欢迎和我交流。

``` css
    /*reset*/
    * {
        border: none;
        padding: 0;
        margin: 0;
    }
    button {
        outline: none;
    }
    li {
        list-style: none;
    }

    .carousel-wrapper {
        width:520px;
        height:280px;
        overflow: hidden;   /*关键*/
        position: relative;
        margin: 100px auto;
    }
    .carousel-item-wrapper {
        width:3640px;
        height:280px;
        position: absolute;
        top: 0;
        left: -520px;
        transition: left .2s ease-in;
    }
    .carousel-item a {
        display: block;
        background-color: red;
        width:520px;
        height: 280px;
    }

    /*使用不同背景色的a替代图片。*/
    .carousel-item:nth-child(1) a {
        background-color: rgb(129,194,214);
        /*第五张图片的复制*/
    }
    .carousel-item:nth-child(2) a {
        background-color: rgb(129,146,214);
    }
    .carousel-item:nth-child(3) a {
        background-color: rgb(217,179,230);
    }
    .carousel-item:nth-child(4) a {
        background-color: rgb(220,247,161);
    }
    .carousel-item:nth-child(5) a {
        background-color: rgb(131,252,216);
    }
    .carousel-item:nth-child(6) a {
        background-color: rgb(129,194,214);
    }
    .carousel-item:nth-child(7) a {
        background-color: rgb(129,146,214);
        /*第一张图片的复制*/
    }

    .carousel-item {
        float: left;
    }
    .carousel-index-mark {
        font-size:60px;
        color: black;
        position: absolute;
        top: 0;
    }
    .carousel-control-wrapper {
        transition: all .2s;
    }
    .carousel-wrapper:hover button {
        display: block;
    }
    .carousel-control-wrapper button {
        transition: all .2s linear;
        display: none;
        width:24px;
        height:36px;
        line-height:36px;
        background-color: rgba(0,0,0,.3);
        color: #fff;
        position: absolute;
        top: 50%;
        cursor: pointer;
    }
    button#prev {
        left:0;
    }
    button#next {
        right:0;
    }
    button i {
        font-size: 18px;
    }
    .carousel-index-wrapper {
        width:65px;
        height:13px;
        overflow: hidden;
        position: absolute;
        bottom:15px;
        left:50%;
        margin-left: -33px;
    }
    .carousel-index-btn {
        width:9px;
        height:9px;
        float: left;
        margin:2px;
        background-color: #b7b7b7;
        border-radius: 50%;
        text-indent: -999em;
        /*这个-999em的文字对齐声明有助于增强可访问性。*/
        cursor: pointer;
    }
    .active-carousel-index-btn {
        background-color: #f44103;
    }
```

### 四、JS部分
这一块是主要部分，内容较多，因此我们逐步来实现各部分功能以便于理解。
#### 0.功能和结构分析：
根据最开始的思路讲解，我们把这个轮播的JavaScript功能大致分为以下4个部分：
1.左右滑动按钮功能
2.索引按钮跳转功能
3.自动播放功能
4.循环播放功能。

我们来分别逐步实现。

#### 1.实现左右滑动按钮功能：
``` javascript
function addLoadEvent(func) {
    var oldLoad = window.onload;
    if (typeof oldLoad != 'function') {
        window.onload = func;
    } else {
        window.onload = function () {
            oldLoad();
            func();
        }
    }
}
//给文档加载完成后的load事件绑定相应的处理函数：
addLoadEvent(preventDefaultAnchors);
addLoadEvent(carouselControl);

/*用一个对象把轮播组件的相关参数封装起来，优点是灵活便于扩展升级；缺点是同时也增加了文件的体积。*/
var carouselInfo = {
    itemWidth: 520,
    trueItemNum: 5,
    itemNum: 7,
    totalWidth: 7 * 520
};

//阻止a标签默认的点击跳转行为
function preventDefaultAnchors() {
    var allAnchors = document.querySelectorAll('a');

    for (var i = 0; i < allAnchors.length; i++) {
        allAnchors[i].addEventListener('click', function (e) {
            e.preventDefault();
        }, false);
    }
}

function carouselControl () {
    var prev = document.querySelector("#prev");
    var next = document.querySelector("#next");
    var carouselWrapper = document.querySelector(".carousel-wrapper");

    prev.onclick = function () {
        slide(-1);
    };
    next.onclick = function () {
        slide(1);
    };
}

function slide(slideItemNum) {
    var itemWrapper=document.querySelector(".carousel-item-wrapper");
    var currentLeftOffset=(itemWrapper.style.left)?parseInt(itemWrapper.style.left): 0,
        targetLeftOffset=currentLeftOffset-(slideItemNum*carouselInfo.itemWidth);

    itemWrapper.style.left=targetLeftOffset+'px';
}
```

第1步的demo：[carousel-step-1](http://juniortour.net/demo/standard-js-carousel/carousel-step-1.html)

#### 2.实现索引按钮跳转功能：
``` javascript
function carouselControl() {
    var prev = document.querySelector("#prev");
    var next = document.querySelector("#next");
    var carouselWrapper = document.querySelector(".carousel-wrapper");
    //添加索引按钮的引用
    var indexBtns = document.querySelectorAll(".carousel-index-btn");

    //标记当前所在的图片编号，用于配合控制.index-btn。
    var currentItemNum = 1;
    prev.onclick = function () {
        //把滑动功能和切换索引按钮功能装入一个函数之中，以便于获取当前索引：
        currentItemNum=prevItem(currentItemNum);
    };
    next.onclick = function () {
        //把滑动功能和切换索引按钮功能装入一个函数之中，以便于获取当前索引：
        currentItemNum=nextItem(currentItemNum);
    };

    for (var i = 0; i < indexBtns.length; i++) {
        //利用立即调用函数，解决闭包的副作用，传入相应的index值
        (function (i) {
            indexBtns[i].onclick = function () {
                slideTo(i+1);
                currentItemNum=i+1;
            }
        })(i);
    }
}

function nextItem(currentItemNum) {
    slide(1);
    currentItemNum += 1;
    if (currentItemNum == 6) currentItemNum = 1;
    switchIndexBtn(currentItemNum);

    return currentItemNum;
}

function prevItem(currentItemNum) {
    slide(-1);
    currentItemNum -= 1;
    if (currentItemNum == 0) currentItemNum = 5;
    switchIndexBtn(currentItemNum);

    return currentItemNum;
}

//添加直接跳转函数：
function slideTo(targetNum) {
    var itemWrapper=document.querySelector(".carousel-item-wrapper");
    itemWrapper.style.left=-(targetNum*carouselInfo.itemWidth)+'px';
    switchIndexBtn(targetNum);
}

function slide(slideItemNum) {
    var itemWrapper = document.querySelector(".carousel-item-wrapper");
    var currentLeftOffset = (itemWrapper.style.left) ? parseInt(itemWrapper.style.left) : 0,
        targetLeftOffset = currentLeftOffset - (slideItemNum * carouselInfo.itemWidth);

    itemWrapper.style.left = targetLeftOffset + 'px';
}

function switchIndexBtn(targetNum) {
    //切换当前的索引按钮
    //删除过去激活的.active-carousel-index-btn
    var activeBtn=document.querySelector(".active-carousel-index-btn");
    activeBtn.className=activeBtn.className.replace(" active-carousel-index-btn","");

    //添加新的激活索引按钮
    var targetBtn=document.querySelectorAll(".carousel-index-btn")[targetNum-1];
    targetBtn.className+=" active-carousel-index-btn";
}
```

第2步的demo：[carousel-step-2](http://juniortour.net/demo/standard-js-carousel/carousel-step-2.html)

#### 3.实现自动播放功能：
``` javascript
function carouselControl() {
    //省略前面重复的代码......

    for (var i = 0; i < indexBtns.length; i++) {
        //利用立即调用函数，解决闭包的副作用，传入相应的index值
        (function (i) {
            indexBtns[i].onclick = function () {
                slideTo(i+1);
                currentItemNum=i+1;
            }
        })(i);
    }

    //添加定时器
    var scrollTimer;
    function play() {
        scrollTimer=setInterval(function () {
            currentItemNum=nextItem(currentItemNum);
        },2000);
    }
    play();

    function stop() {
        clearInterval(scrollTimer);
    }

    //绑定事件
    carouselWrapper.addEventListener('mouseover',stop);
    carouselWrapper.addEventListener('mouseout',play,false);

    /*DOM二级的addEventListener相对于on+sth的优点是：
     * 1.addEventListener可以先后添加多个事件，同时这些事件还不会相互覆盖。
     * 2.addEventListener可以控制事件触发阶段，通过第三个可选的useCapture参数选择冒泡还是捕获。
     * 3.addEventListener对任何DOM元素都有效，而不仅仅是HTML元素。*/
}
```

第3步的demo：[carousel-step-3](http://juniortour.net/demo/standard-js-carousel/carousel-step-3.html)

#### 4.关键点：实现循环播放功能：
``` javascript
function slide(slideItemNum) {
    var itemWrapper = document.querySelector(".carousel-item-wrapper");
    var currentLeftOffset = (itemWrapper.style.left) ? parseInt(itemWrapper.style.left) : 0,
        targetLeftOffset = currentLeftOffset - (slideItemNum * carouselInfo.itemWidth);

    /*不在这里跳转了。先处理偏移值，实现循环，再跳转。*/
    //itemWrapper.style.left = targetLeftOffset + 'px';

    switch (true) {
            /*switch 的语法是：当case之中的表达式等于switch (val)的val时，执行后面的statement（语句）。*/
        case (targetLeftOffset>0):
            itemWrapper.style.transition="none";
            itemWrapper.style.left=-carouselInfo.trueItemNum*carouselInfo.itemWidth+'px';
            /*此处即相当于：itemWrapper.style.left='-2600px';*/
            targetLeftOffset=-(carouselInfo.trueItemNum-1)*carouselInfo.itemWidth;
            //相当于：targetLeftOffset=-2080;
            break;
        case (targetLeftOffset<-(carouselInfo.totalWidth-carouselInfo.itemWidth)):
            //此处即相当于：targetLeftOffset<-3120
            itemWrapper.style.transition="none";
            itemWrapper.style.left=-carouselInfo.itemWidth+'px';
            //相当于：itemWrapper.style.left='-520px';
            targetLeftOffset=-carouselInfo.itemWidth*2;
            //相当于：targetLeftOffset=-1040;
            break;
    }

    /*这里我使用了setTimeout(fn,0)的hack
     * 参考bootstrap的carousel.js源码，似乎也利用了setTimeout(fn,0)这一hack。
     *
     * stackoverflow上有对这一hack的讨论和解释：
     * http://stackoverflow.com/questions/779379/why-is-settimeoutfn-0-sometimes-useful
     * 根据第二个回答，我个人的理解是：setTimeout(fn,0)相当于异步执行内部的代码fn，
     * 具体到这个轮播，就是在上一轮非过渡定位的页面渲染工作（switch语句内部的case）结束之后，再执行setTimeout内部的过渡位移工作。
     * 从而避免了，非过渡的定位还未结束，就恢复了过渡属性，使得这一次非过渡的定位也带有过渡效果。
     **/

    //各位可以试一试，把setTimeout内部的代码放在外部，“循环”时会有什么样的错误效果。
    //itemWrapper.style.transition="left .2s ease-in";
    //itemWrapper.style.left=targetLeftOffset+'px';

    setTimeout(function () {
        itemWrapper.style.transition="left .2s ease-in";
        itemWrapper.style.left=targetLeftOffset+'px';
    },20);
}
```

第4步的demo：[carousel-step-4](http://juniortour.net/demo/standard-js-carousel/carousel-step-4.html)

至此，就完成了一个完整的循环播放图片轮播，欣赏一下自己的杰作吧~~~ヾ(✿ﾟ▽ﾟ)ノ

### 五、源码及示例：

#### 1.GitHub仓库地址及完整代码：[JuniorTour/simple-standard-js-carousel](https://github.com/JuniorTour/simple-standard-js-carousel)

#### 2.在线demo：[原生JS循环播放图片轮播组件](http://juniortour.net/demo/standard-js-carousel/standard-js-carousel.html) 

很惭愧，只做了一点简单的工作。如果觉得本文不错的话，欢迎给[我的GitHub](https://github.com/JuniorTour/simple-standard-js-carousel)点赞！