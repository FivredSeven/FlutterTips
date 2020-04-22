# FlutterTips

#### flutter从二级页面退回到一级页面
正常来说我们在flutter二级页面要退回到一级页面，可以调用Navigator.of(context).pop()
但也有特殊情况
例如我们的ThemeData选择了TargetPlatform.iOS，因为这样能支持向右滑动页面退回到上一级页面。
这种情况下，回到上级页面并不需要调用Navigator.of(context).pop()。

#### flutter与java混合编译，加载黑屏的问题
debug包尤其明显，黑屏的时长甚至能达到2-3秒
解决办法：
* 1、flutter的父view初始化的时候设置为INVISIBLE
* 2、给flutterView设置addFirstFrameListener
* 3、在Listener的onFirstFrame()回调里，设置父view为VISIBLE

#### 使用Offstage遇到的坑
首先看一段简单的Offstage代码
```
Offstage(
    offstage: imageUrl == null || imageUrl.length() == 0,
    child: ElderNetworkImage(
      url: imageUrl,
      width: 100,
      height: 100,
    ),
  ),
```
补一下基础，属性里的offstage是当条件成立的时候，隐藏child里的View。
没错，它只是控制显示隐藏的。并不像其他语言中的if else.
当imageUrl=""满足offstage，不显示child，原以为就不会执行child，而我错了，它会创建ElderNetworkImage，并且执行它的initState()方法。



#### 加载网络图片公共控件
还是基于之前看到的ImageStreamListener
这次其实也不算是最终解，先说说一个好的网络图片加载组件的必要条件
* 基础1：加载监听，包括成功和失败
* 基础2：加载失败的情况下，要支持设置默认图
* 基础3：图片缓存-内存缓存
* 基础4：图片缓存-文件缓存（这个暂时还没处理，理论很简单，在加载成功后写入本地，之后有时间补上）

```

import 'package:flutter/material.dart';

class ElderNetworkImage extends StatefulWidget {
  ElderNetworkImage({@required this.url,
    this.width, this.height, this.defImagePath, this.roundCorner = 0, this.successCloseView});

  final String url; //图片地址
  final double width; //图片宽度
  final double height; //图片高度
  final String defImagePath; //加载失败时显示的默认图
  final double roundCorner; //支持圆角
  final Widget successCloseView;//右上角的关闭按钮（任意组合view）
  ......可以持续自定义

  @override
  State<StatefulWidget> createState() {
    return _StateElderNetworkImage();
  }
}

class _StateElderNetworkImage extends State<ElderNetworkImage> {
  Image _image;
  bool showClose = false;

  @override
  void initState() {
    super.initState();
    KGFlutterLog.d("ElderNetworkImage", "initState url:" + widget.url);
    _image = Image.network(
      widget.url,
      width: widget.width,
      height: widget.height,
    );
    var resolve = _image.image.resolve(ImageConfiguration.empty);
    resolve.addListener(ImageStreamListener((_, __) {
      KGFlutterLog.d("ElderNetworkImage", "加载成功");
      //加载成功
      setState(() {
        showClose = true;
      });
    }, onError: (dynamic exception, StackTrace stackTrace) {
      KGFlutterLog.d("ElderNetworkImage", "加载失败 exception：" + exception + "|stackTrace:" + stackTrace.toString());
      //加载失败
      setState(() {
        _image = Image.asset(
          widget.defImagePath,
          width: widget.width,
          height: widget.height,
        );
      });
    }));
  }

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: <Widget>[
        new ClipRRect(
          borderRadius: BorderRadius.circular(widget.roundCorner),
          child: _image,
        ),
        Offstage(
          offstage: !showClose,
          child: widget.successCloseView,
        ),
      ],
    );
  }
}

```

#### 监听加载网络图片失败
```
getNetworkImage(ClassifyItem item) {
  Image image = Image.network(item.image);
  final ImageStream stream = image.image.resolve(ImageConfiguration.empty);
  stream.addListener(new ImageStreamListener(_updateImage, onError: (dynamic error, StackTrace stackTrace) {
    print('ERROR caught by framework');
    setState(() {
      imageError = true;
    });
  },));
  return image;
}
```

#### for循环添加widget
在一个Container内部，如果要加十个类似的Container，不用费劲地连续写。
以下为伪代码：

```
Widget widget;
List<Widget> list = new List();
for (var i=0;i<10;i++) {
  list.add(Container(Text(i)));
}
widget = new Column(
  children: list
)
return widget;
```