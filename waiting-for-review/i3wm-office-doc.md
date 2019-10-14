# i3 平铺窗口管理器

  在此之前用的是dwm，发生了几次崩溃之后，
想试试i3,dwm崩溃一般发生在打开指定的github页面(可重现)

## 按键

  我的$mod默认设置的是Win键

---

- $mod + t 切换到tabbed模式
- $mod + e 切换到分屏模式
- $mod + s 切换到stacking模式

  tabbed和stacking两种模式的区别在于标题的显示，
tabbed的标签展示方式和浏览器的tab页一个方式，
stacking和显示堆栈信息的方式一样，说白了，
tabbed的标题在一行中展示，stacking的标题在多行中展示。

---

- $mod + p 打开dmenu
- $mod + i 分屏模式改为垂直分
- $mod + v 分屏模式改为水平分
- $mod + f 全屏
- $mod + a 将焦点放到父窗口中,分屏的时候有用
- $mod + enter 打开终端

---

- $mod + h 焦点移到左窗口
- $mod + j 焦点移到下窗口
- $mod + k 焦点移到上窗口
- $mod + l 焦点移到右窗口

---

- $mod Shift + h 窗口左移
- $mod Shift + j 窗口下移
- $mod Shift + k 窗口上移
- $mod Shift + l 窗口右移

---

- $mod Shift + q 杀掉当前窗口
- $mod Shift + e 退出i3
- $mod Shift + r 重启i3
- $mod Shift + c 重新加载i3配置

---

- $mod + space 焦点在平铺窗口和浮动窗口中切换
- $mod Shift + space 切换平铺模式和浮动模式

---

- $mod + r 调整窗口大小 hjkl调整 esc退出

---

## 使用i3

  上面的快捷键包括了i3的常用方法

## 配置i3

  配置文件在~/.config/i3/config，
检查配置修改的是否符合i3要求的格式，
用 i3 config file

  i3配置提供了很多有用的功能，
也提供了很多有用的扩展

## 状态栏配置

  状态栏配置和i3的配置互不影响，两个是单独的模块
