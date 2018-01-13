# WeChatJumpPlugin

微信新版本带来的跳一跳小游戏大火了一阵，随后引各行各业高手来刷分，[点击查看八仙们是怎么过海的](https://m.toutiao.com/i6507191239606010371/?iid=22640951143&app=news_article&timestamp=1515124765&tt_from=weixin_moments&utm_source=weixin_moments&utm_medium=toutiao_ios&utm_campaign=client_share&wxshare_count=3&from=singlemessage&isappinstalled=0&pbid=6507437914757006861)。

于是再下也不甘寂寞，抄起手中的老鼠和机器人写了go+Android版本外挂。  
[go部分源码](https://github.com/xmh19936688/WeChatJumpPlugin-go)  
[Android部分源码](https://github.com/xmh19936688/WeChatJumpPlugin-Android)

## 大致思路

安卓端做一个透明window浮在小游戏上面，在window上之处player应该如何jump，然后发一个get请求到go端，计算后得出的按压时长通过adb命令发送到手机。

## 所用技术

### Android

调用`WndowManager.addView(View,WindowManager.LayoutParams)`向界面添加一个悬浮窗。  
悬浮窗的view中有一个宽高均为`match_parent`的view，主要是为其设置`OntouchListener`来让用户“说明”player应该怎么跳。  
随后Android端将数据（接受点击事件的坐标和jump距离）发送到go端。  
注意悬浮窗不能完全覆盖小游戏，不然不会有任何点击事件都会被悬浮窗拦截，此处悬浮窗宽占满，高占2/3。  
发向go的“点击坐标”即（宽/2,高*5/6）。

### go

用一个`map[int64]int64`来记录每一次jump的距离和按压时长;一个`[]int64`有序数组来记录历史距离。  
每次收到距离请求时首先去map里看有没有记录，如果有就直接返回。  
如果没有，则根据有序数组找出最近的两个距离，拿map根据相似三角形计算大致按压时长，随后返回并记录到map与array。  

### 数据校正

go计算得出的按压时长不一定准确，这就需要反馈结果。  
Android端向go发送jump数据之后，go通过adb向游戏发送按压事件，随后Android在悬浮窗标记落地位置与目标位置的便宜（拖拽小人到落地位置，如果调准了则点击一下屏幕表示校正值为0）。  
go端接收到校正值后调整“上一次”的数据并保存。

### 数据累计

看到这里应该能明白，这种程序“越跳越准”，勉强叫做“学习”吧。  
每次程序退出时，将map与array保存到文件，每次启动时，再从文件加载。

## 为什么放弃

程序写到这就算完成了，但是本人在实际用的时候发现一个问题。游戏原本设计难度在于“精确控制按压时长”，而本程序为了保证数据准确性，则要求“精确控制触屏位置”，不然无法获取正确数据，也就不能“好好学习”，当然也就“指挥”不准了。在测试过程中，用触控笔不断“调教”程序，尚不能保证完美点在中心点，何况是粗大的手指。而外挂之所以叫外挂就是降低“成本”提高“收益”，本程序只是转移了“成本”，所以后续的优化（比如map与array的组织方式、距离计算等）一并放弃……
