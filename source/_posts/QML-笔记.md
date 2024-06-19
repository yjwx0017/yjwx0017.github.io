title: QML 笔记
categories: 日志
tags: [Qt,QML]
date: 2023-01-18 14:44:00
---
## State

配合使用：state、states、PropertyChanges。

Item 属性：state 当前状态，states 定义所有状态。

非 Item 对象通过 StateGroup 来使用 State。

``` qml
 Rectangle {
     id: signal
     width: 200; height: 200
     state: "NORMAL"

     states: [
         State {
             name: "NORMAL"
             PropertyChanges { target: signal; color: "green"}
             PropertyChanges { target: flag; state: "FLAG_DOWN"}
         },
         State {
             name: "CRITICAL"
             PropertyChanges { target: signal; color: "red"}
             PropertyChanges { target: flag; state: "FLAG_UP"}
         }
     ]
 }
```

when 属性：

``` qml
 Rectangle {
     id: bell
     width: 75; height: 75
     color: "yellow"

     states: State {
                 name: "RINGING"
                 when: (signal.state == "CRITICAL")
                 PropertyChanges {target: speaker; play: "RING!"}
             }
 }
```

<!--more-->

## Animations

Animations：为属性应用动画

Transition 配合 Animations：为状态切换应用动画

### 直接属性动画（Direct Property Animation）

``` qml
 Rectangle {
     id: flashingblob
     width: 75; height: 75
     color: "blue"
     opacity: 1.0

     MouseArea {
         anchors.fill: parent
         onClicked: {
             animateColor.start()
             animateOpacity.start()
         }
     }

     PropertyAnimation {id: animateColor; target: flashingblob; properties: "color"; to: "green"; duration: 100}

     NumberAnimation {
         id: animateOpacity
         target: flashingblob
         properties: "opacity"
         from: 0.99
         to: 1.0
         loops: Animation.Infinite
         easing {type: Easing.OutBack; overshoot: 500}
    }
 }
```

### <Animation> on <Property> 语法

``` qml
 Rectangle {
     id: rect
     width: 100; height: 100
     color: "red"

     PropertyAnimation on x { to: 100 }
     PropertyAnimation on y { to: 100 }
 }
```

``` qml
 Rectangle {
     width: 100; height: 100
     color: "red"

     SequentialAnimation on color {
         ColorAnimation { to: "yellow"; duration: 1000 }
         ColorAnimation { to: "blue"; duration: 1000 }
     }
 }
```

### State 过渡动画

``` qml
 Rectangle {
     width: 75; height: 75
     id: button
     state: "RELEASED"

     MouseArea {
         anchors.fill: parent
         onPressed: button.state = "PRESSED"
         onReleased: button.state = "RELEASED"
     }

     states: [
         State {
             name: "PRESSED"
             PropertyChanges { target: button; color: "lightblue"}
         },
         State {
             name: "RELEASED"
             PropertyChanges { target: button; color: "lightsteelblue"}
         }
     ]

     transitions: [
         Transition {
             from: "PRESSED"
             to: "RELEASED"
             ColorAnimation { target: button; duration: 100}
         },
         Transition {
             from: "RELEASED"
             to: "PRESSED"
             ColorAnimation { target: button; duration: 100}
         }
     ]
 }
```

使用通配符：

``` qml
 transitions:
     Transition {
         to: "*"
         ColorAnimation { target: button; duration: 100}
     }
```

### 通过 Behavior 为属性指定默认动画

``` qml
 Rectangle {
     width: 75; height: 75; radius: width
     id: ball
     color: "salmon"

     Behavior on x {
         NumberAnimation {
             id: bouncebehavior
             easing {
                 type: Easing.OutElastic
                 amplitude: 1.0
                 period: 0.5
             }
         }
     }
     Behavior on y {
         animation: bouncebehavior
     }
     Behavior {
         ColorAnimation { target: ball; duration: 100 }
     }
 }
```

### SequentialAnimation 和 ParallelAnimation

``` qml
 Rectangle {
     id: banner
     width: 150; height: 100; border.color: "black"

     Column {
         anchors.centerIn: parent
         Text {
             id: code
             text: "Code less."
             opacity: 0.01
         }
         Text {
             id: create
             text: "Create more."
             opacity: 0.01
         }
         Text {
             id: deploy
             text: "Deploy everywhere."
             opacity: 0.01
         }
     }

     MouseArea {
         anchors.fill: parent
         onPressed: playbanner.start()
     }

     SequentialAnimation {
         id: playbanner
         running: false
         NumberAnimation { target: code; property: "opacity"; to: 1.0; duration: 200}
         NumberAnimation { target: create; property: "opacity"; to: 1.0; duration: 200}
         NumberAnimation { target: deploy; property: "opacity"; to: 1.0; duration: 200}
     }
 }
```

### 其他动画相关类型

- PauseAnimation
- ScriptAction
- PropertyAction
- SmoothedAnimation
- SpringAnimation
- ParentAnimation
- AnchorAnimation