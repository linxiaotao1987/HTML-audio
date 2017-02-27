# HTML5音频插件的踩坑记录

场景和问题描述：做ME课程详情页时需要做一个老师讲课的音频播放。

1、进入页面的时候需要获取到音频的长度以展示给用户；

2、用户点击播放开始load音频资源，然后开始播放。播放中要展示缓冲进度；

3、拖拽或点击音频进度条可以前进或后退；


实现这个需求的第一个难点是进入页面后获得音频长度。
在ios中进入页面是不会去读音频的，所以音频的canplay事件不会触发，即使在标签上写明autoplay。
另外使用window.onload以及使用ajax请求后执行success函数来触发audio.play()都是无效的。
但是，如果是在微信端，那么可以利用WeixinJSBridgeReady事件。
在html头部加入监听这个事件的代码，以保证在这个事件触发前已经完成监听。
```
 document.addEventListener("WeixinJSBridgeReady", function () {
            audio.play();
    }, false);
```
实践中发现，这个方法可以让ios和安卓都在进入页面之后自动播放音频。
然而因为ME项目用的是react做的单页面应用，所以没办法在head插入这段监听代码。
而在其他位置监听则不能保证100%成功，所以就放弃了这种解决方案。

另一个解决的方法是给页面绑定一个touchstart的事件：
```
let audioAutoPlay = ()=> {
            let play = function () {
                audio.play();
                audio.pause();
                document.removeEventListener("touchstart", play, false);
            };

            document.addEventListener("touchstart", play, false);
        };
        audioAutoPlay();
```
通过用户的交互行为来触发audio的播放事件。
虽然可以正常获取到，但并不是真正的进入页面就能读取。
由于找不到其他方法实现，只能和产品沟通下。

通过用户的touchstart事件执行audio.play()之后，会触发loadstart事件，这是时候给play按钮的状态加上loading的class。
load的过程中如果读取到音频的长度会触发loadedmetadata事件，这时候audio.duration的值就是音频的长度，单位是秒。

由于这里只需要获取到长度就可以，所以让audio暂停下来；

接下来是依次监听事件，api网址：http://www.cnblogs.com/top5/archive/2012/02/22/2362486.html

摘几个说一下：

progress 当load到数据时触发，用来显示loading的进度。audio.buffered.end(0)是缓冲的时间长度。

timeupdate 音频播放时间改变时触发，audio.currentTime返回当前的播放时间。由于在ios上需要load到相当长一段音频他才会开始播放，所以在timeupdate之前需要给用户loading提示。首先监听播放按钮的点击事件，点击之后加loading的class。然后执行audio.play()。然后监听timeupdate，事件触发之后再去掉loadingclass。
