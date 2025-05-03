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

## 調査結果

詳細な調査の結果、以下の問題点が特定されました：

1. **特定のプラグインの読み込みに時間がかかっている**
   - nvim-treesitterの読み込みに約25-28秒
   - nvim-ts-autotagの読み込みに約28秒
   - これらのプラグインはTreeSitterパーサーを読み込むため、特に初回起動時に時間がかかる

2. **プラグイン管理の問題**
   - 100個以上の多数のプラグインが導入されている
   - 大部分のプラグインが遅延読み込み設定なしで即時ロードされている
   - lazy.nvmのキャッシュは有効だが、多くのプラグインの初期化に時間がかかっている

3. **キャッシュ効果**
   - Neovim終了後すぐに再起動すると速い → システムキャッシュが効いている
   - しばらく経つとキャッシュが破棄され、再度すべてをロードする必要がある

4. **zshとの関連性**
   - zshもNixで管理されており、シェル環境の初期化に時間がかかっている可能性がある
   - システム全体の初期化が影響している可能性も考えられる

## 実行計画

### 1. プラグインの遅延読み込み対策
   - プラグインを必要になったタイミングでロードするように設定を変更する
   - 特に起動時間に大きく影響するプラグインを特定し、遅延読み込みを設定
   ```lua
   -- 例：nvim-treesitterの遅延読み込み設定
   {
     "nvim-treesitter/nvim-treesitter",
     lazy = true,
     event = { "BufReadPost", "BufNewFile" }, -- ファイルを開いたときにロード
     dependencies = {
       "nvim-treesitter/nvim-treesitter-textobjects",
     },
     -- 他の設定
   }
   
   -- 例：nvim-ts-autotagの遅延読み込み設定
   {
     "windwp/nvim-ts-autotag",
     lazy = true,
     event = { "InsertEnter" }, -- 挿入モードに入ったときにロード
     -- 他の設定
   }
   ```

### 2. 起動時に必要なプラグインの整理
   - 起動時に絶対に必要なプラグインのみを即時ロード、他は遅延読み込みに変更
   - プラグインをイベント、キーマップ、コマンドに基づいて遅延読み込み
   - 設定例:
   ```lua
   -- コマンド実行時にロード
   { 
     "plugin/name", 
     lazy = true, 
     cmd = { "PluginCommand" } 
   }
   
   -- 特定のキーマップを使用時にロード
   { 
     "plugin/name", 
     lazy = true, 
     keys = { "<leader>p" } 
   }
   
   -- 特定のファイルタイプでロード
   { 
     "plugin/name", 
     lazy = true, 
     ft = { "markdown", "lua" } 
   }
   ```

### 3. lazy.nvmの最適化
   - lazy.nvmのパフォーマンス関連設定を最適化
   ```lua
   require("lazy").setup({
     performance = {
       cache = {
         enabled = true,
         path = vim.fn.stdpath("cache") .. "/lazy",
         ttl = 86400, -- 24時間→そのまま
       },
       reset_packpath = true, -- packpathをリセットし、プラグインの競合を防ぐ
       rtp = {
         reset = true, -- rtpをリセット
         disabled_plugins = {
           "netrw", "netrwPlugin", "netrwSettings", "netrwFileHandlers",
           "gzip", "zip", "zipPlugin", "tar", "tarPlugin", 
           "getscript", "getscriptPlugin", "vimball", "vimballPlugin",
           "2html_plugin", "logipat", "rrhelper", "spellfile_plugin", "matchit"
         },
       },
     },
   })
   ```

### 4. プラグインの見直しと整理
   - 使用頻度の低いプラグインを特定し、本当に必要か検討
   - 類似機能を持つプラグインを整理・統合
   - 特に起動時間に影響の大きいプラグインを選別

### 5. TreeSitterのパーサー最適化
   - 必要なパーサーのみをインストール
   - パーサーのコンパイルをバックグラウンドで行うように設定
   ```lua
   require("nvim-treesitter.configs").setup({
     ensure_installed = { "lua", "vim", "markdown" }, -- 本当に必要な言語のみに制限
     sync_install = false, -- 非同期インストール
     auto_install = false, -- 自動インストールをオフに
   })
   ```

### 6. システムキャッシュ関連の最適化
   - Nixの設定を見直し、zshの起動も高速化
   - システムキャッシュが効果的に利用されるよう設定を調整

### 7. 起動時間計測とフィードバック
   - 各改善策適用後に起動時間を計測
   - 最も効果のあった対策を特定し、さらに最適化

この改善策を順番に適用し、その都度効果を検証することで、MacBook ProでもMacBook Airのように高速な起動時間を実現することが期待できます。
