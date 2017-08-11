---
layout: post
title: "react-navigation的使用"
date: 2017-08-11
tag: ReactNative
---

### `react-navigation`分为三个部分

- `StackNavigator`类似顶部导航条，用来跳转页面和传递参数。
- `TabNavigator` 类似底部标签栏，用来区分模块。
- `DrawerNavigator` 抽屉，类似从App左侧滑出一个页面，具体我没有使用过，在这里不做讲解。

### TabNavigator的基本用法

```
const TabNav = TabNavigator(
  {
    MainTab: {
      screen: HomePage,
      path: '/',
      navigationOptions: {
        title: '首页',
        // tabBarLabel: '首页',
        tabBarIcon: ({ tintColor, focused }) => (
          focused
                    ?
                    <Image
                        source={require('../images/tab_icon_homes@2x.png')}
                    />
                    :
                    <Image
                        source={require('../images/tab_icon_home@2x.png')}
                    />
        ),
      },
    },
    MinePage: {
      screen: MinePage,
      path: '/settings',
      navigationOptions: {
        title: '我的',
        tabBarIcon: ({ tintColor, focused }) => (
        focused
                    ?
                    <Image
                        source={require('../images/tab_icon_usercenters@2x.png')}
                    />
                    :
                    <Image
                        source={require('../images/tab_icon_usercenter@2x.png')}
                    />
        ),
      },
    },
  },
  {
    tabBarPosition: 'bottom',
    animationEnabled: false,
    swipeEnabled: false,
    tabBarOptions:{
        activeTintColor: '#5c6de5', // 改变tabBar选中字体颜色
        // inactiveTintColor: '#CC9999', // 文字和图片默认颜色
        // showIcon: true, // android 默认不显示 icon, 需要设置为 true 才会显示
        // indicatorStyle: {
        //     height: 0
        // }, // android 中TabBar下面会显示一条线，高度设为 0 后就不显示线了， 不知道还有没有其它方法隐藏？？？
        // style: {
        //     backgroundColor: '#FFFF99', // TabBar 背景色
        // },
        // labelStyle: {
        //     fontSize: 12, // 文字大小
        // },
    }
  }
);
```

### `StackNavigator`的基本用法

```
const StacksOverTabs = StackNavigator({
  Root: {
    screen: TabNav,
  },
  TwoPage: {
    screen: TwoPage,
    navigationOptions: {
      title: '二级页面',
    },
  },
  Profile: {
    screen: FirstPage,
    path: '/people/:name',
    navigationOptions: ({ navigation }) => {
      title: `${navigation.state.params.name}'s Profile!`;
    },
  },
});

export default StacksOverTabs;
```

### 界面之间跳转

```
const MyNavScreen = ({ navigation, banner }) => (
  <ScrollView>
    <SampleText>{banner}</SampleText>
    <Button
    // 界面之间传值
      onPress={() => navigation.navigate('Profile', { name: 'Jordan' })}
      title="去第一个页面"
    />
    <Button
      onPress={() => navigation.navigate('TwoPage')}
      title="去二级页面"
    />
    <Button
      onPress={() => navigation.navigate('MinePage')}
      title="去我的页面"
    />
    <Button onPress={() => navigation.goBack(null)} title="返回" />
  </ScrollView>
);

const HomePage = ({ navigation }) => (
  <MyNavScreen banner="我是首页" navigation={navigation} />
);

const FirstPage = ({ navigation }) => (
  <MyNavScreen
    banner={`${navigation.state.params.name}s Profile`}
    navigation={navigation}
  />
);

const TwoPage = ({ navigation }) => (
  <MyNavScreen banner="我是二级页面" navigation={navigation} />
);

const MinePage = ({ navigation }) => (
  <MyNavScreen banner="我是我的页面" navigation={navigation} />
);
```

效果图：

![](/images/posts/react-native-navigator.png)


