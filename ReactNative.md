[TOC]

创建项目 react-native init myproject

插件快捷命令:rnf(react native function)、rnfs(带StyleSheet)

# CSS

RN中的样式与CSS的不同：没有继承性（RN中的继承只发生在Text组件上）、样式名采用小驼峰命名、所有尺寸都没有单位、有些特殊的样式名(marginHorizontal水平外边距、marginVertical垂直外边距)

## StyleSheet

可以通过在style属性中调用StyleSheet声明样式(个人觉得有点像scss)

```react
import { StyleSheet, Text, View } from 'react-native'

export default function App () {
  return (
    <View style={styles.container}>
      <Text>Open up App.js to start working on your app!</Text>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
})
```

## Dimensions

Dimensions可以用来获取屏幕的尺寸

```react
import { StatusBar } from 'expo-status-bar'
import { StyleSheet, Text, View, Dimensions } from 'react-native'
export default function App () {
  const windowWidth = Dimensions.get('window').width
  const windowHeight = Dimensions.get('window').height
  return (
    <View style={styles.container}>
      <Text>{windowWidth}</Text>
      <Text>{windowHeight}</Text>
      <StatusBar />
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  }
})
```

# 核心组件

## Button && Alert

```react
import { View, StyleSheet, Button, Alert } from 'react-native'
export default function App () {
  const handlePressTwo = () => {
    Alert.alert(
      "警告标题",
      "警告内容",
      [
        {
          text: "取消",
          onPress: () => {
            console.log('Cancel')
          },
          style: 'cancel'
        },
        {
          text: "确定",
          onPress: () => {
            console.log('OK')
          },
          style: "default"
        }
      ]
    )
  }
  const handlePressThree = () => {
    Alert.alert(
      "更新提示",
      "发现新版本，是否现在更新",
      [
        {
          text: "稍后再试",
          onPress: () => {
            console.log('稍后提醒我')
          },
        },
        {
          text: "取消",
          onPress: () => {
            console.log('Cancel')
          },
          style: 'cancel'
        },
        {
          text: "确定",
          onPress: () => {
            console.log('OK')
          },
          style: "default"
        },
      ]
    )
  }
  return (
    <View style={styles.container}>
      <Button
        title='第一个按钮'
        color={'red'}
        onPress={() => {
          Alert.alert('我是第一个按钮')
        }}
      ></Button>
      <Button
        title='第两个按钮'
        color={'red'}
        onPress={handlePressTwo}
      ></Button>
      <Button
        title='第三个按钮'
        color={'red'}
        onPress={handlePressThree}
      ></Button>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'space-around',
    alignItems: 'center'
  }
})
```

总结：

Button常用属性：title、onPress、color（注意不要以style定义button的颜色）

Alert常用属性：Alert.alert() 可以再里面依次写警告标题、内容、以及N个按钮

## StatusBar && Switch

```react
import { StyleSheet, View, StatusBar, Switch } from 'react-native'
import React, { useState } from 'react'

export default function App () {
  const [hideStatus, setHideStatus] = useState()
  return (
    <View style={[styles.container]}>
      <StatusBar
        hidden={hideStatus}
        barStyle={'dark-content'} />
      <Switch
        trackColor={{ false: 'red', true: 'green' }}
        thumbColor={'blue'}
        value={hideStatus}
        onValueChange={() => {
          setHideStatus(!hideStatus)
        }} />
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  }
})
```

总结：

StatusBar常用属性：hidden、backgroundColor、barStyle

Switch常用属性：trackColor背景色、thumbColor小圆点颜色、value、onValueChange

## ActivityIndicator && Platform

```react
import { StyleSheet, View, ActivityIndicator, Platform } from 'react-native'
import React from 'react'

export default function App () {
  if (Platform.OS === 'android') {
    alert('安卓')
  } else if (Platform.OS === 'ios') {
    alert('IOS')
  }
  return (
    <View style={styles.container}>
      <ActivityIndicator
        color={'red'}
        size={'large'} />
      {/* 数字指定大小只在安卓端有效 */}
      <ActivityIndicator
        color={'red'}
        size={70} />
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center'
  }
})
```

总结：

ActivityIndicator常用属性：color、size

Platform不是组件，可以用来判断当前系统

## Image

