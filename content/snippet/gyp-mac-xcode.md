---
title: "xcode-tools 在每次系统更新后的问题"
date: "2021-06-17T13:50:46+02:00"
tags: ["theme"]
categories: ["starting"]
banner: "img/banners/banner-1.jpg"
---

Oops...今天又遇到 gyp 报错了.. 记得明明安装过 xcode-tools..

```bash
No receipt for 'com.apple.pkg.CLTools_Executables' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLILeo' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLI' found at '/'.

gyp: No Xcode or CLT version detected!
gyp ERR! configure error 
gyp ERR! stack Error: `gyp` failed with exit code: 1
gyp ERR! stack     at ChildProcess.onCpExit (/Users/alberthuang/.nvm/versions/node/v10.24.1/lib/node_modules/npm/node_modules/node-gyp/lib/configure.js:351:16)
gyp ERR! stack     at ChildProcess.emit (events.js:198:13)
gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:248:12)
gyp ERR! System Darwin 20.5.0
gyp ERR! command "/Users/alberthuang/.nvm/versions/node/v10.24.1/bin/node" "/Users/alberthuang/.nvm/versions/node/v10.24.1/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js" "rebuild"
gyp ERR! cwd /Users/alberthuang/code/bellcode/coder/node_modules/native-is-elevated
```


看了一下相关的代码 https://github.com/nodejs/node-gyp/blob/0da2e0140dbf4296d5ba7ea498fac05e74bb4bbb/gyp/pylib/gyp/xcode_emulation.py#L1543

的确是找不到对应的xcode-tools包才会报这个错误

重新安装xcode-tools解决
```
sudo rm -rf $(xcode-select -p)
sudo xcode-select --install
```

后续升级系统再深入调查一下为什么每次更新都会导致这个问题。 (: