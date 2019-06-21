---
title: 修改键盘键位的布局
key: 2019-06-18
tags: windows scancode-map keyboard-layout

---

在 Windows XP/Windows 2000 系统中，提供了一种新的键盘扫描码映射方法，使你能随意地设置键盘键位的布局，这就是注册表中的 “Scancode Map”, 我们巧妙利用 “Scancode Map” 就可将普通键盘改造成随心随意的个性化键盘。

<!--more-->

## 说明

为了正确设置，我们有必要先了解一下“Scancode Map”(扫描码映射)。 

“Scancode Map”是注册表中

```
[HKEY_LOCAL_MacHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]
```

中的一个二进制键值(默认没有，需新建)，并且有固定的格式。Scancode Map 代码的一般格式是：

```
hex:00,00,00,00,00,00,00,00,|02|,00,00,00,|映射之后的扫描码（XX XX）,原扫描码(XX XX)|00,00,00,00”。 
```

其含义为： 

前 8 个 `00` (DWord两个0)是版本号和头部字节， 

接下来的 `02` 表示映射数，其最小为值为 `02`，表示只映射一组（这里的数值是映射数目加上末尾用作结尾的 `00，00，00，00` ，因此总是比实际的映射数目大一），若要映射多组，只需增加相应的值即可，如映射 2 组其值应为 `03`,3 组为 `04`。

后边代码每4个是一组：前两个是映射后键位的扫描码，后两个是键位原扫描码。

如果要交换两个键，则一个有两组映射，四个值的排列形式是：键A，键B，键B，键A——它表示：键A成为键B，键B成为键A。

最后以 `00,00,00,00` 结尾。

> 注意：在注册表中输入时，需要将扫描码的高低字节交换一下。
> 
> 另外，如果想要某个键失效，将它的扫描码映射为“00 00”即可。

若要恢复键盘键位原来的布局，只需定位于注册表[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]，删除"Scancode Map"键值


**特别说明：**

1. 在目前的Windows版本下面，对键盘映射是全局的，而不是针对某个用户的，如果您修改添加或者删除了某个映射，那么不管哪个用户使用，键盘都发生了变化；
2. 另外，如果一台电脑有多个键盘，那这些键盘都会产生同样的变化。
3. 在XP上不能屏闭 `POWER` `SLEEP` `WAKE UP` 这三个键。（没有亲自测试，笔记本上没有这三个键，如果真不行就扣掉吧 ------ 开玩笑，在台式机上这三个键通过bios设置应该可以把相应功能禁用掉）
4. ThinkPad 上的 `Fn` 键是不能被映射的，因为它不能被OS识别，所以不能使用上面的方式进行设置。thinkpad新版的bios里面提供了一个功能，让左侧的“Fn”键和相邻的“Ctrl”键进行功能互换，感觉用处不是很大，有需要的朋友可以去试试看。（如果在笔记本的 bios上找不到这个功能的话，需要刷新版bios，操作很简单，不要恐惧）

导入或设置或修改或删除注册表键值后，**重启**你的电脑，改变就生效了。

