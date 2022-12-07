
# 一、反编译获取河北健康码源码
通过反编译工具获取河北健康码源代码在上一篇文章中讲到过，这里就不在过多讲解!
[关于微信小程序反编译获取源码](https://blog.csdn.net/wds326598/article/details/115519922)
# 二、解决报错问题
获取到源代码以后，目录结构以及编译结果,如图
![请添加图片描述](https://img-blog.csdnimg.cn/3a47c675f6c146b7af3894564a8010b0.png)

## 1.解决以 '__' 为开头和结尾的目录为保留目录。__plugin__ 目录下的所有文件将会被忽略。
次问题只需要删除目录中__plugin__文件即可!

## 2.解决TypeError: _typeof3 is not a function
此问题为反编译经典错误，其解决方案:
在目录中找@babel-runtime-helpers-typeof.js 把里面的代码全部删掉~
![请添加图片描述](https://img-blog.csdnimg.cn/be9b1ffdf51c43ca981dd1f9e278335a.png)
**换成以下代码**
```javascript
function _typeof2(o) {
  "@babel/helpers - typeof";
  return (_typeof2 = "function" == typeof Symbol && "symbol" == typeof Symbol.iterator ? function(o) {
      return typeof o;
  } : function(o) {
      return o && "function" == typeof Symbol && o.constructor === Symbol && o !== Symbol.prototype ? "symbol" : typeof o;
  })(o);
}
 
 
function _typeof(o) {
  return "function" == typeof Symbol && "symbol" === _typeof2(Symbol.iterator) ? module.exports = _typeof = function(o) {
      return _typeof2(o);
  } : module.exports = _typeof = function(o) {
      return o && "function" == typeof Symbol && o.constructor === Symbol && o !== Symbol.prototype ? "symbol" : _typeof2(o);
  }, _typeof(o);
}
 
 
module.exports = _typeof;
```
至此编译顺利通过!!!
![请添加图片描述](https://img-blog.csdnimg.cn/ab2d16e0c82c4c309665f8c4fae23efb.png)
#   三、对源代码进行调试
##  1.去掉首页未登录或者各种实名认证的弹框
全局搜索'去登录'关键字找到弹框的位置，注释掉即可!
以及全局搜索'当前网络繁忙，请稍后重试'弹框，注释掉即可!
这个比较简单就不在截图了
##  2.对首页Index进行分析
打开pages中的index文件
我们从上往下分析寻找源码
### <1.首先我们需要找到的是姓名与身份证号,通过在index搜索或者用开发者工具定位，可以在源码中定位到姓名与身份证号
![请添加图片描述](https://img-blog.csdnimg.cn/0b33ed98a95447368e3292c901ba91f4.png)

名字用的字段是name,身份证号用的字段是identity
也就是说我们只需要修改这2个值，便可以更改首页中的姓名与身份证号

###  <2.寻找绿码
我们顺着代码继续往下寻找，会发现健康码的顶部皇冠图片，但是皇冠图片是通过wx:if="{{fxdjflag!=20&&!newflag&&severalvaccine&&severalvaccine.num>=severalvaccine.maxNum}}"语句来判断是否显示,我们需要把该句代码注释掉,皇冠图片即可显示出来
同时继续往下查看会发现绿码旁边的边框，我们依旧注释或者删除掉他的判断语句wx:if，即可显示出来
![请添加图片描述](https://img-blog.csdnimg.cn/c813edcbe9d049f5b3359aae130f3326.png)
>说明:
> wx:if条件判断语句
>只有满足wx:if= ""里面的判断，此语句才能生效
>以边框图片中的判断条件为例：
>wx:if="{{fxdjflag!=20&&!newflag&&severalvaccine&&severalvaccine.num>=4}}"
>fxdjflag:风险等级   severalvaccine:代表接种疫苗的次数
>也就是只有风险等级!=20并且接种疫苗次数大于4，才显示这个图片

接下来是注释掉条件判断语句的代码:

![请添加图片描述](https://img-blog.csdnimg.cn/dfc467639dca40c7b96e27d592d07f78.png)
完成上述步骤以后，接下来就是二维码，我们大家都知道二维码区分黄码，绿码，红码，从源代码中我们可以看到他是通过参数imgData获取并且展示的。接下来那么我们去js文件中寻找imgData参数，看看他是怎么获取，怎么使用的
![请添加图片描述](https://img-blog.csdnimg.cn/efdfe2bf8f804f8383ba93dbe1ad3958.png)
通过inde.js源码分析可知：
imgData是由m构成! 
m是由一个方法h传递2个参数构成
其中第一个参数是t.data.imgStream
另一个参数是键值对形式{}
而我们发现{}中有color是通过f传递，而f有3种颜色分别是:f = "#39b778"  f = "#f5d000"  f = "#d8242b",那么我们把这3种颜色分别转换一下
![请添加图片描述](https://img-blog.csdnimg.cn/afbf54bdc4004561876a08200f562f07.png)
![请添加图片描述](https://img-blog.csdnimg.cn/c1ab8dcc6b8d42cfb90ee6dd7859279e.png)
![请添加图片描述](https://img-blog.csdnimg.cn/b54995fbae91485192f4b27fff8fc18b.png)
颜色很明显，到这为止我们可以构建出一个键值对，直接把里面的color写死，写成绿色，只需要在寻找到t.data.imgStream参数我们便可以自己构建绿码
继续搜寻imgStream
![请添加图片描述](https://img-blog.csdnimg.cn/0975f1600b6e4541bbb612d604060c69.png)
通过源码可知，imgStream是通过API：/hbjkm/code/createCode获取的，通过单词的imgStream反义可知是照片流!
我们可以这样在网络上随意找一张图片，拷贝其地址当做imgSteam
这样有imgSteam有键值对，我们可以构建m有了m我们可以构建imgData,就可以生成绿色二维码
如图
![请添加图片描述](https://s2.loli.net/2022/12/07/YK5wcEohBL2vJTp.png)

需要注意的是我们需要再onShow方法中构建

###  <2.寻找核酸检测时间
核酸检测时间目前我们已知有24小时，48小时，72小时，4天 ，5天
我们继续在index.wxml源码中寻找:
![请添加图片描述](https://img-blog.csdnimg.cn/087ecb33ad3c459094b850a9e64cb937.png)
通过源码分析可知:
内容展示参数为hsDay(初步猜测字段含义可能是小时天)
hsDay如果是0,1,2就显示单位小时，否则就显示单位天
然后时间展示为hsjcsj(初步猜测为核酸检测时间)
带着疑问前去js寻找
我们在js中发现这样一段代码:
![请添加图片描述](https://img-blog.csdnimg.cn/5dc2e14275a54fb9883f43309b637f21.png)

通过源码查看我们可以大概猜测到hsDay如果是0，那么就会在hsObj中取出24，如果hs是1那么就会是48，
带着疑问我们输入0看看是不是24小时
![请添加图片描述](https://s2.loli.net/2022/12/07/YK5wcEohBL2vJTp.png)

大家也都看到了，接下来还差一个检测时间:通过上一步分析我们已经知道检测时间参数是hsjcsj,
那我们就在onShow中构建随意输入一个看看情况!
![请添加图片描述](https://s2.loli.net/2022/12/07/soZhtLIODcbR9F1.png)


到这，我们对源代码首页中一些重要的信息进行了分析和修改，后期我们甚至可以加入我们自己的代码，在代码中进行手动的自定义设置核酸时间，设置检测时间，以及检测机构等各种信息~
#    总结
此文章仅限于对反编译教程的学习交流使用,并不会用于其它各种用途，本人也不上传任何形式的源代码。
