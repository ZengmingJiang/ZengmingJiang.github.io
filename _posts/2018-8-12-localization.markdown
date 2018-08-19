---
layout: post
title: 手机游戏多语言化方案
date: '2018-08-12 20:13:26 +0800'
categories: [jekyll, Game-D&R]
---

![MultiLangDemo](../../../../../assets/img/blog/MultiLangDemo.PNG)
### 一．	游戏多语言化情况分类
游戏中需要多语言化的资源分为3类：
1. 游戏UI布局文件，一般为xml格式，定义了游戏的各种UI布局，内含有各种静态文本，这些文本都是要进行多语言化的；
2. 游戏自定义动态加载文本，一般为代码中动态加载的文本内容；
3. 游戏配置文件，一般为xml格式的配置文件，这些文件是供游戏逻辑去调用的，这里特指包含需要显示的文字的配置文件。
### 二.最后的多语言化形态
1.最终上述三种语言资源相关文件都会被抽取成JSON格式，以便转换成EXCEL格式
```JSON
{
	"LANG_01":
	{
		"ch":"点击",
        "en":""
	},
	"LANG_02":
	{
		"ch":"确认",
        "en":""
	}
}
```
2.在最终翻译时，将上述JSON格式的语言资源转换成EXCEL格式，以便交付翻译社翻译，一般只有开发时的语言是有填写的，比如下面的CN

Key  | CN  | EN
--|---|--
LANG_01 |  点击  | Click
LANG_02  | 确认  | OK

3.在翻译完成其他类型的语言后，如英文（EN），西班牙文（ES-AR）后，EXCEL格式会再次被转换为JSON格式文件，以便程序识别，再通过一定手段，根据Key将对应的需要显示的语言设置到原本的相关文件中，就完成了软件的多语言化（这里讨论的只是静态的多语言化，并不包括软件在运行时，动态切换语言类型的情况，那种情况下，是需要动态地根据Key及语言环境替换相关控件的语言资源的）
### 三.各类型文件的多语言化方式
#### 1.游戏UI布局文件
UI布局中含有众多的文字，皆有法可依，在游戏引擎中，一般都要以Text控件来表示，可以使用程序，靠抽取UI布局文件中的Text控件内容来实现多语言化。但是为了简单起见，要求Text控件的名称在其UI布局文件中唯一，且不能为空
```xml
<?xml version="1.0"?>
<element type="Button">
	<attribute name="Name" value="Btn_AttributeAddRoot" />
	<attribute name="Size" value="1280 720" />
	<attribute name="Pivot" value="0 0" />
	<attribute name="Color" value="0 0 0 0.5" />
	<element type="Button">
		<attribute name="Name" value="Btn_AttributeAddMain" />
		<attribute name="Position" value="0 -15" />
		<attribute name="Size" value="745 490" />
		<attribute name="Min Anchor" value="0.5 0.5" />
		<attribute name="Max Anchor" value="0.5 0.5" />
		<attribute name="Pivot" value="0.5 0.5" />
		<attribute name="Texture" value="Texture2D;Textures/Friend/bg_haoyoukuang.png" />
		<attribute name="Image Rect" value="0 0 56 60" />
		<element type="Text">
      <!--注意：这里就是Text控件的名称-->
			<attribute name="Name" value="T_AttributeAddTitle" />
			<attribute name="Position" value="0 10" />
			<attribute name="Min Anchor" value="0.5 0" />
			<attribute name="Max Anchor" value="0.5 0" />
			<attribute name="Pivot" value="0.5 0" />
			<attribute name="Color" value="0.98 0.69 0.31 1" />
			<attribute name="Top Left Color" value="0.98 0.69 0.31 1" />
			<attribute name="Top Right Color" value="0.98 0.69 0.31 1" />
			<attribute name="Bottom Left Color" value="0.98 0.69 0.31 1" />
			<attribute name="Bottom Right Color" value="0.98 0.69 0.31 1" />
			<attribute name="Font" value="Font;Fonts/PingFang_Medium.ttf" />
			<attribute name="Font Size" value="20" />
      <!--注意：这里就是Text控件的文本内容-->
			<attribute name="Text" value="加点" />
		</element>
  </element>
</element>
```
对于上述XML文件，自行撰写任意语言程序，遍历Text控件，或任意包含文本的控件，并提取控件名称及控件的值，上述文件的名称和值分别是：**T_AttributeAddTitle**,**加点**
这个条目就会被写入JSON文件中，如下：

Key  | CN  | EN
--|---|--
T_AttributeAddTitle|加点|

#### 2.游戏自定义动态加载文本
由于自定义动态加载文本，只是直接由程序调用的文本，那么满足一个需求就行了，那就是输入Key，语言标识（EN/CN）得到Value。
各游戏引擎或者各软件一般自己都会带有一个本地化工具，用来读取语言资源，然后提供接口，例如：
`String value = localization.Get(String Key, String Lan)`
其中Key就是各单条语言资源的Key了，而Lan就是语言标识，如EN/CN
也有些引擎不需要每次都传Lan进去，而是可以设置全局的Lan，这样取出来的都是该语言的文本
例如：
```C#
localization.Lan = "EN";
String value = localization.Get("T_AttributeAddTitle");
```
使用上述方式的好处是，只要改变了localization对象的Lan值，那么各处调用语言资源的输出就会改变了。
而上述所提及的**本地化工具**一般都是需要读取上面第二点所提及的Json文件了。通过读取Json文件，本地化工具会将所有的语言资源记录成内存中的表，随时等待取值。

