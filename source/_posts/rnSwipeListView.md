---
title: react-native-swipe-list-view
cover: /covers/rnSwipeListView.jpg
date: 2020-01-21 14:21:58
category: 移动端
tags: react-native
excerpt: react-native-swipe-list-view的使用和注意事项
---

## 前言

&emsp;&emsp;`react-native-swipe-list-view`是一个`React-Native`上用于侧滑的组件，基于 RN 原生的`ListView`，网上有很多关于`react-native-swipe-list-view`使用的博客，但是很多都没有注意一些使用的细节，比如如果是删除组件，删除以后的效果如何，是否正常运行？我下面就把作为侧滑删除的时候用法完整的说一下。

## 基本使用

&emsp;&emsp;直入正题，下面是主要部分的代码:

```typeScript
import { SwipeListView } from 'react-native-swipe-list-view';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import ListEmpty from '@/components/ListEmpty';
import { Size, Colors } from '@/config';
import Iconfont from '@/components/Iconfont';

const { px } = Size;

<SwipeListView
  data={addressList}
  closeOnRowPress={true}
  keyExtractor={item => String(item.id)}  // 实现删除后自动关闭的关键
  renderItem={data => {
    const { item } = data;
    const {
      id,
      receiverName = '',
      detailedAddress = '',
      phone = '',
      provinceName = '',
      cityName = '',
      districtName = '',
    } = item;
    return (
      <View style={styles.addressRow}>
        <View style={styles.infoContent}>
            <View style={styles.infoHead}>
              <Text style={styles.userName}>{receiverName}</Text>
              <Text style={styles.userTel}>{phone}</Text>
            </View>
            <Text
              style={styles.userAddress}
            >{`${provinceName}${cityName}${districtName}${detailedAddress}`}</Text>
        </View>
        <TouchableOpacity style={styles.editWrap} onPress={() => handleEdit(id)}>
          <Iconfont name="iconbianjix" size={Size.px(16)} color={Colors.dark} />
        </TouchableOpacity>
      </View>
    );
  }}
  renderHiddenItem={data => (
    <TouchableOpacity style={styles.rowBack} onPress={() => deleteAddress(data.item.id)}>
      <Text style={styles.deleteBtn}>删除</Text>
    </TouchableOpacity>
  )}
  disableRightSwipe={true}
  rightOpenValue={-96}
  ListEmptyComponent={<ListEmpty />}
/>

// 样式代码
const styles = StyleSheet.create({
  rowBack: {
    display: 'flex',
    flexDirection: 'row',
    justifyContent: 'flex-end',
    alignItems: 'center',
    backgroundColor: Colors.red,
    height: '100%',
  },
  editWrap: {
    display: 'flex',
    justifyContent: 'center',
    alignItems: 'center',
  },
  closeBtn: {
    position: 'absolute',
    right: px(0),
    top: -px(30),
  },
  deleteBtn: {
    fontSize: px(16),
    color: Colors.white,
    marginRight: px(25),
  },
  deleteText: {
    textAlign: 'center',
    fontSize: px(14),
  },
  addressRow: {
    width: '100%',
    display: 'flex',
    justifyContent: 'space-between',
    flexDirection: 'row',
    backgroundColor: Colors.white,
    paddingVertical: px(15),
    paddingHorizontal: px(18),
    borderBottomWidth: Size.ONE_PIXEL,
    borderBottomColor: Colors.borderColor,
  },
  infoHead: {
    display: 'flex',
    flexDirection: 'row',
    marginBottom: px(12),
  },
  userName: {
    fontSize: px(16),
    color: Colors.dark,
  },
  userTel: {
    fontSize: px(16),
    color: Colors.dark,
    marginLeft: px(16),
  },
  userAddress: {
    fontSize: px(13),
    color: Colors.labelColor,
  },
  infoContent: {
    display: 'flex',
  },
});

```

&emsp;&emsp;最终实现效果如下，删除后会自动关闭打开的行：

![实现效果](/images/posts/rnSwipeListView/result.jpg)

&emsp;&emsp;解释一下参数：

- data：这个是你要传给 SwipeListView 的数据，是一个数组。
- closeOnRowPress：决定点击行后是否关闭行，其实实际效果是无论是否为`true`，点击行后都会自动关闭行。
- keyExtractor：这个是唯一标识键，实现删除后自动关闭的关键，在很多博客里都没有提到，如果用的是删除组件，没有这个的话就会导致上一行删除后下一行的删除变成打开状态的 bug。接收参数为数组中的每一项。
- renderItem：渲染每一行的元素，要注意的是接收一个`data`，`data.item`才是数组中的每一项。`handleEdit`是点击编辑的操作。
- renderHiddenItem：渲染隐藏的元素，这里隐藏的是一个删除的按钮。`deleteAddress`是执行删除的函数。
- disableRightSwipe：禁止右滑，因为删除组件是只有左滑的，所以设为`true`。
- rightOpenValue：右边隐藏元素打开时的长度，要注意的是这是一个负数。
- ListEmptyComponent：列表为空的时候展示的元素。这里的`<ListEmpty />`就是空数据的组件。

&emsp;&emsp;基本上删除组件会用到的属性就是上面这些，希望对你有所帮助~
