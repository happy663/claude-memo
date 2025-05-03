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

1. **ネットワーク接続の遅延問題**

   - nvim起動時にDenopsプラグインが自動的にDenoサーバーを起動している
   - ネットワーク接続処理（ソケット通信）が行われ、これが遅延の主な原因
   - `lsof -i -n` の結果から、nvimプロセスがDeno（localhost:ポート番号）に接続していることを確認
   - すぐに再起動した場合は、既にDenoサーバーが起動しているため速い

2. **日本語入力システム（skkeleton）の影響**

   - skkeleton（日本語入力プラグイン）がDenopsプラグインに依存している
   - skkeleton初期化時にネットワーク接続が発生し、DNS参照や辞書アクセスが行われる
   - 大量の日本語辞書ファイル（SKK-JISYO）の読み込みが含まれる

3. **環境変数とシステム設定の違い**

   - MacBook ProとMacBook Airで環境変数やシステム設定が微妙に異なる可能性がある
   - Nixの設定によって、ネットワーク関連の挙動が影響を受けている

4. **DNS解決の遅延**
   - DNSの設定（`10.226.15.254`）に対する名前解決に時間がかかっている可能性
   - MacBook Airよりも、MacBook ProのDNS解決が遅い可能性がある

## 解決策

1. **Denopsの設定調整**

   - 起動時のネットワーク接続タイムアウトを短くする

   ```vim
   " ~/.config/nvim/init.luaに追加
   vim.g.denops_server_wait_timeout = 1000  -- 1秒のタイムアウト（デフォルトは30秒）
   ```

2. **ホスト名解決の最適化**

   - `/etc/hosts` にローカルホスト定義を追加し、DNS参照を回避

   ```
   # /etc/hostsに追加
   127.0.0.1 localhost.localdomain
   ::1       localhost.localdomain
   ```

3. **skkeleton設定の遅延読み込み化**

   - skkeleton（日本語入力）を必要になるまで遅延させる

   ```lua
   -- /Users/happy/dotfiles/conf/.config/nvim/lua/plugins/japanese/skkeleton.luaを編集
   return {
     {
       "vim-skk/skkeleton",
       cond = vim.g.not_in_vscode,
       lazy = true,          -- 追加：遅延読み込み設定
       event = "InsertEnter", -- 追加：挿入モード時に読み込み
       dependencies = {
         { "vim-denops/denops.vim" },
       },
       -- 以下は同じ
     }
   }
   ```

4. **Denops自体の遅延読み込み化**

   - Denopsプラグインを直接遅延読み込みにする

   ```lua
   -- /Users/happy/dotfiles/conf/.config/nvim/lua/plugins/japanese/denops.luaを新規作成（またはプラグイン定義を適切な場所に移動）
   return {
     {
       "vim-denops/denops.vim",
       lazy = true,          -- 追加：遅延読み込み設定
       event = "InsertEnter", -- 追加：挿入モード時に読み込み
     }
   }
   ```

5. **起動時のネットワーク接続制限**

   - 起動時のネットワーク接続をブロックする設定を追加

   ```lua
   -- ~/.config/nvim/init.luaに追加
   vim.g.denops_server_addr = ""  -- 共有サーバーアドレスを空にしてネットワーク接続を回避
   ```

6. **システムキャッシュの活用**

   - Nixの設定を見直し、キャッシュ戦略を最適化
   - システム起動時にDenoサーバーを事前に起動することも検討

7. **問題の根本解決：代替プラグインの検討**
   - 日本語入力に別のプラグインを使用することを検討
   - ネットワーク接続に依存しない方式のプラグインを探す

上記の解決策を順番に試し、最も効果的な方法を特定することをお勧めします。特に1, 3, 4の対策は比較的簡単に実施でき、効果が期待できます。
