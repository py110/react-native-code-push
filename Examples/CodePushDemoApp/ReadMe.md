如何使用这个服务
1 先安装 code-push

2 然后增加你需要更新的app ,这里也 demo 为例
code-push app add demo

3 打包你需要更新的资源文件,注意不同平台不同入口文件

react-native bundle \
--platform ios \
--entry-file index.ios.js \
--bundle-output ./release/main.jsbundle \
--assets-dest ./release \
--dev false

注意:可以需要先创建 这个 release 文件,否则 bundle 可能形成不出来

4 更新资源包到 code-push 服务器上

code-push release demo ./release 1.6.0

5 查看你的资源包对应的唯一 api key

code-push deployment ls demo

6 找到你的 api key 到 xcode de infor.plist 文件中进行配置

<key>CodePushDeploymentKey</key>
<string>uRv5z7A2FgT-es1PByKUU1N58kXeVkTVXb7vx</string>
	
7 使用xcode 运行程序	