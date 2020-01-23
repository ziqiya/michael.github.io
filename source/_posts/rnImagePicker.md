---
title: react-native-image-picker
cover: /covers/rnImagePicker.jpg
date: 2020-01-23 11:16:32
category: 技术
tags: react-native
excerpt: react-native-image-picker实战，实现图片上传
---

## 前言

&emsp;&emsp;图片上传的功能在手机 APP 上用的也是很多了，今天主要讲一下怎么用`react-native-image-picker`具体实现安卓和 IOS 上的图片上传并解决一些常见问题。本文还会涉及`rn-fetch-blob`上传插件。

## React Native Image Picker

&emsp;&emsp;首先介绍一下`react-native-image-picker`是什么，官网上的解释：是一个可以让你用原生 UI 的形式选择手机上的图片、视频或者直接拍照的方式来上传文件的`React Native`的组件。但他只是帮我们完成了从手机里拿到照片的那一步，至于如何展示，上传还是需要我们自己实现。

## 实现步骤

- 1.我们需要写一个照片上传的容器来展示图片，抽组件。
- 2.引入`react-native-image-picker`完成图片获取，上传成功后将 source 放到写好的容器中。
- 3.上传成功后执行上传，调用文件服务的上传接口，这里用到了`rn-fetch-blob`插件。
- 4.最后是 ios 端的照片权限兼容。

## 代码分析

### 图片容器

&emsp;&emsp;首先是写一个照片上传的容器来展示图片,这里我写了一个`CustomImagePicker`的组件，`CustomImagePickerProps`中调用的时候要传入的属性：

```typeScript
import React, { useState } from 'react';
import { Text, StyleSheet, Image, ImageBackground, TouchableOpacity, ImageSourcePropType } from 'react-native';
import { Colors, Size } from '@/config';

/** 初始化自定义配置 */
const initialImageOptions: Options = {
  title: '选择图片',
  storageOptions: {
    skipBackup: true,
    path: 'images',
  },
  mediaType: 'photo',
  chooseFromLibraryButtonTitle: '图片库',
  cancelButtonTitle: '取消',
  takePhotoButtonTitle: '拍照',
};

interface CustomImagePickerProps {
  /** 初始化背景图 */
  imgSource?: ImageSourcePropType;
  /** 其他图片自定义配置,详细参考react-native-image-picker的option配置 */
  imgConfig?: Options;
  /** 悬浮文字 */
  title?: string;
  /** 取消事件回调 */
  onCancel?: (response: Response) => void;
  /** 失败事件回调 */
  onFailed?: (response: Response) => void;
  /** 成功事件回调,返回文件路径 */
  onSuccess?: (fileUrl: string) => void;
}
const CustomImagePicker: React.FC<CustomImagePickerProps> = props => {
  const { imgSource = require('@/assets/pic_empty.png'), title = '', imgConfig, onCancel, onFailed, onSuccess } = props;
  const imagePickerOptions = { ...initialImageOptions, ...imgConfig };
  const [currentImgSource, setCurrentImgSource] = useState();

return(
  const [currentImgSource, setCurrentImgSource] = useState();
   <TouchableOpacity onPress={handleUploadImage}>
      <ImageBackground source={currentImgSource || imgSource} style={styles.backgroundImg}>
        {!currentImgSource && (
          <>
            <Image source={require('@/assets/add-upload.png')} style={styles.addImg}></Image>
            <Text style={styles.titleText}>{title}</Text>
          </>
        )}
      </ImageBackground>
    </TouchableOpacity>
    )
  }

// 样式部分
const styles = StyleSheet.create({
  backgroundImg: {
    width: px(277),
    height: px(152),
    marginLeft: px(28),
    zIndex: 0,
  },
  addImg: {
    width: px(64),
    height: px(64),
    marginTop: px(33),
    marginLeft: px(110),
  },
  titleText: {
    fontSize: px(15),
    color: Colors.grey,
    marginTop: px(19),
    textAlign: 'center',
  },
});
```

&emsp;&emsp;需要用到`TouchableOpacity`来触发图片的上传操作，`ImageBackground`用来展示`imagePicker`拿到的图片，`currentImgSource`是当前选中的图片`source`。这样最基本的组件结构已经有了。

