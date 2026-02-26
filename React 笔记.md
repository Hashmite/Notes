# Linux  安装

1.  下载： [VM](https://www.sysgeek.cn/install-vmware-workstation-pro/#0-下载-vmware-workstation-pro-个人免费版) 和 镜像 [centos7](https://wiki.centos.org/Download.html) [centos7](https://vault.centos.org/7.6.1810/isos/x86_64/) [阿里镜像](https://mirrors.aliyun.com/centos/7/isos/x86_64/)

```markdown
最新密匙
    4A4RR-813DK-M81A9-4U35H-06KND    NZ4RR-FTK5H-H81C1-Q30QH-1V2LA     
    JU090-6039P-08409-8J0QH-2YR7F    4Y09U-AJK97-089Z0-A3054-83KLA 
    4C21U-2KK9Q-M8130-4V2QH-CF810   MC60H-DWHD5-H80U9-6V85M-8280D
通用密钥
    JU090-6039P-08409-8J0QH-2YR7F
```

2. 网络配置
   1. 手动配置网络：修改： /etc/sysconfig/network-script/ifcfg-ens33

```mariadb
IPADDR=192.168.1.11 #网卡IP地址
NETMASK=255.255.255.0 #子网掩码
GATEWAY=192.168.1.2 #网卡网关地址
DNS1=8.8.8.8 #网卡DNS地址

重启使其生效。使用命令：service network restart / systemctl restart network
```

3. 修改yum 源
   1. 下载 [wget](https://mirrors.aliyun.com/centos/7.9.2009/os/x86_64/Packages/) 并安装rpm -ivh  xxx.rpm  

```markdown
将yum源设置为国内镜像站点。国内主要开源的开源镜像站点应该是网易和阿里云
    修改CentOS默认yum源为mirrors.aliyun.com
              ◊ 1、首先备份系统自带yum源配置文件/etc/yum.repos.d/CentOS-Base.repo
                        } mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
              ◊ 2、下载ailiyun的yum源配置文件到/etc/yum.repos.d/
                        } wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
              ◊ 3、运行yum makecache生成缓存
                        } yum makecache  
              ◊ 4、这时候再更新系统就会看到以下mirrors.aliyun.com信息 
                        } yum -y update          
```

1. 安装上传下载工具： yum -y install lrzsz 
2.  Linux 系统 必须安装：yum -y install glibc.i686
3.  Linux 上安装找不到无用包解决方案： yum -y install epel-release

4. 修改防火墙

> Linux防火墙类型主要有iptables和firewalld两种
>
> - iptables是一种静态防火墙，早期在Linux系统中广泛使用，其配置文件位于/etc/sysconfig/iptables中。
> - 而firewalld则是一种动态防火墙，目前已经取代了iptables，配置文件位于/usr/lib/firewalld和/etc/firewalld中



常用的firewalld命令：



1. systemctl start  /disable/enable /status  firewalld
2. 名称   执行命令刷新虚拟防火墙:   firewall-cmd --reload 
3. 查看当前防火墙规则：   firewall-cmd --list-all
4. 查看已经开发的端口   Firewall-cmd --list-ports 
5.  允许特定端口的入站连接   a. firewall-cmd --zone=public  --add/--remove-port=<端口号>/tcp  --permanent
6. 允许特定IP地址范围的入站连接

```markdown
firewall-cmd --zone=public --add-source=<IP地址/子网掩码> --permanent
             - 列如：允许来自192.168.0.0/24 子网的连接
        -         firewall-cmd --zone=public --add-source=192.168.0.0/24 --permanent
查看本机已经开启的端口号
              - centos7以下使用: netstat -ant,7使用ss ：ss -ant
              
Firewalld 端口转发
        firewall-cmd --query-masquerade # 检查是否允许伪装IP
        firewall-cmd --add-masquerade # 允许防火墙伪装IP
        firewall-cmd --remove-masquerade# 禁止防火墙伪装IP
            man firewall-cmd                               ##查看帮助
        
    # 将80端口的流量转发至8080
    firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080
    
    # 将80端口的流量转发至192.168.0.1的8080端口
    firewall-cmd --add-forward-port=port=80:proto=tcp:toaddr=192.168.0.1:toport=8080
软连接
   ln -s  目标指令1   目标指令2
        给nginx 建立一个可以全局访问的软连接
     ln -s  /usr/local/nginx/sbin/nginx   /usr/local/bin/nginx
    
配置jdk 环境变量：
    export JAVA_HOME=/opt/jdk1.8
    export PATH=$JAVA_HOME/bin:$PATH
// export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar             
```





5.  处理Linux上时间不同步问题
   1. 查看服务器是否安装了ntp， 系统默认安装了 ntpdate    rpm -qa | grep ntp 
   2. 安装ntp , ntpdate 已系统默认安装过    yum install -y ntp
   3. 修改vi /etc/ntp.conf 配置文件

```markdown
    server 0.centos.pool.ntp.org iburst
    server 1.centos.pool.ntp.org iburst
    server 2.centos.pool.ntp.org iburst
    server 3.centos.pool.ntp.org iburst
然后在下面添加这几行：
    server 0.cn.pool.ntp.org iburst
    server 1.cn.pool.ntp.org iburst
    server 2.cn.pool.ntp.org iburst
    server 3.cn.pool.ntp.org iburst
4. 启动ntp 服务(systemctl restart/enable ntpd.service)，并开机自启， 
    查看是否同步： ntpq -p
    开启默认端口： 123/tcp

```





【备注】 需要注意的是； 如果你需要搭建多个Linux 虚拟机器，此时。你通过上面的步骤已经在VM 上搭建了你的第一个Linux 机器。 此时，你只需要克隆第一个机器。将第一个机器设置域名为master 主机。其他按照克隆出来的机器依次命名 slaver1.slaver2......; (注意，这里增加机器需要考量你的电脑本身，一切的机器都是基于你拥有一个强大的物理机器)

## 环境配置



查看Linux系统是否有自带的jdk

1. 安装ntp , ntpdate 已系统默认安装过
2. 接着进行一个个删除包，输入：rpm -e --nodeps +包名
3. 最后再次：rpm -qa | grep java检查是否删除完即可去[oraclejdk](https://www.oracle.com/java/technologies/downloads/#java8) 官网下载  
4. 使用命令解压： tar -xvf jdk-8u341-linux-x64.tar.gz
5. 配置环境变量： 用vim /etc/profile进入编辑状态，加入下边这段配置
6. 重新加载配置，输入：source /etc/profile

```markdown
export JAVA_HOME=/opt/devtools/jdk1.8.0_341

export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

## Linux 目录介绍

![img](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251225225550311.png)



- **/etc — 系统配置核心**
- **/var/log**，各种服务日志全部存放此处；邮件、缓存、队列等内容也在此目录下维护
- **/bin、/usr/bin**：供普通用户使用的基础指令（如 `ls` 位于 `/bin/ls`  **/sbin、/usr/sbin**：系统管理指令，主要由 root 使用。

# node 配置和安装

Node提供了运行javaScript环境和平台而npm提供了管理这个平台上使用的代码库（包）的工具

## 下载[NVM](https://nvm.uihtm.com/)   或者 [地址1](https://github.com/coreybutler/nvm-windows/releases)

> 下载nvm包管理器: [点击下载](https://github.com/coreybutler/nvm-windows/releases); 并自定义目录下安装，在nvm 目录下 setting.text 文件加入淘宝镜像：
```markdown
 nvm  node_mirror  https://npmmirror.com/mirrors/node/  
 nvm npm_mirror  https://npmmirror.com/mirrors/npm/ 
 ```
## 设置registry地址
> registry = "http://registry.npmmirror.com/"  ||  npm config set registry http://registry.npmmirror.com/

一般情况，安装npm 时，默认会 在C:\Users\yangkaihu 目录下生成一个.npmrc  文件，该文件便是 npm  的 镜像文件

2. 管理员cmd 模式开启node管理版本功能: nvm on , 之后便可以下载 nvm install [版本号];使用指定nodejs 版本：nvm use [版本号]

3. 如果在下载某个版本时用的不是指定的镜像源，也可以在下载时直接指定镜像源：

 ```markdown
  npm install  --registry=https://registry.npmmirror.com  [版本号]
 ```

## 配置 Node 环境变量
* 配置nvm 变量：NVM_SYMLINK  ： D:\Program Files\nodejs   和  NVM_HOME：D:\nvm
* 在安装nodejs 路径下 分别创建node_global(存放全局包的下载)、node_cache 并进行如下步骤： 

```markdown
○ 以管理员方式运行cmd: 
	§ npm config set prefix "D:\Program Files\nodejs\node_global"
	§ npm config set cache "D:\Program Files\nodejs\node_cache"
  可以使用npm config  get 或者 npm config list   或者 npm config get registry  查看是否设置成功
```

 - 配置环境变量：和 JavaJDK 一样:
       新建：NODE_PATH；"输入自己安装的nodejs文件的存储路径
			测试： 输入npm install express -g


## nvm查看node的版本

- nvm list 命令 - 显示版本列表
```
nvm list // 显示已安装的版本（同 nvm list installed）
nvm list installed // 显示已安装的版本
nvm list available // 显示所有可以下载的版本

```
## nvm 常用指令

```
$ nvm version         // 查看nvm版本
$ nvm install 4.6.2   // 安装node4.6.2版本（附带安装npm）
$ nvm uninstall 4.6.2 // 卸载node4.6.2版本
$ nvm list            // 查看node版本
$ nvm use 4.6.2       // 将node版本切换到4.6.2版本
$ nvm root　　　　     // 查看nvm安装路径 
$ nvm install latest  //下载最新的node版本和与之对应的npm版本
```

## Node 与 Npm 的关系

官方人员意识到这一点，将 Npm 安装集成到了 Node，也就是说现在你只需要安装一个 Node，就同时安装了 Npm。

[NPM 介绍](https://dkvirus.gitbooks.io/-npm/content/chapter1.html)    [NPM 版本](https://www.npmjs.com/package/npm?activeTab=versions)   [华为开源镜像库](https://mirrors.huaweicloud.com/home)
***



# React 工程化 

## JSX 渲染底层逻辑



**第一步：** 

​     把编写的jsx 语法，编译为虚拟**DOM对象** （框架内部构件的一套对象体系），基于这些属性描述出来，所构建视图中的DOM节点的相关特征



- 基于该语法包 

  ```json
  "bable": {
    "presets": [
      "react-app"
       ]
    }
  ```

  把jsx语法编译为 React.createElement(...)格式；格式如下参数：

- 再把creatElement 方法执行，创建出虚拟DOM对象

  ```jsx
  jsx 语法
  
  <>
     <h2 className="title" style={styObj}>hello world</h2>
     <div className="box">
              <span>{x}</span>
              <span>{y}</span>
     </div>
  </>
  
  编译为函数
  
  React.createElement(
  React.Fragment,
      null,
  React.createElement(
      "h2", { className: "title", style: sty0bj },  "\u73E0\u5CF0\u57F9\u8BAD"
  ),
  React.createElement( "div",{ className: "box"},
                      React.createElement("span", null, x), 
                      React.createElement( "span", null,  y))
  );
  
  渲染为虚拟DOM
  
  virtualDOM ={
     $$typeof: Symbol(react.element),
     ref: null,
     key: null,
     type: 标签名 或者组件
    // 存储了元素的相关属性 &&  字节点信息
     props:{
           元素的相关信息
           children: 子节点信息，若没有子节点则没有这属性
        }
  }
  
  把虚拟DOM 渲染为 真实DoM 
  v18
    const root=ReactDOM.createRoot(document.getElementById('root'));
    root.render(
      <>
        ...
      
      </>
  );
  ```

**第二步**：把虚拟DOM 渲染为真实DOM（浏览器页面中最后 渲染出阿里，让用户看见的ODM 元素）

> 补充说明： 第一次渲染页面是直接从虚拟DOM -> 真实DOM，但是后期视图更新的时候，需要经过一个DOM-DIFF 的对比，计算出补丁包PATCH （也就是两次视图差异部分）进行渲染

1. 第一次渲染完成后， 保存 虚拟DOM 缓存=> oldVirtualDOM
2. 按照最新的数据，把jsx重新编译为 ‘全新的virtualDom’ (全部重新编译一遍)，拿新的虚拟DOM 和之前缓存起来的虚拟DOM 进行对比 -> DOM-DIFF , 之后生成-> PATCH 补丁包， 再把该补丁包渲染



> 总结： jsx 语法通过调用语法包编译并创建createElement，然后执行该函数生成虚拟DOM ，最后通过render 将 虚拟DOM 渲染为真实DOM

## props 基本认识



- **props :** **属性是只读属性，不可以修改，原理是props 对象被冻结了**
  - 关于对象的规则设置
    - 冻结对象： Object.freeze(obj); 检查是否被冻结： Object.isFrozen(obj) => true/false
      - 被冻结对象，不能修改成员值、不能新增成员、删除现有成员、给成员做劫持 ( Object.defineProperty)
    - 密封
      - 检查被密封: Object.seal(obj) ; 检查是否被密封： Object.isSealed(obj)可以修改，其他操作不可做
    - 不可拓展； 把对象设置为不可拓展： Object.preventExtensions(obj)
      - 检查是否可拓展： Object.isExtensible(obj)
        - 被设置不可拓展对象： 除不能新增成员，其余的擦欧总都可以处理
- 作用： 父组件{index.jsx} 调用子组件{DemonOne.jsx}的时候，可以基于属性，把不同的信息传递给子组件，子组件接收响应的属性值，呈现不同的效果，让组件的复用性增强
  - 虽然对于传递过来的属性，不能直接修改，但是可以做一些规则校验
    - 通过把函数当作对象，设置静态的私有属性方法，来给其设置属性的校验规则
      - 通过函数组件.defaultProps = {} 设置
      - 设置其他规则，例如： 数据值格式、是否必传...... 依赖于这个插件 ： prop-types
        - 对应官网： [props](https://github.com/facebook/prop-types)
  - 被冻结对象，即是不可拓展，也是密封的，同理，被密封的对象，也是不可拓展的

## 插槽处理机制											



> 1. 传递数据值： 用属性
>2. **传递HTML结构： 用插槽**
> 
> - 工作原理：在 < Card > 和 < /Card > 之间的所有内容（元素、文本、甚至其他组件）都会被捆绑起来，作为一个**props.children**传递给Card 组件

JSX 中通过 {children} 决定这些内容应该被放置在何处。

- 这里放置的位置如何？ 需要将传递过来的children 做一个类型转化，React 内部提供了该功能
  - React 的实现方式是通过普通的 props 传递 JSX 元素，通过转化为数组通过下标定义
    - 你可以将任何 JSX 元素作为普通的 prop 传递给组件。这为你提供了极大的灵活性来定义多个“具名插槽”（如 header、body、footer）

```jsx
children = React.Children.toArray(children)
```



```jsx
// 定义组件 - Card.js
function Card({ children }) {
  return (
    <div className="card">
      <div className="card-content">
        {children} {/* 这就是“插槽”的位置 */}
      </div>
    </div>
  );
}

// 使用组件 - App.js
function App() {
  return (
    <Card>
      <h2>这是标题</h2> {/* 这些内容会作为 `children` 传入 Card 组件 */}
      <p>这是一些描述性的文字，会被注入到 Card 组件的 children 位置。</p>
    </Card>
  );
}
```

# 类组件（生命周期）

类组件的生命周期： 组件的挂载 -> 组件的更新 -> 组件的卸载 

- Mounting(挂载)：已插入真实 DOM
- Updating(更新)：正在被重新渲染
- Unmounting(卸载)：已移出真实 DOM!



old lifecycle

![old lifecycle](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e12b2e35c8444f19b795b27e38f4c149~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)





new lifecycle

![new lifecycle](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7d8676f379d4d96bbf0ebd9a8528594~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)



废弃原因： 在v15的版本，更新过程是同步的，往往一个主线程长时间被占用，会导致页面性能问题；**React Fiber的机制:** 利用浏览器 `requestIdleCallback` 将可中断的任务进行分片处理，每一个小片的运行时间很短，这样唯一的线程就不会被独占



**对于states生命周期 更新来说**

1. shouldComponentUpdate(nextProps,nextState)
   - newState: 存储要修改的最新状态
   - this.state: 存储的还是修改前的状态，此时状态还没有改变
   - 该函数周期一般返回true ， 允许更新

2. 触发componentWillUpdate 周期函数： 更新之前
   - 此周期函数也是不安全的
   - 在这个阶段吗，状态还没有被修改

3. 修改状态值/属性值 让this.state.xxx 改为最新的值

4. 触发render 周期函数进行渲染
   - 按照最新的状态、属性，把返回的jsx 编译为虚拟DOM
   - 和第一次 渲染出来的虚拟DOM 进行对比 DOM-DIFF
   - 把差异的那分进行渲染 渲染为真实的DOM

5. 触发componentDidUpdate , 更新完毕

# Hooks 函数

## 状态管理

React组件分类

- **函数组件**
  - **不具备“状态、ref、周期函数”等内容**,第一次渲染完毕后,无法基于组件内部的操作来控制其更新,因此称之为静态组件!
  - 但是具备属性及插槽,父组件可以控制其重新渲染渲染流程简单,渲染速度较快!
  - 基于FP(函数式编程)思想设计,提供更细粒度的逻辑组织和复用!

- **类组件**
  - **具备“状态、ref、周期函数、属性、插槽”等内容**,可以灵活的控制组件更新,基于钩子函数也可灵活掌控不同阶段处理不同的事情!
  - 渲染流程繁琐,渲染速度相对较慢!基于OOP(面向对象编程)思想设计,更方便实现继承等!

react hook组件，就是基于react 中新提供 的hook 函数，可以让函数组件动态化

### setState 进阶





this.setState({partialState},{callback})

- partialState: 支持局部状态更改 

  ```jsx 
  this.setState({
      x:100  // 不论总共有多少状态，我们只修改了x,其余的状态不动
  })
  ```

  **callback**: 在状态更改/视图更新完毕后执行（也可以说只要执行了setState， callback 一定会执行）

  - 发生在componentDidUpdate 周期函数之后（componentDidUpdate 会在任何状态更改后都触发执行，而回调函数方式，可以在指定状态更新后处理）
  - 特殊，即使我们基于shouldComponentUpdate阻止了状态/视图更新，componentDidUpdate 周期函数肯定不会执行，但是我们设置的这个Callback 回调函数依然会被触发执行

```jsx

class Demo extends React.Component {
  state={
      x:10,
      y:4,
      z:1
  };
  handle=()=>{
      let{x,y,z} = this.state;
      this.setState({x:x+1});
      console.log(x);
      this.setState({y:y+1});
      console.log(y);
      this.setState({z:z+1});
      console.log(z);
  }
  render() {
      console.log("视图渲染")
      let {x,y,z} = this.state;
      return<div>
          x:{x}   -   y:{y}   -   z:{z}
          <br/>
          <button onClick={this.handle}>按钮</button>
      </div>
  }
}
```



如上 9~15行代码；当触发事件调用到该处函数时， 不会立即更新状态和视图而是将他加入到队列中（更新队列），当上下文中的代码都处理完毕后，会让更新队列中的任务，统一渲染或者更新一次（批处理）

涉及生命周期：

- shouldUpdate
- willUpdate
- this.setState.xxx
- render
- didUpdate
- callback   

当我们需要根据前两个参数更新后的值作为第三个参数的值时，这个时候react 提供了一个方法， 来自于 react-dom 中解构出来的 **flushSync**

- flushSync: 可以刷新 "updater 更新队列",也就是 让修改状态的任务立刻批处理一次！！！

### useState 处理机制 



> useState ： React Hook  函数之一；**目的是在函数组件中使用状态**，**并且后期基于状态的修改，可以让组件更新，类似于类组件中的setState**
>
> let [num,setNum] = useState(initialValue);
>
> 返回一个state,以及更新state 的函数

- 执行useState，传递的initialValue 是初始状态值，返回结果是一个数组：[状态值，修改状态的方法]

  - num 变量存储的是获取的状态值
  - setNum()变量存储的是修改状态的方法，并且会通知视图更新

  函数组件（或者Hooks 组件）不是类组件，所以没有实例的概念，调用组件不再是创建类的实例，而是把函数执行，产生与i给私有上下文而已，在所以，在函数组件中不涉及this 的处理   


#### 处理机制



> 函数组件的每一次渲染（或者更新），都是把**函数重新执行**，产生全新的“私有上下文”
>
> - 内部的代码都需要重新执行一次
> - 涉及到的函数需要重新的构建，这些函数的作用域（私有上下文）是每一次执行的DEMO 产生的闭包
> - 每一次执行DEMO 函数，也会把useState 重新执行。
>   - 执行useState，只有第一次，设置的初始值会生效，其余以后再执行，获取的状态都是最新的状态值（而不是初始值）
>   - 返回的修改状态的方法，每一次都是返回一个新的     



![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251209204817802.png)

基本源码原理如下代码： 

```js
var _state;
function useState(initialValue){
    if(typeof  _state === "undefined") _state = initialValue;
    var setState=function setState(value){
        _state=value;
    };
    return [_state,setState];
}
```





![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251209211912376.png)



- 当我们在给对应的方法中加入计时器后，在规定的时间生效后会去上下文查找对应的变量值，并打印




#### 同步异步处理

 执行sueState: 把需要的状态信息都放在对象中统一管理

- 执行setState方法的时候，传递的是啥，就把状态“整体”改为啥值，并不会象类组件中的this.setState 一样，支持部分状态的更新
- 处理办法就是，将存储的值进行解构出来，通常 这样写  …state   
- 官方的建议是： 需要多个状态，就把useState执行多次即可  

![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251213122854008.png)

1. 同步的话，需要使用flushSync 方法进行指定
2. 在react16 中，也和this.setState一样，放在合成事件&周期函数中，都是异步的操作，但是放在其他的异步操作中，（列如： 定时器，手动绑定的事件中等，它是同步的）

#### 函数更新和优化



useState 自带了性能优化的机制：

- 每一次修改状态值的时候，会拿最新要修改的值和之前的状态值做比较（基于object.is 作比较）
- 如果发现两次的值是一样的，则不会修改状态，也不会通知视图更新（可以理解为，类似于PureComponent，在shouldComponentUpdate 中做了浅比较和优化）
- usetState 方法中，也可以在该方法中包含写一个函数，
  -    ![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251213133415955.png)
  - 该函数只在第一次初始化的时候执行，之后的操作中不再执行该方法
- 如下图所示，实际拿到的数据是11， 那如何实现累计呢，setXxx 方法中，需要将函数中的prev 进行累计返回即可

![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251213125650745.png)



![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251213130724776.png)





## 函数生命周期

### useEffect

> 作用： 在函数组件中，使用生命周期函数,
>
> `useEffect`是React中用于处理**副作用**（Side Effects）的Hook，它让函数组件能够执行在渲染过程之外的操作，如**数据请求、DOM操作、设置订阅/定时器、日志记录**等，可以看作是[componentDidMount](https://www.google.com/search?q=componentDidMount&oq=useEffect+的作用是什么呢+csdn&gs_lcrp=EgZjaHJvbWUyBggAEEUYOdIBCDYyNTVqMGo0qAIAsAIB&sourceid=chrome&ie=UTF-8&mstk=AUtExfDm-KYSnTBFCTf-ajAwy6h7VPkHv1OJZd7CXUXR9xx1Am-y85RfmlMHa_sEL4rqgZ53Bh-9p3X4gUg7YQUzz1Ph2Lt8E0HgR0Tyf_cJ5e2L7CNfhM1taxj5BpBvGys_NItCJLdCPlUt9ALmdp4-4DzSaLzkvDPCjfuVNN1dpI2WWZI&csui=3&ved=2ahUKEwiCx_2Nu5eSAxW5sVYBHbLRIyMQgK4QegQIARAB)、`componentDidUpdate`和[componentWillUnmount](https://www.google.com/search?q=componentWillUnmount&oq=useEffect+的作用是什么呢+csdn&gs_lcrp=EgZjaHJvbWUyBggAEEUYOdIBCDYyNTVqMGo0qAIAsAIB&sourceid=chrome&ie=UTF-8&mstk=AUtExfDm-KYSnTBFCTf-ajAwy6h7VPkHv1OJZd7CXUXR9xx1Am-y85RfmlMHa_sEL4rqgZ53Bh-9p3X4gUg7YQUzz1Ph2Lt8E0HgR0Tyf_cJ5e2L7CNfhM1taxj5BpBvGys_NItCJLdCPlUt9ALmdp4-4DzSaLzkvDPCjfuVNN1dpI2WWZI&csui=3&ved=2ahUKEwiCx_2Nu5eSAxW5sVYBHbLRIyMQgK4QegQIARAC)这三个类组件生命周期方法的组合，通过控制第二个**依赖数组参数**来决定执行时机

- useEffect(callback): 没设置依赖
  - **第一次渲染完成后**，执行callback ，等价于 componentDidMount 
  - 在组件每一次更新完毕后，也会执行callback，等价于componentDidUpdate’
  - callback 中，获取的都是最新状态值
- useEffect(callback，[]):设置了，但是无依赖
  - **只有第一次渲染完毕后**，才会执行callback,每一次视图更新完毕后，callback 不再执行
  - 类似于，componentDidMount 
- useEffect(callback，[num]): 有依赖
  - 数组当中还可以放多个状态值，**第一次渲染完毕后**，执行callback，
  - 当依赖的状态值，或多个状态值中 的一个发生改变，也会触发callback 执行
  - 但依赖的状态如果没有改变，组件更新的时候callback 不会执行

- 如果 callback 返回了一个小函数，那么，该小函数会在第一次渲染完毕后把组件释放的时候把上一次返回的小函数执行


![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251213141904694.png)







###  uselayoutEffect  和 useEffect 的细节



- useEffect 必须在函数的最外层上下文中调用，不能把其嵌入到条件判断、循环等操作语句中
- useEffect 如果设置了返回值，则返回值必须是一个函数（代表组件销毁时触发）；如果经过async 修饰，返回都是一个实例，而不是函数
  -  此时，我们可以使用 Xxx.then(箭头函数)，即可忽略掉返回一个实例的问题  
  - 或者，在内部包括一个小函数 并执行该小函数![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251213214030352.png)

如果链表中的callback 执行又修改了状态值（视图更新）

- 对于useEffect 来讲： 第一次真实DOM 已经渲染，组件更新会重新渲染真实的DOM
- 对于useLayoutEffect来讲，第一次真实DOM 还未渲染，遇到callback中修改了状态，视图立即更新，创建出来的虚拟DOM，然后和上一次的虚拟DOM 合并在一起渲染为真实DOM. 也就是此类场景下，真实DOM 只渲染一次，不会出现图像的闪烁
  -  useLayoutEffect 会阻塞浏览器渲染真实DOM（真实DOM 已经创建），优先执行Effect链表中的callback  

 useLayoutEffect 优先于useEffect  第一个首先输出更新

- useEffect: 浏览器肯定会把第一次的真实DOM 绘制了，再去渲染第二次真实DOM 
- useLayoutEffect: 浏览器是把两次真实DOM 的渲染，合并到一起渲染    



![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251213222326214.png)

视图更新步骤

1. 第一步： 基于 bable-preset-react-app 把jsx编译为createElement 格式
2. 把reacteElement 执行，创建出虚拟DOM
3. 基于root,render 方法把虚拟DOM 变为真实DOM 对象（DOM-DIFF）
   1. useLayoutEffect 阻塞第四步操作，先去执行Effect 链表中的方法（同步操作）
   2. useEffect 第四步操作和Effect链表中的方法执行，是同时进行的（异步操作） 
4. 浏览器渲染和绘制真实DOM 对象

## useRef& uselmperativeHandle 的使用



> 类组件中，我们基于ref 可以做的事情 ：
>
> 1. 赋值给一个标签，获取DOM 元素
> 2. 赋值给一个类子组件： 获取子组件实例（可以基于实例调用子组件中的属性和方法等）
> 3. 赋值给一个函数子组件： 报错（需要配合React.forwardRef 首先ref 转发，获取子组件中的某一个 DOM 元素）
>
> ref 的使用方式：
>
> - ref=‘box’; this.refs.box 获取（不推荐使用）
>
> - ref={x=>this.box=x}; this.box 获取
>
> - this.box=React.createRef(); 创建一个ref 对象
>
>   < ref={this.box}> this.box.current  获取DOM 元素

如何在函数中 使用ref 呢？

1. 基于 “ref={函数}” 的方式，可以把创建的DOM 元素（或者子组件的实例）赋值给box 变量
2. 也可以基于React.createRef() 创建一个ref 对象
3. 推荐使用useRef函数直接创建ref 对象，该方式也只能在函数组件中使用	 

​        let box=useRef(null)



- useRef ,  每一次在组件更新时，再一次执行该方法时，获取的还是第一次创建的对象（推荐使用）
- React.createRef(); 每一次在组件更新时，再一次执行该方法时，都会创建一个全新的ref 对象。比较狼垡资源





总结： 在类组件中，创建ref 对象，我们基于React.createRef 处理：但是在函数组件中，为了保证性能，我们应该使用专属的 useRef 处理



在父组件中调用子组件情况：

- 当调用的是函数子组件时： 需要将该函数组件放在React.forwardRef(函数组件名(props,ref)) 进行转发使用，父组件想要获取子组件中的某个元素可以直接在该属性上加入 ref={ref}

  -   **如果想获取子组件中的内部的属性和方法，需要使用useImperativeHandle（ref,()=>{return(返回的是什么都会被ref 对象获取到) }）**

- 当调用的时函数的类子组件时： 直接在函数父组件中 加入ref={子组件的变量或者属性} 即可获取对应 的子租价的属性和值

  

## useMemo 构建计算缓存



useMemo（callback,[依赖变量]）

- 第一次渲染的时候，callback 会执行，后期只有 依赖的状态发生变化，callback 才会再执行，每一次把callback 执行的返回结果赋值给 xxx
- useMemo 具备缓存的效果，再依赖的状态值没有改变，callback 没有 触发执行的时候，xx 获取的是上一次计算出来的结果（计算缓存）

![useMemo](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251215213620020.png)

- 作用：是一个优化的Hook 函数， 如果函数组件中，又消耗性能、时间的计算操作，则尽可能的使用useMemo 缓存起来设置对应的依赖


## usecallback缓存函数引用

> useCallback 用于得到一个固定引用值的函数，通常用它进行性能优化 其基本使用格式：
>
> ​       const cachedFn = useCallback(fn, dependencies)
>
> - 组件第一次渲染，useCallback 执行， 创建一个函数 callback  ，赋值给xxx
> - 组件后续每一次更新，判断依赖的状态值是否改变，如果改变，则重新创建新的函数堆，赋值给xxz， 如果没有改变或者没有设置依赖，则xxx获取的一直是第一次创建的函数堆，不会创建新的函数堆

**简单来讲： useCallback 可以保证，函数租金的每一次更新，不再把里面的小函数重新创建，用的都是第一次创建的**



useCallback不能乱用，并不是所有的租金内部的函数，都要其处理会更好

1. 虽然可以减少了堆内存的开辟，但是useCallback本省也有自己的处理逻辑和缓存机制，这个也消耗时间
2. 如何使用呢：
   1.  父组件嵌套子组件，父组件要把一个内部的函数，基于属性传给子组件，此时传入这个方法，我们基于useCallback 处理以下会更好 

诉求：

-  当父组件更新的时候，因为传递的子组件的属性仅仅是一个函数（特点：基本应该是不变的），所以不想再让子组件也跟着更新了
  - 第一条：传递的 子组件的属性（函数），每一次需要是相同的堆内存地址（是一致的），基于useCallback 处理
  - 第二条：在子组件内部也要做一个处理，验证父组件传递的属性是否发生改变，如果没有改变，则让子组件不能更新，有变化才需要更新，继承React.PureComponent 即可（shouldCompnentUpdate中对新老属性做了浅比较）  （该方法属于类组件）  ![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251217220539719.png)
  -  第三条：如果是函数组件，那么则是基于React.memo 函数，对新老传递的属性做比较，如果不一致，则执行函数组件，如果一致，则不让子组件更新  ![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251217221444795.png)

![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251217213729347.png)







![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251217220216532.png)









![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251217220341172.png)





## 自定义Hook提取公共逻辑

> 使用自定义hook 可以将某些组件逻辑提取到可重用的函数中










# 合成事件

==**语法是基于 onXxx={函数}进行事件绑定**==

**this值的处理**

- 当绑定的不再是箭头函数而是普通函数，当调用该普通函数时需要 使用this.xxx，此时**this 的值是undefined**而不是我们需要的对象或者实例本身；那**该如何解决**
  - 可以基于js中的bind 方法，预先处理函数中的this 和实参（this.普通函数.bind(this)）
    - 只要该方法进过 bind 处理了，那么最后一个实参，就是传递的合成事件对象
    - bind 还可以预先传参（通常传入的参数和实际参数个数不符合时，优先匹配最右侧信息）
  - 也可以直接使用箭头函数；代表了当前上下文的实例对象    
  - 或者也可以直接声明该函数为箭头函数   

**合成事件对象**

SyntheticBaseEvent 合成事件对象， react 内部特殊处理，把各个浏览器的事件对象统一化后，构建的一个事件对象

-   ```jsx
        handle2 = (ev) => {
            console.log(this);
            console.log(ev); // ev: SyntheticBaseEvent 
        };
    ```

- 合成事件对象中，也包含了浏览器内置事件对象中的一些属性和方法（常用的基本有）

  - clientx/clienty
  - pagex/pagey
  - target
  - type
  - preventDefault
  - stopPropagation
  - …..
  - nativeEvent: 基于这个属性，可以获取浏览器内置原生的事件对象

## 合成事件原理 - 事件传播

> - 例如： 一个容器中，很多元素都要在点击的时候做一些事情
>  - 事件委托： 只需要给容器 做一个事件绑定（点击内部的任何元素，根据事件的冒泡传播机制，都会让同期的点击事件也触发，我们在这里，根据**事件源**，做不同的事情就可以了）
>   - 限制：当前操作的事件必须支持冒泡传播机制才可以
>     - 例如： mouseenter / mouseleave 等事件美誉偶冒泡 传播机制的
>   - 如果单独做到事件绑定中，做了事件传播机制的阻止，那么事件委托中的操作也不会生效！！！



事件具备传播机制，例如：当触发inner 的点击行为的时候

1. 从最外层向最里层逐一查找（捕获阶段：分析出路径）
2. 把事件源（点击的这个元素）的点击行为触发（目标阶段）
3. 按照捕获阶段分析出来的路径，从里到外，把每一个元素的行为也触发（冒泡阶段）   

从这里也能看出事件的传播机制是： **先捕获后目标再冒泡**



- ev.stopPropation(): 阻止事件传播，包含捕获和冒泡
- ev.stopImmediatePropagation() :立刻阻止当前的事件的其他方法未执行的不再执行（同级）

## 合成事件原理-事件委托



> 事件委托： 
>
>   利用事件的传播机制，实现的一套事件绑定的处理方案；根据**事件源target**，做不同的的事情就可以了



优势：

- 提高JS 运行的性能，并且把处理的逻辑都集中到一起
- 某些需求必须基于事件委托处理，
- 给动态绑定的 元素做事件绑定

### 合成事件原理

> 合成事件处理原理：
>
> - “**绝对不是**”给当前元素基于addEventListener 单独做的事件绑定，React 中的合成事件，都是基于“**事件委托**” 机型处理
>   - 在react17及以后版本，都是委托给 #root 这个容器（捕获和冒泡都做了委托）
>   - 在17版本以前，都是委托给document 容器的（且只做了冒泡阶段的委托） 
>   - 对于没有事件传播机制的这样的一个事件，才是单独做事件绑定 ；例如： onMouseEnter \ onMouseLeave       



**在组件渲染的时候**,如果发现JSX元素属性中有 onXxx/onXxxCapture 这样的属性,不会给当前元素直接做事件绑定,只是把绑定的方法赋值给 元素的相关属性!!例如:

```jsx
outer.onClick=() => {console.log('outer 冒泡「合成」');}

outer.onClickCapture=() => {console.log('outer 捕获「合成」');}
```



DOM 零级事件绑定是 outer.**onclick**, 而上述是大写的outer.**onClick** 绑定，故此，他不是给当前元素基于addEventListener 单独做的事件绑定，而它是基于事件委托处理的

然后对#root这个容器做了事件绑定「捕获和冒泡都做了」

​    **原因:**因为组件中所渲染的内容,最后都会插入到#root容器中,这样点击页面中任何一个元素,最后都会把#root的点击行为触发!! 而在给#root绑定的方法中,把之前给元素设置的onXxx/onXxxCapture属性,在相应的阶段执行!!

**其基本执行源码如下所示：** 

```jsx
< script >

const root = document.querySelector('#root'),
      outer = document.querySelector('#outer'),
      inner = document.querySelector('#inner');

// 经过视图渲染解析,outer/inner上都有onXxx/onXxxCapture这样的属性

outer.onClick = () => { console.log('outer 冒泡「合成」')}
outer.onClickCapture = () => { console.log('outer 捕获「合成」')}
inner.onClick = () => {console.log('inner 冒泡「合成」')}
inner.onClickCapture = () => {console.log('inner 捕获「合成」')}

// 给#root做事件绑定
    
root.addEventListener('click', (ev) => {
    let path = ev.path; // path: [事件源->....->window] 所有祖先元素

    [...path].reverse().forEach(ele => {

        let handle = ele.onClickCapture;
        if (handle) handlex();  // handlex(),如果不经过处理，方法中的this 是undefined,如果是箭头函数，则找函数上下文中的this
      });
}, true);

root.addEventListener('click', (ev) => {
    let path = ev.path;

    path.forEach(ele => {

        let handle = ele.onClick;
        if (handle) handle();
       });
}, false);
    
</script>
```



1. 视图渲染的时候，遇到合成事件绑定并没有给元素做事件绑定而是给元素做了设置对应的属性（合成事件属性）
2. 给#root做了事件绑定 ，在#root 上绑定的方法执行，把所有规则的路径中，有合成事件属性的都执行即可（捕获和冒泡）



所谓合成事件绑定，其实并没有给元素本身做事件绑定，而是给元素设置 onXxx/onXxxCapture 这样的合成事件属性。当事件行为触发，根据原生事件传播机制，都会传播到#root 容器上， react 内部给#root 容器做了事件绑定（捕获&冒泡）

当react内部绑定的方法执行的时候，会根据ev.path 中分析的路径，依次把对应阶段的onXxx/onXxxCapture 等事件合成属性触发执行 ====> 合成事件是利用事件委托（事件传播机制）完成的！！！

![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251206224532766.png)

### 合成事件机制

1. ev.stoPropagation():  合成事件中的“阻止事件传播”, 阻止原生的事件传播 & 阻止合成事件中的事件传播。 
2. ev.nativeEvent.stopPropagation(); 原生事件对象中的“阻止事件传播”；只能阻止原生事件 的传播



在16版本中，合成事件的处理机制，不再是把事件委托给#root元素，而是委托给docuent 元素，并且只做了冒泡阶段的委托，在委托的方法中，把 onXxx/onXxxCapture 合成事件属性进行执行

​       

# 组件通信

## 实现父子组件通信

### 类组件件基于props 属性

![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251220153038389.png)



![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251220155706706.png)



- 属性规则校验
  -  PropTypes from ‘prop-types’ 

总结： 类组件中父子组件之间的传值基于属性props 属性完成，配合属性规则校验使用





**父子组件通信:**

@1就是以父组件为主导,基于“属性”实现通信

原因:只有父组件可以调用子组件,此时才可以基于属性,把信息传递给子组件

- 父组件基于属性,可以把信息传递给子组件「父->子」
- 父组件基于属性「插槽」,可以把HTML结构传递给子组件「父->子」
- 父组件把方法基于属性传递给子组件,子组件把传递的方法执行「子->父」

@2父组件基于ref获取子组件实例「或者子组件基于useImperativeHandle暴露的数据和方法」

### 函数组件通信

> 函数组件也是通过props 进行通信，语法和类组件中的使用变了，**函数组件中需要标注props 形参**



- 这里可以有两步优化
  - 基于useMemo 实现复杂逻辑的“计算缓存”
  - 基于useCallback 实现每次函数更新创建的地址为唯一地址值,依赖一定要加上
  - memo作用于函数组件上，做一个浅比较，导出去的时候之间把该函数抱起来

![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251220163541061.png)





## 跨组件通信

### 类组件通信



> **优点:** 解决了跨多层组件传递 props 的繁琐（Props Drilling），代码更简洁
>
> - **适用场景:** 主题 (theme)、用户认证信息 (user auth)、国际化 (i18n) 等全局或跨层级状态.

1. 创建一个上下文对象ThemeCentext.js

2. 让祖先组件具备状态和修改的方法，同时还需要将这些信息存储到上下文中(value存储) <ThemeContext.Provider   value={{。。。}}> <ThemeContext.Provider> 

   ```js
   import React from "react";
   
   const ThemeContext = React.createContext();
   
   export default ThemeContext;
   ```

![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251221130512012.png)



3. 在后代组件中，我们需要获取上下文中的信息（两种方式）

   1.  导入上下文创建的对象并解构出静态对象(provider模式)

      ![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251221131212127.png)

   2. 通过consumer 消费消息（消费者模式： 推荐使用）

      ![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251221131818124.png)



### 函数组件通信

![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251224200459878.png)



```jsx

import ThemContext from  "../ThemeContext";
const VoteMain = function VoteMain() {

    let {supNum,oppNum}=useContext(ThemeContext);   ## 函数中也可以之间这样使用
    return (
              <div className="main">
              <p>支持人数:{supNum}人</p>
              <p>反对人数:{oppNum}人</p>
              </div>;
    )

};

export default VoteMain;


# 或者如下写法

const VoteMain = function VoteMain() {
    
return <ThemeContext.Consumer>
   {context => {

        let { supNum, oppNum } = context;

            return (
            <div className="main">

              <p>支持人数:0人</p>
              <p>反对人数:0人</p>
              </div>;
            )
              }}
</ThemeContext.Consumer>;
};
export default VoteMain;
```



![推荐使用的方法](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251224201743830.png)



# react样式私有化

## 基本方案



> 组件之间将多个组件合并在一个组件中时，其组件内的各个样式会混乱所以需要样式的私有化处理

- 内联样式： 内联样式就是jsx 元素中，直接定义行内的样式 style={{……..}}

- 第二种，保证最外层样式类名称classname 不一样

  - 保证组件最外层样式类名的唯一性，例如： 路径名称+组件名称 作为样式类名
  - 基于less,sas、styleus等css 预编译语言的嵌套功能，保证组件后代元素的样式，都嵌入在外层样式类中！！！ 

  ![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251228143704688.png)

- **css modules （推荐）:**  css 的规则是全局的，任何一个租金的样式规则，都对整个页面有效，产生局部作用域的唯一方法，就是使用一个独一无二的class 名字，这就是css modules 的做法
  - 名称一般为： xxx.modules.css 文件，那么，在属性上就是  对象.xxx 调用即可   
  - 全局样式；加入 :global(.clearfix),使用global 包裹起来。这样样式就 不经过编译了
  - 继承样式 使用 composes

![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251228170624743.png)



- React JSS （推荐使用）:  实质是创建了一个对象，最终调用对象的方式进行样式的设置

  1.  导入依赖，创建对象，调用对象

  - 优点是可以实现一些动态化的实质性动态修改
  - 可以作为函数进行样式的返回   15~21
  - 

```jsx
import React from 'react'
import {render} from 'react-dom'
import {createUseStyles} from 'react-jss'
// 1、创建样式
const useStyles = createUseStyles({
  myButton: {
    color: 'green',
    margin: {
      // jss-plugin-expand插件让语法可读性更高
      top: 5, // jss-plugin-default-unit插件补全单位
      right: 0,
      bottom: 0,
      left: '1rem'
    },
    '& span': props=>{
        return{
                 // jss-plugin-nested 插件将样式应用到子节点
             fontWeight: 'bold' // jss-plugin-camel-case插件将fontWegith转化为font-weight
            fontSize: props.size +'px',
            color: '#000'
        };
    }
  },
 myLabel: {
    fontStyle: 'italic'
  },
})
 
// 2、使用这些样式定义组件，并将其传递给classes属性，使用它来分配作用域类名
const Button = ({children}) => {
  const classes = useStyles({
      
      size: 14,
      color: 'orange'
      
  })  // 使用样式
  return (
    <button className={classes.myButton}>
      <span className={classes.myLabel}>{children}</span>
    </button>
  )
}
// 3、使用组件
const App = () => <Button>Submit</Button>
 
render(<App />, document.getElementById('root'))
```



![jss-](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251229195404245.png)



## Hoc 高阶组件处理

- 创建一个代理组件（Hoc 函数组件），获取基于ReactJSS 便于i的样式，把获取的样式基于属性传递给类租价

![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20251229200531932.png)





## styled-components(推荐使用)

> 目前在react 中，还流行css-in-js 的模式，也就是把css 像js 一样进行编写； 其中较为常用的插件就是 `styled-components`
>
> - yarn  add styled-components
> - 对应官网可以参考： [官网](https://styled-components.com/docs/basics#getting-started)



1. 导入  styled-components 插件中的 styled
2. 基于 `styled.标签名` 这种方式编写需要的样式
   1. 样式要写在 “ES6模板字符串” 中
   2. 返回并且导出的结果是一个自定义的组件 
   3. 安装插件： styled-components

```js
 export const NavBox= styled.nav
   background-color: lightblue;
   width:300px;
   .title{
       font-size:20px;
       color:red;
       &:hover{
           color: green;
       }
   }
```

















# redux 工程化



> 官网： https://cn.redux.js.org/    
>
> 在react 框架中，我们也有**公共状态管理的解决方案： 也就是公共管理容器**
>
> - redux+react-redux
> - dva [redux-saga] 或者 umi
> - Mobx
>
> redux: -是JavaScript 引用的状态容器，将提供可预测的状态管理，除了react 一起使用外，还支持其他框架：体精悍，却有很强大的插件拓展生态，提供的模式和工具使您更容易理解应用程序中的 状态何时、何地。为什么以及如何更新，以及就当这些更改发生时，您的引用程序逻辑将如何表现



什么时候使用readux?

- 在应用的大量地方，**存在大量的状态**，**状态会随着实际的推移而频繁更新**
- 更新该状态的逻辑可能比较复杂，中型和大型代码量的应用。很多人协同开发

Readux 是一个小型的独立js 库、但是他通常与其他几个包一起使用：

- **React-readux**: 它让react 组件与redux 有了交互，可以从store 读取一些state,可以通过 dispatch actions 来更新store
- **Redux Toolkit**： 是我们推荐的编写redux 逻辑的方法，包含了我们认为i对于构建redux 应用程序不可少的包和函数，
- **Redux DevTools** 拓展： 可以显示redux 存储中的状态随时间变化的历史纪录，这允许您有效的调式应用程序

## redux 执行流程图示

1. 创建store 公共容器（该容器中有两部分，一部分是 公共状态，另外一部分皴法事件池）
2. 创建上下文对象ThemeContext；在项目根路径index.js 中 配置上下文
3. 类组件和函数组件从上下文中 获取最新状态
   - 函数组件：使用useContext(ThemeContext) 解构出store 对象，并通过store.getState() 解构出对应的状态变量
   -  类组件： 通过ThemeContext 对象解构 contextType
     -  在render()方法中，通过this.context  解构出store ，然后在通过store.getState() 解构出 状态变量 
4. 然后通过store.dispatch 派发行为触发事件池中的方法进行状态的修改

![redux执行流程图示](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20260101210127060.png)





## redux 的基础操作和思想



```json
下载相关依赖：
  - @reduxjs/toolkit
  - redux
  - redux-logger
  - redux-promise
  - redux-thunk
  - react-redux
```



### 创建store 公共容器

1.  创建一个store 全局公共容器，用来存储各组件需要的公共信息。

      **一般在src 目录下创建store 包下创建下index.js 存储**

   - 可以把它理解为一个状态改变的控制器      

   ```js
   import {createStore} from 'redux'  
   /*管理员： 修改stroe 容器的公共状态*/
   let initial{
       supNum: 10,
       oppNum: 5
   }
   const reducer = function reducer(state=initial,action){
   // state: 存储store 容器种的公共状态（最开始美誉偶的时候，赋值初始状态值Initial）
   // action: 每一次基于dispatch 派发的时候，传递进来的行为对象（要求必须具备type 属性，存储派发的行为标识）
   // 接下来我们需要基于派发的行为标识，修改store 容器中的公共状态信息(要等到最后return的时候，我们需要先克隆)
     state=_.clone(true,state) 或者 state = {...state} // 浅克隆， 最开始，我们应该把获取的state 状态克隆一份，只是对第一级克隆，第二季无法克隆
   // 接下来我们需要基于派发的行为标识，修改store 容器中的公共状态信息
       swithc (action.type){
           case 'VOTE_SUP':
             state.supNum++;
             break;
           case 'VOTE_OPP':
             state.oppNum++;
             break;
           default:   
       }
   
       //return 的内容，会整体替换store容器中的内容
       return state;
   };
   
   /*创建store 公共容器*/ 
   const store=createStore(reducer);
   export default store;
   
   ```

   - 第一次派发，state 没有值，会把initial 的值赋值给state
     - 第一次派发是在redux 内部派发的 （目的： 给state 赋值初始值）； 第一次派发，传递的action.type 不会和任何的逻辑匹配 
     -  第二次派发 对业务逻辑进行匹配；第一次dispatch 派发，都会把reducer 执行   

   手动派发如下： 

```jsx
store.dispatch({
    type: 'VOTE_SUP',
    step: 10
});
```



### 将store 容器设置为上下文共享

> - 在index.jsx 中，基于ThemeContext.Provider 把创建的store 放在上下文中
> - 因为所有的组件最后都是在index.jsx中渲染，所有的组件都可以理解为 index.jsx 的后代组件，基于上下文方案，获取在上下文存储的store就可以了

创建上下文对象： ThemeContext.js

```js
import React from "react";

const ThemeContext = React.createContext();
export default  ThemeContext; 
```



index.jsx

```jsx
import ThemeContext from './ThemeContext';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
    <ConfigProvider locale={zhCN}>
        <ThemeContext.Provider
            value={{
            store
            }}>
            <Vote/>
        </ThemeContext.Provider>
    </ConfigProvider>
);
```







### 子组件使用上下文



函数组件和类组件 获取状态信息更新

- 使用useContext(ThemeContext) 解构出store
- 时殷弘store.getState()解构出具体需要的状态变量
  - 函数组件中，通知组件更新必须要构建状态值的变化  
  - 类组件中，通知组件更新直接使用强制方法 this.forceUpdate();

**函数子组件使用上下文**

```jsx
import React, {useContext, useState, useEffect} from "react";
import './Vote.less';
import VoteMain from './VoteMain';
import VoteFooter from './VoteFooter';
import ThemeContext from "../ThemeContext";

const Vote = function Vote() {

  const {store} = useContext(ThemeContext);

     // 获取容器中的公共状态
    let {supNum, oppNum}= store.getState();

    // 组件第一次渲染完成后，将组件更新方法放在 STORE 的事件池中
    let [num, setNum] = useState(0);
    const update = () => {
        setNum(num + 1);
    };
    useEffect(() => {
        // let unsubscribe = store.subscribe(让组件更新方法)
        // + 将组件更新方法放在 STORE 的事件池中
        // + 返回的 unsubscribe 方法执行，可以把刚才放入事件池中的方法移除
        let unsubscribe = store.subscribe(update);
        return （） =>{
            unsubscribe(); // 在上一次组件释放的时候，把上一次放在事件池中的方法释放掉
        }
    }, [num]);

   return(
       <div className="vote-box">
           < div className="header">
               < h2 className="title"> React是一个很棒的前端框架 < /h2>
               <span className="num">{supNum + oppNum}</span>
           </div>
           <VoteMain/>
           <VoteFooter/>
       </div>
   )
};

```





**类子组件使用上下文**

```jsx
import React from "react";
import ThemeContext from "../ThemeContext";

class VoteMain extends React.Component {

    static contextType = ThemeContext;
    render() {
        const {store} = this.context;
        let {supNum, oppNum} = store.getState();
        return <div className="main">
            <p>支持人数：{supNum}人</p> // 获取公共状态信息做绑定
            <p>反对人数：{oppNum}人</p>
        </div>;

    }
    componentDidMount() { // 类组件第一次渲染完毕，把 “让鞍更新的办法”，基于store。subsoribe 放入到事件池中；类组件直接基于 forceUpdate 强制更新即可
        const {store} = this.context;
        store.subscribe(() => {
            this.forceUpdate();

        });
}
export  default  VoteMain;
```



**action 派发**





```jsx
import React, {useContext} from "react";
import {Button} from 'antd';

import ThemeContext from "../ThemeContext";

const VoteFooter = function VoteFooter() {
    const {store} = useContext(ThemeContext);
    return <div className="footer">

        <Button type="primary"
                onClick={() => {
                    store.dispatch({
                        type: 'VOTE_SUP'
                    });
                }}>
            支持
        </Button>
        <Button type="primary" danger
                onClick={() => {
                    store.dispatch({
                        type: 'VOTE_OPP'
                    });
                }}>
            反对
        </Button>
    </div>;
};
export default VoteFooter;
```

































### 优化redux

#### reducer的拆分和合并

> 将多个reducer 合并为一个reducer，赋值给我们创建的store

​                                         **模板如下：** 

```js

import {createStore} from 'redux'  
/*管理员： 修改stroe 容器的公共状态*/
let initial{
    
}
export default function Xxxreducer(state=initial,action){
  state=_.clone(true,state) 或者 state = {...state} // 浅克隆， 最开始，我们应该把获取的state 克隆一份，只是对第一级克隆，第二季无法克隆
    // 接下来我们需要基于派发的行为标识，修改store 容器中的公共状态信息
    swithc (action.type){
  
    }

    //return 的内容，会整体替换store容器中的内容
    return state;
};

```





**合并多个reducer**

```js
reduer.js

/+
 合并多个reducer ，最后合并为一个reducer 
    +/

import {combineReducers} from 'redux';
import voteReducer form './voteReducer';
import personlReducer from './personalReducer';


const reducer = combineReducers({
    vote: voteReducer,
    personal: personlReducer
});
export default reducer;


===========================================================
  index.js


import {createStore} from 'redux';
import reducer from './redcuers';

const store = createStore(reducer);
export default store;
    
    

```





派发的行为不需要修改，每次派发的时候都会去所有的reducer 进行逐一匹配，（用派发的行为标识和每一个模块的reducer 中的判断进行匹配，谁匹配成功就执行谁）

#### 派发行为标识宏管理



> redux工程化的第二步：
>
> - 第一次dispatch 派发的时候，都会去每个模块的reducer 中找一遍，把所有和派发行为标识匹配的逻辑执行
>   -  可能存在，团队协做的时候，可能存在 派发的行为标识存在冲突； 所以我们要求派发的行为标识必须是唯一的  
>   - 基于宏管理（统一管理）： 让所有需要派发的行为标识，唯一性



store /**action-types**.js  ： **统一管理需要派发的行为标识：**

```js
/*
统一管理需要派发的行为标识：
  + 为了保证不冲突，我们一般都是这样命名： 模块名——派发的行为标识（大写）
  + 变量和存储的值是一致的
  + 所有需要派发的行为标识，都在这里定义
*/

export const VOTE_SUP = "VOTE_SUP";
export const VOTE_OPP = "VOTE_OPP";
```





#### actionCreator 的创建

voteAction.js



```js
/*
vote 板块要派发的行为对象管理
  voteAction 包含很多方法，每一个方法返回都会返回派发的行为对象

*/
import * as  TYPES from '../action-types';
const voteACtion ={
    support (){
        return {
            type: TYPES.VOTE_SUP
        }   
    },
    ......
}
    。。。。。。。。 很多给xxxAction.js  
```

合并多个xxXAction.js

```jsx

/*

导入多个xxxAction.js 对象，合并为一个action 即可
*/
import voteAction from './voteAction';
import personalAction from './personalAction'；

const action ={
    vote:voteAction,
    personal: personalAction;
}
export default action

```

如何整合action

```jsx
import React, {useContext} from "react";
import {Button} from 'antd';
/*
导入action 对象
**/
import action from './action';
import ThemeContext from "../ThemeContext";

const VoteFooter = function VoteFooter() {
    const {store} = useContext(ThemeContext);
    return <div className="footer">

        <Button type="primary"
                onClick={() => {
                    store.dispatch({
                       action.vote.support（）； // 整改后
                    });
                }}>
            支持
        </Button>
        <Button type="primary" danger
                onClick={() => {
                    store.dispatch({
                        action.vote.oppose();
                    });
                }}>
            反对
        </Button>
    </div>;
};
export default VoteFooter;
```

















# reduce-redux 的基础应用



1.  上下文可以直接不再使用，直接react-redux 中导入 Provider,并在该标签中 传入store 属性值即可，就不需要自己创建上下文对象了
2.  在react-redux 中解构出 ： connect , 直接需要在导出时 connect()(渲染的组件)；

对于函数组件来讲：

```js 



const Vote = function Vote (props) {

let { supNum, oppNum } = props;
return <div className="vote-box">

      <div className="header">
       <h2 className="title">React 是一个很棒的前端框架</h2>
       <span className="num">{supNum + oppNum}</span>
      </div>
      <VoteMain />
      <VoteFooter />
      </div>;
};

export default connect(state => state.vote) (Vote);

/* export default connect(state => {

return {
    supNum: state.vote.supNum,
    oppNum: state.vote.oppNum,
    num: state.vote.num
    }
}) (Vote); */


/*
connect(mapStateToProps, mapDispatchToProps)(要渲染的组件)
1. mapStateToProps：可以获取到 Redux 中的公共状态，将需要的信息作为属性传递到组件即可
connect(state=>{
   // state：存储 Redux 容器中所有模块的公共状态信息
   // 返回对象中的信息，即要作为属性传递到组件
return {
    supNum:state.vote.supNum,
    info:state.personal.info
})(Vote);

*/
```







对于类组件来讲

```js
import {connect} from 'react-redux';

class VoteMain extends React.Component {

    render() {
        let {supNum, oppNum} = this.props;
        return <div className="main">
            <p>支持人数：{supNum}人</p>
            <p>反对人数：{oppNum}人</p>
        </div>;
    }
}
export default connect(state => state.vote)(VoteMain);
```





```markdown
connect(mapStateToProps, mapDispatchToProps)(要渲染的组件)

2.mapDispatchToProps：将需要派发的任务作为属性传递组件

connect(null,
    dispatch => {
// dispatch:store.dispatch 派发任务的方法

// 返回对象中的信息，将作为属性传递组件
        return {
            ...
        };
 })(Vote);
```





```js
import {connect} from 'react-redux'

const VoteFooter = function VoteFooter(props) {
    let {support, oppose} = props;
    return <div className="footer">
        <Button type="primary" onClick={support}>支持</Button>
        <Button type="primary" danger onClick={oppose}>反对</Button>
    </div>;
};
export default connect(
    null,
    action.vote // 等同于如下：
    
   /* dispatch => {
        return {
            support() {
                dispatch(action.vote.support());
            },
            oppose() {
                dispatch(action.vote.oppose());
            }
        }; */
    }
)(VoteFooter);


```





![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20260125160112247.png)







# redux 和react-redux 的归纳梳理





1.  把reducer/状态按照模块进行划分和管理；把所有模块的reducer 合并为一个即可！！!
2.  每一个任务派发，都会把所有模块的reducer,依次执行，派发时候传递的行为对象（行为标识）是统一的！！！所以我们要保证； 各个 模块之间，派发的行为标识的唯一性 ===> 派发行为标识的统一管理
3. 创建action/ actionCreator 对象，也是按照模块管理我们需要开发的行为对象

以上，就是我们的redux









**在组件中使用时，如何使用 Redux：**

1. 我们需要创建上下文对象，基于其 Provider 将创建的 store 放在根组件的上下文信息中；不同组件需要基于上下文对象，则获取上下文中的 store！
2. 需要用于公共状态的组件
   - store.getState() 获取公共状态
   - store.subscribe（让组件更新的函数）放在事件池中

3. 需要派发的组件
   - store.dispatch(actionCreator)







**react-redux ```就是帮助我们简化redux 在组件中的使用```**

1. 提供Provider 组件，可以自己在内部创建上下文对象，把store 放在根组件的上下文中

     ```js
     import {Provider} from 'react-redux';
     
     const root = ReactDOM.createRoot(document.getElementById('root'));
     
     root.render(
         <ConfigProvider locale={zhCN}>
             <Provider store={store}>
                 <Vote/>
             </Provider>
         </ConfigProvider>
     );
     ```

2. 提供了connect 函数，在函数内部，可以快速获取到上下文中的的store,快速的把公共状态以及需要开发的操作基于属性传递给组件; 

   ```js
   connect(mapStateToProps,mapDispatchToProps)(渲染的组件)
   
   import React from "react";
   import './Vote.less';
   import VoteMain from './VoteMain';
   import VoteFooter from './VoteFooter';
   import {connect} from 'react-redux';
   
   const Vote = function Vote(props) {
       let {supNum, oppNum} = props;
       return <div className="vote-box">
           <div className="header">
               <h2 className="title">React 是一个很棒的前端框架</h2>
               <span className="num">{supNum + oppNum}</span>
           </div>
           <VoteMain/>
           <VoteFooter />
       </div>;
   };
   
   export default connect(state => state.vote)(Vote);
   
   ```

   





# redux 中间件及处理机制



- redux 中间件
  - redux-logger : 每一次派发，在控制台输出派发日志，方便对redux  操作进行调试， 输出的内容： 派发之前的状态，派发的行为以及派发后的状态
  - redux-thunk/redux-promise： 实现异步派发（每一次派发的时候，需要传递给reducer 的action 对象中的内容，是需要异步获取的）
  - redux-saga  

![](https://raw.githubusercontent.com/Hashmite/PicGo/main/img/20260131091710176.png)

## 使用redux 中间件





- 中间件日志

```jsx
import {createStore, applyMiddleware} from 'redux';
import reducer from './reducers';
import reduxLogger from 'redux-logger';

const store = createStore(
     reducer,
    applyMiddleware(reduxLogger)  // 引入中间件：applyMiddleware ，并加redux 日志加载其中
);

export default store;
```



- 中间件异步派发： 实际开发中是需要异步操作  ，例如： 在派发的时候，我们需要先向服务器发送请求，把数据拿到后，再进行派发，此时需要中间件 redcuxThunk
  - await关键字必须写在被async修饰的函数中,但是他会阻塞进程,await是等待的意思，那么它等待什么呢，它后面跟着什么呢？其实它后面可以放任何表达式，**不过我们更多的是放一个返回promise 对象的表达式**。注意await 关键字只能放到async 函数里面，列入执行一个方法立刻返回结果
  - 在不使用任何中间件的情况下，**actionCreator 对象中，是不支持异步操作的；**我们要保证方法执行，要必须立刻返回标准的action 对象，

```jsx
const delay =(interal =100)=>{
    return new Promise(resolve =>{
        setTimeout(()=>{
            resolve();
            
        },interval);
    })
}
## 一个函数中如果加了async 和await ，拿到的这个对象是promise 实例，且状态是pendings，当 awati delay() 没有立马拿到的结果，而是返回一个promise 实例，则会抛出异常
let fn = async()=>{
    await delay();
    return {x:100};
}
fn().then(value =>{
    console.log(value);
})


```





故此，我们需要这样一个异步操作的

```js
// 导入reduxthunk  

import reduxThunk from 'redux-thunk'


const store = createStore(
     reducer,
    applyMiddleware(reduxLogger,reduxThunk)  // 引入中间件：applyMiddleware ，并加redux 日志加载其中
);

export default store;
```



```jsx

//  错误的使用方式
const voteAction ={
    async support(){
        await delay();
        return {
            type: TYPES.VOTE.SUP
        }
    }
}

// 正确的使用方式, redux-thunk  中间件语法, 其实就是返回据一个函数
const voteAction ={
    support(){
       return async (dispatch) =>{
        await delay();
        return {
            type: TYPES.VOTE.SUP
        };
       }
       
   }
}

```





```jsx
react-redux 中
{
  support(){
      dispatch(voteAction.support())
  }
}
点击支持按钮，执行suppoort 方法，把voteAction.support() 执行，执行的返回值进行派发
1. 首先方法执行返回一个函数(也是对象)。内部给函数设置了一个type 属性，属性值不会和reducer 中的逻辑匹配
     第一次派发，dispatch(函数)
     + type 不会和reducer 中的逻辑匹配，所以没有修改任何的状态
2. 把返回的函数执行，把派发的方法dispatch 传递给函数
     接下来我们在函数中，自己搞异步的操作，当异步操作成功后，我们在手动基于dispatch.进行派发即可
```



 



其实真正的源码和这样的处理； redux-thunk 中间件会重写dispatch 方法，第一次点击按钮，用到的dispatch 其实是重写 这个dispatch,传递给dispatch 的是一个函数(没有type),此时即不会报错，也不会通知reducer，仅仅只是返回这个函数执行





```jsx
//redux-promise 中间件
const voteAction ={
    
// redux-promise 中间件， 
    async support(){
        await delay();
        return {
            type: TYPES.VOTE.SUP
        }
    }
}
}

那么在 store 中 ，就要这样配置
import {createStore, applyMiddleware} from 'redux';
import reducer from './reducers';
import reduxLogger from 'redux-logger';
import reduxThunk from 'redux-thunk';
import reduxPromise from 'redux-promise';

const store = createStore(
     reducer,
    applyMiddleware(reduxLogger,reduxThunk,reduxPromise)  // 引入中间件：applyMiddleware ，并加redux 日志加载其中
);

export default store;



============================================================================================================
    第一次点击返回按钮
     dispatch (voteAciton.oppose())
1. 此dispatch也是被重写的，传递进来的是promise 实例；
  + 没有type 属性，但是不报错
  + 也不会通知reducer 执行
2. 但是他会监听voteAction.oppose 执行的返回值（promise实例）,等待实例为成功的时候，他内部会自定在派发一次任务
  + 用到的是 store.disapthc 派发（会通知reducer 执行了）
  + 把实例为成功会的结果（也即是我们安徽的标准action 独享） 传递给reducer，实现状态更新
    
```



redux-promise 和 redux-thunk 中间件，都是处理异步派发的，它们相同的地方

1. 都是两次派发

      +第一次派发是重写后的dispatch；这个方法不会去校验对象是否有 type 属性；也不会在传递对象时校验对象是否为标准普通对象！

      +这次派发，仅仅是为了第二次派发

   - redux-thunk：执行返回的函数，执行真正的 dispatch 传递函数
   - redux-promise：监听返回的 promise 实例，在实例为成功后，需要基于

真正的 dispatch，执行成功的结果，再进行派发！！！



2.  区别

   reduc-thunk 的第二次派发是手动处理的

   redux-promise 是自动处理的



# 基于redux 重构TASKOA 案例











# 封装企业级fetch请求库①

## fetch 语法



```markdown
/*
向服务器发送数据请求的方案：
  第一类： XMLHttpRequest
     + ajax: 自己辨析请求的逻辑和步骤
     + axios: 第三方库，对XMLHttpRequest 进行封装（基于promise 进行封装）
  第二类： fetch
     ES6 内置的API，本省 即是基于promise ，用权限的方案实现客户端和服务器端的数据请求
     + 不兼容IE
     + 机制的完善度上，还是不如XMLHttpRequest 的，（列入： 无法设置超时时间。没有内置的请求中断的处理）
  第三类： 其他方案。主要是跨域为主
     + jsonp
     + postMessage
     + 利用img的src 发送请求，实现数据埋点和上报   

let promise 实例(p)=fetch(url,配置项)
   + 当请求成给，p 的状态是fulfilled, 值是请i去回来的内容，如果请求失败，p 的状态是rejected ，值是失败原因
   + fetch  和 axios 区别：
       ++ 在fetch 中，只要服务器有反馈信息，都说明网络请求成给，组后的实例p 都是fulfilled
       ++ 在axios 中， 只有返回黄台吗是以2开始的，才会u让实例是成功状态，只有服务器没有任何反馈，实例p 才是rejected
配置项： 
   + method: 请求的方式，默认是GET(GET、HEAD、DELETE、OPTIONS、POST、PUT、PATCH)  
   + mode  请求模式： (no-cors、*cors、 same-origin)
   + catch: 缓存： 默认default, (*default,no-catche,reload,force-catche,only-if-cached)
   + credentials 资源凭证(列如：cookie)(include,*same-origin,omit)
       ++ fetch 默认情况下，跨域请求中，是不允许携带资源凭证的，只有同源下才允许
       ++ include: 同源和跨域下都可以和
       ++ same-origin: 同源下才可以
       ++ omit: 都不可以
  + headers: 普通对象{}/Headers 实例
    自定义的请i去头信息
  + body: 设置请求主体信息
    只适用于Post 系列请求
    body 设置的格式有要求，
       ++ JSON 字符串 
       ++ URLENCODED application/x-www-form-urlencoded 字符串('xxx=xxx&xxx=xxx')
       ++ 普通字符串 text/plain
       ++ FormData 对象， 主要 运用在文件上传( 或者表单提交)的操作中
            let  fm = new FormData();
            fm.append('file',文件)
       ++ 二进制 或者buffer格式   
我们发现，相比axios ， fetch 没有对GET 系列请i去，问好传参的信息做特殊的处理(axios 中基于params 设置了问好参数信息，) ，需要自己手动凭借到URL的末尾才可以
*/

/*
Headers 类，头部处理(请求头或者响应头)  Headerws.protoType
  Headers.prototype
   + append 新增头信息
   + deleted  删除头部信息
   + forEach（value,key）: 迭代获取所有头部信息
   + get： 获取某个头部信息
   + has: 验证是否包含某个信息
*/
    
let head = new Headers;
  head.append('Context-Type','application/json');
 
let p = fetch('/api/getTaskList');
   console.log(p)
   headers: head
   
p.then(response =>{
   
}).catch(reson =?{

})


```





## **服务器返回状态处理逻辑**



```jsx
服务器返回的response 独享(Response 类的实例)
    私有属性
       body 响应 主题信息 ( 它是一个ReadableStream 可读流)
       headers 响应头信息 (它是Headers 类的实例)
       statuc/statusText  返回的HTTP 状态码及其描述
let p = fetch('/api/getTaskList');
   console.log(p)
   headers: head
   
p.then(response =>{
   // 进入THEN 中的时候，不一定代表成功了，可能是各种状态码
       let {headers,status,statusText} =response;
       if(/^(2|3)\d{2}$/.test(status)){
            console.log('成功'，response)
             return;
       }
       // 获取失败对象，状态码不对
       return Promise.reject({
           code: -100,
           status,
           statusText
        });
}).catch(reson =?{
          
})       
       
```



## 服务器返回状态成功处理逻辑



> Response.protoType
>
> -  arrayBuffer
> - blob
> - formData
> - json
> - text
> - …….
>
>  这些方法即使用来处理body 可读流信息，可以把可读流替换成我们需要的格式，只不过返回值是一个promise 实例，这样可以避免返回的信息在转化中 出现问题（列如：服务器返回的是一个可读流，我们转化为json 对象肯定是不对的，此时可以 让其返回一个失败的实例即可）





```jsx



fetch(url,{
    method: 'post',
    headers:{
        'Content-Type':'application.x-www-form-urlencoded'
    }
    boyd:  qs.stringify({
      task: '我学会了fetch 操作'
})
})
```



fetch  中的请求中断



```jsx


/* fetch 中的请求中断 */
let ctrol = new AbortController();

fetch('/api/getTaskList', {
// 请求中断
    signal: ctrol.signal
}).then(response => {

    let {status, statusText} = response;
    if (/^(2/3)\d{2}$/.test(status))
    return response.json();

    return Promise.reject({

        code: -100,
        status,
        statusText
    });

}).then(value => {
    console.log('最终处理后的结果:', value);
}).catch(reason => {
//
    console.dir(reason)
});
// 立即中断请求 ctrol.abort();
```











# 封装企业级fetch 请求库②

```jsx
// 核心解构模型
http([config])
 + url 请求地址
 + credentials 携带资源凭证 *including/same-origin/omit
 + headers: null 自定义的请求头信息(格式必须是纯粹对象)
 + body: null 请求主题信息()
 + params: null  设定问好传入参数信息，
 + responseType  预设服务器退回结果的读取方式 *json/text/arrayBuffer/blob
 + signal 中断请求的信息
 
http.get/head/delete/options(url,config) 
http.post/put/patch(url,body.config)
```

**封装http请求**

```jsx
/*
 http([config])
   + url 请求地址
   + method 请求方式  *GET/DELETE/HEAD/OPTIONS/POST/PUT/PATCH
   + credentials 携带资源凭证  *include/same-origin/omit
   + headers:null 自定义的请求头信息「格式必须是纯粹对象」
   + body:null 请求主体信息「只针对于POST系列请求，根据当前服务器要求，如果用户传递的是一个纯粹对象，我们需要把其变为urlencoded格式字符串(设定请求头中的Content-Type)...」
   + params:null 设定问号传参信息「格式必须是纯粹对象，我们在内部把其拼接到url的末尾」
   + responseType 预设服务器返回结果的读取方式  *json/text/arrayBuffer/blob
   + signal 中断请求的信号
 -----
 http.get/head/delete/options([url],[config])  预先指定了配置项中的url/method
 http.post/put/patch([url],[body],[config])  预先指定了配置项中的url/method/body
 */

import _ from '../assets/utils';
import qs from 'qs';
import { message } from 'antd';

/* 核心方法 */
const http = function http(config) {
  // initial config & validate 「扩展：回去后，可以尝试对每一个配置项都做校验?」
  if (!_.isPlainObject(config)) config = {};
  config = Object.assign({
    url: '',
    method: 'GET',
    credentials: 'include',
    headers: null,
    body: null,
    params: null,
    responseType: 'json',
    signal: null
  }, config);
  if (!config.url) throw new TypeError('url must be required');
  if (!_.isPlainObject(config.headers)) config.headers = {};
  if (config.params !== null && !_.isPlainObject(config.params)) config.params = null;

  let { url, method, credentials, headers, body, params, responseType, signal } = config;
  // 处理问号传参
  if (params) {
    url += `${url.includes('?') ? '&' : '?'}${qs.stringify(params)}`;
  }

  // 处理请求主体信息：按照我们后台要求，如果传递的是一个普通对象，我们要把其设置为urlencoded格式「设置请求头」？
  if (_.isPlainObject(body)) {
    body = qs.stringify(body);
    headers['Content-Type'] = 'application/x-www-form-urlencoded';
  }

  // 类似于axios中的请求拦截器：每一个请求，递给服务器相同的内容可以在这里处理「例如：token」
  let token = localStorage.getItem('tk');
  if (token) headers['authorization'] = token;

  // 发送请求
  method = method.toUpperCase();
  config = {
    method,
    credentials,
    headers,
    cache: 'no-cache',
    signal
  };
  if (/^(POST|PUT|PATCH)$/i.test(method) && body) config.body = body;
  return fetch(url, config)
    .then(response => {
      let { status, statusText } = response;
      if (/^(2|3)\d{2}$/.test(status)) {
        // 请求成功:根据预设的方式，获取需要的值
        let result;
        switch (responseType.toLowerCase()) {
          case 'text':
            result = response.text();
            break;
          case 'arraybuffer':
            result = response.arrayBuffer();
            break;
          case 'blob':
            result = response.blob();
            break;
          default:
            result = response.json();
        }
        return result;
      }
      // 请求失败：HTTP状态码失败
      return Promise.reject({
        code: -100,
        status,
        statusText
      });
    })
    .catch(reason => {
      // 失败的统一提示
      message.error('当前网络繁忙，请您稍后再试~');
      return Promise.reject(reason); //统一处理完提示后，在组件中获取到的依然还是失败
    });
};

/* 快捷方法 */
["GET", "HEAD", "DELETE", "OPTIONS"].forEach(item => {
  http[item.toLowerCase()] = function (url, config) {
    if (!_.isPlainObject(config)) config = {};
    config['url'] = url;
    config['method'] = item;
    return http(config);
  };
});
["POST", "PUT", "PATCH"].forEach(item => {
  http[item.toLowerCase()] = function (url, body, config) {
    if (!_.isPlainObject(config)) config = {};
    config['url'] = url;
    config['method'] = item;
    config['body'] = body;
    return http(config);
  };
});

export default http;
```















# redux-toolkit 的应用①



> redux toolkit 最大的特点
>
> - 基于切片机制，把reducer 和 actionCreator 混合在一起了



/store/index.js  类似store



```jsx
import {configureStore} from '@reduxjs/toolkit';
import reduxLogger from 'redux-logger';
import reduxThunk from 'redux-thunk';
import taskSliceReducer from './features/taskSlice'; // 导入切片
const store = configureStore ({
    //  指定reducer
    reducer: {
        // 按照模块管理各个切片导出的reducer, 类似reducer 的合并
        task: taskSliceReducer
        
    }
    // 使用中间件（默认集成了reduxThunk,但是一旦设置，会替换默认值，需要手动指定thunk 中间件）
    middleware:[reduxLogger,reduxThunk]
});

export default store;

```



/features/taskSlice.js  类似reducer

```jsx
// task 板块的切片, 每个切片包含了 REDUCER 和 ACTION-CREATORE 
import {createSlice} from '@reducxjs/toolkit';
  
const taskSlice = createSlice({
    // 设置切片的名字
    name: 'task',
    // 设置此切片对应的reducer 中的初始状态
    initialState:{
        taskList: null
    },
    // 编写不同业务逻辑下，对公共状态的更改
    reducers:{
        getAllTaskList(state，action){
            // state: redux 中的公共状态信息(基于immer 库管理，无需自己再克隆)，传递进来的其他信息，行为标识action也不需要要判断了,传递的其他信息，都是以action,payload 传递进来的值
            state.taskList = action.payload;
        }，
        removeTask(state,{payload}){
        let taskList =state.taskList;
        if(!Array.isArray(taskList)) return;
        state.taskList =taskList.filter(item =>{
            //payload: 接收春娣进来的，要删除哪一项的ID 
            return +item.id!== +payload;
        });
    }，
    updateTask(state,{payload}){
    ......
      }
    }, 
})；
// 从切片中获取actionCreator后期通过dispatch  派发renwu
export let {getAllTaskList,removeTask,UpdateTask}= taskSlice.actions;



// 从切片中获取reducer
export default taskSlice.reducer;

```







# 类 的装饰器

> 类的装饰器在类声明之前被声明，可以用来监视，修改或者替换类的定义
>
> 引入插件支持该语法（webpack 中的babel 支持装饰器）
>
> - yarn add @babel/plugin-proposal-class-properties @babel/plugin-proposal-decorators
> - 修改bable配置项
> - yarn add roadhog: 2.5.0-beta.1   解决bable语法包和插件之间的兼容性问题







**在package.json 配置**



```json
"bable":{
    "react-app"
},
"plugins":{
    "@bable/plugin-proposal-decorators",
    {
    "legacy": true
},
"@bable/plugn-proposal-class-properties",
{
    "loose": true
}
}
```





**类的装饰器**



```js
// target: 当前装饰的这个类，同一个装饰器可以使用在多个类上
const text=(target) =>{
    target.num=100,
        // 装饰器函数执行的返回结果，会替换原有的类，。return 100; Demo 就是100了
    target.getnum=function getNum(){}
}

@test
class Demo{ }
```







**类的装饰器如何处理呢？**

同一个类可以使用多个装饰器，处理顺序是从下到上处理，而函数中，则是先按照函数从上而下外层函数执行获取函数黄诗琪之后 ，再从下而上返回

```jsx
编译后的源码
var _class;
const test = target =>{
    target.num =100;
};
let Demo =test(_calss =calss Demo {}) || _class;
```







# 属性和方法的装饰器



```markdown
const test = (target, name, descriptor) => {
  
    target:Demo.prototype
    name:'x'
    descriptor:{configurable: true, enumerable: true, writable: true, initializer: ƒ}  修饰的属性，则初始值是基于initializer函数设置的！！ 


    target:Demo.prototype
    name:'getX'
    descriptor:{configurable: true, enumerable: false, writable: true, value: ƒ}  修饰的函数，则初始值是基于value属性设置的！！ 
  
    console.log(target, name, descriptor);
};


```







# mobx5 基础及知识

> https://cn.mobx.js.org
>
> mobx 是一个简单可拓展的状态管理库，相比较于redux ,它：
>
> 1. 开发难度低
> 2. 开发代码量少
> 3. 渲染性能好
>
> 想要使用mobx ，首先需要让项目支持JS 装饰器语法
>
> yarn add @babel/plugin-proposal-decorators @babel/plugin-proposal-class-properties



1. reaction &&  autorun(()=>{…}) 区别; 首先会立刻执行一次，自动建立起来依赖关系监听，当依赖的状态值变化，callback 会立刻执行
   - 检测的状态需要装饰为可检测   
   - 不同点：
     - reaction:默认不执行 ；第一个参数是一个返回数组的函数，该函数是需要检测的状态，第二参数便是callback 函数回调
     - autorun:默认执行 





函数组件时没有装饰器的，但是可以直接使用observer() 包裹在该函数中调用即可

```js
import React from 'react';
import {observable,action} from 'mobx';
import {observer} from 'mobx-react';

// 创建一个公共容器
class Store{
    // 公共状态
    @observable num=10;
    //修改公共状态的方法
    @action change(){
        this.num ++;
    }
}
let store =new Store;

@observer
export default class Demo extends React.Component{
    render(){
        return<div>
            <span>{store.num</span>
            <br/>
            <button onClick={()=>{
                store.change();
            }}>按钮</button>
            </div>
    }
}
===================================函数组件=======================================
export default  oberver(
   function Demo {
     render(){
        return<div>
            <span>{store.num</span>
            <br/>
            <button onClick={()=>{
                store.change();
            }}>按钮</button>
            </div>
    })
}
```







# 路由管理

## react-router-dom基础运用和细节

> 当代前端开发，大部分呢都以SPA 但也面应用开发为主
>
> 而前端路由机制，就是构建SPA单页应用的的利器
>
> https://reactrouter.com/en/main

1. 版本号： 5.3.4

2.  基于HashRouter把所有要渲染的内容包起来，开启HASH 路由

   1.  后续用到的<Route> 、<Link> 等，都需要在HashRouter 中使用

   2.  开启后，整个页面地址，默认会设置 一个 #/  哈希值

      1.  Linki 实现路由 切换，跳转 的组件

         1.  最后渲染完毕的结果依然是A标签
         2. 可以根据路由模式，自动设定点击A 切换的方式  

         ```jsx
         <Link to='/'>A</Link>
         ```

3. 路由匹配配置

    路径地址匹配股则：

   ​         路由地址：Route 中的path 字段指定的地址

   ​          页面地址：浏览器URL 后面的哈希值

   ```jsx
   <div className="context">
   <Route exact path="/" component={A}/>
       。。。。。
       <Redirect to="/"/>
   </div>
   ```

   当这样配置时，会出现首个组件页签依然会显示，该如何解决呢？

   ​     将路由配置 放在<Switch> 标签内即可实现精准匹配，并将Route 中 加入 exact 设置精准匹配

```jsx
import {HashRouter，Switch,Route,Switch,Redirect,Link} from 'react-router-dom'
const App = function App(){
    return <HashRouter>
        <NavBox>
            <a href="" >a</a>
            <a href="" >b</a>
            <a href="" >c</a>
        </NavBox>
        <div classname=content>
        </div>
        </HashRouter>
}
```



## 专属路由表管理机制

```jsx
//路由配置
<Route path="/" render={
        ()=>{
            // 返回跳转的页面组件
        }
    }/>


// 重定向：从哪里重定向到哪里

<Redirect from ="/a"  to="/a/a1" 
```

```js
配置路由表： 数组： 数组中每一项就是每一个需要配置的路由规则
    redirect: true 此匹配是否重定向
    from： 来源的地址
    to: 重定向的地址
    exact: 是否精准匹配
    path: 匹配的路径
    component: 渲染的组件
    name: 路由 名称（命名路由）
    meta: 路由元信息
    children: 

const routes =[{
    redirect： true,
    from:'/',
    to: '/a',
    exact: true
},{
    name:'a',
    path:'/a',
    component: A,
    meta:{},
    children: AA
}
               ......
               {
                redirect： true,
               to:'/a'
               }
]


export default routes;
```



多个路由表实在太麻烦，可以直接配置

```jsx
import React, { Suspense } from "react";
import { Switch, Route, Redirect } from 'react-router-dom';

/* 调用组件的时候，基于属性传递路由表进来，我们根据路由表，动态设定路由的匹配规则 */
const RouterView = function RouterView(props) {
    // 获取传递的路由表
    let { routes } = props;
    return <Switch>
        {/* 循环设置路由匹配规则 */}
        {routes.map((item, index) => {
            let { redirect, from, to, exact, path, component: Component } = item,
                config = {};
            if (redirect) {
                // 重定向的规则
                config = { to };
                if (from) config.from = from;
                if (exact) config.exact = true;
                return <Redirect key={index} {...config} />;
            }
            // 正常匹配规则
            config = { path };
            if (exact) config.exact = true;
            return <Route key={index} {...config} render={(props) => {
                // 统一基于RENDER函数处理，当某个路由匹配，后期在这里可以做一些其它事情
                // Suspense.fallback：在异步加载的组件没有处理完成之前，先展示的Loading效果！！
                return <Suspense fallback={<>正在处理中...</>}>
                    <Component {...props} />
                </Suspense>;
            }} />;
        })}
    </Switch>;
};
export default RouterView;
```



## 路由懒加载

> 实现的目的是它将不同路由对应的组件拆分成独立的代码块，只有在用户访问特定路由时才加载对应的代码，从而减少首屏加载的资源体积，提升性能

```js

const routes =[{
    redirect： true,
    from:'/',
    to: '/a',
    exact: true
},{
    name:'a',
    path:'/a',
    component:  lazy( ()=>{
        return import ('../vies/B');
    }),
    或者
    component: lazy(()=>import('../vies/B,'),
    meta:{},
    children: AA
}
               ......
               {
                redirect： true,
               to:'/a'
               }
]
```

然而，处理这懒加载是异步的，需要时间去加载，这个时候就需要欧美使用 Suspense  标签包裹起来该路由,并指定 `fallback`（加载过程中显示的 UI，比如 loading 提示）



指定js 打包后的js 文件

```jsx
component : lazy(()=>import(/* webpackChunkName:"Achild" */ '../views/a'))
```

## 在组件中获取路由对象

在react-router-dom v5中，基于Route 路由匹配渲染的组件，路由默认给每个组件传递三个属性: <Route path='/a' component={A}>

- history
- location
- match

后期我们基于props/this.props 获取传递的属性值， 在render 中可以获取传递的属性，也可以通过hook 函数获取

当不是通过route 渲染的，我们将如何获取呢？

- 基于函数高阶组件，自己包裹一层进行处理

```jsx
class HomeHead extends React.Component{
    render(){
       return <></> 
    }
}

const Hadle = function Handle(Component){
    return function Hoc(props){
        这里面技能获取到对象
        return <Component {...props} />
    }
}
export default Handle(HomeHead)
```

- 在react-router-dom v5 版本中，自带了一个高阶组件withRouter，就是用来解决这个问题的

```jsx
export default withRouter(HomeHead);
```

## router 传参或跳转

路由跳转方案一共有两种，跳转过程中传参

- 通过link 进行跳转

- 编程时导航

  ```jsx
  history.push('/c');
  history.push({
      pathname: '/c',
      search: '', // 存储的是问号传参信息，要求是url encoded 字符串
      state:{}
  });
  history.replace('/c')
  
  需要注意的是，在search 中只能是 url encoded 字符串，如果需要使用对象则需要借助qs 库进行转化
  import qs from 'qs';
  search: qs.stringify({
      id:100,
      name:'yangkaihu'
  })
  ```

如何获取换地过来的值呢？

- 通过 useloaction

  ```jsx
  import {uselocation，useRouteMatch，useParams} from 'react-route-dom'
  import qs from 'qs'
  const C = function C(){
      const loaction = useLocatio();
       const match= useRouteMatch();\
      const params =useParams();  直接获取对象
      match.jparams; 是一个对象，是url 传参的存储
      let query =qs.parse(location.search.substring(1));
  }
  
  或者通过URLSearchParams 来处理
  let usp =new URLSearchParams(location.search);
  usp.get('name')
  ```

  

```jsx

{
    name:'a',
    path:'/a/:id?/:name?', // url 路径传参配置
    component:  lazy( ()=>{
        return import ('../vies/B');
    }),
    或者
    component: lazy(()=>import('../vies/B,'),
    meta:{},
    children: AA
}
```

- 隐式传参

  - 传递的信息不会显示在URL 地址中，安全美观也没有限制
    - ​	在目标组件内刷新，传递的信息就丢失了

  ```jsx
  history.push({
      pathname: '/c',
      state:{
          id: 100,
          name:yangkaihu
      }
  })
  在目标组件内获取
  const loaction= useLocation();
  location.state;
  ```


**Navlink 和link 的区别**

> 都是实现路由跳转，语法上基本一样
>
> - 每次页面加载或者路由切换完毕，都会拿最新的路由地址，和NavLink 中的 to 指定的地址（或者pathname 地址） 进行匹配
>   -    默认会设置active 渲染样式类，可通过activeClassName  重写设置选中的样式或者类名
>   - 故此，我们可以给选中的导航设置对应的样式











# routev6基本操作 

> 1. v6中，移除了 
>    -  switch
>    - redirect   —–> 代替方案：Navigate
>    - withRouter —–> 代替方案：自己写一个

v6 中，各级路由统一写在一个js 中		

```jsx
routes.js

const App = function  App(){
    return <HashRouter>
        <HomeHead />
        <div className='content'>
            <Routes>
                {/*
                  所有的理由匹配规则，放在<Routes> 中；
                  每一条规则的匹配，还是基于<Route>;
                    + 路由匹配成功，不再基于component/render 控制渲染组件，而是基于element 
                    + 不再需要Switch ,默认匹配成功，就不再匹配下面的
                    + 不再需要exact, 默认都是精确匹配
                  原有的<Redirect> 操作，被<Navigate to='/'  /> 代替
                    + 遇到<Navigate />组件，路由就会跳转，跳转到to 指定的路由地址
                    + 设置 replace 属性，则不会新增立即记录，而是替换现有记录
                    + 遇到<Navigate />组件，路由就会跳转，跳转到to 指定的路由地址， to 的值可以是一个对象： pathname 需要跳转的地址，search 问号传参 
                */}
             <Route path='/' element={<Navigate to="/a" />} />
            <Route path='/a' element={<A />}>
                {/*
                v6 中，各级路由统一写在一个js 中		
                */}
                <Route path='/a/a1' element={<a1 />} />
                <Route path='/a/a2' element={<a2 />} />
                </Route>
            </Routes>
        </div>
    </HashRouter>
}
                
   

A.js
import React from 'react'
import {Link ,Outlet} from 'react-router-dom'
const A= function A(){
    return <DemoBox>
        <div className="menu">
            <link to='/a/a1'>a1</link>
        </div>
        <div className='view'>
            {/*
             路由容器，用来渲染二级或者多级路由
            */}
        <Outlet />
        </div>
    
    </DemoBox>
}
                
```





## 跳转和传参

```markdown
在react-router-dom v6中，虽然基于<Route> 匹配渲染，但是不会基于属性将history、location、match传递给组件，想获取相关的信息，我们只能基于Hook 函数处理
  + 首先确保，需要使用Hook 的组件，是在Router （HashRouter or  BrowserRouter）内部包着的
  + 只要子<Router>内部包裹的组件，不论是否是基于<Route>匹配渲染的
     ++ 默认都不可能再基于props 获取相关的对象信息
     ++ 只能基于 路由Hook 获取

为了在类组件中也可以获取路由信息：    
  1.我们在构建路由表的时候，我们会想办法： 继续让基于<Router> 匹配渲染的组件，可以基于属性获取需要的信息
  2.不是基于<Route>匹配渲染的组件，我们重写withRouter（v6 中干掉了这个API），让其和基于<Route>匹配渲染的组件具备相同的属性
 
在react-router-dom v6中，实现路由跳转的方式：
  1.<Link / NavLink  to="/a"> 点击跳转路由
  2.<Navigate to="/a" /> 遇到这个组件就会跳转
  3.编程时导航； 取消了history 对象，基于navigate 函数实现路由跳转
     import {useNavigate} from 'react-router-dom'
     const navigate =useNavigate();
    使用方式：  navigate({
         pathname: '/a',
         search: "", // 显示问号传参
         state:{}// 隐式问号传参
    })
    获取该传入进来的参数使用 useSearchParams() 函数 或者使用 new URLSearchParams(location.search)，或者 直接使用 useParam()函数
    
```





总结：

```markdown

/*

在 react-router-dom v6 中，常用的路由 Hook

+ useNavigate -> 代替 v5 中的 useHistory：实现编程式导航

+ useLocation（v5 中也有）：获取 location 对象信息 pathname/search/state....

+ useSearchParams（新增的）：获取问号传递参点信息，获取结果是一个 URLSearchParams 对象

+ useParams（v5 中也有）：获取路径参数匹配的信息

+ useMatch (pathname) -> 代替 v5 中的 useRouteMatch（v5 中的这个 Hook 很有用，可以基于 params 获取路径参数匹配的信息

息；但在 v6 中，这个 Hook 需要我们自己传递地址，而且 params 中也不能获取匹配的信息，用得比较少了！！

/*

```





## v6 中 路由表及统一管理

路由表

```jsx
routes.js
import {Navigate} from 'react-router-dom'
二级路由
const aRoutes=[
    {  
    path: '/a',
    component: ()=><Navigate to='/a/a1'/>   
    },
    {
    path:'/a/a1',
    name:'a-a1',
    component: lazy(()=>import(/* webpackChunkName:"a" */'../view/a/a1'))
    meta:{}
    }
];

一级路由
const routes =[{
    path:'/',
    component: ()=><Navigate to="/c" />
},{
    path:'/a',
    name:'a',
    component: A，
    meta:{},
   children: aRoutes
               
}，{
    path:'/b',
    name:'b',
    component: lazy(()=>import('../view/b'))
    meta:{},
             
}，{
    path:'/c/:id?/:name?',
    name:'c',
    component: lazy(()=>import('../view/c'))
    meta:{},
            
}
               
];
export default routes;

```



使用路由表index.js 

```jsx
import routes from '/routes'
import {Routes,Route,useNavigate,useLocation,useParam,useSearchParams} from 'react-router-dom'

统一渲染的组件，实现权限校验，传递属性 等
const Element =function Element(props){
    let {component: Component}=props;
    //把路由信息先获取，最后基于属性传递给组件： 只要基于<Route> 匹配渲染的组件，都可以基于属性获取路由信息
    const navigate=useNavigate(),
          location=useLocation(),
          params=useParams(),
          [usp]=useSearchParams();

    //最后要把Component 进行渲染
    return <Component localtion={location}  />;
}




递归创建routes
const createRoute = function createRoute(routes){
    return<>
      {routes.map((item,index)=>{
          let {path，children}=item
          // 每一次路由匹配成功，不直接渲染我们设定的组件，而是渲染Element： 在Element 做一些特殊处理后，再去渲染我们真实要渲染的组件
        return<Route key={index} path={path}  element={<Element {...item} />}>
              // 二级路由，基于递归的方式绑定子路由
              {
                  Array.isArray(children)? createRoute(children) : null
              }
              
          </Route>  
      })}
    </>
}

// 创建路由容器
export default function RouteView(){
    return <Suspense fallback={<>正在加载...</>}> 
        <Route>
   { createRoute(routes) }
    </Route> 
    </Suspense>
   
}；
// 创建按withRoter
export const withRouter =funciotn withRouter(Component){
    // Component 真实渲染的组件
    return function Hoc(props){
        // 提前获取路由信息，作为属性传递给Component
            const navigate=useNavigate(),
          location=useLocation(),
          params=useParams(),
          [usp]=useSearchParams();
    
        return <Component {...props} navigate={navigate}>
    }
};

export default withRouter(HomeHead);
<HomeHead x={10} bool={true}></HomeHead>
```



#   useReducer 实现对状态统一管理

> useReducer 是useState 的升级，而sueState 是 state 的升级
>
> 1. 普通需求处理时候，基本都是useState 处理，不会直接使用useReducer
> 2. 但是如果一个组件的逻辑很复杂，需要大量的状态，此时使用useReducer 管理这些状态更好一些

````jsx
import React,{useReducer,useState} from 'react'

//组件的局部容器 store
const initialState={
    num: 0
};
const reducer =function reducer (state,action){
    state= {...}
    switch(action.type){
            case:'plus':
              state.num++;
              break;
            case: 'minus':
               state.num--;
               break;
            default;
    }
    return state;
}

// action 
const A1= function A1(){
   let {state,dispatch}=useReducer(reducer,initialState);
    return <div className="bax">
    <span >{state.num}</span>
    <br/>
        <button onClick={()=>{
               dispatch({
                   type: 'plus'
               }) 
            }}>增加</button>
         <button onClick={()=>{
               dispatch({
                   type: 'minus'
               }) 
            }}>减少</button>
    </div>
}

export default A1
````













































































































































































































































































































































































































