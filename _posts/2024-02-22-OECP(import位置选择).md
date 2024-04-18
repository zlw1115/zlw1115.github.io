---
layout:     post
title:      import位置选择
subtitle:   
date:       2024-02-22
author:     Zeng Liwen
header-img: 
catalog: 	 true
tags:
    - work
---

[toc]

1. import位置选择是写在模块头还是函数里？
2. 写法不同有什么区别？
3. 怎么选择导入的位置？

### 0.import 介绍

模块的搜索路径：当执行导入一个模块时(import xxx)，默认情况下python解释器会搜索当前目录，已安装的内置模块和第三方模块。搜索路径存放在sys模块的path中。如果缺省的sys.path中没有含有自己的模块或包的路径，可以动态的加入(sys.path.apend)即可。

- sys.path的添加规则:

  1. sys.path第一个路径是包含输入脚本的目录。在交互环境下，会添加一个空项，对应当前目录。

  2. 如果PYTHONPATH环境变量存在，sys.path会加载此变量指定的目录。

  3. 尝试查找Python Home，如果设置了PYTHONHOME环境变量，将认为这就是Python Home。否则，将使用python.exe所在的目录找到lib/os.py去推断Python Home。

     如果确实找到了Python Home，则相关的子目录(Lib、plat-win、lib-tk等)将以Python Home为基础加入到sys.path，并导入（执行）lib/site.py，将site-specific目录及其下的包加入。

     如果没有找到Python Home，则把注册表Software/Python/PythonCore/2.5/PythonPath的项加入到sys.path(HKLM和HKCU合并后加入)，但相关的子目录不会自动添加的。

  4. 如果我们没有找到Python Home，并且没有PYTHONPATH环境变量，并且不能在注册表中找到PythonPath，那么缺省相对路径将加入(如：./Lib;./plat-win等)

即：

当在安装好的主目录中运行Python.exe时，首先推断Python Home，如果找到了Python Home，注册表中的PythonPath将被忽略；否则将注册表中的PythonPath加入。

如果PYTHONPATH环境变量存在，sys.path肯定会加载此变量指定的目录。

如果Python.exe在另外的一个目录下(不同的目录，比如通过COM嵌入到其他程序)，Python Home将不推断，此时注册表的PythonPath将被使用；如果Python.exe不能发现他的主目录(PythonHome)，并且注册表也没有PythonPath，则将加入缺省的相对目录。

- 标准import

  Python中所有加载到内存的模块都放在sys.modules。当import一个模块时首先会在这个列表中查找是否已经加载了此模块，如果加载了则只是将模块的名字加入到正在调用import的模块的Local名字空间中；如果没有加载则从sys.path目录中按照模块名查找模块文件，模块文件可以是py、pyc、pyd，找到后将模块载入到内存，并加入到sys.modules中，并将名称导入到当前的Local名字空间。由此，一个模块不会重复载入。多个不同的模块都可以用import引入同一个模块到自己的Local名字空间，其背后的PyModuleObject对象只有一个。

- 嵌套Import

  - 本地模块导入A模块(import A)，而A中又有import语句，会激活另一个import动作，入import B，而B模块又可以import其他模块，一直下去。此种情况需要注意的一点是，各个模块的Local名字空间是独立的，所以，本模块import A完了后本模块只能访问模块A，不能访问B及其他模块。虽然模块B已经加载到内存了，如果要访问还要在明确的在本模块中import B。

  - 在模块A中import B，而在模块B中import A。这是会怎样了？

    - [A.py]

      from B import D

      class C:pass

    - [B.py]

      from A import C

      class D:pass

    为什么执行A的时候不能加载D了？

    如果将A.py改为import B就可以了。

    这是怎么回事？

    from B import D，python内部会执行下面几个步骤：

    1. 在sys.modules中查找符号“B”。

    2. 如果符号B存在，则获得符号B对应的module对象\<module B>，从\<module B>的\__dict__中获得符号“D”对应的对象，如果“D”不存在，则抛出异常。

    3. 如果符号B不存在，则创建一个新的module对象\<module B>，注意，这时module对象的\__dict__为空。执行B.py中的表达式，填充\<module B>的\__dict__。

       从\<module B>的\__dict__中获得“D”对应的对象，如果“D”不存在，则抛出异常。

所以，这个例子的执行顺粗如下：

