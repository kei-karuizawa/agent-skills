---
name: create-xcode-project
description: 创建/重构/修改 xcode 项目时，项目的组织方式指导，主要是为了确保项目是使用 Swift Package Manager 来管理依赖以及组件化。
---

# 创建/重构/修改 xcode 项目时的组织方式指导

## 前提条件

应用需要满足以下任一条件才可使用此文档：

1. 新建一个 Xcode 项目时。
2. 用户明确说明重构一个 Xcode 项目时，若无明确说重构，那么就按照项目现有的组织方式运行，不必强制使用 Swift Package Manager。
3. 若发现当前 Xcode 项目既没有采用 CocoaPods 组件化代码，也没有其他特殊的组织方式，那么在用户允许的情况下，帮用户重构项目，使用 Swift Package Manager 来管理依赖以及组件化。注意，需要询问用户，且最好只问一次，在用户拒绝后就不再询问了。
4. 对于 3 的补充，若用户的 CocoaPods 只是管理第三方框架，并没有用来组件化用户自己的代码，则这个项目不代表使用了 CocoaPods 组件化，则需要重构为 Swift Package Manager 组件化（用户允许前提下）。

## 如何使用此技能

满足任一前提条件时，那么需要用 Swift Package Manager 来管理依赖以及组件化。代码结构应如下：

MyProduct/
├─ AppShell/
│   ├─ Podfile
│   ├─ Pods/
│   └─ MyProduct.xcodeproj
│
└─ Packages/
    ├─ Core/
    ├─ Domain/
    ├─ Network/

注意，上面给你的只是结构，并不意味着命名要按照上面的命名，也并不意味着一定要有网络模块、域名模块等不需要的东西。我需要你分析项目的实际需求按照上面的结构重构。

调整完项目后，必须确保所有 Packages 都可以正常编译，且整个项目可以完整编译。