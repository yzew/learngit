## 视觉软件开发

结果获取与保存：实现了65+个工具12个派生类的结果获取函数，通过工厂模式派分，并使用多重嵌套的Dictionary保存；

通信交互协议：通过TCP通信控制图像任务执行，参数设置，结果输出，并兼容VM协议；
（1）支持 任务指令关联关键字配置，任务窗口关联配置
（2）支持 任务输出参数格式配置
（3）支持 任务切换
（4）支持 服务IP和端口设置

显示界面配置：图像任务显示窗口配置，图像实时显示，任务配置入口
（1）支持 单窗口和多窗口模式
（2）支持 窗口默认配置设置
（3）支持 任务绑定显示窗口
（4）支持 实时显示（绑定指定任务）开启和关闭
（5）支持 指令控制配置界面弹出，交互配置界面弹出，修改为UI直接配置





**技术栈：**C# Winform 

反编译工具：dnSpy、序列化与反序列化：Newtonsoft.Json

**项目背景：**已有的视觉软件不支持通信指令交互，无法结集成到自研平台中，且任务执行结果的获取受限。

**项目描述：**对视觉软件进行二次开发，提供界面与接口进行任务执行结果的配置、获取与返回，并可通过网络调试工具及自有平台进行调用。

**代码量：**1000行

**项目难点：**视觉软件存在65+个可选工具，每个工具对应着若干派生类及不同样式的多层嵌套的结果。对任务结果的获取与存储、配置文件的设计、界面的交互及展示的设计等是项目的重点。

比如结果字典中是某个派生类，需要输出的结果有location、matrix、line等，line中又会有角度、point等，point中又会有坐标。
因此以什么样的格式来存储这些结果，来方便界面软件的显示

**实现功能：**

1、对于新加入的视觉工具，可自动获取工具的输出结果格式并保存到JSON配置文件中

2、通过 Winform 搭建界面进行交互

3、启动程序后，软件界面会自动获取任务列表并展示。在任务列表中选中任务后，会在四级嵌套的菜单中显示当前任务可输出的结果列表，可对需要获取的结果进行选择，后续任务的执行便会返回设定格式的结果。 

4、配置好输出结果后，发送任务指令就会执行程序，通过多态对视觉软件执行的结果进行类型转换及函数调用获取需要的结果，并使用多重嵌套的Dictionary<string, dynamic>保存任务执行结果



**代码：**

changeJson类库（100）：changeJSON命名空间，下面有outputJSON类，存储工具名及Dictionary<string, object>格式的结果；JsonConfig类负责对outputJSON对象进行序列化与反序列化，读取、写入配置文件等

OutputProcess类库（320）：主函数getResult，输入类型及基类对象，根据不同的结果类型将基类转换为不同的派生类，再调用对应的函数一层层获取结果，存储在层层嵌套的Dictionary<string, dynamic>中，再以dynamic格式返回。

Winform（467）：

Config功能的点击事件：执行一次任务并获取结果，将结果格式存储在配置文件中

四级菜单列表的触发事件：对配置文件中任务名对应的结果进行解析与展示，点击后将要输出的结果以string类型及“,”分隔符增加到输出列表中，并展示在textbox中

CmdServer类：调用了异步tcp的dll库进行数据的通信，其中比较重要的是接收到数据后的触发事件。

对输入的命令进行完全匹配/部分匹配，然后调用自研软件的DLL库执行任务，获取任务执行结果后通过OutputProcess类库对结果进行解析，并根据设定的输出结果进行筛选及展示

**执行流程：**

启动程序后，软件界面会自动获取任务列表并展示。

比如我们要执行一个任务，就在任务列表中选择，然后四级嵌套的菜单中会自动获取此任务的最后一个工具（任务的输出）对应的结果列表。

在结果列表中选择的结果选项会被存在程序中，之后执行任务时会输出选择的这些结果；若是不进行结果选择的话，会默认输出x y angle这些通用的输出。

配置好输出结果后，发送任务指令就会执行程序，通过多态对执行结果进行转换，使用多重嵌套的Dictionary<string, dynamic>保存任务执行结果





```c#
public Dictionary<string, string> GetLocation(dynamic result, dynamic tmp)
{
    Dictionary<string, string> location = new Dictionary<string, string>();
    Type aa = tmp.GetType();
    MethodInfo[] methods = aa.GetMethods();
    // 筛选返回值是list<>形式的
    IEnumerable<MethodInfo> listMethods = methods.Where(m =>
        m.ReturnType.IsGenericType &&
        m.ReturnType.GetGenericTypeDefinition() == typeof(List<>));
    // 调用每个方法
    foreach (MethodInfo method in listMethods)
    {
        object lst = Activator.CreateInstance(method.ReturnType);
        lst = method.Invoke(result, null);
        if (lst is IEnumerable)
        {
            foreach (var item in (IEnumerable)lst)
            {
                location.Add(method.Name.Substring(4), item.ToString());
                // HwvLogManager.AddOkLog($"{item}");
            }
        }
    }
    return location;
}
```



```c++
lst2.ForEach(val => this.listBox2.Items.Add(val)); 
```



```c#
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace changeJSON
{
    public class outputJSON
    {
        /// <summary>
        /// 指令结构
        /// </summary>
        public string Name;

        /// <summary>
        /// 返回值
        /// </summary>
        public string value;
        
        
        /// <summary>
        /// TOP500W,1,message,x,y
        /// </summary>
        public outputJSON(string Name_, string value_)
        {
            Name = Name_;
            value = value_;
        }
    }

  public class JsonConfig
    {
 
        private static Dictionary<int, string> configDic = new Dictionary<int, string>();
 
        /// <summary>
        /// 根据key查value
        /// </summary>
        /// <param name="key"></param>
        /// <returns></returns>
        public static outputJSON ReadConfig(int key)
        {
            if (File.Exists("D:\\TestVision\\TestVision\\config.json") == false)//如果不存在就创建file文件夹
            {
                FileStream f = File.Create("D:\\TestVision\\TestVision\\config.json");
                f.Close();
            }
            string s = File.ReadAllText("D:\\TestVision\\TestVision\\config.json");
            try
            {
                // 反序列化
                configDic = JsonConvert.DeserializeObject<Dictionary<int, string>>(s);
            }
            catch
            {
                configDic = new Dictionary<int, string>();
            }
 
            if (configDic != null && configDic.ContainsKey(key))
            {
                return JsonConvert.DeserializeObject<outputJSON>(configDic[key]);
            }
            else
            {
                return JsonConvert.DeserializeObject<outputJSON>(string.Empty);
            }
        }
 
        /// <summary>
        /// 添加key-value信息
        /// </summary>
        /// <param name="key"></param>
        /// <param name="value"></param>
        public static void WriteConfig(int key, outputJSON value)
        {
            if (configDic == null)
            {
                configDic = new Dictionary<int, string>();
            }
            
            configDic[key] = JsonConvert.SerializeObject(value);
            // 序列化
            string s = JsonConvert.SerializeObject(configDic);
            File.WriteAllText("D:\\TestVision\\TestVision\\config.json", s);
        }
 
        /// <summary>
        /// 删除key对应的配置信息
        /// </summary>
        /// <param name="key"></param>
        public static void DeleteConfig(int key)
        {
            if (configDic != null && configDic.ContainsKey(key))
            {
                configDic.Remove(key);
                string s = JsonConvert.SerializeObject(configDic);
                File.WriteAllText("D:\\TestVision\\TestVision\\config.json", s);
            }
        }
    }
}
```
