---
layout: post
title: "ReactNative-底部TabBar react-native-tab-navigator"
date: 2017-08-13
tag: ReactNative
---

首先需要安装 `react-native-tab-navigator`

`npm install react-native-tab-navigator –save`

导入组件

```
import TabNavigator from 'react-native-tab-navigator'
```

详细代码如下：

```
import React, { Component } from 'react';
import {
  AppRegistry,
  ScrollView,
  StyleSheet,
  TouchableOpacity,
  Text,
  Image,
  View,
} from 'react-native';

import { StackNavigator } from 'react-navigation';

import TabNavigator from 'react-native-tab-navigator'

export default class DemoApp extends Component {

    constructor(props){  
        super(props)  
        this.state={  
            selectedTab:'首页',  
        }  
    } 

  render() {
    return (
      <View style={styles.container}>
        <TabNavigator>
            <TabNavigator.Item
                selected={this.state.selectedTab === '首页'}
                title = '首页'
                titleStyle={styles.tabText}
                selectedTitleStyle={styles.selectedTabText} 
                renderIcon={() => <Image style={styles.icon} source={require("./images/tab_icon_home@2x.png")} />}

                renderSelectedIcon={() => <Image style={[styles.icon,{tintColor:'red'}]} source={require("./images/tab_icon_homes@2x.png")} />}
                
                onPress={() => this.setState({ selectedTab: '首页' })}
            >
                <View style={styles.page0}>
                    <Text style={{fontSize:18,padding:15,color: 'blue'}}>首页</Text>
                </View>
            </TabNavigator.Item>

            <TabNavigator.Item
                selected={this.state.selectedTab === '购物车'}
                title = '购物车'
                titleStyle={styles.tabText}
                selectedTitleStyle={styles.selectedTabText} 
                renderIcon={() => <Image style={styles.icon} source={require("./images/tab_icon_home@2x.png")} />}

                renderSelectedIcon={() => <Image style={[styles.icon,{tintColor:'red'}]} source={require("./images/tab_icon_homes@2x.png")} />}
                
                onPress={() => this.setState({ selectedTab: '购物车' })}
            >
                <View style={styles.page1}>
                    <Text style={{fontSize:18,padding:15,color: 'blue'}}>购物车</Text>
                </View>
            </TabNavigator.Item>

            <TabNavigator.Item
                selected={this.state.selectedTab === '目的地'}
                title = '目的地'
                titleStyle={styles.tabText}
                selectedTitleStyle={styles.selectedTabText} 
                renderIcon={() => <Image style={styles.icon} source={require("./images/tab_icon_destn@2x.png")} />}

                renderSelectedIcon={() => <Image style={[styles.icon,{tintColor:'red'}]} source={require("./images/tab_icon_destns@2x.png")} />}
                
                onPress={() => this.setState({ selectedTab: '目的地' })}
            >
                <View style={styles.page0}>
                    <Text style={{fontSize:18,padding:15,color: 'blue'}}>目的地</Text>
                </View>
            </TabNavigator.Item>

            <TabNavigator.Item
                selected={this.state.selectedTab === '我的'}
                title = '我的'
                titleStyle={styles.tabText}
                selectedTitleStyle={styles.selectedTabText} 
                renderIcon={() => <Image style={styles.icon} source={require("./images/tab_icon_usercenter@2x.png")} />}

                renderSelectedIcon={() => <Image style={[styles.icon,{tintColor:'red'}]} source={require("./images/tab_icon_usercenters@2x.png")} />}
                
                onPress={() => this.setState({ selectedTab: '我的' })}
            >
                <View style={styles.page1}>
                    <Text style={{fontSize:18,padding:15,color: 'blue'}}>我的</Text>
                </View>
            </TabNavigator.Item>
        </TabNavigator>
      </View>
    );
  }
}

const styles = StyleSheet.create({
    container: {
        flex:1
    },
    tabText: {
        fontSize: 10,
        color: 'black'
    },
    selectedTabText: {
        fontSize: 10,
        color: 'red'
    },
    icon: {
        width: 22,
        height: 22
    },
    page0: {
        flex: 1,
        backgroundColor: 'yellow'
    },
    page1: {
        flex: 1,
        backgroundColor: 'red'
    }
});

AppRegistry.registerComponent('DemoApp', () => DemoApp);
```

效果图：

![](/images/posts/2017-08-13-tabbar.png)

