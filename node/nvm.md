## NVM

nvm是`node`版本管理工具，全称为`node.js version management`，可以用于安装和切换不同版本的`node.js`。

在`windows`操作环境中，我们可以通过[github](https://github.com/coreybutler/nvm-windows/releases)下载最新版本。

- nvm-noinstall.zip：绿色免安装版，但使用时需进行配置。
- nvm-setup.zip：安装版，推荐使用

注意：**安装路径和`node`存放路径不要选择在`c:/program files`下**，避免未知的异常。安装完成后，打开`CMD`，输入命令`nvm`可以列出各种命令。

![image/nvm.png](image\nvm.png)

在使用之前我们可以通过`nvm node_mirror`和`nvm npm_miror`设置镜像地址提高下载速度，也可以在`nvm`安装目录下的`setting.txt`文件进行修改。值得注意的是，**最后的`/`符号一定不能省**，否则会出现`nvm install`的时候报错的情况。

```javascript

```

 之后我们就可以正常使用`nvm`的命令了，常用的几个如下

```javascript
nvm list // 列出当前的node版本
nvm install <version> // 安装指定版本的node,或者写成latest以安装最新版本
nvm use version // 切换当前系统的node版本
```

最后我们设置一个全局的`npm`地址，让各个版本的`node`公用

```javascript
1. npm config set prefix "D:\nvm\npm" //配置用npm下载包时全局安装的包路径
// 安装全局npm，不同的node都使用这个npm，想更新全局的npm的话首先删除全局路径(就是上一行命令的地址,可以使用npm config ls查看)下的npm,再执行一次这个命令即可
2. npm install -g cnpm --registry=https://registry.npm.taobao.org
3. 在用户变量中添加 NPM_HOME=D:\nvm\npm，PATH中添加%NPM_HOME%
```

