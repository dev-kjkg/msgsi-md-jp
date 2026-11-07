## 公式とは一切関係性がない。日本人向けに翻訳したものです
公式はこちらです
- https://github.com/MysticGSI/mysticgsi
# MysticGSI

Androidの純正ファームウェアからGSI（汎用システムイメージ）を作成するツールです。

対応するファームウェア形式：

- フルOTA ZIP（`payload.bin`）

- fastbootパッケージ

- `super.img`、sparseイメージ

- `system.new.dat`

- Samsung TAR

- Huawei `UPDATE.APP`

- Unisoc `.pac`

- LG `.kdz`

- Oppo `.ozip`

- QFILパッケージ

- Sony `.sin`

- Pixelファクトリーイメージ

パーティション形式はext4、EROFS、F2FSに対応しています。

## セットアップ

このプロジェクトにはPython 3.10以降が必要です。

### 自動セットアップ

#### macOS / Linux

macOSでは、事前にHomebrewとXcode Command Line Toolsをインストールしてください。

```
git clone https://github.com/MysticGSI/mysticgsi.git && cd mysticgsi
./setup_host.py
```

#### Windows

Python 3とGitがまだインストールされていない場合は、先にインストールしてください。どちらもPATHに追加されていることを確認してください。
WinGetの利用を推奨します。

```
winget install --id Python.Python.3.14 --exact --source winget
winget install --id Git.Git --exact --source winget
```

```
git clone https://github.com/MysticGSI/mysticgsi.git && cd mysticgsi
python setup_host.py
```

macOS、Debian/Ubuntu、Arch、Fedora、Windowsでは、このスクリプトがシステムパッケージをインストールし、`.venv`を作成します。NixOSでは開発シェルを確認します。ネイティブ画像ツールとApktoolのJARファイルはリポジトリに同梱されています。

### 手動セットアップ

開発ツールも必要な場合は、`requirements.txt`を`requirements-dev.txt`に置き換えてください。同梱のApktool JARを使うにはJavaが必要です。ネイティブ画像ツールは`tools/bin/<platform>/<architecture>`にあります。

<details>
<summary>macOS</summary>

```
xcode-select --install
brew install python@3.13 curl aria2 gpatch openssl@3 openjdk@17
export PATH="$(brew --prefix openjdk@17)/bin:$PATH"
"$(brew --prefix python@3.13)/bin/python3.13" -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

</details> <details>
<summary>Ubuntu / Debian</summary>

```
sudo apt-get update
sudo apt-get install -y python3 python3-venv python3-pip curl aria2 \
    openssl patch ca-certificates libarchive-tools default-jre-headless
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

</details> <details>
<summary>Arch</summary>

```
sudo pacman -Syu --needed python python-pip curl aria2 openssl patch \
    libarchive jre17-openjdk-headless
python -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

</details> <details>
<summary>Fedoraおよびその他（RHELなど）</summary>

```
sudo dnf update -y
sudo dnf install -y --skip-unavailable python3 python3-pip curl aria2 openssl patch \
    ca-certificates libarchive java-openjdk-headless
python -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

</details> <details>
<summary>NixOS</summary>

```
nix --extra-experimental-features 'nix-command flakes' develop
python3 cli.py build <name> <firmware> --type <type>
```

</details> <details>
<summary>Windows</summary>

```
winget install --id 7zip.7zip --exact --source winget
winget install --id aria2.aria2 --exact --source winget
winget install --id EclipseAdoptium.Temurin.26.JDK --exact --source winget
python -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
```

</details>

## 使い方

```
.venv/bin/python cli.py build <name> <firmware|URL> [--type <type>] [--format <lz4|lz4hc|lzma>] [--compress]
.venv/bin/python cli.py rebuild <name> [--compress]
.venv/bin/python cli.py list
.venv/bin/python cli.py clean
```

Windowsでは、リポジトリのルートディレクトリから次のように実行します。

```
.venv\Scripts\python.exe cli.py build test D:\path\to\firmware.tar
```

WindowsでF2FSを展開するには、大文字と小文字を区別できるローカルNTFS作業ディレクトリが必要です。ディレクトリ権限を適切に扱えるよう、最新のPython for Windowsを使用してください。

### コマンド

```
build - 新しいROMイメージを作成
rebuild - 既存のROMイメージを再作成
list - すべてのビルドを一覧表示
clean - すべてのビルドを削除
```

### コマンドラインオプション

```
name - ビルド名
firmware or URL - ファームウェアのパス、またはダウンロード元URL
--type <type> - ROMの種類。既定値はauto（自動判定に失敗する場合は種類を明示してください）
--compress - 出力イメージをZIPに圧縮
--add <tag> - ビルド名にタグを追加
--no-debloat - パッチセットが通常削除するアプリを残す
--no-merge - パーティションをsystemに統合せず、別々のイメージを作成
--format <lz4|lz4hc|lzma> - ext4の代わりに、指定した圧縮方式のEROFSを作成
--avb-key /path/to/key.pem - 指定したRSA秘密鍵で署名
```

既定では、AOSPのAVB RSA-2048テスト鍵でイメージに署名します。独自のRSA秘密鍵で署名するには、`--avb-key /path/to/key.pem`を指定してください。

systemパーティションとproductパーティションを分けて保持するには、次のように実行します。

```
.venv/bin/python cli.py build lg firmware.kdz --no-merge --compress
```

このコマンドは`*-system.img`、`*-product.img`と、通常systemに統合される他のパーティションのイメージを出力します。もともとsystem内にあるパーティションはそのままです。ZIPには各イメージが`<partition>.img`という名前で格納され、`rebuild`でもこの構成が維持されます。これらのイメージを使用するには、パーティションを個別にマウントできる端末が必要です。

既定のイメージ形式はext4です。`--format`を指定するとEROFSで作成でき、通常はサイズを30～50%削減できます。`lz4hc`は`lzma`より削減幅が小さい一方、端末上での読み込みが速くなります。対象端末のカーネルとfstabがEROFSに対応している必要があります。また、lzmaを使うには`CONFIG_EROFS_FS_ZIP_LZMA`が必要です。`rebuild`では元のビルドの形式が維持されます。

実行例：

```
.venv/bin/python cli.py build raven \
    https://dl.google.com/dl/android/aosp/raven-up1a.231105.003-factory-76a795d5.zip \
    --type pixel --compress
```

## 開発

プルリクエストを作成する前に、[CONTRIBUTING.md](CONTRIBUTING.md)を確認してください。

```
.venv/bin/ruff check .
```

サイズが50 MiB以上のパッチファイルはxz圧縮（`<name>.xz`）して保存し、ビルド時に展開します。ファイルを追加したら、`./tools/assets.py pack`を実行し、`.xz`ファイル（分割アーカイブの場合は`.xz.000`、`.xz.001`など）をコミットしてください。

`./tools/assets.py status`を実行すると、圧縮済みのファイルを確認できます。

## ライセンス

Apache License 2.0です。詳細は[LICENSE](LICENSE)を参照してください。同梱のavbtoolはMITライセンスです。詳細は[tools/avb/LICENSE](tools/avb/LICENSE)を参照してください。`patches/`以下のサードパーティ製ファイルには個別の利用条件があります。詳細は[NOTICE](NOTICE)を参照してください。