也可以用将下面的文本存成 `scancode.reg`，双击导入注册表。键值可通过查上面提到的键位表查询，找到你要替换的 Scan Code码，把##,##替换掉就可以了。 

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout] 
"Scancode Map"=hex:00,00,00,00,00,00,00,00,02,00,00,00,##,##,##,##,00,00,00,00 
```

了解了“Scancode Map”之后，我们就可以来利用添加功能键了。比如WIN键扫描码为：`E0 5B`，Esc 为 `00 01` 左边的 Ctrl 为 `00 1D`更详细的扫描码请见键盘扫描码。


## 举例：

比如：前面提到的IBM ThinkPad键盘，为了把那个浏览器网页前后浏览的键禁止，可以设置为：  

```
"Scancode Map"=hex:00,00,00,00,00,00,00,00,03,00,00,00,00,00,6A,E0,00,00,69,E0,00,00,00,00 
```

比如：说我们想把F9，F10键修改成为音量调整键，通过查表，可以得知：

F9、F10扫描码分别为(00，43)、(00,44),

Volume Up、Volume Down的扫描码分别为(E0，30)、(E0,2E)，

这样只要将Scancode设置为如下就可以了： 

```
"Scancode Map"=hex:00,00,00,00,00,00,00,00,03,00,00,00,30,E0,43,00,2E,E0,44,00,00,00,00,00 
```
        （ 含义为：          |    版本号和头部字节 | 两组映射 |   第一组 | 第二组 | 结尾终止 | ）

我是把“后退”和“前进”两个按键映射为“上翻页”和“下翻页”，注册表文件如下：

```
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]
"Scancode Map"=hex:00,00,00,00,00,00,00,00,03,00,00,00,49,e0,6a,e0,51,e0,69,e0,00,00,00,00
```

再次说明：所有对注册表的修改，必须在重新启动电脑后，才能生效。 


## Linux

Linux系统中可以用下面命令:

```bash
[root@localhost ~]# setkeycodes e06a 104
[root@localhost ~]# setkeycodes e069 109
```


在linux环境下可以用 `showkey` 命令测试按键的键盘码或者扫描码（加 `-s` 选项），注意按键“按下”和“松开”的扫描码可能不一样，例如大小写切换键，按下是3A，松开是BA

键盘扫描码对应表            

```
Backspace       00 0E
Caps Lock       00 3A
Delete          E0 53
End             E0 4F
Enter           00 1C
Escape          00 01
HOME            E0 47
Insert          E0 52
Left Alt        00 38
Left Ctrl       00 1D
Left Shift      00 2A
Left Windows    E0 5B
Num Lock        00 45
Page Down       E0 51
Page Up         E0 49
Power           E0 5E
PrtSc           E0 37
Right Alt       E0 38
Right Ctrl      E0 1D
Right Shift     00 36
Right Windows   E0 5C
Scroll Lock     00 46
Sleep           E0 5F
Space           00 39
Tab             00 0F
Wake            E0 63
0               00 52
1               00 4F
2               00 50
3               00 51
4               00 4B
5               00 4C
6               00 4D
7               00 47
8               00 48
9               00 49
-               00 4A
*               00 37
.               00 53
/               00 35
+               00 4E
Enter           E0 1C
F1              00 3B
F2              00 3C
F3              00 3D
F4              00 3E
F5              00 3F
F6              00 40
F7              00 41
F8              00 42
F9              00 43
F10             00 44
F11             00 57
F12             00 58
F13             00 64
F14             00 65
F15             00 66
Down            E0 50
Left            E0 4B
Right           E0 4D
Up              E0 48
Calculator      E0 21
E-Mail          E0 6C
Media Select    E0 6D
Messenger       E0 11
My Computer     E0 6B
' "             00 28
- _             00 0C
, <             00 33
. >             00 34
/ ?             00 35
; :             00 27
[ {             00 1A
\ |             00 2B
] }             00 1B
` ~             00 29
= +             00 0D
0 )             00 0B
1 !             00 02
2 @             00 03
3 #             00 04
4 $             00 05
5 %             00 06
6 ^             00 07
7 &             00 08
8 *             00 09
9 (             00 0A
A               00 1E
B               00 30
C               00 2E
D               00 20
E               00 12
F               00 21
G               00 22
H               00 23
I               00 17
J               00 24
K               00 25
L               00 26
M               00 32
N               00 31
O               00 18
P               00 19
Q               00 10
R               00 13
S               00 1F
T               00 14
U               00 16
V               00 2F
W               00 11
X               00 2D
Y               00 15
Z               00 2C
Close           E0 40
Fwd             E0 42
Help            E0 3B
New             E0 3E
Office Home     E0 3C
Open            E0 3F
Print           E0 58
Redo            E0 07
Reply           E0 41
Save            E0 57
Send            E0 43
Spell           E0 23
Task Pane       E0 3D
Undo            E0 08
Mute            E0 20
Next Track      E0 19
Play/Pause      E0 22
Prev Track      E0 10
Stop            E0 24
Volume Down     E0 2E
Volume Up       E0 30
? -             00 7D
              E0 45
Next to Enter   E0 2B
Next to L-Shift E0 56
Next to R-Shift E0 73
DBE_KATAKANA    E0 70
DBE_SBCSCHAR    E0 77
CONVERT         E0 79
NONCONVERT      E0 7B
Internet        E0 01
iTouch          E0 13
Shopping        E0 04
Webcam          E0 12
Back            E0 6A
Favorites       E0 66
Forward         E0 69
HOME            E0 32
Refresh         E0 67
Search          E0 65
Stop            E0 68
My Pictures     E0 64
My Music        E0 3C
Mute            E0 20
Play/Pause      E0 22
Stop            E0 24
+ (Volume up)   E0 30
- (Volume down) E0 2E
|<< (Previous) E0 10
>>| (Next)      E0 19
Media           E0 6D
Mail            E0 6C
Web/Home        E0 32
Messenger       E0 05
Calculator      E0 21
Log Off         E0 16
Sleep           E0 5F
Help(on F1 key) E0 3B
Undo(on F2 key) E0 08
Redo(on F3 key) E0 07
Fwd (on F8 key) E0 42
Send(on F9 key) E0 43
```

现在使用的是 Fedora 20，将这两句写在了 `/etc/rc.d/rc.local` 中，保证能够在启动后被加载


## 我的

我把右 ctrl 和 Caps-Lock 键进行了互换, menu 换成 win,  左 alt 和 左 win 进行了互换. reg 文件如下:

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]
"Scancode Map"=hex:00,00,00,00,00,00,00,00,07,00,00,00,\
1d,00,3a,00,\
3a,00,1d,e0,\
5c,e0,5d,e0,\
5B,E0,38,00,\
38,00,5b,e0,\
00,00,00,00

```
<< EOF >>

参考:

- [修改windows的注册表以实现修改键盘按键的映射](https://blog.csdn.net/u010695008/article/details/51752278)
- [通过注册表修改键盘按键的映射](http://blog.chinaunix.net/uid-174325-id-3912617.html)

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
