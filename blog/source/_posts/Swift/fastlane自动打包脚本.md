---
layout: swift
title: fastlane自动打包脚本
date: 2018-03-19 22:31:13
tags:
- Swift
categories: 
- Swift
---

> Fastlane是麻省理工学院批准的开源项目，可以将Mac、iOS、android项目的自动打包、发布等一系列繁琐的任务自动化。

使用 fastlane 可以极大的节约打包上架的时间。
<!-- more -->
#### 准备工作
关于 fastlane 的安装初始化，就不做说明了，网上很容易搜索到。当环境搭建完成后，我们可以在自己的项目工程中会看到多出来一个 fastlane 的文件。关于打包的配置在这个文件夹里面都可以进行配置。

首先，我们需要使用 fastlane 来完成我们的打包工作，我们需要弄清楚我们脚本的工作流程，简单整理一下需求。

我们需要执行一个脚本，脚本填写我们的<font color=#d13f28>**项目路径**</font>，输入我们想要修改的<font color=#d13f28>**版本号**</font>。然后用时间戳设置项目的<font color=#0277BD>**build号**</font>。在脚本的当前路径创建一个build文件夹，并且在里面创建一个 archive 文件夹用来放置 .xcarive 文件。在 build 文件夹下将打包出来的 ipa 文件放置在以自己 scheme 名命名的文件夹中。
简单总结下：
输入： 项目路径、版本号。
生成：build 号、xcarchive 文件 和 ipa 文件。
上传：自动上传 ipa 文件到 fir。
#### 编写脚本
创建一个 .sh 文件，第一次执行 .sh 文件的时候可能会报 `zsh: permission denied:` 警告。这需要我们允许一下脚本执行权限 `chmod -R 777 XXX.sh`。再次执行不会报错。

我们需要接受第一个参数为路径地址，和第二个参数为版本号。

```
# 参数传递目标工程 和 版本号
rootPath=$1
version=$2
```
然后处理一下当前时间作为 build 号，并获取一下工程文件夹的名称

```
# 工程名默认与文件夹同名
project_name=$(basename $rootPath)
# scheme 默认与工程名同名
scheme_name=$project_name
# 打包时间 设置为 build 号码
BuildDate=$(date +%Y%m%d%H%M)
```
如果需要打包不同的 scheme 可以再对脚本做修改。

接下来需要指定打包输出文件的位置

```
# 指定项目地址
workspace_path="$rootPath/$scheme_name.xcworkspace"
# 指定输出路径
output_path=$(pwd)/build
# 指定输出归档文件地址
archive_path="$output_path/archive/${scheme_name}_${BuildDate}.xcarchive"
# 指定输出ipa地址
ipa_path="$output_path/$scheme_name"
# 指定输出ipa名称
ipa_name="${scheme_name}_${BuildDate}.ipa"
```

#### 修改 版本号、build号
由于我们用到了 fastlan 进行打包，这里就有2个思路来解决修改版本号的问题。第一种就是直接在脚本上找到工程的 Info.plist 文件修改 `CFBundleShortVersionString` 和 `CFBundleVersion` 的值。第二种就是使用 fastlane 的插件修改，在 Fastfile 中安装 versioning 插件来修改。工程目录下执行 `fastlane add_plugin versioning` 安装。注：之前使用 fastlane 自带的修改方法，由于苹果有设置，所以修改不了，使用插件才可以。

##### 直接修改 Info.plist 文件
需要找到 Info.plist 位置直接进行修改。可能每个工程的 Info.plist 文件位置不同，所以需要指定一下文件位置

```
# Info.plist 文件路径
infoPath=$project_name/Info.plist
echo "============================ 正在 修改Build ============================"
cd $rootPath
/usr/libexec/PlistBuddy -c "Set :CFBundleShortVersionString $version" $infoPath || exit
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $BuildDate" $infoPath || exit
echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!! 修改 完成 ${version} ${BuildDate}   !!!!!!!!!!!!!!!!!!!!!!!!!!!!"
```
##### 在 Fastfile 修改
插件安装成功后，可以使用插件的方法来修改。

```
# 修改版本号和build号函数 例: version:1.1.1 build:123
	def prepare_version(options)
    increment_build_number_in_plist(
  		# target: [target_name],
  		build_number: options[:build]
  	)
  	increment_version_number_in_plist(
    	version_number: options[:version]
    ) 
```
这样调用的时候只需要将 version 和 bild 作为参数传递过去就可以。

#### 打包成 ipa 文件
打包使用 fastlane 很方便，需要自定义 lane，将参数传递过去就可以。
在 Fastfile 文件中编写

```
desc "打包上传到 Fir"
	lane :fir do |options|
		#sigh(adhoc:true)  #如果要使用ad-hoc打包, 则需打开此项配置
		gym(scheme: options[:scheme],
		# workspace: "xxx.xcworkspace", # 可省略
		configuration: "Debug",         # Debug or Release
		archive_path: options[:archive_path],
		clean: true,                 #清空上次打包信息
		output_directory: options[:outPath],
		output_name: options[:ipa_name],
		export_method:"development"  # app-store, ad-hoc, package, enterprise, development, developer-id
		)	
		firim(firim_api_token:"554c539d32252425f397dbf39e1831d9")
		#使用自动证书管理
		# enable_automatic_code_signing(path: "HUIpaTest.xcodeproj")
	end
```
在脚本中只需要将关键信息传递过去就可以了调用了

```
echo "============================ 开始打包 ipa 文件 ============================"
echo ${scheme_name}
fastlane fir scheme:"${scheme_name}" \
			 buildDate:"${BuildDate}" \
			 archive_path:"${archive_path}" \
			 outPath:"${ipa_path}" \
			 ipa_name:"${ipa_name}"
echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!! 打包完成 !!!!!!!!!!!!!!!!!!!!!!!!!!!!"
```
至此，打包的流程就可以实现了，我们会在脚本的根目录下看到一个 build 文件夹，其中包括一个 archive 文件夹，用来存放 .xcarchive 文件。还有一个以 scheme 命名的文件夹，其中放置我们上传需要的 ipa 文件。

#### 上传 ipa 文件到 
很多时候需要将打包出来的 ipa 文件进行发布测试，这些工作脚本也可以替我们完成。这里还是有2个方法来处理，第一种还是使用 fir 提供的脚本指令直径进行上传。第二种就是使用 fastlane 的插件来完成。注：安装指令为 `fastlane add_plugin firim`。
##### sh 指令上传
指令很简单，安装好 fir 的脚本环境后，直接使用。

```
上传到fir
echo "============================ 开始上传 ipa 文件 ============================"
fir publish "${ipa_path}/${ipa_name}" -T "${Fir_token}" -c
echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!! 上传完成 !!!!!!!!!!!!!!!!!!!!!!!!!!!!"
```
##### fastlane 指令
在 fastfile 文件中直接在打包完成后添加指令即可。

```
	firim(firim_api_token:"XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX")
```
如果需要上传到 testflight，大概的思路也差不多，不过 fastlane 已经提供了对应的方法，不需要在导入插件了。
#### 最后
至此，打包测试脚本就可以使用了。fastlane 还有很多功能需要慢慢去学习掌握，后面如果用到了，再来做一些更新吧。完整的 sh 文件在[这里](https://github.com/hGhostD/Shell/blob/master/ipa%E6%89%93%E5%8C%85/ipa.sh)。
<br>
<br>
> [fastlane 官方网文档](https://docs.fastlane.tools/getting-started/ios/appstore-deployment/)
> [小团队的自动化发布－Fastlane带来的全自动化部署](https://zhuanlan.zhihu.com/p/23180455)