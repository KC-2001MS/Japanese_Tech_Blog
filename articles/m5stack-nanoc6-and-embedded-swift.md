---
title: "Embedded SwiftでM5Stack NanoC6を動かす"
emoji: "💡"
type: "tech"
topics: ["Swift", "ESP32-C6", "Embedded Swift"]
published: false
---

これから、Embedded Swiftを使用して、M5Stack NanoC6でLEDを点滅させる。

# はじめに
WWDC2024では、Embedded Swiftが発表され、SwiftでAppleデバイスではなく、組み込み機器の開発ができるようになる予定だ。（現在は、正式なリリースはされていないようで、SwiftのSnapshotをインストールして使用する必要があるようだ。）
そこで、Appleデバイスの開発だけでなく組み込み機器の開発に挑戦してみることにした。**今回は、M5Stack NanoC6を使用してLEDを点滅させることを目指す。**組み込み機器については初めてなので間違っていることがあるかもしれないが、暖かく見守っていただけると幸いだ。

# Embedded Swiftとは
Embedded Swiftは、Swiftを組み込み機器の開発で扱うことができるものだ。Embedded Swiftは、今までのSwiftとは異なり、Embedded Swift用のコンパイルモードを使用する。そのため、依存関係が大きく異なっているが、ほとんどのSwiftの構文を利用できる。逆に言うと、利用できない構文があるとも言える。
大きく注目を集めたのはWWDC 2024のセッション[「Embedded Swiftでサイズを縮小」](https://developer.apple.com/jp/videos/play/wwdc2024/10197/)ではあるが、実は発表されたのはそれ以前のSwift.orgのブログ記事の[「Byte-sized Swift: Building Tiny Games for the Playdate」](https://www.swift.org/blog/byte-sized-swift-tiny-games-playdate/)や[「Get Started with Embedded Swift on ARM and RISC-V Microcontrollers」](https://www.swift.org/blog/embedded-swift-examples/)だ。
このSwift.orgのブログ記事のタイトルからわかる通り、全ての組み込み機器でEmbedded Swiftが利用できるわけではない。ARMアーキテクチャとRISC-Vアーキテクチャのものに限られている。もし、みなさんがEmbedded Swiftの開発をしたい場合、対応した組み込み機器を入手する必要があるため注意が必要だ。

# 組むこみ機器の選定方法について
私は、M5Stack NanoC6を使用してみることにしたため、これ以降はM5Stack NanoC6での話となる。みなさんは異なる機器を使用するかもしれないが、Embedded Swiftは以下のものが対応しているのでそれを元に選定する。
## マイクロコントローラ
マイクロコントローラにはCPUがあるため、こちらを見れば良い。おそらく他にもEmbedded Swiftで利用できるマイクロコントローラはあるとは思うが、ここではサンプルプロジェクトで確実に動作することが確認されているマイクロコントローラだけをあげてみた。
- ARM
  - STM32F746
  - RP2040(製品名Raspberry Pi Picoのほうが有名)
  - nRF52840
- RISC-V
  - ESP32C6

この中から選べば安全だと思われる。組み込みの知識がある人であればこれ以外のマイクロコントローラでも動作させられるのかもしれないが、現在の私にはわからない。
# 環境構築
では環境構築に進む。基本的には、[Swift Matter Examples TutorialsのSwift Matter Examples Tutorials](https://apple.github.io/swift-matter-examples/tutorials/swiftmatterexamples/setup-macos/)に記載されている通りに進める。
```zsh
$ ls ~/Library/Developer/Toolchains/
swift-DEVELOPMENT-SNAPSHOT-2024-06-03-a.xctoolchain
swift-latest.xctoolchain

$ plutil -extract CFBundleIdentifier raw \
  -o - \
  ~/Library/Developer/Toolchains/swift-DEVELOPMENT-SNAPSHOT-2024-06-03-a.xctoolchain/Info.plist
org.swift.59202406031a

$ TOOLCHAINS=org.swift.59202406031a swift --version
Apple Swift version 6.0-dev (LLVM c7c87ee42989d4b, Swift 0aa0687fe0f4047)
Target: arm64-apple-macosx14.0
```
homebrewのインストールはわかると思うので省略する。
```zsh
$ brew install cmake ninja dfu-util
```

```zsh
$ mkdir -p ~/esp

$ cd ~/esp
$ git clone \
  --branch v5.2.1 \
  --depth 1 \
  --shallow-submodules \
  --recursive https://github.com/espressif/esp-idf.git \
  --jobs 24

$ cd ~/esp/esp-idf
$ ./install.sh

$ cd ~/esp
$ git clone \
    --branch release/v1.2 \
    --depth 1 \
    --shallow-submodules \
    --recursive https://github.com/espressif/esp-matter.git \
    --jobs 24

$ cd ~/esp/esp-matter
$ ./install.sh

$ export TOOLCHAINS=org.swift.59202406031a

$ . ~/esp/esp-idf/export.sh

$ . ~/esp/esp-matter/export.sh
```

その後、以下のコマンドを順に実行する。
```zsh
$ cd swift-embedded-examples
$ python3 -m venv .venv
$ source .venv/bin/activate
$ python3 -m pip install -r Tools/requirements.txt
```
これで、環境の構築は完了だ。
# サンプルプロジェクトのファイル構成を確認
では、Swiftのプロジェクトがどのように構成されているのかを確かめてみる。esp32-led-blink-sdkのファイル構成を一つ一つ説明する。
- main
  - BridgingHeader.h
  C言語のヘッターファイルであることは明らかであるのだが、不思議なことに
  - CMakeLists.txt
  - idf_component.yml
  - Led.swift
  LED構造体を定義している。
  - Main.swift
  エントリーポイントとなるファイルだ。
- CMakeLists.txt


## わかること
このプロジェクトの特徴的な点としてSwift Package Managerを使用していない点が挙げれられる。ただ、同じくEmbedded Swiftを利用している[swift-playdate-examples](https://github.com/apple/swift-playdate-examples)においてはSwift Package Managerを使用していることから、今後Swift Package Managerを使用したプロジェクト構成が可能かどうかは検証する余地がある。

# サンプルプロジェクトを動作させてみる
では、サンプルプロジェクトを動作させてLEDを点滅させてみる。M5Stack NanoC6はESP32C6であることは確認しているので、今回はesp32-led-blink-sdkのプロジェクトを使用してみることにする。
では、基盤を接続したら以下のコマンドを叩いて動作させよう。
## そのままだとビルドはできても動かない
ビルドできているはずなのに動かないはずだ。これは、サンプルプロジェクトでは、異なる基盤[Esp32-C6-Bug]()を使用しており、このボードが動くようにプログラミングされているためだ。
[M5Stack NanoC6のドキュメント](https://docs.m5stack.com/ja/core/M5NanoC6)を確認してみよう。注目して欲しいのは、ピンマップに関する記載だ。LED(Blue)はGPI07に割り当てられている。対して、サンプルプロジェクトでの図によるとGPI08に割り当てられていることに気づくはずだ。
つまり、数字を変更するだけで動作させることができる。
```swift

```
を
```swift

```
に変えれば良い。

# 最後に
どうだっただろうか。ESP32-C6-BUGは簡単には手に入らず、値段も高めの4785円のため、1276円のM5Stack NanoC6でEmbedded Swiftを体験できることを伝えられるのはとても有意義なことではないだろうか。特に、Embedded Swiftに関して実際に動かしてみた記事は日本語圏では多くないため、皆の役に立てば幸いだ。