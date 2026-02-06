---
title: "環境を入れたくない私 vs 環境を入れさせるNeovim"
emoji: "🦾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
publication_name: "uniformnext"
---

# はじめに
最近、大学生というブランドがもう少しという事実に気付き、悲しんでいるたくみです。
まず皆さんに問い掛けたい。

VSCode重くないですか？？？
あとVim使えるのかっこよくないですか？

本命は下なんですけど、上もちゃんと感じてます。

ということで、VSCodeからNeovimへ移行したい、のですがNeovimさんは環境を入れさせようとしてきます。
私はMacに環境を入れたくないので、思想の違いが出て全面戦争が勃発しています。

一緒に戦っていただける方をこの記事で見つけられればなと思っています。

# 背景
前提として、私はVSCodeのNeovim拡張機能を使用して開発を行なっています。
設定も簡単ですしそこまで不満はなかったのですが、最近この拡張機能がよくバグります。
`hjkl`で移動できなくなったり、`i`でインサートに入れなくなったり、マウスを使わされたりと、結構えぐめなバグによく遭遇します。
また、VSCode自体も重く起動に時間がかかったり重すぎてサーバーが落ちたりと、不満がたくさん出てきました。

以上のことから、Neovimに移行しようと思いました。

# 問題
さて、Neovimへ移行すると決意しましたが、この移行には絶望が待っていました。
るんるんでプラグインを設定していたところ、LSPやCopilotなど、Node.jsやnpmを必要とするプラグインがちらほらありました。
**MacにNode.jsやnpm、bunなどは入れたくない**という意地があり、正直挫折しました。
しかし、LSP、Copilotがなかったら開発なんてできない...
やはりNeovimは私を見捨てるのかと思いました。

# 解決策
geminiやclaudeに話を聞いてもらいながら、「Dockerでneovimを使えば良いのでは？」という発想に至りました。
DockerをかますことでNeovimの良さである速度が落ちるということは重々承知ですが、そこはNeovimへの気持ちがあれば無視できるとしています。
LSPは`lsp-containers`というプラグインがあり、それでも対応できそうだとは思いましたが、対応していないlspサーバーも存在することや、そもそも開発の中でのnpmやpythonも必要だと思ったので、この解決法を採用しています。

## 設定方法
さて、一家に一台は必ずDockerが存在すると思います。
設定方法はとても簡単で、Neovimの環境を持った`Dockerfile`とそれを起動する`docker-compose.yaml`だけ作ります。

私が設定したファイル等は以下です。

```Dockerfile:Dockerfile
# Debianイメージをベースとする。
# alpineでも良かったが、Masonで対応していない部分があった。
FROM debian:bookworm-slim

# 依存パッケージのインストール
RUN apt-get update && apt-get install -y --no-install-recommends \
  curl \
  bash \
  build-essential \
  git \
  openssh-client \
  ripgrep \
  xclip \
  unzip \
  ca-certificates \
  gnupg \
  software-properties-common \
  wget \
  && rm -rf /var/lib/apt/lists/*

# Node.js最新版のインストール
RUN curl -fsSL https://deb.nodesource.com/setup_current.x | bash - \
  && apt-get install -y --no-install-recommends nodejs \
  && rm -rf /var/lib/apt/lists/*

# Neovim最新版のインストール
RUN curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim-linux-arm64.tar.gz \
  && tar xzf nvim-linux-arm64.tar.gz -C /opt \
  && ln -s /opt/nvim-linux-arm64/bin/nvim /usr/local/bin/nvim \
  && rm nvim-linux-arm64.tar.gz

# lazygitのインストール
RUN LAZYGIT_VERSION=$(curl -s https://api.github.com/repos/jesseduffield/lazygit/releases/latest | grep '"tag_name":' | sed 's/.*"v\([^"]*\)".*/\1/') \
  && curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/latest/download/lazygit_${LAZYGIT_VERSION}_Linux_arm64.tar.gz" \
  && tar xzf lazygit.tar.gz lazygit \
  && install lazygit /usr/local/bin \
  && rm lazygit lazygit.tar.gz

# Claude Codeのインストール
RUN npm install -g @anthropic-ai/claude-code

# GitHubのSSHホストキーを事前登録
RUN mkdir -p /root/.ssh \
  && ssh-keyscan github.com >> /root/.ssh/known_hosts 2>/dev/null

ENV PATH="/root/.local/share/nvim/mason/bin:${PATH}"

WORKDIR /workspace
CMD ["nvim", "."]
```

