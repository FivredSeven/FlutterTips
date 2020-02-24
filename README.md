# FlutterTips
#### for循环添加widget
在一个Container内部，如果要加十个类似的Container，不用费劲地连续写。    
以下为伪代码：    

``Widget widget;
List<Widget> list = new List();
for (var i=0;i<10;i++) {
  list.add(Container(Text(i)));
}
widget = new Column(
  children: list
)
return widget;
``
#### 监听加载网络图片失败
``getNetworkImage(ClassifyItem item) {
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
``
