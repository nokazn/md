# WSL  上で  Nuxt.js  を開発する際に https でアクセスできるようにする

## mkcert のインストール

ローカル開発環境用の認証局作成ツールである mkcert をインストールする。 Go 言語で作られており、ビルドするには Go 1.13+ が必要。

Go が入ってなければ Go をインストールする。

```bash
$ cd /usr/local/src
$ curl -O https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz
$ tar -C /usr/local -xzvf /usr/local/src/go1.14.2.linux-amd64.tar.gz
$ ln -s /usr/local/go/bin/go /usr/local/bin/go
$ go version
go version go1.14.2 linux/amd64
```

mkcert をダウンロードしてビルドし、`/usr/local/bin/mkcert` にシンボリックリンクを通している。

```bash
sudo apt install libnss3-tools
git clone https://github.com/FiloSottile/mkcert && cd mkcert
go build -ldflags "-X main.Version=$(git describe --tags)"
ln -s ./mkcert /usr/local/bin/mkcert
```

## ルート証明書の作成

WSL 上でルート証明書をインストールし、Windows で使用可能なファイルとして出力する。

```bash
$ mkcert install
$ cd $(mkcert -ROOTCA)
$ openssl pkcs12 -export -inkey ./rootCA-key.pem -in ./rootCA.pem -out rootCA.pfx
```

## ルート証明書のインストール

WSL 上にインストールした`rootCA.pfx`をWindowsにダウンロードする。
ダウンロードした`rootCA.pfx`を右クリックし、`PFX のインストール`→ 証明書ストアの、`証明書をすべて次のストアに配置する`→`信頼されたルート証明機関` を選択する。

## SSL証明書と認証キーを作成

WSL 内のアプリケーションのプロジェクトルートに戻る。
認証済にしたドメイン名を引数に追加し、証明書と認証キーを得る。

```bash
$ mkcert localhost 127.0.0.1
Using the local CA at "/path/to/rootCA.pem" ✨

Created a new certificate valid for the following names 📜
 - "localhost"
 - "127.0.0.1"

The certificate is at "./localhost+1.pem" and the key at "./localhost+1-key.pem" ✅
```

## Nuxt で証明書と認証キーを設定

```bash
$ mv ./localhost+1.pem ./localhost.pem
$ mv ./localhost+1-key.pem ./localhost-key.pem
```

```typescript
// nuxt.config.ts
import fs from 'fs';
import path from 'path';
import { Configuration } from '@nuxt/types';

const config: Configuration = {
  //...
  server: {
      https: {
        key: fs.readFileSync(path.resolve(__dirname, 'localhost-key.pem')),
        cert: fs.readFileSync(path.resolve(__dirname, 'localhost.pem')),
      },
    },
  //...
};

export default config;

```

`yarn dev`として、新しいタブで開くと`https://127.0.0.1:3000`でアクセスできるようになっている。

## 参考

[FiloSottile/mkcert: A simple zero-config tool to make locally trusted development certificates with any names you'd like.](https://github.com/FiloSottile/mkcert)