# Mac系统在进行npm install时报错问题

```
xcode-select: error: tool 'xcodebuild' requires Xcode, 
but active developer directory '/Library/Developer/CommandLineTools' is
 a command line tools instance
```

查了很多解决方案，有说node版本问题的，有说xcode-select安装不对的，有说需要安装xcode的（太大了，有7个G）。

第一种：

更换node版本：升级node版本或降低版本，推荐使用nvm可以任意切换，大家自行百度安装nvm吧，是真的好用，强烈安利。

第二种：

```shell
xcode-select --install  //如果提示已经安装成功大家就卸载重新安装一遍
sudo rm -rf $(xcode-select -print-path)  //卸载
code-select --install     //重装
sudo xcode-select --switch /Library/Developer/CommandLineTools
xcode-select --version       //查看版本，即安装成功
```

安装xcode大家自行百度，大家可以挨个试一下，反正我是没成功。

后面找到了官网：

https://github.com/nodejs/node-gyp/blob/master/macOS_Catalina.md

官网提供了两条命令：

1. `npm explore npm -g -- npm install node-gyp@latest`
2. `npm explore npm -g -- npm explore npm-lifecycle -- npm install node-gyp@latest`

大家可以试一下，我升级了`node-gyp`之后确实好了。然后npm install的时候还是各种报错，但是已经不是之前的问题，是因为vue-loader的问题，有说vue-loader版本过高需要加一下它的plugin，即：

```//js
const VueLoaderPlugin = require('vue-loader/lib/plugin');

module.exports = {
  // ...
  plugins: [
    new VueLoaderPlugin()
  ]
}
```

大家可以试一下。我把pack.json文件恢复到了和主分支一样的样子，然后重新npm install的时候就OK了。

有时候有些bug真的就是看缘分。