1. 执行A.py中的from B import D

   由于是执行的python A.py，所以在sys.modules中并没有\<module B>存在，首先为B.py创建一个module对象(\<module B>)，注意，这时创建的这个module对象是空的，里边啥也没有，在python内部创建了这个module对象之后，就会解析执行B.py，其目的是填充\<module B>这个dict。

2. 执行B.py中的from A import C

   在执行B.py的过程中，会碰到这一句，首先检查sys.modules这个module缓存中是否已经存在\<module A>了，由于这是缓存还没有缓存\<module A>，所以类似的，Python内部会为A.py创建一个module对象(\<module A>)，然后，同样的，执行A.py中的语句。

3. 再次执行A.py中的from B import D

   这时，由于在第1步时，创建的\<module B>对象已经缓存在了sys.modules中，所以直接就得到了\<module B>，但是，注意，从整体过程来看，我们知道，这时\<module B>还是一个空的对象，里面啥也没有，所以从这个module中获得符号"D"的操作就会抛出异常。如果这里只是import B，由于"B"这个符号再sys.module中已经存在，所以不会抛出异常。

### 1. import位置介绍

有如下两个模块Main、Zoom，现需在Main模块里导入Zoom模块里的ZoomList列表

```
Zoom.py
ZoomList = [
	"Cat",
	"Dog",
	"Fish"
]
-------
Main.py -- 1
from Zoom import ZoomList

def Demo():
  len(ZoomList)

Demo()
-------
Main.py -- 2
def Demo():
  from Zoom import ZoomList
  len(ZoomList)

Demo()

```

无论是在模块头import还是函数里import，都能达到运行的目的

### 2. 写法不同有什么区别？

**内存效率角度**

涉及到在模块import和函数import出来的东西在存储方式上的不同：

- 在Main模块头import

  在Main模块头import了ZoomList，本质是，模块生成了一个全局引用，引用了ZoomList，这个引用会作为这份模块的属性，常驻在Main模块里。实际效果等同于执行了Main.ZoomList = Zoom.ZoomList，此后模块里访问ZoomList，直接访问的是Main.ZoomList，而非Zoom.ZoomList，虽然此刻Main.ZoomList与Zoom.ZoomList引用的是同一个列表。

- 在Demo函数里import

  在Demo函数里import了ZoomList，本质是，函数里生成了一个局部变量，引用了ZoomList，这个变量与函数里其他局部变量一样，它是存在函数的栈里，当函数执行结束后，该引用会被释放。即，函数里import出来的东西，这个引用跟Main模块没什么关系。

通过对比，相对于函数里import，在模块头import时Main模块多生成一个引用，虽然一个引用的内存大小微乎其微几乎可以无视，但严格来说确实在模块头import它消耗的内存会相对多一点。

**运行效率角度**

这涉及到在访问ZoomList时，分别干了什么。

- 在Main模块头import

  Demo访问的ZoomList直接从全局引用拿即可

- 在Demo函数里import

  Demo需要额外运行一次导入操作，即from Zoom import ZoomList，也就是说Python解析器需要多解析了一条语句，去完成一次import流程。

理论上从运行效率而言，显然在Main模块头import时更好。

**逻辑稳定性和数据一致性角度**

- 在Main模块头import

  从Main模块头导入ZoomList时，Main模块生成了一个引用Main.ZoomList，它与Zoom.ZoomList引用了同一个列表。

  但一般从代码逻辑而言，无论何时，在逻辑上，是要访问Zoom.ZoomList，而非Main.ZoomList；当Zoom.ZoomList引用了新列表，Main.ZoomList仍然引用了旧列表，随后Main模块所有的代码引用ZoomList的旧列表，而非当前Zoom.ZoomList引用的新列表，这就导致了不一致，代码发生逻辑错误。

  ```
  Main.py
  
  import Zoom
  from Zoom import ZoomList
  
  def Demo():
    print("Main模块里的ZoomList列表", ZoomList)
    
  Zoom.ZoomList = ["Godeng"]
  Demo()
  print("Zoom模块里的ZoomList列表", Zoom.ZoomList)
  ```

- 在Demo函数里import

  由于在函数里执行了导入语句，导入的就是当前的Zoom模块的ZoomList引用的列表，在逻辑上是绝对稳定一致的。

  ```
  Main.py
  
  import Zoom
  def Demo():
    from Zoom import ZoomList
    print("Main模块里的ZoomList列表",ZoomList)
    
  Zoom.ZoomList = ["Godeng"]
  Demo()
  print("Zoom模块里的ZoomList列表",Zoom.ZoomList)
  Demo()
  ```

