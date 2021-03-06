# Kirara-Fantasia-Script #
## *@author:EnderLop* ##
## 脚本仅供玩家活动末期赶成就或刷作品珠、专武之用，请勿恶意或依赖使用！ ##
## 脚本仅供玩家活动末期赶成就或刷作品珠、专武之用，请勿恶意或依赖使用！ ##
## 脚本仅供玩家活动末期赶成就或刷作品珠、专武之用，请勿恶意或依赖使用！ ##
---
---
### 写个*Python*脚本拯救模拟器玩家的肝 ###
+ 使用*Pillow*库扫色获取游戏进程信息并给出回应
+ 实现战斗结束自动点击再来一关
+ 珍藏珠就绪自动进行芳文跳，跳完开启*AUTO*模式
+ 支持全屏幕和窗口化游戏
#### 目前功能较单一，后期可能会添加其他功能 ####
1. 尝试自动续表、设置最大游戏次数
2. 尝试兼容安卓系统
3. 尝试句柄操作
---
### 更新记录 ###
+ 2019.9.27:
  1. 更新了窗口模式，这下可以边刷*Kirara Fantasia*边补番力！
+ 2020.1.1:
	1. 使用四分色块法精确查找游戏窗口的左上角及右下角的绝对坐标，避免窗口模式下手工打点误差
+ 2020.1.18：
	1. 补充循环次数限制，并去除了持续检测的等待时间，增加程序运行的效率
#### 更新笔记 ####
+ 2021.1月初更新补充：
  1. 理论上只要模拟器能将游戏画面开到全屏就可以使用全屏模式
  2. 如果模拟器四面都有边框且颜色一致，理论上可以使用窗口模式（*推荐用NOX模拟器，贴吧有使用教程*）
---
## 使用方法 ##
1. 启动：
  + 使用*NOX*模拟器开启*Kirara Fantasia*，进入将要刷次数的对战，开启脚本，选择模式后两秒脚本启动（*成功跳忙音*）
  + **注意：刷次数过程中*NOX*模拟器必须时刻处于最上层**
2. 停止：
  + 将光标移至显示器左上角，一段时间后程序自动停止运行（*成功跳忙音*）
  + **注意：若出现错误，请重启脚本（如遇死循环，请打开任务管理器关闭脚本）**
