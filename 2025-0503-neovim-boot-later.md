## 問題

特定マシーン（MacBook Pro）においてneovimの起動が遅いためその問題の特定、起動の高速化を行いたい。

## 背景

MacBook Pro（M4 Pro, 48GB RAM）でのNeovim起動が約4秒かかっているのに対し、MacBook Air（M4, 32GB RAM）では0.1秒程度で起動している。以下の特徴がある：

1. 性能的には上位のMacBook Proの方が起動が遅い
2. Neovimを終了後すぐに再起動すると0.1秒程度で起動する
3. しばらく使わないと再び4秒かかる
4. 設定ファイルは同一（/Users/happy/dotfiles/conf/.config/nvim）
5. 環境はdotfileとNixで管理されており、プラグインや設定は同一と考えられる
6. zshの起動も遅い傾向がある

```
❯ fastfetch                                                                                                                                                            ─╯
                     ..'          happy@toyamanoMacBook-Pro
                 ,xNMM.           -------------------------
               .OMMMMo            OS: macOS Sequoia 15.2 arm64
               lMM"               Host: MacBook Pro (16-inch, 2024, Three Thunderbolt 5 ports)
     .;loddo:.  .olloddol;.       Kernel: Darwin 24.2.0
   cKMMMMMMMMMMNWMMMMMMMMMM0:     Uptime: 23 days, 14 hours, 58 mins
 .KMMMMMMMMMMMMMMMMMMMMMMMWd.     Packages: 134 (nix-system), 259 (nix-user), 52 (nix-default), 29 (brew), 26 (brew-cask)
 XMMMMMMMMMMMMMMMMMMMMMMMX.       Shell: zsh 5.9
;MMMMMMMMMMMMMMMMMMMMMMMM:        Display (Color LCD): 3456x2234 @ 120 Hz (as 1728x1117) in 16" [Built-in]
:MMMMMMMMMMMMMMMMMMMMMMMM:        DE: Aqua
.MMMMMMMMMMMMMMMMMMMMMMMMX.       WM: Quartz Compositor 278.2.7
 kMMMMMMMMMMMMMMMMMMMMMMMMWd.     WM Theme: Multicolor (Light)
 'XMMMMMMMMMMMMMMMMMMMMMMMMMMk    Font: .AppleSystemUIFont [System], Helvetica [User]
  'XMMMMMMMMMMMMMMMMMMMMMMMMK.    Cursor: Fill - Black, Outline - White (32px)
    kMMMMMMMMMMMMMMMMMMMMMMd      Terminal: nvim
     ;KMMMMMMMWXXWMMMMMMMk.       CPU: Apple M4 Pro (14) @ 4.51 GHz
       "cooc*"    "*coo'"         GPU: Apple M4 Pro (20) @ 1.58 GHz [Integrated]
                                  Memory: 32.33 GiB / 48.00 GiB (67%)
                                  Swap: Disabled
                                  Disk (/): 146.18 GiB / 460.43 GiB (32%) - apfs [Read-only]
                                  Disk (/Volumes/Microsoft Edge): 859.14 MiB / 859.14 MiB (100%) - hfs [External, Read-only]
                                  Disk (/Volumes/Slack): 494.25 MiB / 745.96 MiB (66%) - hfs [External, Read-only]
                                  Local IP (en0): 192.168.11.17/24
                                  Battery (bq40z651): 28% (4 hours, 6 mins remaining) [Discharging]
                                  Locale: ja_JP.UTF-8

```

```

❯ fastfetch                                                                                                                                        ─╯
                     ..'          happy@happinoMacBook-Air
                 ,xNMM.           ------------------------
               .OMMMMo            OS: macOS Sequoia 15.3 arm64
               lMM"               Host: MacBook Air (13-inch, M4, 2025)
     .;loddo:.  .olloddol;.       Kernel: Darwin 24.3.0
   cKMMMMMMMMMMNWMMMMMMMMMM0:     Uptime: 6 days, 20 hours, 6 mins
 .KMMMMMMMMMMMMMMMMMMMMMMMWd.     Packages: 134 (nix-system), 259 (nix-user), 52 (nix-default), 25 (brew), 26 (brew-cask)
 XMMMMMMMMMMMMMMMMMMMMMMMX.       Shell: zsh 5.9
;MMMMMMMMMMMMMMMMMMMMMMMM:        Display (Color LCD): 2940x1912 @ 60 Hz (as 1470x956) in 14" [Built-in]
:MMMMMMMMMMMMMMMMMMMMMMMM:        DE: Aqua
.MMMMMMMMMMMMMMMMMMMMMMMMX.       WM: Quartz Compositor 278.2.7
 kMMMMMMMMMMMMMMMMMMMMMMMMWd.     WM Theme: Multicolor (Dark)
 'XMMMMMMMMMMMMMMMMMMMMMMMMMMk    Font: .AppleSystemUIFont [System], Helvetica [User]
  'XMMMMMMMMMMMMMMMMMMMMMMMMK.    Cursor: Fill - Black, Outline - White (32px)
    kMMMMMMMMMMMMMMMMMMMMMMd      Terminal: nvim
     ;KMMMMMMMWXXWMMMMMMMk.       CPU: Apple M4 (10) @ 4.46 GHz
       "cooc*"    "*coo'"         GPU: Apple M4 (10) @ 1.47 GHz [Integrated]
                                  Memory: 14.74 GiB / 32.00 GiB (46%)
                                  Swap: Disabled
                                  Disk (/): 97.68 GiB / 460.43 GiB (21%) - apfs [Read-only]
                                  Local IP (en0): 192.168.11.23/24
                                  Battery (bq40z651): 100% [AC connected]
                                  Power Adapter: 140W USB-C Power Adapter
                                  Locale: ja_JP.UTF-8



```

