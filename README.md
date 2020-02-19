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
####
