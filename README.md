# dingdong-helper
叮咚自动下单 并发调用接口方式 多人实战反馈10秒以内成功 自动将购物车能买的商品全部下单 只需自行编辑购物车和最后支付即可

不考虑分批多次使用优惠券下单（会选择一张推荐优惠券），当前情形下能下单就很好了，所以我会将购物车里所有能买的东西一次下单，博最高的概率买到东西才是正道

# 特别强调
1. 不要长时间运行程序，目前已知有人被风控了，暂时未确认风控的因素是ip还是用户或设备相关信息，如果要测试用单次执行模式，并发只能用于6点、8点半的前一分钟，然后执行时间不能超过2分钟，如果买不到就不要再执行程序了，切忌切忌，如果已经被风控的可以尝试改一下ip，或者换号

## 注意事项

1. 如果在6点和8点30（进APP看公告有写开放时间）没买到不要一直跑程序，长时间并发接口会被对方拉黑到IP黑名单中，接口返回405。
3. 根据反馈有少部分人的站点是假库存，可能是怕大家闹事，开放购买之前能看到购物车里有，但是根本就不可能买到，第一秒下单瞬间很多东西就没了，我也是，几百块的购物车最后下单几十块，我同时用app人工操作了购物车确实是没货了，不是程序问题。

## 快捷抓包

忘记小程序已经有PC版了，手机进入小程序右上角3个点->在电脑中打开即可，送上一个参考文章https://blog.csdn.net/z2181745/article/details/123002569 比手机抓包方便太多。

注意事项
1. Charles安装和配置好后再打开或重新打开电脑端叮咚小程序，如果在之前打开可能会抓不到
2. 如果使用电脑端小程序抓包，则不要去碰手机微信里的叮咚小程序，否则session会失效，反过来也一样，其他操作在app上操作不影响，但不能同时在两个端的小程序操作，互斥

## 步骤

1. 通过Charles（我截图和教程是Charles，用Charles会更方便对比）等抓包工具抓取微信中叮咚买菜小程序中的接口信息中的用户信息配置到UserConfig.java中，比如openId、userId，详情见下截图，此操作每个用户只需要做一次。注意事项：其中有一个参数叫station_id，headers和body中都有，一定要确认抓包时你的站点信息设置是对的（非常重要 非常重要 非常重要），进入小程序后左上角确认站点信息后再抓包
2. 运行UserConfig.java获取addressId填入addressId变量
3. 将需要买的菜自行通过APP放入购物车并 打开全选按钮（一定要打开 非常重要 非常重要 非常重要）
4. 等待叮咚开放购买前1分钟运行Application.java，如不出意外到时间点后10秒以内会提交成功，该程序会自行更新购物车（新增或者无货）和配送时间
5. 等待程序结束，如果成功则自行打开叮咚买菜app-我的订单-待支付-点击支付
6. 每次抢之前跑一下UserConfig中的main方法确认登录状态是否准确，如果状态不对则重新抓包更新UserConfig数据
7. 如果想用自己的号帮别人下单，只需要在小程序中选择对方所在的站点和设置默认地址，然后抓包获取到station_id替换UserConfig中station_id（注意headers和body中都有），重新跑一次UserConfig获取addressId并替换


## 程序自动结束的几个条件

1. 购物车无可购买商品
2. 下单成功
3. 用户登录信息失效
4. 无配送时间还会继续，为了考虑在叮咚开放时间前提前打开程序，如果反复确认不能配送了就自己停止服务吧

## 设备分工（同上面的步骤，把设备关系说清楚）

#### 手机&电脑

1. 打开微信小程序中的叮咚买菜，通过电脑抓包软件抓取信息填入代码中，在token不失效的情况下可以一直使用
2. 也可以使用电脑端小程序进行抓包会比手机方便很多

#### 手机

1. 在开放购买前选择商品到购物车并打开全选按钮（一定要打开 非常重要 非常重要 非常重要）
2. 等待下单成功后去待支付订单页面支付

#### 电脑

1. 运行UserConfig获取addressId并填入变量addressId，如果地址不变只需要运行一次
2. 开放购买前1分钟运行Application，前30秒左右会获取基本信息直到看到提交订单失败的信息则代表基本信息获取完毕等待开放时间一到即可成功

## 思路

虽然我家吃的很多，但是时间长了也受不了这几天每天早上起来抢菜，手都点抽经了都买不到，看着购物车里的菜越来越少心急如焚，作为程序员只能靠自己的双手了，吃完午饭开干，晚上6点成功下单
1. 抓app的包没抓到
2. 抓小程序的包可以，但是小程序无法做登录，拿不到open id，所以只能通过自行抓包解决。另看到请求参数中有一些签名字段，心想麻烦哟
3. 准备研究如何签名，解包微信小程序，初步研究签名相关代码，搞不定就去研究app hook，但那耗费精力太大，留着当后手
4. 先写一个获取地址的请求，发现那几个看着像签名的参数可以不用传，省了一大笔精力，应该一开始就用Charles的breakpoint删除参数再repeat尝试无签名是否可访问，被唬住了，早知道就可以省略步骤3
5. 梳理下单需要的参数和步骤，数据量非常庞大，眼睛都看晕了，需要细心
6. 看到下单成功很开心，这就是乐趣

最后希望疫情早日结束大家伙都能吃上饭

## 更新记录

### 2022.04.11 
1. 新增自动勾选购物车
2. 优化请求量过大和持续时间过长被网关拦截提示
3. 执行UserConfig时新增站点信息确认，如站点信息错误将导致购物车在手机上看有货程序执行无货或无法下单

## 抓包截图 将你的信息填入

这个图有时候会挂，直接从项目里面看也一样，就是路径image/headers.jpeg 和 body.jpeg  对应到UserConfig中的headers和body方法里的参数
![请求头信息](https://github.com/JannsenYang/dingdong-helper/blob/f6e20d377aa482063732a5be614e3dae3d4c5091/image/headers.jpeg)
![请求体信息](https://github.com/JannsenYang/dingdong-helper/blob/f6e20d377aa482063732a5be614e3dae3d4c5091/image/body.jpeg)

## 20220410实战记录

用了的全部秒抢，我自己傻逼了，为了提交github，收货地址id在运行的时候忘记填了，跑了几分钟才后知后觉，随即补上了失败时的返回信息。
![实战记录1](https://github.com/JannsenYang/dingdong-helper/blob/2c61ce883a7092efb93481253083b79ea66c62fe/image/20220410-1.png)
![实战记录2](https://github.com/JannsenYang/dingdong-helper/blob/f6e20d377aa482063732a5be614e3dae3d4c5091/image/20220410-2.png)


