---
title: IntelliJ IDEA 共享 IDE 设置
date: 2018-08-12 15:36:33
categories: 工具
tags:
  - Android Studio
  - IntelliJ IDEA
---

IntelliJ IDEA 支持安装在不同计算机上的 IntelliJ IDEA（或其他基于 IntelliJ 平台的）产品的不同实例之间共享 IDE 设置。如果您使用不同的计算机工作，这可以帮助您重新创建一个舒适的工作环境，而且不会让你对事物的外观和行为感到厌烦。

您可以通过以下方式之一共享 IDE 设置：

 - 通过配置 [Settings Repository](https://www.jetbrains.com/help/idea/sharing-your-ide-settings.html#settings-repository)。这允许您同步任何可配置的组件（启用和禁用插件列表除外），但需要根据您想要共享的设置创建 Git 存储库。如果要在团队成员中实施相同的设置，此选项很有用。

 - 通过使用 [IDE Settings Sync](https://www.jetbrains.com/help/idea/sharing-your-ide-settings.html#IDE_settings_sync) 插件。它使用了 JetBrains 服务器，因此不需要额外的配置。已同步的设置与您的 [JetBrains 帐户](https://sales.jetbrains.com/hc/en-gb/articles/208459005-What-is-JetBrains-Account-)相关联 ，因此其他用户无法使用这些设置。

可以同步的设置包括：IDE 主题、键盘映射、配色方案、系统设置、UI设置，菜单和工具栏设置、项目视图设置，编辑器设置、代码完成设置、参数名称提示、实时模板、代码样式和列表启用和禁用插件。

## 通过 Settings Repository 共享设置 ##
### 使用条件 ###
在开始使用 Settings Repository 之前，请确保 **Settings Repository** 插件已启用。该插件与IntelliJ IDEA 捆绑在一起，默认情况下处于启用状态。如果该插件未启用，请在 **Settings / Preferences Dialog** 对话框的 **Plugins** 页上启用它。

### 配置 Settings Repository ###
如果要共享 IDE 设置，请执行以下步骤：
1. 在任何托管服务上创建 Git 存储库，例如 [Bitbucket](https://bitbucket.org/) 或 [GitHub](https://github.com/)。
2. 在安装了要共享其设置的 IntelliJ IDEA 实例的计算机上，导航到 **File | Settings Repository**。指定创建的远程仓库的 URL，然后点击 **Overwrite Remote**。
3. 在要应用设置的每台计算机上，在 **Settings/Preferences dialog** 对话框中，展开 **Tools** 节点并选择 **Settings Repository**，指定创建的远程仓库的 URL，然后点击 **Overwrite Local**。
如果想要储存库保留远程设置和本地设置的组合，可以点击 **Merge**。如果检测到任何冲突，将显示一个对话框，可以在其中解决这些冲突。如果要使用本地设置覆盖远程设置，请单击点击 **Overwrite Remote**。
> 提示：如果选择使用 [Bitbucket](https://bitbucket.org/) 托管你的存储库，建议使用 [App passwords](https://bitbucket.org/account/admin/app-passwords) 进行身份验证。您需要为存储库设置读/写权限。

每次执行 **Update Project** 或 **Push** 操作时，或者当关闭项目或退出 IntelliJ IDEA 时，计算机的本地设置将自动与远程仓库中的设置同步。

在第一次同步时，系统将提示您指定用户名和密码。建议使用 [access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/) 进行 GitHub 身份验证。如果由于某种原因，您想要使用用户名和密码而不是 access token，或者您的 Git 托管服务提供商不支持它，建议您配置 [Git credentials helper](https://help.github.com/articles/caching-your-github-password-in-git/)。

如果要禁用自动设置同步，请导航到 **File | Settings | Tools | Settings Repository** 并禁用 **Auto Sync** 选项。您可以通过从主菜单选择 **VCS | Sync Settings** 来手动更新设置。
> 请注意：[macOS Keychain](https://support.apple.com/kb/PH20093) 是受支持的，这意味着您可以在所有基于IntelliJ 平台的产品之间共享凭据（如果原始 IDE 与请求方 IDE 不同，系统将提示您授予访问权限）。

### 通过其他只读存储库共享更多设置 ###
除了 **Settings Repository**，还可以配置任意数量的其他存储库，其中包含要共享的任何类型的设置，包括实时模板、文件模板、方案、部署选项等。

这些存储库被称为**只读源**，因为它们不能被覆盖或合并，仅用作设置源。

要配置此类存储库，请执行以下操作：
1. 在 **Settings / Preferences Dialog** 对话框中，展开 **Tools** 节点，然后选择 **Settings Repository**。
2. 单击“+”并添加包含要共享设置的 GitHub 仓库的 URL。

与只读源中的设置进行同步的方法与 **Settings Repository** 的方式相同。

## 通过 Settings Sync plugin 共享设置 ##
### 使用条件 ###
在开始使用 Settings Sync 之前，请确保 **Settings Sync** 插件已启用。如果该插件未启用，请在 **Settings / Preferences Dialog** 对话框的 **Plugins** 页上启用它。

### 配置 Settings Sync plugin ###
如果要共享 IDE 设置，请执行以下步骤：
1. 登录以下任一项：
 - 您的 IDE：从主菜单中选择 **Help | Register**，选择使用 [JetBrains 帐户](https://sales.jetbrains.com/hc/en-gb/articles/208459005-What-is-JetBrains-Account-) 激活您的许可证并输入您的凭据。
 - [Toolbox App](https://www.jetbrains.com/toolbox/app/)：单击应用程序右上角的齿轮图标，然后选择 **Settings** 并单击 **Log in** 按钮。请注意，通过登录 Toolbox App，您将自动登录到您运行的所有 JetBrains 产品。
2. 在 IntelliJ IDEA 窗口的右下角，单击齿轮图标并选择 **Enable Settings Sync**。您的本地设置将导出到关联您的帐户的 JetBrains 存储库。
3. 如果想要自动同步所有已启用和已禁用插件的列表，请选择 **Sync plugins silently** 选项。有关如何禁用手动同步插件的说明，请参阅 [Sync plugins](https://www.jetbrains.com/help/idea/sharing-your-ide-settings.html#sync-plugins)。
4. 在要应用这些设置的其他计算机上，单击齿轮按钮并选择 **Enable Sync**。在打开的对话框中，单击 **Get Settings from Account** 以从存储库导入设置。如果要使用本地设置覆盖存储库，请单击 **Keep and Sync Local Settings**。

每次运行不同的 IDE 实例时（或者在超过一小时不活动后激活它），或者当任何这些设置被修改并且已应用此更改时，本地设置将自动与存储在存储库中的设置同步。

### Sync plugin ###
安装或卸载插件或更改其状态（启用/禁用）时，可以将这些更改应用于所有 IDE 安装。

如果想要在 IDE 实例之间自动同步插件，请在启用设置同步时选择 **Sync plugins silently** 选项。

手动同步插件的步骤：
1. 在 IntelliJ IDEA 窗口的右下角，单击齿轮图标并选择 **Sync Plugins**。
2. 打开一个对话框，显示自上次同步以来修改的所有插件的列表。单击每个插件旁边的箭头按钮，然后选择修改插件的状态、将存储库状态应用于所有安装、在本地跳过此更改或跳过所有 IDE 实例。
3. 在为每个插件选择了要执行的操作后，单击 **Apply Changes**。

## 参考 ##
[Sharing IDE Settings](https://www.jetbrains.com/help/idea/sharing-your-ide-settings.html)