```yaml:docker-compose.yaml
services:
  nvim:
    build: .
    volumes:
      - ${TARGET_PATH}:/workspace
      - ~/Documents/dotfiles/nvim:/root/.config/nvim
      - ~/.local/share/docker-nvim:/root/.local/share/nvim
      - ~/.local/state/docker-nvim:/root/.local/state/nvim
      - ~/.cache/docker-nvim:/root/.cache/nvim
      - ~/.config/github-copilot:/root/.config/github-copilot
      - ~/.gitconfig:/root/.gitconfig:ro
      - /run/host-services/ssh-auth.sock:/run/host-services/ssh-auth.sock
    environment:
      - SSH_AUTH_SOCK=/run/host-services/ssh-auth.sock
    stdin_open: true
    ports:
      - 8000:8000
    tty: true
```

ポイントは、GithubのSSHキーを使用するためのssh-agentの共有と、ボリューム付けするパスを毎回変えることができるところです。
また、毎回`docker-compose run --rm ...`と実行するのは大変なので、`.zshrc`で関数を作成しています。
```bash:.zshrc
# docker-nvimの設定
function dnvim() {
  # 現在のパスを取得
  local target_path
  target_path=$(pwd)

  # Dockerfileなどのパスを取得
  local config_dir="$HOME/Documents/dotfiles/nvim/docker"

  # docker compose runコマンドでコンテナの実行
  TARGET_PATH="$target_path" docker compose \
    -f "$config_dir/docker-compose.yaml" \
    -p "nvim-env" \
    run \
    --service-ports \
    --rm nvim
}
```

ここでのポイントとして、ボリューム付けするパスを`pwd`に設定していること、`--service-ports`で`docker-compose.yaml`に書いた`ports`をフォワーディングすることです。

以上の設定を行うことで、`dnvim .`と入力してDocker内のNeovimを触ることができています。

# この方法の課題
さて、この方法でいけると確信した私にまた絶望が降りかかります。
この方法には以下の課題があります。
1. `dnvim`を2つ以上開けない。
    `docker-compose`でポートフォワーディングしていますが、2つ以上開こうとするとそのポートが競合し、開くことができません。
2. 開発で使用する環境全てを入れる必要がある。
    例えば、`terraform`を使うプロジェクトがあれば、`aws cli`を使うプロジェクト、`python`や`node`など、それぞれのプロジェクトで使用する環境は違うと思います。
    現在の設定だと、この`Dockerfile`に全ての環境のインストールを書く必要があり、とても面倒です。
3. 使用するポートは書かないといけない。
    そのサービスで使用するポートは全て事前に`docker-compose.yaml`に書く必要があります。
    また、`aws cli`でのログインなどは、`callbacl_url`にローカルのランダムポートを指定しているものもあります。設定をいじることで固定することはできますが、それを固定するのは違うと思います。
4. クリップボードの設定が面倒
    操作しているneovimはあくまでDockerコンテナ内なので、クリップボードの共有や画像、ファイルの転送が面倒です。

たくさんの課題が出てきており、やはりNeovimは私を見捨てる気なんじゃないのかと思っています。
`devcontainer-cli`を使用することで解決できる説はあるんですが、あれ`node`入れないと動かないはずなんですよね...

ここら辺の課題は開発しながら気長に解決していきたいと思います。

# まとめ
Macに環境を入れたくない私と、環境を入れざるを得ない状況を作ってくるNeovimとの格闘の物語でした。
絶対勝って見せます。応援してください。もし解決策など知っている方いたらコメントで指摘ください。

では、良いNeovimライフを！

