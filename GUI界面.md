# QT

安装的QT5.15，2020年6月12日发布，提供3年的支持到2023年6月

QT6比较新，就算了

## QT教程

[教程](http://c.biancheng.net/view/1804.html)

黑马资料：D:\基础学习\QT从入门到实战完整版资料\Qt基础教程V2.0.doc

### 名词

#### Project

Project 的中文翻译是“项目”或者“工程”，这里的项目是指为实现某个相对独立功能的程序代码合集，这些代码不单单是放在一块，而是有相互之间的关联性，并且有专门负责管理该项目的项目文件，比如：

* Qt 使用 .pro 文件管理项目；
* VC++ 则使用 .vcproj 作为项目文件。


 集成开发环境通常都是依据项目文件（.pro/.vcproj）管理和构建项目。



#### Debug 和 Release

Debug 即调试，Release 即发行。代码编写之后，生成的目标程序或库文件通常不会绝对正确，或多或少有些毛病（bug），  因此需要进行纠错调试（Debug）。调试过程中需要源代码和二进制目标程序之间一一对应的关系， 这样才能定位到错误代码，所以 Debug  版本的程序是臃肿而不进行优化的。

 与之相对的是 Release 发行版，在纠正了发觉到的错误后，需要发布程序用于实际用途，实际应用时强调运行效率高，减少冗余代码，因此会对二进制程序进行大量优化，提升性能。这样发布的二进制目标程序就是 Release 版。

 Debug 版本和 Release 版本使用的库文件不一样：

* Debug 版本程序通常链接的也是 Debug 版本的库文件，比如 libQt5Guid.a/Qt5Guid.dll，库文件的简短名（不含扩展名）都是以 d 结尾的，Debug 库通常都比较大 。
* Release 版本程序链接的通常就是 Release 版本的库文件，Release 版本库文件名字比 Debug 版本库文件少一个字母 d  ，如 libQt5Gui.a/Qt5Gui.dll，而且 Release 版本库一般都比 Debug 版本小很多，运行效率也高很多。

#### Dynamic Link 和 Static Link

Dynamic Link 即动态链接，Static Link 即静态链接。

##### 动态链接库

目标程序通常都不是独立个体，生成程序时都需要链接其他的库，要用到其他库的代码。对于多个程序同时运行而言，内存中就可能有同一个库的多个副本，占用了太多内存而干的活差不多。

 为了优化内存运用效率，引入了动态链接库（Dynamic Link Library），或叫共享库（Shared  Object）。使用动态链接库时，内存中只需要一份该库文件，其他程序要使用该库文件时，只要链接过来就行了。由于动态库文件外置，链接到动态库的目标程序相对比较小，因为剥离了大量库代码，而只需要一些链接指针。

 使用动态库，也意味着程序需要链接到如 *.dll 或 *.so 文件，得提前装好动态库文件，然后目标程序才能正常运行。

##### 静态链接库

静态库就是将链接库的代码和自己编写的代码都编译链接到一块，链接到静态库的程序通常比较大，但好处是运行时依赖的库文件很少，因为目标程序自己内部集成了很多库代码。

##### 库文件后缀

Linux/Unix 系统里静态库扩展名一般是 .a，动态库扩展名一般是 .so 。Windows 系统里 VC 编译器用的静态库扩展名一般是 .lib，动态库扩展名一般是 .dll 。

 MinGW 比较特殊，是将 GNU 工具集和链接库从 Linux/Unix 系统移植到 Windows 里， 有意思的情况就出现了，MinGW  使用的静态库扩展名为 .a ，而其动态库扩展名则为 .dll， .a 仅在生成目标程序过程中使用，.dll 则是在目标程序运行时使用。

#### Explicit Linking 和 Implicit Linking

Explicit Linking 即显式链接，Implicit Linking 即隐式链接，这两种都是动态链接库的使用方式。

 动态链接库通常都有其导出函数列表，  告知其他可执行程序可以使用它的哪些函数。可执行程序使用这些导出函数有两种方式：一是在运行时使用主动加载动态库的函数，Linux 里比如用  dlopen 函数打开并加载动态库，Windows 里一般用 LoadLibrary  打开并加载动态库，只有当程序代码执行到这些函数时，其参数里的动态库才会被加载，这就是显式链接。显式链接方式是在运行时加载动态库，其程序启动时并不检查这些动态库是否存在。

 隐式链接是最为常见的，所有的编译环境默认都是采用隐式链接的方式使用动态库。隐式链接会在链接生成可执行程序时就确立依赖关系，在该程序启动时，操作系统自动会检查它依赖的动态库，并一一加载到该程序的内存空间，程序员就不需要操心什么时候加载动态库了。比如 VC 编译环境，链接时使用动态库对应的 .lib 文件（包含动态库的导出函数声明，但没有实际功能代码），在 .exe  程序运行前系统会检查依赖的 .dll，如果找不到某个动态库就会出现类似下图对话框：

![image-20220711181621355](D:\MarkDown\picture\image-20220711181621355.png)

MinGW 是将动态库的导出函数声明放在了 .a 文件里，程序运行依赖的动态库也是 .dll 。

 请注意，VC 链接器使用的 .lib 文件分两类，一种是完整的静态库，体积比较大，另一种是动态库的导出声明，体积比较小。MinGW 链接器使用的 .a 文件也是类似的，Qt 官方库都是按照动态库发布的，静态库只有自己编译才会有。

# tkinter

## tkinter教程

[Tkinter教程](http://c.biancheng.net/tkinter/)

[教程2](https://blog.csdn.net/weixin_33739541/article/details/93964322?utm_medium=distribute.pc_relevant_t0.none-task-blog-2~default~BlogCommendFromMachineLearnPai2~default-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2~default~BlogCommendFromMachineLearnPai2~default-1.control)

## 代码

D:\研究生\项目\十所\十所\有界面版源码\ImgGenerate.py

```python
from ctypes.wintypes import tagMSG
from re import A
import tkinter as tk  # 使用Tkinter前需要先导入
from tkinter import filedialog
from tkinter import *
import tkinter.ttk  # Combobox
from PIL import Image, ImageTk
import threading
import cv2
import random
import os
from numpy import *
import numpy as np
import multiprocessing
#encoding:utf-8
import xlrd
from xlrd import xldate_as_tuple # 读取表
import math
import scipy as sp
import matplotlib.pyplot as plt
# import multiprocessing_win
from tkinter.messagebox import *
import sys
# -*- coding:utf-8 _*-


##########################################  将高斯斑图和背景图混合  ########################################
def blend_two_images(book,img1,a):
    sheet1 = book.sheets()[0]  # 获取第一个sheet页
    rows = sheet1.nrows  # 获取总行数
    # 得到.exe所在目录的路径
    gml = os.path.realpath(os.path.dirname(sys.argv[0]))
    path = gml + "\\generated-image\\"
    buf = os.listdir(path)
    for i in buf:
        img2 = Image.open(path + i)#带有目标位置信息的高斯斑图
        img2 = img2.convert('RGBA')
        img = Image.blend(img1, img2, a)#参数，可以改变目标和背景的对比度
        path2 = gml + "\\image\\"
        isExists=os.path.exists(path2)
        # 判断结果
        if not isExists:
            # 如果不存在则创建目录
            # 创建目录操作函数
            os.makedirs(path2)
        img.save(path2 + i.split(".")[0] + ".png")#保存生成图像
    return

##########################################  图像平移  ########################################
def image_trans(img):
    rows, cols = img.shape[:2]
    #图像平移矩阵
    M = np.float32([[1, 0, 100], [0, 1, 50]])
    #图像平移
    result = cv2.warpAffine(img, M, (cols, rows)) 
    return result

##########################################  通过线性变化增强对比度  ########################################
def contrast_change(img,a):
    out_image = float(a) * img
    # 进行数据截断, 大于255的值要截断为255
    out_image[out_image > 255] = 255
    # 数据类型转化
    out_image = np.round(out_image)
    out_image = out_image.astype(np.uint8)
    return out_image

#################################################  噪声  ######################################################
#椒盐噪声
def PepperandSalt(src,percetage):
    NoiseImg=src
    NoiseNum=int(percetage*src.shape[0]*src.shape[1])
    for i in range(NoiseNum):
        randX=random.randint(0, 1023 + 1)
        randY=random.randint(0, 2047 + 1)
        if random.randint(0, 1 + 1)<=0.5:
            NoiseImg[randX,randY]=0
        else:
            NoiseImg[randX,randY]=255
    return NoiseImg
#高斯噪声
def GaussianNoise(src,means,sigma,percetage):
    NoiseImg=src
    NoiseNum=int(percetage*src.shape[0]*src.shape[1])
    for i in range(NoiseNum):
        randX = random.randint(0, 1023 + 1)
        randY = random.randint(0, 2047 + 1)
        NoiseImg[randX, randY]=NoiseImg[randX,randY]+random.normal(means,sigma)
        if  NoiseImg[randX, randY].any()< 0:
                 NoiseImg[randX, randY]=0
        elif NoiseImg[randX, randY].any()>255:
                 NoiseImg[randX, randY]=255
    return NoiseImg

##########################################  将函数打包进线程  ########################################
def thread_it(func, *args):
    # 创建线程
    t = threading.Thread(target=func, args=args)
    # 守护线程
    t.setDaemon(True)
    # 运行线程
    t.start()

# 选择png格式的图片，返回图片路径
def openimg():
    sfname = filedialog.askopenfilename(title='选择图片', filetypes=[('E', '*.png'), ('All Files', '*')])
    return sfname

# 选择文件夹，返回文件夹路径
def openfile():
    sfname = filedialog.askdirectory(title='选择文件夹')
    return sfname

# 图像合成
def hc():
    global book
    global sfnameaa
    sfname = filedialog.askopenfilename(title='选择背景图', filetypes=[('E', '*.jpg'), ('All Files', '*')])
    img1 = Image.open(sfname)  # 背景图
    img1 = img1.convert('RGBA')
    a=0.3 #图像合成函数，影响图像的对比度
    book = xlrd.open_workbook(sfnameaa)
    blend_two_images(book,img1,a)
    # 消息提示框
    tkinter.messagebox.showinfo(message='done！') 

# 几何变换
def APP():
    index = getInput2("选择图片处理数量")
    gml = os.path.realpath(os.path.dirname(sys.argv[0]))

    # 0的话处理单幅图像
    if  index == 0:
        filename = openimg()
        img = cv2.imdecode(np.fromfile(filename,dtype=np.uint8),cv2.IMREAD_GRAYSCALE)
        # cv2.imdecode也是两个参数，第二个参数和cv.imread一致
        # cv2.IMREAD_COLOR：默认参数，读入一副彩色图片，忽略alpha通道，可用1作为实参替代
        # cv2.IMREAD_GRAYSCALE：读入灰度图片，可用0作为实参替代
        # cv2.IMREAD_UNCHANGED：顾名思义，读入完整图片，包括alpha通道，可用-1作为实参替代
        img = image_trans(img)
        path = gml + "\\geometric_trans-image\\"
        isExists=os.path.exists(path)
        # 判断结果
        if not isExists:
            # 如果不存在则创建目录
            # 创建目录操作函数
            os.makedirs(path) 
        
        imgname = filename.split("/")[-1]
        portion=os.path.splitext(imgname)  # 将文件名称和后缀分开
        newname=portion[0]+"_jh"+".png" 
        cv2.imencode('.png',img)[1].tofile(path+newname)
    #     imencode(ext, img[, params])
    #    ext: 图像后缀，".bmp"、".jpg"等cv2模块支持的图像格式
    #    img: 数据类型为numpy.ndarray的图像，shape为height × \times × width × \times ×channel
    #    params: 图像保存时的参数，与cv2.imwrite(filename, img[, params])中参数一致
        #cv2.imwrite(path, img)
    
    # 1的话批量处理
    elif index == 1:
        path = gml + "\\geometric_trans-image\\"
        isExists=os.path.exists(path)
        # 判断结果
        if not isExists:
            # 如果不存在则创建目录
            # 创建目录操作函数
            os.makedirs(path) 
        filename = openfile()
        buf = os.listdir(filename)  # os.listdir() 方法用于返回指定的文件夹包含的文件或文件夹的名字的列表
        for i in buf:
            img = cv2.imdecode(np.fromfile(filename + "/" + i,dtype=np.uint8),cv2.IMREAD_GRAYSCALE)
            img = image_trans(img)
            portion=os.path.splitext(i)  # 将文件名称和后缀分开
            newname=portion[0]+"_jh"+".png" 

            cv2.imencode('.png',img)[1].tofile(path+newname)
    tkinter.messagebox.showinfo(message='done！') 



# 噪声模拟
def APP1():
    a = combobox1.get()
    
    # indexs = listbox1.curselection()  # .curselection()函数返回的是一个只有一个元素的元组
    # index = indexs[0]
    index = getInput2("选择图片处理数量")
    gml = os.path.realpath(os.path.dirname(sys.argv[0]))
    # 0的话处理单幅图像
    if  index == 0:
        filename = openimg()
        img = cv2.imdecode(np.fromfile(filename,dtype=np.uint8),1)
        img1 = PepperandSalt(img, 0.002) #添加椒盐噪声
        img2 = GaussianNoise(img, 20, 40, 0.01) #添加高斯噪声
        path = gml + "\\noise-image\\"
        isExists=os.path.exists(path)
        # 判断结果
        if not isExists:
            # 如果不存在则创建目录
            # 创建目录操作函数
            os.makedirs(path) 
        imgname = filename.split("/")[-1]
        portion=os.path.splitext(imgname)
        if  a == "椒盐噪声": 
            imgg=img1
            newname=portion[0]+"_jy"+".png" 
        elif a == "高斯噪声": 
            imgg=img2
            newname=portion[0]+"_gs"+".png" 
        cv2.imencode('.png',imgg)[1].tofile(path + newname)#可以选择添加噪声类型
    
    # 1的话批量处理
    elif index == 1:
        filename = openfile()
        path = gml + "\\noise-image\\"
        isExists=os.path.exists(path)
        # 判断结果
        if not isExists:
            # 如果不存在则创建目录
            # 创建目录操作函数
            os.makedirs(path) 
        buf = os.listdir(filename)
        for i in buf:
            img = cv2.imdecode(np.fromfile(filename + "/" + i,dtype=np.uint8),1)
            img1 = PepperandSalt(img, 0.002) #添加椒盐噪声
            img2 = GaussianNoise(img, 20, 40, 0.01) #添加高斯噪声
            portion=os.path.splitext(i)

            if  a == "椒盐噪声": 
                imgg=img1
                newname=portion[0]+"_jy"+".png" 
            elif a == "高斯噪声": 
                imgg=img2
                newname=portion[0]+"_gs"+".png" 
            cv2.imencode('.png',imgg)[1].tofile(path+newname)
    tkinter.messagebox.showinfo(message='done！') 
        

# 对比度变化
def APP2():
    # 对比度变换有高低两种典型值，可以设置他们分别对应一个参数
    # a>1时对比度增加, a<1时对比度降低
    if combobox.get() == "增加对比度": a=2
    elif combobox.get() == "降低对比度": a=0.5

    index = getInput2("选择图片处理数量")
    gml = os.path.realpath(os.path.dirname(sys.argv[0]))
    path = gml + "\\contrast_change-image\\"
    isExists=os.path.exists(path)
    # 判断结果
    if not isExists:
        # 如果不存在则创建目录
        # 创建目录操作函数
        os.makedirs(path)
    # 0的话处理单幅图像
    if  index == 0:
        filename = openimg()
        img = cv2.imdecode(np.fromfile(filename,dtype=np.uint8),cv2.IMREAD_GRAYSCALE)
        img = contrast_change(img,a)


        imgname = filename.split("/")[-1]
        portion=os.path.splitext(imgname)

        if  a == 2: 
            newname=portion[0]+"_high"+".png" 
        elif a == 0.5: 
            newname=portion[0]+"_low"+".png" 

        cv2.imencode('.png',img)[1].tofile(path + newname)
    
    # 1的话批量处理
    elif index == 1:
        filename = openfile()
        buf = os.listdir(filename)
        for i in buf:
            img = cv2.imdecode(np.fromfile(filename + "/" + i,dtype=np.uint8),cv2.IMREAD_GRAYSCALE)
            img = contrast_change(img,a)
            portion=os.path.splitext(i)
            if  a == 2: 
                newname=portion[0]+"_high"+".png" 
            elif a == 0.5: 
                newname=portion[0]+"_low"+".png" 

            cv2.imencode('.png',img)[1].tofile(path+newname)
    tkinter.messagebox.showinfo(message='done！') 

# 弹窗选择单幅图像处理还是批量图像处理
def getInput2(title):
    def return_callback():
        global a
        a = 0
        root.quit()
    def return_callback2():
        global a
        a = 1
        root.quit()
    root = Tk(className=title)
    root.wm_attributes('-topmost', 1)
    screenwidth, screenheight = root.maxsize()
    width = 270
    height = 90
    size = '%dx%d+%d+%d' % (width, height, (screenwidth - width)/2, (screenheight - height)/2)
    root.geometry(size)
    root.resizable(0, 0)

    b1 = tk.Button(root, text="单幅图像处理", pady=5, anchor='center',command=return_callback,font=("黑体",15)).pack()
    b2 = tk.Button(root, text="批量图像处理", pady=5, command=return_callback2,font=("黑体",15)).pack()

    root.mainloop()
    root.destroy()
    return a

def getInput(title):
    def return_callback(event):
        root.quit()
    root = Tk(className=title)
    root.wm_attributes('-topmost', 1)
    screenwidth, screenheight = root.maxsize()
    width = 300
    height = 60
    size = '%dx%d+%d+%d' % (width, height, (screenwidth - width)/2, (screenheight - height)/2)
    root.geometry(size)
    root.resizable(0, 0)
    label = tk.Label(root, text="高斯斑图数量对应合成图数量",font=("黑体",15))
    label.pack()
    entry = Entry(root,font=("黑体",15))
    entry.bind('<Return>', return_callback)
    entry.pack()
    entry.focus_set()
    root.mainloop()
    str = entry.get()
    root.destroy()
    return str

def image_generated(sfnameaa):
    global book
    aa = getInput("生成高斯斑图数量")
    aa = int(aa)
    
    book = xlrd.open_workbook(sfnameaa)
    o1=5 #x方差
    o2=20 #y方差
    p=0  #关联系数，会影响目标形状
    sheet1 = book.sheets()[0] # 获取第一个sheet页
    rows = sheet1.nrows # 获取总行数
    cols = sheet1.ncols # 获取总列数
    gml = os.path.realpath(os.path.dirname(sys.argv[0]))
    #gml = os.path.split(os.path.abspath(__file__))[0]

    for i in range(0, aa):
        phi_up = sheet1.cell_value(i, 0) #获取表格中的phi_up(俯仰角）
        psi_up = sheet1.cell_value(i, 1) #获取表格中的psi_up(偏航角）
        f = sheet1.cell_value(i, 2) #获取表格中的相机焦距
        X=1023+f*math.tan(psi_up*sp.pi/180)/11 #计算X值,即均值
        Y=511-f*math.tan(phi_up*sp.pi/180-27.5*sp.pi/180)/11 #计算Y值，即均值
        x,y=np.mgrid[0:1023:1024j,0:2047:2048j] #绘制网格
        # 求概率密度函数
        z = (1 / (2 * math.pi * o1 * o2 * pow(1 - pow(p, 2), 0.5))) * np.exp((-1 / (2 * (1 - p * p)) )* (((y - X) * (y - X)) / (o1 * o1) - 2 * p * (y - X) * (x - Y) / (o1 * o2) + (x- Y) * (x - Y) / (o2 * o2)))
        plt.rcParams['figure.figsize'] = (26.43, 52.85) #调整保存图像尺寸大小
        plt.axis('off') #去掉坐标轴
        plt.imshow(z,cmap='gray')
        
        path = gml + "\\generated-image\\"
        isExists=os.path.exists(path)
        # 判断结果
        if not isExists:
            # 如果不存在则创建目录
            # 创建目录操作函数
            os.makedirs(path) 

        plt.savefig(path + "{}.jpg".format(i),bbox_inches='tight',pad_inches=0.0) #保存图像
    tkinter.messagebox.showinfo(message='done！')   

def main():
    global combobox
    global combobox1
    global varlist
    global listbox1
    global sfnameaa
    global book
    

    root.title("图像合成仿真")
    root.geometry("500x280")

    B = tk.Button(root, text="图像合成",command=lambda:thread_it(hc),font=("黑体",15))
    B.pack(anchor='n', pady=10)

    # 创建列表选项
    # 这里用弹窗代替了
    # varlist = tkinter.StringVar()
    # listbox1 =Listbox(root, width=11, height=2,listvariable=varlist)
    # listbox1.pack(anchor='w')
    # # i表示索引值，item 表示值，根据索引值的位置依次插入
    # for i,item in enumerate(["单幅图像处理","批量处理"]):
    #     listbox1.insert(i,item)
    
    B1 = tk.Button(root, text="几何变换", command=lambda:thread_it(APP),font=("黑体",15))
    B1.pack(anchor='center', pady=6)
    B2 = tk.Button(root, text="噪声模拟", command=lambda:thread_it(APP1),font=("黑体",15))
    B2.pack(anchor='center', pady=6)
    var1 = tkinter.StringVar()
    combobox1 = tkinter.ttk.Combobox(root, textvariable=var1, value=("高斯噪声","椒盐噪声"),font=("黑体",15))
    # 设置默认选项
    combobox1.current(0)
    combobox1.pack(pady=3)
    B3 = tk.Button(root, text="对比度变化", command=lambda:thread_it(APP2),font=("黑体",15))
    B3.pack(anchor='center', pady=6)


    var = tkinter.StringVar()
    combobox = tkinter.ttk.Combobox(root, textvariable=var, value=("增加对比度","降低对比度"),font=("黑体",15))
    # 设置默认选项
    combobox.current(0)
    combobox.pack()
    
    root.mainloop()

if __name__=='__main__':
    # plt在设计上存在一个问题，就是只能在主线程中运行，因此无法放入Button中，会直接报错
    # 所以这里在main函数中创建了个multiprocessing来运行这个用到plt处理图像的函数
    multiprocessing.freeze_support()
    root = tk.Tk()
    sfnameaa = filedialog.askopenfilename(title='选择小目标位置数据文件')
    job_for_another_core = multiprocessing.Process(target=image_generated,args=(sfnameaa,))
    job_for_another_core.start()
    main()
```



## tkinter使用中需要注意的问题

### 1.解决函数执行时界面卡死

在python主进程中创建一个个线程，将函数扔进里面

```python
import threading
def thread_it(func, *args):
    # 创建线程
    t = threading.Thread(target=func, args=args)
    # 守护线程
    t.setDaemon(True)
    # 启动线程
    t.start()
 
B1 = tk.Button(root, text="几何变换", command=lambda:thread_it(APP),font=("黑体",15))
```

### 2.通过multiprocessing多进程解决plt无法在线程中运行的问题

plt在设计上存在一个问题，就是只能在主线程中运行，因此无法放入Button中，会直接报错。所以这里在main函数中创建了个multiprocessing来运行这个用到plt处理图像的函数

```python
import multiprocessing

if __name__=='__main__':
    # 新增下面一行代码即可打包多进程，注意这句话要放在最上面
    multiprocessing.freeze_support()
    root = tk.Tk()
    sfnameaa = filedialog.askopenfilename(title='选择小目标位置数据文件')
    # 创建进程
    job_for_another_core = multiprocessing.Process(target=image_generated,args=(sfnameaa,))
    # 启动进程
    job_for_another_core.start()
    main()
```

### 3.绝对路径

不能有相对路径，如果有的话可需要获取.exe所在目录，然后转换成绝对路径

### 4.获取当前路径的正确方式

```python
# 正确方法：通过sys.argv[0]的值来判断
# sys.argv[0] 是指命令行上可执行文件的路径.
os.path.realpath(os.path.dirname(sys.argv[0]))

# 正确方法二：通过sys.executable的值来判断
# 这个方法只在打包成exe后可以正确获取到路径，所以不建议用
# sys.executable是 Python 解释器的可执行二进制文件的绝对路径,直接在pycharm里面运行时候,sys.executable获取的python.exe的路径,但是python脚本打包成exe之后,sys.executable就是exe所在的文件路径了.
os.path.realpath(os.path.dirname(sys.executable))


# 错误方法：通过python的__file__来判断脚本路径以此作为当前文件路径
# 通过pyinstaller打包的exe,再通过这种方式获取当前路径,获取出来的却是C:\Users\Windows账号名\AppData\Local\Temp\_xxxx 这样的路径.
os.path.realpath(os.path.dirname(__file__))


# 补充
os.path.dirname()  # 去掉脚本的文件名，返回目录。
os.path.realpath(__file__)  # 获取当前执行脚本的绝对路径。
```

### 5.解决cv2.imread等函数不能存在中文路径问题

```python
# 代替imread
img = cv2.imdecode(np.fromfile(filename,dtype=np.uint8),cv2.IMREAD_GRAYSCALE)
# cv2.imdecode也是两个参数，第二个参数和cv.imread一致
# cv2.IMREAD_COLOR：默认参数，读入一副彩色图片，忽略alpha通道，可用1作为实参替代
# cv2.IMREAD_GRAYSCALE：读入灰度图片，可用0作为实参替代
# cv2.IMREAD_UNCHANGED：顾名思义，读入完整图片，包括alpha通道，可用-1作为实参替代

# 代替imwrite
 cv2.imencode('.png',img)[1].tofile(path+newname)
# imencode(ext, img[, params])
# ext: 图像后缀，".bmp"、".jpg"等cv2模块支持的图像格式
# img: 数据类型为numpy.ndarray的图像，shape为height × \times × width × \times ×channel
# params: 图像保存时的参数，与cv2.imwrite(filename, img[, params])中参数一致
```

### 6.自动创建文件夹

```python
isExists=os.path.exists(path)
# 判断结果
if not isExists:
# 如果不存在则创建目录
# 创建目录操作函数
	os.makedirs(path) 
```

### 7.将文件夹下所有文件加一个后缀

```python
buf = os.listdir(path)  # os.listdir() 方法用于返回指定的文件夹包含的文件或文件夹的名字的列表
    for i in buf:
        portion=os.path.splitext(i)  # 将文件名称和后缀分开
        # imgname = filename.split("/")[-1]是取最后一个/后的文件名
        newname=portion[0]+"_jh"+".png" 
```

### 8.通过弹窗的窗口在其他函数内进行功能选择或参数输入

```python
# 弹窗选择单幅图像处理还是批量图像处理
def getInput2(title):
    def return_callback():
        global a
        a = 0
        root.quit()
    def return_callback2():
        global a
        a = 1
        root.quit()
    root = Tk(className=title)
    root.wm_attributes('-topmost', 1)
    screenwidth, screenheight = root.maxsize()
    width = 270
    height = 90
    size = '%dx%d+%d+%d' % (width, height, (screenwidth - width)/2, (screenheight - height)/2)
    root.geometry(size)
    root.resizable(0, 0)

    b1 = tk.Button(root, text="单幅图像处理", pady=5, anchor='center',command=return_callback,font=("黑体",15)).pack()
    b2 = tk.Button(root, text="批量图像处理", pady=5, command=return_callback2,font=("黑体",15)).pack()

    root.mainloop()
    root.destroy()
    return a

def getInput(title):
    def return_callback(event):
        root.quit()
    root = Tk(className=title)
    root.wm_attributes('-topmost', 1)
    screenwidth, screenheight = root.maxsize()
    width = 300
    height = 60
    size = '%dx%d+%d+%d' % (width, height, (screenwidth - width)/2, (screenheight - height)/2)
    root.geometry(size)
    root.resizable(0, 0)
    label = tk.Label(root, text="高斯斑图数量对应合成图数量",font=("黑体",15))
    label.pack()
    entry = Entry(root,font=("黑体",15))
    entry.bind('<Return>', return_callback)
    entry.pack()
    entry.focus_set()
    root.mainloop()
    str = entry.get()
    root.destroy()
    return str
```

### 9.通过from xx import减小exe程序的大小

尽量使用 `from xx import xxx`方式导入，这样打包后的exe程序的大小相比直接使用`import xx` 会小很多。我本来写了一个很小的程序，结果打包好后竟然达到了213M，后来精简导包方式，压缩到34M。





# 打包成exe：

## tkinter

* pip install pyinstaller

* cd 到所在目录下

* pyinstaller -F -w ax.py
  * **-F：**打包后只生成单个exe格式文件；
  *  **-D：**默认选项，创建一个目录，包含exe文件以及大量依赖文件；
  * **-c：**默认选项，使用控制台(就是类似cmd的黑框)；
  * **-w：**不使用控制台；但要是命令行程序的话就需要保留控制台
  * **-p：**添加搜索路径，让其找到对应的库；
  * **-i：**改变生成程序的icon图标。

* 之后除了exe的其他文件都可以删除掉

