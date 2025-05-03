## 問題

特定マシーンにおいてneovimの起動が遅いためその問題の特定、起動の高速化

## 背景

問題で遅いという表現をしているのは他のPCと比較して遅いという話をしています。

比較した対象PCの情報

- Neovimの起動が遅い方

```

❯ fastfetch                                                                                                                                                            ─╯
                     ..'          happy@toyamanoMacBook-Pro
                 ,xNMM.           -------------------------
               .OMMMMo            OS: macOS Sequoia 15.2 arm64
               lMM"               Host: MacBook Pro (16-inch, 2024, Three Thunderbolt 5 ports)
     .;loddo:.  .olloddol;.       Kernel: Darwin 24.2.0
   cKMMMMMMMMMMNWMMMMMMMMMM0:     Uptime: 21 days, 8 mins
 .KMMMMMMMMMMMMMMMMMMMMMMMWd.     Packages: 132 (nix-system), 259 (nix-user), 52 (nix-default), 8 (brew), 25 (brew-cask)
 XMMMMMMMMMMMMMMMMMMMMMMMX.       Shell: zsh 5.9
;MMMMMMMMMMMMMMMMMMMMMMMM:        Display (Color LCD): 3456x2234 @ 120 Hz (as 1728x1117) in 16" [Built-in]
:MMMMMMMMMMMMMMMMMMMMMMMM:        DE: Aqua
.MMMMMMMMMMMMMMMMMMMMMMMMX.       WM: Quartz Compositor 278.2.7
 kMMMMMMMMMMMMMMMMMMMMMMMMWd.     WM Theme: Multicolor (Dark)
 'XMMMMMMMMMMMMMMMMMMMMMMMMMMk    Font: .AppleSystemUIFont [System], Helvetica [User]
  'XMMMMMMMMMMMMMMMMMMMMMMMMK.    Cursor: Fill - Black, Outline - White (32px)
    kMMMMMMMMMMMMMMMMMMMMMMd      Terminal: nvim
     ;KMMMMMMMWXXWMMMMMMMk.       CPU: Apple M4 Pro (14) @ 4.51 GHz
       "cooc*"    "*coo'"         GPU: Apple M4 Pro (20) @ 1.58 GHz [Integrated]
                                  Memory: 33.68 GiB / 48.00 GiB (70%)
                                  Swap: Disabled
                                  Disk (/): 145.30 GiB / 460.43 GiB (32%) - apfs [Read-only]
                                  Disk (/Volumes/Microsoft Edge): 859.14 MiB / 859.14 MiB (100%) - hfs [External, Read-only]
                                  Disk (/Volumes/Slack): 494.25 MiB / 745.96 MiB (66%) - hfs [External, Read-only]
                                  Local IP (utun4): 10.226.42.137/32
                                  Battery (bq40z651): 52% (6 hours, 51 mins remaining) [Discharging]
                                  Locale: ja_JP.UTF-8




```

- Neovimの起動が早い方

```

MacBook Air M4
メモリ:32GB
```

MacBook ProのNeovim起動が4秒
MacBook AirのNeovim起動が0.1秒

性能的には明らかによいPCの方がNeovimの起動が遅いです。
Neovimの設定ファイル等は/Users/happy/dotfiles/conf/.config/nvimに存在します。
環境はdotfileとNixで管理しているためプラグインの数やshellの環境自体はPC間で同じだと考えられます。

MacBook ProのNeovimの起動は4秒かかりますが、Neovimを終了後すぐに開いた際は0.1秒ほどで起動します。
しかし、しばらくNeovimを起動しない状態で再びNeovimを起動すると初期と同じように4秒ほどの時間がかかります。

この現象と関係しているのはかわかりませんが、zshの起動も遅いです

## 実行計画