```react
import { StyleSheet, Text, View, Image, Dimensions } from 'react-native'
import React from 'react'

export default function App () {
  return (
    <View style={styles.container}>
      <Text>App</Text>
      <Image style={styles.itemImage}
        source={require('./assets/favicon.png')}></Image>
      <Image source={{
        uri: 'http://www.xxx.com/logo.png'
      }}></Image>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  itemImage: {
    height: Dimensions.get('window').height / 5,
    width: Dimensions.get('window').width / 5,
    marginVertical: 20
  }
})
```

总结：

Image常用属性：style、source(本地图片用require，线上图片用{uri:xxxx})

## TextInput

```react
import { StyleSheet, View, TextInput, Dimensions, Button } from 'react-native'
import React, { useState } from 'react'

export default function App () {
  const [username, setUsername] = useState('')
  const [password, setPassword] = useState()
  const [phone, setPhone] = useState('')
  const [text, setText] = useState()
  const doLogin = () => {
    alert(username + password + text)
  }
  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        placeholder="请输入用户名"
        value={username}
        onChangeText={(val) => {
          setUsername(val)
        }} />
      <TextInput
        style={styles.input}
        placeholder="请输入密码"
        value={password}
        secureTextEntry={true}
        onChangeText={(val) => {
          setPassword(val)
        }} />
      <TextInput
        style={styles.input}
        placeholder="请输入手机号"
        keyboardType='number-pad'
        value={phone}
        onChangeText={(val) => {
          setPhone(val)
        }} />
      <TextInput
        style={styles.input}
        placeholder="请输入自我介绍"
        multiline={true}
        numberOfLines={5}
        textAlignVertical="top"
        value={text}
        onChangeText={(val) => {
          setText(val)
        }} />
      <View>
        <Button
          title='登录'
          onPress={doLogin}></Button>
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  input: {
    width: Dimensions.get('window').width - 20,
    height: 40,
    margin: 10,
    borderWidth: 1,
    borderColor: 'red',
    paddingHorizontal: 5
  }
})
```

总结：

TextInput常用属性要根据输入框类型，具体可以看示例

## Touchable

TouchableHighlight 触碰后，高亮显示

TouchableOpacity 触碰后，透明度降低（模糊显示）

TouchableWithoutFeedback 触碰后，无任何相应

```react
import {
  StyleSheet, View, Text,
  TouchableHighlight,
  TouchableOpacity,
  TouchableWithoutFeedback
} from 'react-native'
import React from 'react'

export default function App () {
  return (
    <View style={styles.container}>
      <TouchableHighlight
        onPress={() => console.log('触碰高亮显示')}>
        <View style={styles.item}>
          <Text>触碰高亮</Text>
        </View>
      </TouchableHighlight>
      <TouchableOpacity
        onPress={() => console.log('触碰透明度变化')}>
        <View style={styles.item}>
          <Text>触碰透明度变化</Text>
        </View>
      </TouchableOpacity>
      <TouchableWithoutFeedback
        onPress={() => console.log('触碰无响应')}>
        <View style={styles.item}>
          <Text>触碰无响应</Text>
        </View>
      </TouchableWithoutFeedback>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  item: {
    marginBottom: 20,
    padding: 10,
    borderWidth: 1,
    borderColor: 'red'
  }
})
```

## ScrollView && SafeAreaView

SafeAreaView相比普通的View会避免刘海屏问题

ScrollView在安卓下会有一个显示不全的问题，可以在ScrollView内部的最下方加一个View，根据Platform来给iOS0高度，安卓根据丢失文本高度给高度

```react
import {
  StyleSheet, View, Text,
  ScrollView
} from 'react-native'
import React from 'react'

export default function App () {
  return (
    <View>
      <Text>111</Text>
      <Text>111</Text>
      <Text>111</Text>
      <ScrollView
        horizontal={true}>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
      </ScrollView>
      <ScrollView
        contentContainerStyle={{ margin: 30 }}
        showsVerticalScrollIndicator={false}>
        <Text style={styles.text}>
          aaaaaa
          aaaaaa
          aaaaaa
          aaaaaa
          aaaaaa
          aaaaaa
          aaaaaa
        </Text>
      </ScrollView>
    </View>
  )
}

const styles = StyleSheet.create({
  text: {
    fontSize: 130
  },
  nav: {
    margin: 10,
    height: 50,
    fontSize: 20
  }
})
```

