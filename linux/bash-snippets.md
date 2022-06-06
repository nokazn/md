# シェルスクリプトの表現

## ファイル名を変更する

### `find` / `fd` コマンドと `xargs` コマンドを組み合わせる

```bash
$ find . -name "components" | sed 'p; s/header/footer/' | xargs -n 2 mv

$ fd "components" | sed 'p; s/header/footer/' | xargs -n 2 mv
```

### `rename` コマンドを使用する

```bash
$ rename /header/footer/ ./src/components
```
