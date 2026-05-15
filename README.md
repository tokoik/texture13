# texture13 - 第１８回 テクスチャオブジェクト サンプルプログラム

## 1. 概要

このプログラムは、OpenGL における「テクスチャマッピング (Texture Mapping)」の基礎を学ぶための、学生向けのサンプルプログラムです。本プログラムは、以下のブログ記事の解説に沿って学習を進めるための雛形として提供されています。

- [第１８回 テクスチャオブジェクト](https://tokoik.github.io/blog/%E3%83%86%E3%82%AF%E3%82%B9%E3%83%81%E3%83%A3%E5%85%A5%E9%96%80/2005/06/14/texture.html)

今回は次の部屋のテクスチャをマッピングした立方体の中心に視点を置いて周囲を眺めるプログラムにおいて、視点の前方にティーポットを置き、それに部屋のテクスチャを使ってキューブマッピングを施します。キューブマッピングを用いた環境マッピングにより、ティーポットに部屋の光景を映り込ませます。

- [部屋の中心であたりを見回すサンプルプログラム](https://github.com/tokoik/texture12)

## 2. ビルド方法

このプログラムは CMake を用いてビルドを構成しています。各プラットフォームごとの手順は以下の通りです。なお、プログラムをビルドするためのバイナリディレクトリは、バージョン管理ファイル（.gitignore）の設定に合わせて build という名前にします。

### 2.1 Windows (Visual Studio 2022 の場合)

1. コマンドプロンプトまたは PowerShell を開き、このプロジェクトのディレクトリに移動します。
2. 以下のコマンドを実行してビルドディレクトリを作成し、CMake で構成を行います。

   ```bat
   mkdir build
   cd build
   cmake .. -G "Visual Studio 17 2022"
   ```

3. 生成された build フォルダ内の texture13.sln を Visual Studio で開きます。
4. ソリューションエクスプローラーで texture13 プロジェクトを右クリックし、「スタートアップ プロジェクトに設定」を選択します。
5. 「ローカル Windows デバッガー」をクリックするか、F5 キーを押してビルドおよび実行します。

### 2.2 macOS (Xcode の場合)

1. ターミナルを開き、このプロジェクトのディレクトリに移動します。
2. 以下のコマンドを実行してビルドディレクトリを作成し、Xcode 用のプロジェクトを生成します。

   ```sh
   mkdir build
   cd build
   cmake .. -G Xcode
   ```

3. 生成された build/texture13.xcodeproj を Xcode で開きます。
4. 左上のスキーム選択（再生ボタンの横）が texture13 になっていることを確認します。
5. 「Run」ボタン（再生ボタン）をクリックするか、Command + R を押してビルドおよび実行します。

### 2.3 Ubuntu Linux

1. ターミナルを開き、このプロジェクトのディレクトリに移動します。
2. 必要なパッケージ（freeglut3-dev や pkg-config など）がインストールされていることを確認し、以下のコマンドでビルドします。

   ```sh
   mkdir build
   cd build
   cmake ..
   make
   ```

## 3. 使い方

### 3.1 プログラムの起動方法

各OSとも、ビルド後に生成されるバイナリディレクトリ (build) やそのサブフォルダから起動します。（※ CMake の設定により、Windows や Xcode では Debug などのフォルダ下に実行ファイルが置かれることがあります）

- **Windows**

  Visual Studio 上で「ローカル Windows デバッガー」をクリックして実行するか、またはコマンドプロンプトから以下のコマンドで起動します。

  ```cmd
  cd build\Debug
  texture13.exe
  ```

- **macOS**

  Xcode 上で左上の「Run（再生ボタン）」をクリックするのが楽です。これにより texture13.app アプリケーションバンドルとして自動的に実行されます。アプリケーションバンドルを直接起動するなら、Finder から build/Debug/texture13.app をダブルクリックするか、ターミナルから open build/Debug/texture13.app を実行します (この場合はエラーメッセージ等が表示されません)。

- **Ubuntu Linux**

  ターミナルから以下のコマンドで実行ファイル（バイナリ）を直接起動します。

  ```sh
  cd build
  ./texture13
  ```

### 3.2 操作方法

- **マウスの左ボタンでドラッグ**

  視点を回転（実際には箱の方を回転）させ、部屋（箱）の内部を見回すことができます。マウスの動きに合わせて周囲の壁、天井、床（テクスチャ）が移動します。

- **キーボードの q, Q または ESC キー**

  プログラムを終了します。

## 4. 解説

プログラムのソースコード（主に main.cpp と box.cpp）をもとに、このプログラム内で行われている処理手順を解説します。このプログラムは、テクスチャオブジェクトを利用して、部屋（立方体）用の 2D テクスチャと、ティーポットの環境マッピング用のキューブマップテクスチャを別々に管理し、それらを切り替えて描画する処理を実装しています。

### 4.1 テクスチャの読み込みとテクスチャオブジェクトの生成 (main.cpp 内の init 関数)

1. **2D テクスチャの領域確保:**

    [`glTexImage2D()`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glTexImage2D.xml) 関数を使って、幅 1024、高さ 128 の 2D テクスチャ領域を確保します。これは部屋（箱）の描画用となるデフォルトのテクスチャ（無名テクスチャ）です。

2. **テクスチャオブジェクト（キューブマップ用）の生成:**

    [`glGenTextures()`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glGenTextures.xml) によって、独立したテクスチャを取り扱うためのテクスチャ名（ID）を 1 つ生成し、`cubemap` 変数に格納します。

3. **画像の読み込みとテクスチャの設定:**

    6 つの画像ファイル (`room2ny.raw` など) を順番に読み込みます。このループの中で、以下の 2 つの処理を同時に行っています。
    - **2D テクスチャへの部分的な置き換え:** 読み込んだ画像を [`glTexSubImage2D()`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glTexSubImage2D.xml) を使って、先に確保した 1024x128 の 2D テクスチャ領域の一部に順次転送します。
    - **キューブマップへの割り当て:** [`glBindTexture(` `GL_TEXTURE_CUBE_MAP, cubemap)`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glBindTexture.xml) でテクスチャオブジェクトをバインドしたのち、[`glTexImage2D()`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glTexImage2D.xml) のターゲットに `GL_TEXTURE_CUBE_MAP_POSITIVE_X` などを指定して、各面の画像をキューブマップのデータとして設定します。設定が終わったら、`glBindTexture(` `GL_TEXTURE_CUBE_MAP`, 0 `)` を呼び出して無名テクスチャに戻しています。

4. **環境マッピングのためのテクスチャ座標の自動生成:**

    再度キューブマップのテクスチャオブジェクトをバインドした状態で、[`glTexGeni()`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glTexGen.xml) を使用してテクスチャ座標 (S, T, R) の生成モードを `GL_REFLECTION_MAP`（環境マッピング）に設定します。

### 4.2 面ごとのテクスチャ座標の割り当て (box.cpp 内の `box()` 関数)

部屋として描画する立方体の処理です。結合された 1 つの 2D テクスチャを 6 つの面に分割して貼り付けるため、[`glTexCoord2dv()`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glTexCoord.xml) を使って面ごとに 1/8（0.125）ずつの幅でテクスチャ座標を切り分けて割り当てています。

```c
  /* 頂点のテクスチャ座標 */
  static const GLdouble texcoord[][4][2] = {
    { { 0.0,   1.0 }, { 0.125, 1.0 }, { 0.125, 0.0 }, { 0.0,   0.0 } },
    { { 0.125, 1.0 }, { 0.25,  1.0 }, { 0.25,  0.0 }, { 0.125, 0.0 } },
    /* ... 他の4面も同様に 0.125 ずつずらす ... */
  };
```

### 4.3 オブジェクトごとのテクスチャの切り替えと描画 (main.cpp 内の scene() 関数)

`scene()` 関数では、テクスチャの種類を切り替えながら、ティーポットと部屋（箱）を描画します。

1. **ティーポット（環境マッピング）の描画:**

    - `glBindTexture(` `GL_TEXTURE_CUBE_MAP`, cubemap `)` でキューブマップ用のテクスチャオブジェクトを有効にします。
    - `glEnable(` `GL_TEXTURE_CUBE_MAP` `)` と、各テクスチャ座標の自動生成 (`GL_TEXTURE_GEN_S` など) を有効にします。
    - 視点が常に原点にあるため、背景（部屋）の回転に合わせてティーポットへの環境マッピングの映り込みも回転させる必要があります。そのため、`glMatrixMode(` `GL_TEXTURE` `)` でテクスチャ行列にトラックボールの回転 (`trackballRotation()`) を適用します。このとき、[`glLoadTransposeMatrixd()`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glLoadTransposeMatrix.xml) を使うことで、映り込みが背景に対して正しい方向に回転するようにします。
    - 視点の前方 (`Z = -3.0`) に `glutSolidTeapot(` 1.0 `)` でティーポットを描画します。
    - 描画後、テクスチャ行列を元に戻し、キューブマップと自動生成を無効化し、バインドを `0`（無名テクスチャ）に戻します。

2. **部屋（2D テクスチャ）の描画:**

    - `glEnable(` `GL_TEXTURE_2D` `)` で 2D テクスチャ（無名テクスチャ）を有効にします。
    - `GL_MODELVIEW` 行列に対してトラックボールの回転 (`glMultMatrixd(` `trackballRotation()` `)`) を適用し、部屋全体を回転させることで、見回しているような効果を作ります。
    - 原点にある視点の前に置いたティーポットが箱（部屋）の中に収まるように、箱を `box(` 5.0, 5.0, 5.0 `)` というように大きくします。箱の内側を見るため、裏側を向いた箱の面がカリングされないように、初期化 (`init()`) 時に `glDisable(` `GL_CULL_FACE` `)` を指定します。

![ティーポットへの映り込みも回転する](https://tokoik.github.io/blog/assets/images/texture13.webp)