### 引入 ImagePicker 回调方法

&emsp;&emsp;接下来就是点击`TouchableOpacity`以后调用`ImagePicker`的`showImagePicker`的 API 拿到本地图片的地址`uri`,文件名称`fileName`,文件类型`type`并作为参数上传文件。`imagePickerOptions`是图片上传弹窗的基本配置。

```typeScript
import ImagePicker from 'react-native-image-picker';

  /** ImagePicker上传调用 */
  const handleUploadImage = () => {
    ImagePicker.showImagePicker(imagePickerOptions, async response => {
      if (response.didCancel) {
        // 用户取消上传 回调
        onCancel && onCancel(response);
      } else if (response.error) {
        // 上传失败 回调
        onFailed && onFailed(response);
      } else {
        const source = { uri: response.uri };
        const file = {
          fileName: response.fileName!,
          fileType: response.type!,
          uri: response.uri,
        };
        // 上传成功 回调
        const uploadResult = await uploadFile(file);
        if (uploadResult.success) {
          setCurrentImgSource(source);
          onSuccess && onSuccess(uploadResult.file);
        } else {
          toastFail('上传失败');
        }
      }
    });
  };
```

### 图片上传

&emsp;&emsp;图片上传因为传统的 postForm 的上传方式后台无法接受到文件参数，用了`formData.append`效果也不太理想。最后用了`rn-fetch-blob`插件完美解决，`rn-fetch-blob`是一个用于简化文件上传的插件，具体可看官网: <a href="https://www.npmjs.com/package/rn-fetch-blob">rn-fetch-blob</a>。图片上传逻辑如下：

```typeScript
import RNFetchBlob from 'rn-fetch-blob';

  /** 上传文件 */
  const uploadFile = async ({ fileName, fileType, uri }: { fileName: string; fileType: string; uri: string }) => {
    try {
      const resultData = await RNFetchBlob.fetch(
        'POST',
        `${UPLOAD_URL}/upload/public?access_token=${UPLOAD_TOKEN}`,
        {
          'Content-Type': 'multipart/form-data',
        },
        [
          {
            name: 'file',
            filename: fileName,
            type: fileType,
            data: RNFetchBlob.wrap(uri.replace('file://', '')),
          },
        ],
      );
      const result = resultData.json();
      if (!result.success) {
        toastFail(result.message);
      }
      return {
        success: result.success,
        file: result.data || '',
      };
    } catch (error) {
      if (error.message) {
        toastFail(error.message);
      }
      return {
        success: false,
        file: '',
      };
    }
  };
```

&emsp;&emsp;其中`UPLOAD_URL`是文件服务的地址，`UPLOAD_TOKEN`是文件上传需要的 `accessToken`。

### rn-fetch-blob 的一些问题

- Q: `rn-fetch-blob`依赖装上以后报错`Cannot read property 'DocumentDir' of undefined`？  
  A: `rn-fetch-blob`版本需要是`"0.10.15"`,否则不兼容 android。

- Q: NetWork 里看不到请求？  
  A: `rn-fetch-blob` 的文件请求走的不是 netWork，所以在 netWork 里是看不到请求的，可以用 `console.log` 的方式查看请求。

### IOS 端权限兼容

&emsp;&emsp;最后是 IOS 端的权限兼容，因为 ios 的 APP 会限制照片，照相，存照片的权限，如果什么都不做就会报`Thread x: signal SIGABRT`的错误，导致 APP 闪退卡死。  
&emsp;&emsp;所以需要在项目的`ios/项目名称`文件夹中找到`Info.plist`文件，在最后加上三个 `key` 和对应的信息 `string`:

```javascript
  <key>NSPhotoLibraryAddUsageDescription</key>
	<string>请允许APP保存图片到相册</string>
	<key>NSPhotoLibraryUsageDescription</key>
	<string>请允许APP访问您的相册</string>
	<key>NSCameraUsageDescription</key>
	<string>请允许APP访问您的相机</string>
```

问题解决完以后，最后实现效果如下:
![ios实现效果](/images/posts/rnImagePicker/ios-result.jpg)

![安卓实现效果](/images/posts/rnImagePicker/android-result.jpg)