3. 具体细节可参考“示例”文件夹中示范视频
---
---
## 源代码 ##
```
#Kirara Fantasia代肝脚本
# -*- coding: utf-8 -*-
#@author:EnderLop
from os import system
from PIL import Image,ImageGrab
from pymouse import PyMouse
import numpy as np
import time
totalRound = 0
x0Amb,y0Amb,x1Amb,y1Amb = (0,0,1919,1079)
x0Acc,y0Acc,x1Acc,y1Acc = (0,0,1919,1079)


"""游戏预先处理单元"""
'''小窗适配'''
def windowsConfig():
    global x0Amb,y0Amb,x1Amb,y1Amb
    system("cls")
    print("下面开始检查游戏界面左上角边缘点")
    time.sleep(1)
    print("\a请在2s内将鼠标移至游戏界面左上角")
    time.sleep(2)
    x0Amb,y0Amb = m.position()[0],m.position()[1]#获取游戏界面左上角模糊绝对位置
    system("cls")
    print("下面开始检查游戏界面右下角边缘点")
    time.sleep(1)
    print("\a请在2s内将鼠标移至游戏界面右下角")
    time.sleep(2)
    x1Amb,y1Amb = m.position()[0],m.position()[1]#获取游戏界面右下角模糊绝对位置

'''模糊绝对位置精确化（四分色块法）'''
def cornerConfig(info):
    global x0Amb,y0Amb,x1Amb,y1Amb,x0Acc,y0Acc,x1Acc,y1Acc
    system("cls")
    print("让我康康你的边框颜色")
    time.sleep(1)
    print("\a请在2s内将鼠标移至模拟器边框上")
    time.sleep(2)
    [R,G,B] = info[m.position()[0],m.position()[1]].tolist()
    x0Min,y0Min = max(1,x0Amb-5),max(1,y0Amb-5)
    x1Max,y1Max = min((x1Amb+5),1918),min((y1Amb+5),1078)
    for i in range(x0Min,(x0Amb+5)):
        for j in range(y0Min,(y0Amb+5)):
            if not colorCompare(info,i,j,R,G,B) and colorCompare(info,(i-1),j,R,G,B) and colorCompare(info,i,(j-1),R,G,B) and colorCompare(info,(i-1),(j-1),R,G,B):
                x0Amb,y0Amb = i,j
    for i in range((x1Amb-5),x1Max):
        for j in range((y1Amb-5),y1Max):
            if not colorCompare(info,i,j,R,G,B) and colorCompare(info,(i+1),j,R,G,B) and colorCompare(info,i,(j+1),R,G,B) and colorCompare(info,(i+1),(j+1),R,G,B):
                x1Amb,y1Amb = i,j
    x0Acc,y0Acc,x1Acc,y1Acc = x0Amb,y0Amb,x1Amb,y1Amb
    system("cls")
    print("\a已完成数据采集,即将开始工作")

'''1080P→窗口化绝对位置调整'''
def stateTransform(xOri,yOri):
    global x0Acc,y0Acc,x1Acc,y1Acc
    xNew = x0Acc + xOri * (x1Acc - x0Acc) / 1919#X轴绝对位置变换
    xNew = int (xNew)
    yNew = y0Acc + yOri * (y1Acc - y0Acc) / 1079#Y轴绝对位置变换
    yNew = int (yNew)
    return xNew,yNew


"""游戏信息获取单元"""
'''截取当前屏幕色块信息'''
def detectFlash():
    pic = ImageGrab.grab()#截屏
    info = np.asarray(pic,'int')#图像数字化
    info = info.swapaxes(0,1)#Shape转置
    return info

'''判断当前位置颜色是否含有特殊信息'''
def colorCompare(info,stateX,stateY,R,G,B):
    if info[stateTransform(stateX,stateY)[0],stateTransform(stateX,stateY)[1]].tolist() == [R,G,B]:
        return True
    else:
        return False


"""对战进程判断单元"""
'''判断专武技是否准备就绪※施工中※'''
'''def spSkillShine(info):
    if colorCompare(info,840,930,255,176,82) and colorCompare(info,803,859,250,250,235):#检查专武技为亮 且 周遭界面亮
        time.sleep(0.5)
        if colorCompare(info,840,930,255,176,82):#检查专武技亮是否偶然
            return True
    elif colorCompare(info,840,930,127,88,41) and colorCompare(info,803,859,125,125,117):#检查专武技为暗 且 周遭界面暗
        return False
    else:
        return None'''

'''判断珍藏技是否准备就绪'''
def starShine(info):
    if colorCompare(info,655,965,7,227,209) and colorCompare(info,655,920,250,250,235):#检查珍藏第一颗星星为亮蓝 且 周遭界面亮
        return True
    elif colorCompare(info,655,965,119,92,92) and colorCompare(info,655,920,125,125,117):#检查珍藏第一颗星星为深粉 且 周遭界面暗
        return False

'''判断当前是否为AUTO模式'''
def autoShine(info):
    if colorCompare(info,1825,30,255,255,255) and colorCompare(info,1820,55,255,255,255):#检查AUTO箭头头部为纯白
        return True
    elif colorCompare(info,1825,30,110,65,53) and colorCompare(info,1850,40,255,253,228):#检查AUTO箭头头部为深棕 且 周遭界面白
        return False

'''判断当前是否已完成对战'''
def finishShine(info):
    if colorCompare(info,1080,225,255,154,185) and colorCompare(info,1450,655,255,255,255):#检查表头为亮粉 且 两按钮间为纯白
        return True
    else:
        return False

'''在珍藏技准备就绪的情况下释放芳文跳'''
def fangWenJump(info):
    if starShine(info):
        if autoShine(info):#关闭AUTO模式
            for round in range(5):
                if autoShine(info):
                    m.click(stateTransform(1825,30)[0],stateTransform(1825,30)[1],1)
                    time.sleep(0.25)
                    info = detectFlash()
                elif not autoShine(info) and not (autoShine(info) is None):
                        break
                else:
                    return None
            else:
                return None
        m.click(stateTransform(655,965)[0],stateTransform(655,965)[1],1)
        time.sleep(0.5)
        info = detectFlash()
    elif colorCompare(info,1350,920,3,113,104):
        m.click(stateTransform(1690,190)[0],stateTransform(1690,190)[1],1)
        time.sleep(0.5)
        m.click(stateTransform(1565,965)[0],stateTransform(1565,965)[1],1)
        time.sleep(0.5)
        for i in range(7):
            m.click(stateTransform(960,540)[0],stateTransform(960,540)[1],1)
            time.sleep(1)
        print("{}：完成一次芳文跳释放".format(time.strftime("%Y/%m/%d %H:%M:%S")))

'''在珍藏技未就绪的情况下开启AUTO模式'''
def feelFish(info):
    if not starShine(info) and not autoShine(info) and not (starShine(info) is None) and not (autoShine(info) is None):
        for round in range(5):
            if not autoShine(info) and not (autoShine(info) is None):
                m.click(stateTransform(1825,30)[0],stateTransform(1825,30)[1],1)
                time.sleep(0.5)
                info = detectFlash()
            elif autoShine(info):
                print("{}：完成AUTO模式的开启".format(time.strftime("%Y/%m/%d %H:%M:%S")))
                break
            else:
                return None
        else:
            return None

'''在对战结束后点击跳过结算界面'''
def finallyFinsih(info):
    if finishShine(info):
        time.sleep(1)
        for round in range(3):
            m.click(stateTransform(960,800)[0],stateTransform(960,800)[1],1)
            if colorCompare(info,630,280,255,154,185) and colorCompare(info,630,130,127,77,92):
                m.click(stateTransform(960,680)[0],stateTransform(960,680)[1],1)
            time.sleep(1)
        print("{}：完成一场大对战刷轮".format(time.strftime("%Y/%m/%d %H:%M:%S")))

'''在对战结束且仍有剩余体力的情况下自动再开一把对战'''
def restartGame(info):
    global totalRound
    if colorCompare(info,580,85,255,154,185) and colorCompare(info,695,885,200,152,110):
        m.click(stateTransform(630,990)[0],stateTransform(630,990)[1],1)
        time.sleep(3)
        totalRound += 1
        print("{0}：已完成{1}场战斗,正在开始第{2}场战斗".format(time.strftime("%Y/%m/%d %H:%M:%S"),totalRound,totalRound+1))


'''主函数'''
def main():
    fangWenJump(detectFlash())
    feelFish(detectFlash())
    finallyFinsih(detectFlash())
    restartGame(detectFlash())


'''运行函数'''
m = PyMouse()
endTrigger = m.position()
userChoice = eval(input("以何种模式进行游戏？\n（输入0表示窗口化游戏，输入其他数字表示全屏游戏）\n"))
if userChoice == 0:
    windowsConfig()
    cornerConfig(detectFlash())
time.sleep(2)
system("cls")
print("\a{}：代肝工作开始".format(time.strftime("%Y/%m/%d %H:%M:%S")))
timeStart = time.perf_counter()
while endTrigger != (0,0):
    endTrigger = m.position()
    main()
dur = time.perf_counter() - timeStart
print("\a{}：代肝工作结束".format(time.strftime("%Y/%m/%d %H:%M:%S")))
print("共完成{3}场对战的刷轮，总耗时为:{2:.0f}h {1:.0f}min {0:.2f}s".format(dur % 60,(dur / 60) % 60,dur / 3600,totalRound))
input("\a输入任何字符以结束:\n")
```
