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

Button&&Alert

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
