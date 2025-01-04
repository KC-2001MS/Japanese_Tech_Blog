---
title: "Embedded SwiftでM5Stack NanoC6を動かす"
emoji: "💡"
type: "tech"
topics: ["Swift", "ESP32-C6", "Embedded Swift"]
published: false
---

# はじめに
WWDC2024では、Embedded Swiftが発表され、SwiftでAppleデバイスではなく、組み込み機器の開発ができるようになる予定です。（現在は、正式なリリースはされていないようで、SwiftのSnapshotをインストールして使用する必要があるみたいです。）
そこで、Appleデバイスの開発だけでなく組み込み機器の開発に挑戦してみることにしました。今回は、M5Stack NanoC6を使用してLEDを点滅させることを目指します。組み込み機器については初めてなので間違っていることがあるかもしれませんが、暖かく見守っていただけると幸いです。

# Embedded Swiftとは
Embedded Swiftは、Swiftを組み込み機器の開発で扱うことができるものです。Embedded Swiftは、今までのSwiftとは異なり、Embedded Swift用のコンパイラを使用します。このコンパイラでは、依存関係が大きく異なっていますが、ほとんどのSwiftの構文を利用できます。逆に言うと、利用できない構文があるとも言えます。
大きく注目を集めたのはWWDC 2024のセッション[「Embedded Swiftでサイズを縮小」](https://developer.apple.com/jp/videos/play/wwdc2024/10197/)ではありますが、実は発表されたのはそれ以前のSwift.orgのブログ記事[「Get Started with Embedded Swift on ARM and RISC-V Microcontrollers」](https://www.swift.org/blog/embedded-swift-examples/)です。
このSwift.orgのブログ記事のタイトルからわかる通り、全ての組み込み機器でEmbedded Swiftが利用できるわけではありません。ARMアーキテクチャとRISC-Vアーキテクチャのものに限られています。もし、みなさんがEmbedded Swiftの開発をしたい場合、対応した組み込み機器を入手する必要があるため注意が必要です。

# 組むこみ機器を選定
私は、M5Stack NanoC6を使用してみることにしました。みなさんは異なる機器を使用するかもしれませんが、Embedded Swiftは以下のものが対応しているのでそれを元に選定しましょう。
## CPUアーキテクチャ
以下のCPUアーキテクチャに対応します。ただ、これらのアーキテクチャが対応するからと言ってマイコンを購入するのはおすすめはしません。
- ARM
- RISC-V
## マイクロコントローラ
マイクロコントローラはCPUアーキテクチャに依存して決まります。おそらく他にもEmbedded Swiftで利用できるマイクロコントローラはあるとは思いますが、環境構築やC/C++のフレームワークに深い理解が必要になると思います。そのため、サンプルプロジェクトにあるマイクロコントローラだけをあげてみました。
- ARM
  - STM32F746
  - RP2040(製品名Raspberry Pi Picoのほうが有名)
  - nRF52840
- RISC-V
  - ESP32C6

# 環境構築

# サンプルプロジェクトを修正
サンプルプロジェクトでは、