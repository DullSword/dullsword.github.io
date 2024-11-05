---
title: Cocos Creator 3 官方热更新方案实践
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2024-09-19 17:51:15
categories:
  - Cocos Creator
  - Cocos Creator 3
tags: Cocos Creator
---

{% message color:warning %}
本文中的操作实践基于 Cocos Creator 3.8.3 版本进行，但文中所述的热更新方案并不特定于此版本。请根据实际情况进行调整和应用。
{% endmessage %}

![hot_update](hot_update.gif)

官方 [资源热更新教程 - 3.8](https://docs.cocos.com/creator/3.8/manual/zh/advanced-topics/hot-update.html) 感觉更多的是描述热更新方案的原理，没有明确的步骤顺序，所以本文基于文档和提供的范例工程实践一下，把过程步骤记录下来。

本文旨在循序渐进的同时介绍一些注意事项，因此会有一些绕弯的行为。如果你想直接跑通范例项目，可以直接跳转到 [简洁步骤](#简洁步骤) 。

## 0. 准备工作

1. 范例工程可以从 [GitHub](https://github.com/cocos-creator/cocos-tutorial-hot-update/tree/master) | [Gitee](https://gitee.com/mirrors_cocos-creator/cocos-tutorial-hot-update/tree/master) 获取（master 分支）。

    {% message color:info %}
    官方文档中提供的 Gitee 地址是无效的，我不清楚是改名了，还是说就是搞错了，项目名是“cocos-tutorial-hot-update”，跳转链接地址里的项目名却是“tutorial-hot-update”。
    {% endmessage %}

2. 打开 Cocos Dashboard，导入范例工程，我这里显示它是基于 `3.3.2` 版本的，将它升级成 `3.8.3` 版本。
3.
    {% blockquote @CocosCreatorDocs https://docs.cocos.com/creator/3.8/manual/zh/advanced-topics/hot-update.html#%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF%E5%92%8C%E8%AE%BE%E8%AE%A1%E6%80%9D%E8%B7%AF 资源热更新教程 - 使用场景和设计思路 %}
    由于 WEB 版本可以通过服务器直接进行版本更新，所以资源热更新只适用于原生发布版本。
    {% endblockquote %}
    本文选择 Android，因此需要 Android 原生开发环境配置，可以参考文档 [安装配置原生开发环境](https://docs.cocos.com/creator/3.8/manual/zh/editor/publish/setup-native-development.html) 和 [Android 原生开发环境配置](https://docs.cocos.com/creator/3.8/manual/zh/editor/publish/android/build-setup-evn-android.html)。

## 1. 构建生成

选择 项目 -> 构建发布 菜单，点击面板上的“新建构建任务”选项：

1. 选择发布平台“安卓”。
2. 填写“游戏名称”。
3. 填写“应用 ID 名称”。
4. 选择“Target API Level”。

    {% message color:info %}
    这里出现的选项跟你的 Android SDK 有关，需要和你要测试的真机或模拟器安卓版本对应。我选择是 **android-28** ，因为我模拟器系统是 **Android 9** 。
    {% endmessage %}

5. 其他选项可以根据自己的情况进行调整。
6. 点击“构建”，等待构建完成。

![build_config](build_config.png)

构建成功后，如果不是首次编译 Android 项目，继续点击旁边的“生成”按钮，等待生成成功即可。

否则需要通过 Android Studio 编译：

1. 找到生成好的 Android Studio 工程。

    ![open_build_directory](open_build_directory.png)

    ![build_android_path](build_android_path.png)

2. 用 Android Studio 打开已经构建好的项目。

    ![open_android_project](open_android_project.png)

    打开 Android Studio 后，会花一段时间进行准备工作，待 Android Studio 将项目准备完成后，即可打包 APK。

3. 打开构建菜单选择 Build Bundle(s) / APK(s) -> Build APK(s)。

    ![build_apk](build_apk.png)

4. 成功后可以在 proj/build 目录内找到 APK。

    ![apk_path](apk_path.png)

{% message color:info %}
Android 平台通过编辑器和 Android Studio 编译后的结果有些区别：

- 通过编辑器执行“生成”步骤后，会在发布路径下生成 build 目录，.apk 生成在 build 目录的 build\app\publish\ 目录下。
- 通过 Android Studio 编译后，.apk 则生成在 proj\app\{游戏名称}\outputs\apk 目录下。
{% endmessage %}

相关：

- [安卓构建示例 - 发布流程](https://docs.cocos.com/creator/3.8/manual/zh/editor/publish/android/build-example-android.html#%E5%8F%91%E5%B8%83%E6%B5%81%E7%A8%8B)
- [打包发布到原生平台 - 编译和运行](https://docs.cocos.com/creator/3.8/manual/zh/editor/publish/native-options.html#%E7%BC%96%E8%AF%91%E5%92%8C%E8%BF%90%E8%A1%8C)
- [打包发布到原生平台 - 注意事项](https://docs.cocos.com/creator/3.8/manual/zh/editor/publish/native-options.html#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)

## 2. 安装 APK & 试运行

将 APK 安装到模拟器或者真机上，打开应用会显示这个界面：

![app_first_open](app_first_open.png)

点击“检查更新”，过会会显示“Fail to download manifest file, hot update skipped.”。这是因为并没有运行对应的服务器，获取不到远程资源包。

![fail_to_download_manifest_file](fail_to_download_manifest_file.png)

## 3. 运行服务器

为了让游戏可以检测到远程版本，可以在本机上模拟一个远程服务器。

我选择的是 Node.js + http-server 的方案：

1. 安装 Node.js。
2. 安装 http-server 模块：

    ```cmd
    npm install -g http-server
    ```

3. 导航到你想要作为服务器根目录的文件夹（没有的话可以自己新建）。例如：

    ```cmd
    cd D:\Program\cocos\cocos-tutorial-hot-update-server
    ```

4. 在该目录下运行以下命令启动服务器：

    ```cmd
    http-server -p 7879
    ```

    ![http_server](http_server.png)

这将启动一个在端口 7879 上运行的 HTTP 服务器。

你可以在浏览器中访问 http-server 打印出的：

- http://{本地 IP}:7879
- <http://127.0.0.1:7879>

或 <http://localhost:7879> 其中任意一个地址来查看服务器内容。

![access_server](access_server.png)

## 4. 部署远程版本

{% blockquote @CocosCreatorDocs https://docs.cocos.com/creator/3.8/manual/zh/advanced-topics/hot-update.html#%E5%9C%A8-cocos-creator-%E9%A1%B9%E7%9B%AE%E4%B8%AD%E6%94%AF%E6%8C%81%E7%83%AD%E6%9B%B4%E6%96%B0 资源热更新教程 - 在 Cocos Creator 项目中支持热更新 %}
游戏发布后，若需要更新版本，则生成一套远程版本资源，包含 assets 目录、src 目录和 Manifest 文件，将远程版本部署到服务端。
{% endblockquote %}

所以我们需要准备一套远程版本资源。

1. 打开发布路径，找到 assets 目录（不是工程的 assets 目录）。

    ![builded_assets](builded_assets.png)

2. 将所有文件复制到服务器根目录下。

    ![push_assets_to_server](push_builded_assets_to_server.png)

3. 打开范例工程的 assets 目录。

    ![assets](assets.png)

4. 将 project.manifest 和 version.manifest 复制到服务器根目录下。

    ![push_manifest_to_server](push_manifest_to_server.png)

这时查看服务器内容，应该如下图所示：

![deploy_remote_version](deploy_remote_version.png)

再次点击“检查更新”，会发现还是提示“Fail to download manifest file, hot update skipped.”。

其实查看打包后的 manifest 文件会发现 `packageUrl` 、 `remoteManifestUrl` 和 `remoteVersionUrl` 的 IP 是 `192.168.130.1` 。

![search_manifest](search_manifest.png)

这是范例工程遗留下来的，所以我们需要更新 manifest 文件，让应用可以成功找到远程包。

相关：

- [原生项目目录 - 项目目录](https://docs.cocos.com/creator/3.8/manual/zh/advanced-topics/native-secondary-development.html#%E9%A1%B9%E7%9B%AE%E7%9B%AE%E5%BD%95)

## 5. 使用 Version Generator 来生成 Manifest 文件

在范例工程中，官方提供了一个 `version_generator.js` ，这是一个用于生成 Manifest 文件的 NodeJS 脚本。使用方式如下：

```cmd
> node version_generator.js -v 1.0.0 -u http://your-server-address/tutorial-hot-update/remote-assets/ -s native/package/ -d assets/
```

`http://your-server-address/tutorial-hot-update/remote-assets/` 、 `native/package/`、`assets/` 相当于都是占位符，需要根据自己的情况进行替换。

下面是参数说明：

- `-v` 指定 Manifest 文件的主版本号。
- `-u` 指定服务器远程包的地址，这个地址需要和最初发布版本中 Manifest 文件的远程包地址一致，否则无法检测到更新。
  - 需要以 `/` 结尾，因为最后生成的 Manifest 文件会在后面拼接上“project.manifest”和“version.manifest”。
- `-s` 本地原生打包版本的目录相对路径，比如 ./build/android/assets。
- `-d` 本地保存 Manifest 文件的相对路径。

根据参数说明，我们需要修改“服务器远程包的地址”和“本地原生打包版本的目录相对路径”：

```diff
- > node version_generator.js -v 1.0.0 -u http://your-server-address/tutorial-hot-update/remote-assets/ -s native/package/ -d assets/
+ > node version_generator.js -v 1.0.0 -u http://192.168.0.105:7879/ -s ./build/android/assets -d assets/
```

{% message color:info %}
因为我们在 [4. 部署远程版本](#4-部署远程版本) 中是把远程版本资源直接放在服务器根目录下，所以这里直接就是 `http://192.168.0.105:7879/` （端口号要跟 [3. 运行服务器](#3-运行服务器)中设置的一致），否则需要提供像 `http://192.168.0.105:7879/tutorial-hot-update/remote-assets/` 这样具体的地址。
{% endmessage %}

然后在范例工程根目录下执行修改后的命令：

![generator_manifest](generator_manifest.png)

## 6. 重新打包并部署

移除之前的构建任务并选择“删除源文件”，接着重新按照 [1. 构建生成](#1-构建生成) 步骤新建一个构建任务并构建生成。

{% message color:info %}

- “清空构建缓存”没有用。
- 也可以直接新建一个构建任务并构建生成，但构建任务列表会有多个安卓构建任务，可能会导致管理混乱或磁盘吃紧，而且这种情况下 [5. 使用 Version Generator 来生成 Manifest 文件](#5-使用-version-generator-来生成-manifest-文件) 时要注意本地原生打包版本的目录相对路径，比如 ./build/**android-001**/assets。
  {% asset_img rebuild_finished.png %}
- 第二次**生成 APK** 可以直接在 Cocos 编辑器“构建发布”面板中点击“生成”，不需要启动 Android Studio，但要注意 `.apk` 生成在 build 目录的 `build\app\publish\` 目录下

{% endmessage %}

可以看到新打包后的 manifest 文件里的 IP 已经是 `192.168.0.105` 了。

![sear_rebuild_manifest](sear_rebuild_manifest.png)

然后将新生成的远程版本资源和 `project.manifest` 、`version.manifest` 覆盖掉服务器上之前的旧资源。

在模拟器或者真机上安装新生成的 APK 并打开，点击“检查更新”，会显示“Already up to date with the latest remote version.”，也就是说已经更新到最新的远程版本。

![app_check_update](app_check_update.png)

## 7. 热更新

终于要真正开始热更新了。

1. 为了看出更新前后的区别，我把背景的颜色改成了黑色。
2. 重新构建（不需要生成 APK）并把新生成的远程版本资源覆盖掉服务器上之前的旧资源。
3. 重新 [5. 使用 Version Generator 来生成 Manifest 文件](#5-使用-version-generator-来生成-manifest-文件) 并把新生成的 `project.manifest` 、`version.manifest` 覆盖掉服务器上之前的旧资源。
   - 记得升版本号，例如升级到 `1.0.1` ：

     ```cmd
     node version_generator.js -v 1.0.1 -u http://192.168.0.105:7879/ -s ./build/android/assets -d assets/
     ```

重启应用，点击“检查更新”，显示“New version found, please try to update.”。

![new_version_found](new_version_found.png)

点击“立即更新”。

![update_finished](update_finished.png)

更新完毕后会自动进行重启。

{% blockquote @CocosCreatorDocs https://docs.cocos.com/creator/3.8/manual/zh/advanced-topics/hot-update-manager.html#%E9%87%8D%E5%90%AF%E7%9A%84%E5%BF%85%E8%A6%81%E6%80%A7 热更新管理器 AssetsManager - 技术细节解析 - 重启的必要性 %}
如果要使用热更新之后的资源，需要重启游戏。有两个原因，第一是更新之后的脚本需要干净的 JS 环境才能正常运行。第二是场景配置，AssetManager 中的配置都需要更新到最新才能够正常加载场景和资源。
{% endblockquote %}

![update_result](update_result.png)

## 简洁步骤

1. [使用 Version Generator 来生成 Manifest 文件](#5-使用-version-generator-来生成-manifest-文件)
   - 会报错 `Error: ENOENT: no such file or directory`，这是因为没有构建过的话，没有本地原生打包版本的目录。这个错误不影响，主要是修改 Manifest 文件中的 IP，当然你也可以自己手动修改。
    ![version_generator_error](version_generator_error.png)
2. [构建生成](#1-构建生成)
3. [运行服务器](#3-运行服务器)
4. [部署远程版本](#4-部署远程版本)
5. [热更新](#7-热更新)

## 实际项目应用

只需要将实际项目设置成类似范例工程的初始环境，后续的步骤就是跟上文一样了。

1. 复制范例工程相关组件、插件和脚本到实际项目中。
    - 将范例工程中的热更新组件 `assets/hotupdate/` 复制到项目中。

        {% blockquote @CocosCreatorDocs https://docs.cocos.com/creator/3.8/manual/zh/advanced-topics/hot-update.html#%E7%83%AD%E6%9B%B4%E6%96%B0%E7%BB%84%E4%BB%B6 资源热更新教程 - 热更新组件 %}
        在范例工程中，热更新组件的实现位于 assets/hotupdate/HotUpdate.ts（GitHub | Gitee）中，开发者可以参考这种实现，也可以自由的按自己的需求修改。
        {% endblockquote %}

        ![hot_update_script](hot_update_script.png)
    - 将范例工程中的 hot-update 插件 `extensions/hot-update` 复制到项目中并启用。

        {% blockquote @CocosCreatorDocs https://docs.cocos.com/creator/3.8/manual/zh/advanced-topics/hot-update.html#%E6%89%93%E5%8C%85%E5%8E%9F%E7%94%9F%E7%89%88%E6%9C%AC 资源热更新教程 - 打包原生版本 %}
        并且应该确保在工程目录的 extensions 文件夹里导入 hot-update 编辑器插件（范例工程里已经导入了该插件）
        该编辑器插件会在每次构建结束后，自动给 main.js 附加上搜索路径设置的逻辑和更新中断修复代码。
        {% endblockquote %}

        ![hot_update_extension](hot_update_extension.png)
        ![hot_update_extension_enable](hot_update_extension_enable.png)
    - 将范例工程中的 version_generator.js 复制到项目中。
2. [使用 Version Generator 来生成 Manifest 文件](#5-使用-version-generator-来生成-manifest-文件)以供后续绑定时使用。
3. 绑定脚本和属性以及事件。
    下列图中有些属性没有进行绑定，是因为没有实现或者没有用到对应功能，可以根据自己的情况来处理。

    - HotUpdate
        ![hot_update_binding](hot_update_binding.png)
    - UpdatePanel
        ![update_panel_binding](update_panel_binding.png)
    - CheckButton
        ![check_button_binding](check_button_binding.png)
    - UpdateButton
        ![update_button_binding](update_button_binding.png)

## 参考

- [资源热更新教程](https://docs.cocos.com/creator/3.8/manual/zh/advanced-topics/hot-update.html)
- [热更新管理器](https://docs.cocos.com/creator/3.8/manual/zh/advanced-topics/hot-update-manager.html)