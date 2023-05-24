# PlantUML
注意:在vscode中安装plantuml插件后，编辑plantuml时会提示有问题，这可能是插件问题导致的，实际预览是正常的，github不支持plantuml图预览
## 思维导图
表达发散性思维的有效图形思维工具

```plantuml
'以下是思维导图语法示例
@startmindmap

+[#cyan] 思维导图

'以下代码声明右边的分支
right side

**:分
段;

++[#pink] 右边的主题1
***: 
这是
分段
输入
;

++ 右边的主题2

'以下代码声明左边的分支，当左边需要多行输入时，必须加上，否则会出现在右边
left side

--[#114514] 左边的主题1
--- 左边的主题1.1
---[#orange] 左边的主题1.2是橘色的

-- 左边的主题2
***:
这是
分段
输入
;
@endmindmap
```

## 时序图
时序图，又名序列图、循序图，是一种UML交互图。它通过描述对象之间发送消息的时间顺序显示多个对象之间的动态协作。它可以表示用例的行为顺序，当执行一个用例行为时，其中的每条消息对应一个类操作或状态机中引起转换的触发事件。
```plantuml
@startuml

box #cyan
participant alice
participant bob
endbox
participant coco

autonumber

group  #green message 
alice -[#red]> bob:hello
bob -> alice:hello
end

alice -> coco:hello
@enduml
```
## 甘特图
甘特图又称为横道图、条状图(Bar chart)。其通过条状图来显示项目、进度和其他时间相关的系统进展的内在关系随着时间进展的情况。
```plantuml
@startgantt


[Prototype design] lasts 15 days
[Test prototype] lasts 10 days
-- All example --
[Task 1 (1 day)] lasts 1 day
[T2 (5 days)] lasts 5 days
[T3 (1 week)] lasts 1 week
[T4 (1 week and 4 days)] lasts 1 week and 4 days
[T5 (2 weeks)] lasts 2 weeks
@endgantt
```
# mermaid
## 流程图
​所有流程图都由节点、几何形状和边、箭头或线组成。mermaid代码定义了这些节点和边的制作和交互方式。它还支持不同类型的箭头、多方向箭头以及与子图的链接。

```mermaid
graph LR
    1[方形] --带文字的无向连接线--- 2(圆角) --> 3((圆形)) --> 4>非对称] --> 5{菱形} --> 6{{六角形}} --> 7[\平行四边形\] --> 8[/平行四边形/] --> 9[/梯形\] --> 10[\梯形/]
```

``` mermaid
graph LR
    执行1[i = 1]
  执行2[j = 0]
  执行3[i ++]
  执行4["a = arr[j], b = arr[j + 1]"]
  执行5[交换 a, b]
  执行6[j ++]
    判断1["i < n"]
    判断2["j < n - i"]
  判断3["a > b"]
  开始 --> 执行1
  执行1 --> 判断1
  判断1 --Y--> 执行2
  执行2 --> 判断2
  判断2 --Y--> 执行4
  判断2 --N--> 执行3
  执行3 --> 判断1
  执行4 --> 判断3
  判断3 --N--> 判断2
  判断3 --Y--> 执行5
  执行5 --> 执行6
  执行6 --> 判断2
  判断1 --N--> 结束
```
```mermaid
graph LR
  执行1[i = 1]
  执行2[j = 0]
  执行3[i ++]
```
## 时序图
```mermaid
sequenceDiagram
Client ->> Server: SYN，seq=x
Server ->> Client: SYN，ACK=x+1，seq=y
Client ->> Server: ACK=y+1，seq=x+1
```