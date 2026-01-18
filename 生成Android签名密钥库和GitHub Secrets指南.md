# 生成Android签名密钥库和GitHub Secrets指南

为了使用这个GitHub Actions工作流构建签名的APK，你需要生成一个Android密钥库（Keystore），并将其相关配置添加到GitHub仓库的Secrets中。以下是完整的操作步骤。

## 第一步：生成Android签名密钥库

Android应用签名需要一个密钥库文件（.jks或.keystore格式）。你可以使用Java的keytool命令来生成。在终端中执行以下命令：

```bash
keytool -genkeypair -v -storetype JKS -keyalg RSA -keysize 2048 \
  -validity 10000 \
  -keystore android_keystore.jks \
  -alias myappkey \
  -keypass your_key_password \
  -storepass your_keystore_password
```

执行此命令后，系统会要求你填写一些信息，包括姓名、组织、单位、所在城市、所在省份、国家代码等。填写完成后，会在当前目录下生成一个名为`android_keystore.jks`的密钥库文件。

**重要提醒**：请务必妥善保管这个密钥库文件，不要上传到公开仓库。一旦丢失，你将无法更新已发布的应用。另外，建议将有效期设置为较长的时间（如10000天），避免密钥过期导致应用无法更新。

## 第二步：生成Base64编码的密钥库

GitHub Actions工作流中使用`secrets.APP_KEYSTORE_BASE64`来传递密钥库文件，这个secret需要的是密钥库的Base64编码内容。在终端中执行以下命令生成编码：

```bash
base64 android_keystore.jks > android_keystore_base64.txt
```

然后查看生成的文件内容：

```bash
cat android_keystore_base64.txt
```

复制这个Base64字符串，稍后需要添加到GitHub Secrets中。

## 第三步：配置GitHub仓库Secrets

打开你的GitHub仓库页面，依次点击页面上方的**Settings**（设置）选项卡，然后在左侧菜单中找到**Secrets and variables**下的**Actions**选项。点击**New repository secret**按钮，按照以下信息创建四个secret：

第一个secret的名称为`APP_KEYSTORE_BASE64`，值为第一步生成的Base64编码字符串。第二个secret的名称为`APP_KEYSTORE_PASSWORD`，值为生成密钥库时设置的密钥库密码（storepass）。第三个secret的名称为`APP_KEYSTORE_ALIAS`，值为生成密钥库时设置的别名（alias），即`myappkey`。第四个secret的名称为`APP_KEY_PASSWORD`，值为生成密钥库时设置的密钥密码（keypass）。

创建完成后的Secrets列表应该包含这四个条目，确保每个secret的名称和值都准确无误。

## 第四步：验证配置是否正确

完成上述配置后，你可以手动触发GitHub Actions工作流进行测试。在仓库页面点击**Actions**选项卡，选择"Build APK"工作流，然后点击**Run workflow**按钮手动触发。构建完成后，检查Artifacts是否成功上传，以及下载的APK是否正确签名。

## 安全注意事项

保护好你的密钥库文件和相关密码是至关重要的。如果这些信息泄露，攻击者可以冒充你发布恶意应用更新。建议采取以下安全措施：不要将密钥库文件上传到任何公开仓库或代码托管平台；定期备份密钥库文件到安全的离线存储介质；在不同环境使用不同的密码；监控GitHub账户的异常活动。

如果你需要将应用发布到Google Play商店，还需要准备Google Play App Signing。Google Play会自动对APK进行重新签名，但前提是你将发布的APK使用应用签名密钥签名。你可以导出应用的签名密钥并上传到Google Play控制台进行配置。
