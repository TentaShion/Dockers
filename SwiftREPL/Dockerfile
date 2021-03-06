# sourcekit-lsp の取得
FROM alpine:3.14.0 AS fetcher
RUN apk add git

ENV SourceKitTag=swift-DEVELOPMENT-SNAPSHOT-2021-07-15-a
RUN cd /tmp \
    && git clone --depth 1 -b ${SourceKitTag} https://github.com/apple/sourcekit-lsp.git



# Language Server の構築
FROM swift:5.4.2-bionic AS build-lang-server

RUN apt-get -y update \
    && apt-get -y upgrade \
    && apt-get install -y libsqlite3-dev

COPY --from=fetcher /tmp/sourcekit-lsp /tmp
RUN cd /tmp \
    || swift build



# VSCode 拡張機能のビルド
FROM node:14.17.3-alpine3.14 AS build-extension
COPY --from=fetcher /tmp/sourcekit-lsp/Editors/vscode /tmp
RUN cd /tmp \
    && npm install \
    && ./node_modules/.bin/vsce package -o ./out/sourcekit-lsp-vscode-dev.vsix



# 実行環境のビルド
FROM swift:5.4.2-bionic

# 作業フォルダーのマウントポイントの設定
VOLUME [ "/home/worker" ]

# Language Server のコピー
COPY --from=build-lang-server /usr/bin/sourcekit-lsp /bin/sourcekit-lsp

# Visual Studio Code 拡張機能のコピー
COPY --from=build-extension /tmp/out/sourcekit-lsp-vscode-dev.vsix /root/

# 作業フォルダーに変更
WORKDIR /home/worker