#### 3.游戏配置文件
1.游戏配置文件的特点
游戏配置文件一般为XML文件（这里只讨论XML文件的情况，因为Json的配置文件，都是类似的处理方式）。而XML文件的特点就是**层级有可能很多，属性有可能也很多**。
策划们在编写游戏配置文件时，并不会关注配置文件中相关文本需要多语言化的问题，于是文本很分散，且有可能会重复。对于程序，在读取相关配置文件中的属性，用来显示时，也不会关注多语言化的问题。需要将配置文件的多语言化对两者透明。
2.配置示例
```xml
<XML>
	<Object ID="t001" Type="0" Name="急急忙忙" Desc1="" Desc3="客官别走呀，这里瞧瞧。" Desc2="我现在需要一些豆腐，去杂货店帮我买一些" Next="t002" />
	<Object ID="t002" Type="0" Name="匆忙上路" Desc1="大老爷就在转角处，快去找他吧。" Desc3="完成地很漂亮，可以领取奖励了。" Desc2="前往大老爷处报到" Next="t003"/>
	<Object ID="t003" Type="0" Name="准备充分" Desc1="我这里什么都有，快来看看。" Desc3="快去做事情，想偷懒啊你。" Desc2="值得表扬，你这小伙子不错" Next="t004" />
</XML>
```
3.配置文件中多语言资源的标识
目前我们并不清楚配置文件中哪些属性是需要做多语言化的（不能笼统地认为其中的中文属性值都是需要进行多语言化的，因为有些属性可能在中文模式下，反而需要显示成空或者英文等，而在其他语言模式下，要显示成别的文字），也就是说，只有配置的人，才知道那些属性可能要考虑多语言化的问题，那么我们另外做一个配置，用来记录所有的配置文件中，各自有哪些属性需要进行多语言化即可，这里简称这个配置文件为《配置文件多语言化标识文件》（以下简称为《配置标识文件》），此文件示例如下
```xml
<XML>
	<File name = "Config/UI/NPC.xml">
		<!--name:属性名称;path:属性所在节点的路径，用，隔开，依次写出该属性所在的每一层的节点名称（不包括根节点）-->
		<Attribute name = "Name" path = "Object" />
	</File>
</XML>
```
接着就可以使用程序，对照这个《配置标识文件》去将对应配置文件中的语言资源提取出来了
4.配置文件中语言资源提取方式
现在知道了配置文件中哪些是需要进行提取的，接着就要寻找一种直接将配置文件中的文本作为Key的方式，这样就可以直接抽取配置文件中的文本转化成多语言Json文件了，如下：

Key  | CH  | EN
--|---|--
急急忙忙  |  急急忙忙 |
客官别走呀，这里瞧瞧。  |  客官别走呀，这里瞧瞧。 |

问题很明显，那就是文字特别长的话，Key也特别长，这个明显不合适。
那有没有可以保留这些文本唯一的特性，又不至于太长的办法呢？
那就是利用MD5来简化文本了。
5.将配置文件MD5化步骤
使用程序，读取配置文件，并且将配置文件中的文本使用对应的MD5码替换，然后抽出对应的MD5码，及原文本，写入Json多语言文件中，最终转换成Excel,如下：

Key  | CH  | EN
--|---|--
096B68074FADEA61  |  急急忙忙 |
00CC84A913FE6A0E  |  客官别走呀，这里瞧瞧。 |

各自的配置文件也已经被MD5化了（这个MD5化的配置文件，在进行多语言转化时要用），如下：
```xml
<XML>
	<Object ID="t001" Type="0" Name="096B68074FADEA61" Desc1="" Desc3="00CC84A913FE6A0E" Desc2="791DA552FA664690" Next="t002" />
</XML>
```
利用这个思路，就可以将配置文件中的多语言文本全部抽出来了。
6.还原步骤
当翻译社翻译完Excel的多语言资源后，我们本地转换为Json格式，然后利用自己的工具，借助《配置描述文件》遍历上述的MD5化的配置文件，找到当中的MD5码，使用Json中MD5码对应的语言资源替换即可

#### 4.名词解释
1.**MD5化**：MD5是一种确保信息传输完整性的算法，这里利用了它的唯一性，简短性，用来代替要进行多语言化的中文，充当多语言资源的Key（每个Key会对应多个国家的文字值）。
2.**多语言化**：多语言化是指，将配置文件中的指定属性的值，进行对多国语言的适配。也就是说配置文件中的有些属性值，在另一个语言中，是要用另外一个值来替换的。于是需要用工具去控制对这些配置的属性值的替换，已完成对另一个语言的适配。
