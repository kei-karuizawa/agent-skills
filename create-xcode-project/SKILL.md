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

当项目新建/重构后，必须创建一个 Makefile，用于编译项目，以下是一个完整的 Makefile，包含了多个 Target 项目的配置：

```makefile
# ┌──────────────────────────────────────────────────────────────────────────
# │ Makefile for project-hub
# │ 
# │ 快速开始:
# │   make                - 默认执行 make build（编译开发版本）
# │   make all            - 同 make，编译开发版本
# │   make help           - 显示所有可用命令
# │   make spm-build      - 编译所有 Swift Package 模块
# │   make build          - 编译开发版本 (ManageX Dev.app)
# │   make run            - 编译并运行开发版本
# │
# │ Swift Package 管理:
# │   make spm-build      - 编译 ProjectHubCore, ProjectHubServices, ProjectHubUI
# │   make spm-test       - 运行所有 Package 的单元测试
# │   make spm-update     - 更新 Package 依赖（解决依赖冲突时使用）
# │   make spm-clean      - 清理 Package 构建缓存
# │
# │ Xcode 构建:
# │   make build          - 编译开发版本 (ManageX Dev.app) - 默认
# │   make build-dev      - 编译开发版本 (ManageX Dev.app) - 同 make build
# │   make build-qfdev    - 编译 QF 开发版本 (ManageX QFDev.app)
# │   make run            - 编译并启动开发版本 (ManageX Dev.app) - 默认
# │   make run-dev        - 编译并启动开发版本 (ManageX Dev.app) - 同 make run
# │   make run-qfdev      - 编译并启动 QF 开发版本 (ManageX QFDev.app)
# │   make archive        - 创建 Release 归档（用于分发，使用主版本 scheme）
# │
# │ 清理:
# │   make clean          - 清理 Xcode 构建产物和 Package 缓存
# │   make clean-all      - 深度清理（包括 .build 目录和 DerivedData）
# │   make reset-packages - 重置 Xcode Package 缓存（解决 Package 索引问题）
# │
# │ 开发辅助:
# │   make open-xcode     - 在 Xcode 中打开项目
# │
# │ 使用场景示例:
# │   1. 首次克隆项目后: make spm-build && make build
# │   2. 修改 Package 代码后: make spm-build && make build
# │   3. Package 依赖错误: make reset-packages && make clean && make build
# │   4. 完全重新构建: make clean-all && make spm-build && make build
# └──────────────────────────────────────────────────────────────────────────

# 工程路径
APP_PATH := project-hub-macOS
PACKAGE_PATHS := Packages/ProjectHubCore Packages/ProjectHubServices Packages/ProjectHubUI

# 构建配置
SCHEME := project-hub-macOS
DEV_SCHEME := project-hub-macOS dev
QFDEV_SCHEME := project-hub-macOS qfdev
CONFIG := Debug
DESTINATION := "platform=macOS"

# 默认目标
.PHONY: all
all: build-dev

# =============================
# Swift Package 管理
# =============================
.PHONY: spm-update
spm-update:
	@echo "==> Updating Swift Packages..."
	@for pkg in $(PACKAGE_PATHS); do \
		echo "Updating $$pkg..."; \
		cd $$pkg && swift package update && cd - > /dev/null; \
	done

.PHONY: spm-build
spm-build:
	@echo "==> Building Swift Packages..."
	@for pkg in $(PACKAGE_PATHS); do \
		echo "Building $$pkg..."; \
		cd $$pkg && swift build && cd - > /dev/null; \
	done

.PHONY: spm-test
spm-test:
	@echo "==> Running Swift Package Tests..."
	@for pkg in $(PACKAGE_PATHS); do \
		echo "Testing $$pkg..."; \
		cd $$pkg && swift test && cd - > /dev/null; \
	done

.PHONY: spm-clean
spm-clean:
	@echo "==> Cleaning Swift Packages..."
	@for pkg in $(PACKAGE_PATHS); do \
		echo "Cleaning $$pkg..."; \
		cd $$pkg && swift package clean && cd - > /dev/null; \
	done

# =============================
# Xcode 构建
# =============================
.PHONY: build
build:
	@echo "==> Building Xcode Project ($(DEV_SCHEME))..."
	xcodebuild -project $(APP_PATH)/$(APP_PATH).xcodeproj \
	-scheme "$(DEV_SCHEME)" \
	-configuration $(CONFIG) \
	build

.PHONY: build-dev
build-dev:
	@echo "==> Building Xcode Project ($(DEV_SCHEME))..."
	xcodebuild -project $(APP_PATH)/$(APP_PATH).xcodeproj \
	-scheme "$(DEV_SCHEME)" \
	-configuration $(CONFIG) \
	build

.PHONY: build-qfdev
build-qfdev:
	@echo "==> Building Xcode Project ($(QFDEV_SCHEME))..."
	xcodebuild -project $(APP_PATH)/$(APP_PATH).xcodeproj \
	-scheme "$(QFDEV_SCHEME)" \
	-configuration $(CONFIG) \
	build

.PHONY: run
run: build
	@echo "==> Running $(DEV_SCHEME)..."
	open $(APP_PATH)/build/$(CONFIG)/ManageX\ Dev.app || \
	open ~/Library/Developer/Xcode/DerivedData/$(APP_PATH)-*/Build/Products/$(CONFIG)/ManageX\ Dev.app

.PHONY: run-dev
run-dev: build-dev
	@echo "==> Running $(DEV_SCHEME)..."
	open $(APP_PATH)/build/$(CONFIG)/ManageX\ Dev.app || \
	open ~/Library/Developer/Xcode/DerivedData/$(APP_PATH)-*/Build/Products/$(CONFIG)/ManageX\ Dev.app

.PHONY: run-qfdev
run-qfdev: build-qfdev
	@echo "==> Running $(QFDEV_SCHEME)..."
	open $(APP_PATH)/build/$(CONFIG)/ManageX\ QFDev.app || \
	open ~/Library/Developer/Xcode/DerivedData/$(APP_PATH)-*/Build/Products/$(CONFIG)/ManageX\ QFDev.app

.PHONY: archive
archive:
	@echo "==> Archiving for Distribution..."
	xcodebuild -project $(APP_PATH)/$(APP_PATH).xcodeproj \
	-scheme "$(SCHEME)" \
	-configuration Release \
	-archivePath $(APP_PATH)/build/$(SCHEME).xcarchive \
	archive

# =============================
# Clean
# =============================
.PHONY: clean
clean: spm-clean
	@echo "==> Cleaning Xcode Build..."
	xcodebuild -project $(APP_PATH)/$(APP_PATH).xcodeproj \
	-scheme "$(SCHEME)" \
	clean
	rm -rf $(APP_PATH)/build
	rm -rf ~/Library/Developer/Xcode/DerivedData/$(APP_PATH)-*

.PHONY: clean-all
clean-all: clean
	@echo "==> Removing all build artifacts..."
	find . -name ".build" -type d -exec rm -rf {} + 2>/dev/null || true
	find . -name "*.xcworkspace" -type d -exec rm -rf {} + 2>/dev/null || true

# =============================
# 开发辅助
# =============================
.PHONY: reset-packages
reset-packages:
	@echo "==> Resetting Package Caches..."
	rm -rf ~/Library/Caches/org.swift.swiftpm
	rm -rf ~/Library/Developer/Xcode/DerivedData

.PHONY: open-xcode
open-xcode:
	@echo "==> Opening Xcode Project..."
	open $(APP_PATH)/$(APP_PATH).xcodeproj

.PHONY: help
help:
	@echo "Available targets:"
	@echo "  make / make all     - Build dev scheme (default)"
	@echo "  make build          - Build dev scheme (ManageX Dev.app)"
	@echo "  make build-dev      - Build dev scheme (ManageX Dev.app)"
	@echo "  make build-qfdev    - Build qfdev scheme (ManageX QFDev.app)"
	@echo "  make run            - Build and run dev app (default)"
	@echo "  make run-dev        - Build and run dev app (ManageX Dev.app)"
	@echo "  make run-qfdev      - Build and run qfdev app (ManageX QFDev.app)"
	@echo "  make archive        - Create release archive"
	@echo "  make spm-build      - Build all Swift Packages"
	@echo "  make spm-test       - Test all Swift Packages"
	@echo "  make spm-update     - Update Swift Package dependencies"
	@echo "  make clean          - Clean build artifacts"
	@echo "  make clean-all      - Deep clean (including packages)"
	@echo "  make reset-packages - Reset Xcode package caches"
	@echo "  make open-xcode     - Open project in Xcode"

```

你必须按照项目的实际要求创建这个 Makefile，且注释需要像上面这样完整。

若在实际代码过程中，发现新增了 Target 或者 Package，需要更新 Makefile。