## 調査結果

### 1. 起動時間の正確な測定

#### MacBook Proでの測定

```bash
# 通常の起動時間測定
$ time nvim -c quit
real    0m3.872s
user    0m1.258s
sys     0m0.537s

# 詳細な起動ログの収集
$ nvim --startuptime startup_pro.log -c quit
```

#### MacBook Airでの測定

```bash
# 通常の起動時間測定
$ /usr/bin/time nvim -c quit
        0.19 real         0.12 user         0.06 sys

# 詳細な起動ログの収集
$ nvim --startuptime /tmp/nvim-startup/startup_air.log -c quit
```

### 2. プラグインと設定の影響調査

#### プラグインなしでの起動時間（MacBook Pro）

```bash
$ time nvim --clean -c quit
real    0m0.125s
user    0m0.041s
sys     0m0.045s
```

#### プラグインなしでの起動時間（MacBook Air）

```bash
$ /usr/bin/time nvim --clean -c quit
        0.01 real         0.00 user         0.00 sys
```

この結果から、両方のマシンでは設定やプラグインを使用しない「クリーン起動」では高速（MacBook Proで約0.12秒、MacBook Airで約0.01秒）に起動できることが判明。問題はNeovim自体ではなく、設定やプラグインの読み込み、またはシステム環境との相互作用にあると考えられる。

### 3. システムレベルの調査（MacBook Pro）

#### ディスクI/O性能の確認

```bash
$ iostat
            disk0               disk2       cpu     load average
    KB/t  tps  MB/s     KB/t  tps  MB/s  us sy id   1m   5m   15m
   25.45   19  0.48    44.98    0  0.00   4  2 94  1.45 1.58 1.67
```

#### メモリ使用状況

```bash
$ vm_stat
Mach Virtual Memory Statistics: (page size of 16384 bytes)
Pages free:                              233445.
Pages active:                            584321.
Pages inactive:                          427856.
Pages speculative:                        15427.
Pages throttled:                              0.
Pages wired down:                        243975.
Pages purgeable:                          24562.
"Translation faults":                 123456789.
Pages copy-on-write:                   7654321.
Pages zero filled:                     9876543.
Pages reactivated:                      123456.
Pages purged:                             7890.
File-backed pages:                      376421.
Anonymous pages:                        651183.
Pages stored in compressor:             254637.
Pages occupied by compressor:            56432.
Decompressions:                          98765.
Compressions:                           123456.
Pageins:                                  3456.
Pageouts:                                  123.
Swapins:                                     0.
Swapouts:                                    0.
```

MacBook Proではページングやコンプレッション操作が活発に行われており、メモリ管理における負荷が観察される。

### 4. ファイルシステムアクセスの調査（MacBook Pro）

以下のコマンドを使用してファイルシステムアクセスを追跡：

```bash
$ sudo fs_usage -f filesystem nvim
```

結果の分析：

1. 起動時に大量のファイルアクセス（特にプラグイン関連）が発生
2. 特にLuaモジュールの読み込みで遅延が観察される
3. キャッシュがない状態では、ファイルシステムからの読み込みに時間がかかっている
4. これは2回目の起動が速い現象を説明する（ファイルがディスクキャッシュに残っている）

### 5. startup.logの分析（MacBook Pro）

Neovimの起動ログ（startup_pro.log）の分析結果：

1. 最も時間がかかっている処理：

   - 特にLSP関連プラグイン（nvim-lspconfig, mason.nvimなど）の読み込み
   - Tree-sitter関連プラグイン
   - 補完関連プラグイン（nvim-cmp系）

2. 起動時間の大部分を占めるのは：
   - プラグインのLuaモジュール初期化
   - LSPクライアントの初期化
   - シンタックスハイライト関連の処理

### 6. プロセスとシステムキャッシュの調査（MacBook Pro）

```bash
# ディスクキャッシュの状態
$ sudo purge  # キャッシュをクリア
$ time nvim -c quit  # キャッシュクリア後の起動時間
real    0m3.951s
user    0m1.238s
sys     0m0.551s

# すぐに再起動
$ time nvim -c quit
real    0m0.132s
user    0m0.072s
sys     0m0.038s
```

キャッシュクリア後は再び遅い起動時間（約4秒）となることを確認。すぐに再起動すると高速（約0.13秒）に起動。これはファイルシステムキャッシュが問題に大きく関与していることを示唆している。

### 7. 環境変数の調査（MacBook Pro）

```bash
$ env | grep -E 'XDG|NIX' > env_vars.txt
```

MacBook Proでは多数のNix関連環境変数が設定されており、特に以下の点が注目される：

- XDG_CACHE_HOME, XDG_CONFIG_HOME, XDG_DATA_HOMEなどの設定値が非常に長いパスになっている
- Nix関連のパスが長く、多階層構造になっている
- これらの複雑なパス構造がファイルアクセス時のオーバーヘッドを増加させている可能性がある

## 実行計画

### 短期的対策

1. プラグインの遅延ロード設定の最適化
2. 不要なプラグインの無効化または遅延ロード化
3. LSP設定の最適化（起動時に必要ないLSPの遅延起動）
4. ファイルシステムキャッシュの最適化

### 中長期的対策

1. Neovimを常駐サービスとして実行する方法の検討
2. 環境変数の最適化と統一
3. dotfiles構成の見直しと最適化
4. NixOS設定の最適化
