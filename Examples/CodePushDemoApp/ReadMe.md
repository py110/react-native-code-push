如何使用这个服务

1 先安装 code-push 并注册,参考相关文档

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

6 找到你的 api key 到 xcode de infor.plist 文件中进行配置,自定义服务

<key>CodePushDeploymentKey</key>
<string>uRv5z7A2FgT-es1PByKUU1N58kXeVkTVXb7vx</string>

<key>CodePushServerURL</key>
<string>http://s0393.com/wxPay/appCheck/</string>

	
7 使用xcode 运行程序	


========================================================================================
相关代码分析
CodePush.js

方法 checkForUpdate

1- getConfiguration ,这里主要是返回 api key 及升级 地址,如果服务是我们自己实现,配置自己的服务地址

2- getCurrentPackage,返回当前包的相关信息

3- 比对二者相关信息,然后调用服务检查更新接口进行检查 ,对应 queryUpdateWithCurrentPackage

对应的是 acquisition-sdk.js 这个文件中的方法 20 行

        var requestUrl = this._serverUrl + "updateCheck?" + queryStringify(updateRequest);
        
https://codepush.azurewebsites.net/updateCheck?deploymentKey=uRv5z7A2FgT-es1PByKUU1N58kXeVkTVXb7vx&appVersion=2.6.0
        

这里就是检查服务,我们可以自定义自己的服务接口,传递过来

的参数样式如下:

[isCompanion:, packageHash:d7b7b4e295a0de6df3b0b1788c83cdab3c0c5d27c13ad23d7b6d09c84390f80e,

自定义服务接口实现

 def updateCheck() {
        def data = [:]
        println params.toString()
        def updateInfo = [:]
        updateInfo.put('description', '升级更新')
        updateInfo.put('label', '幸福中原')
        updateInfo.put('appVersion', '2.6.0')
//        updateInfo.put('updateAppVersion','1.8.0')
        updateInfo.put('label', "v8")
        updateInfo.put('isAvailable', true)
        updateInfo.put('isMandatory', true)
        updateInfo.put('updateAppVersion', false)
        updateInfo.put('packageHash', UUID.randomUUID().toString())
        updateInfo.put('packageSize', 168478)
        updateInfo.put('downloadURL', webRequest.baseUrl + "/${controllerName}/downloadZip")
        data['updateInfo'] = updateInfo
        render data as JSON
    }

    /**
     * 下载zip文件
     */
    def downloadZip() {
        println params.toString()
        def file = new File(servletContext.getRealPath('/apk/release.zip'))
        if (file.exists()) {
            response.setContentType("application/octet-stream")
            // or or image/JPEG or text/xml or whatever type the file is
            response.setHeader("Content-disposition", "attachment;filename=\"${file.name}\"")
            response.setContentLength(file.bytes.length)
            response.outputStream << file.bytes
        } else render "Error!" // appropriate error handling
    }
    

 appVersion:2.6.0, deploymentKey:uRv5z7A2FgT-es1PByKUU1N58kXeVkTVXb7vx ]
 
 
   DEFAULT_UPDATE_DIALOG: {
     appendReleaseDescription: false,
     descriptionPrefix: " Description: ",
     mandatoryContinueButtonLabel: "Continue",
     mandatoryUpdateMessage: "An update is available that must be installed.",
     optionalIgnoreButtonLabel: "Ignore",
     optionalInstallButtonLabel: "Install",
     optionalUpdateMessage: "An update is available. Would you like to install it?",
     title: "Update available"
   }
 
 
 如果升级不执行 检查下代码 CodePush.m 中的
 
if ([binaryDate compare:packageDate] == NSOrderedAscending && [binaryAppVersion isEqualToString:packageAppVersion]) 
 
看看升级版本是否一致,文件日期等等