**极端情况**

对于上一个场景的解决方案：Zoom模块是全局变量，类、函数等等都有可能发生变化，但是Zoom模块没有变，只需在模块头import的最小导入层级是模块，不就可以解决了吗？

绝大部分情况下，即便Reload热更模块或是其他操作，模块对象是相对恒定不变的，可以说是完全解决了运行过程中数据逻辑不一致问题。

```
Main.py

# 限定import最低层层级为Zoom
import Zoom

def Demo():
  print("Main模块里的ZoomList列表",Zoom.ZoomList)

print("变更前")
Demo()
print("Zoom模块里的ZoomList列表",Zoom.ZoomList)
print("变更后")
Zoom.ZoomList = ["Godeng"]
Demo()
print("Zoom模块里的ZoomList列表",Zoom.ZoomList)

```

看似解决了问题，但其实是依赖于某种常见条件下的解决方案，在某些极端情况下其实它仍然保证不了。

限定模块为最小导入层级，只是保证了模块里的东西的逻辑一致，但当模块他本身就会被替换时，那么仍然会发生同样的逻辑错误。

模块Main导入ZoomList时，会生成一个引用Zoom.ZoomList引用；当模块Main导入模块Zoom时，模块Main也相似的产生一个Main.Zoom引用了Zoom模块，如果逻辑上Zoom模块被替换了了？

```
Zoom2.py

ZoomList = [
  "Sheep",
]
```

在程序运行过程中，逻辑上，当Zoom2取代Zoom时，代码里的所有关于Zoom模块的操作都会操作到Zomm2模块。

- 在Main模块头import

```
Main.py
在模块头import模块情况下，Zoom模块被替换后发生逻辑错误

import sys
import Zoom

def Demo():
  print("逻辑上一样的Zoom模块，在模块里内容",Zoom.ZoomList)
  
  
# 动态替换，逻辑上是Zoom，实际上是Zoom2的内容
import Zoom2
sys.modules.pop("Zoom")
sys.modules["Zoom"] = Zoom2

#当前模块的Zoom
Demo()
#实际上最新的Zoom
import Zoom
print("实际上一样的Zoom模块，在模块里内容",Zoom.ZoomList)

```

上例在运行过程中，将sys.modules里的名为"Zoom"的模块对象替换成Zoom2模块对象，逻辑上是希望程序里访问Zoom.ZoomList是将会访问Zoom2.ZoomList，而不是被替换掉的那个Zoom模块的ZoomList。同时由于模块Main引用了旧的Zoom模块，关于逻辑上的Zoom的访问都是错误的。

- 在Demo函数里import

  ```
  Main.py
  在函数里import，逻辑上的Zoom一致
  
  import sys
  
  def Demo():
    import Zoom
    print("逻辑上一样的Zoom模块，在模块里内容", Zoom.ZoomList)
    
  #动态替换模块，逻辑上是Zoom，实际上是Zoom2的内容
  import Zoom2
  sys.modules["Zoom"] = Zoom2
  # 当前模块的Zoom
  Demo()
  # 实际上最新的Zoom
  import Zoom
  print("逻辑上一样的Zoom模块，在模块里内容", Zoom.ZoomList)
   
  ```

**不可替代性情景的角度**

模块间的循环导入

在实际开发中，在模块头import非常容易导致循环导入，使得程序报错无法继续跑；而在函数里import，几乎不会产生此类问题。

模块的循环引用，包括双向引用，表示模块间联系紧密，理想的解决方式是，抽出一个公共的中介模块完成两个模块间的沟通，或者将多个模块合为一体。但是当项目规模变大，模块数量极其庞大且联系复杂时，将会是个灾难。

这个时候选择在函数里import避开循环导入，往往是比较好的选择。

### 3.选择

1. - **Main**模块里**只有**一个地方需要**Zoom**时，在函数里**import**舒服，毕竟不需要翻到模块头**import。**
   - **Main**模块多个地方使用，在模块头**import**舒服，毕竟只需要写一句import，而不需要在每个函数里写，这也说明**Main**模块和**Zoom**模块息息相关，往后极大概率会继续使用。
2. 如果没循环引用的情况下，在模块头**import**的最小导入层级限制为模块，就可以了