1.进一步汉化
2.背景色调改为蓝白
editor/editors/ConfTreeNodeEditor.py
if self.IsEnabled():  
    dc.SetTextForeground(self.GetForegroundColour())  
else:
dc.SetTextForeground(wx.SystemSettings.GetColour(wx.SYS_COLOUR_GRAYTEXT))
更改为固定蓝色
***
/home/mj/Documents/openplc/OpenPLC_Editor/editor/editors/CodeFileEditor.py
添加禁用默认主题
3.将有openplc的元素进行替换
	editor/BeremizIDE.py中init函数更改图标
