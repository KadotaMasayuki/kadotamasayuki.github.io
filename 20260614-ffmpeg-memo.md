# FFmpegでいろいろ（メモ）

調べて使って、その都度追加してゆく・・



## 付帯情報：現在のディレクトリ以下のすべてのディレクトリを辿って処理する場合のOSごとの処理。出力ファイル名は末尾に'_output'を付けてみた。

for windows(.bat) - バッチファイルにするときは%%iにしなければいけないような気がする。未確認。
```
for /R %i in (*.mp4) do ffmpeg -i "%i"  (何かのフィルタとか)  "%~dpni_output.mp4"
```

for Linux(.sh)
```
find . -type f -name "*.mp4" -exec sh -c 'ffmpeg -i "$0"  (何かのフィルタとか)  "${0%.mp4}_output.mp4"' {} \;
```




## 画面サイズを変更する。縦横比を維持、指定サイズに収まるよう縮小する。拡大はしない。

```
ffmpeg -i input.mp4 -vf "scale=2000:1500:force_original_aspect_ratio=decrease" output.mp4
```

※ mp4の場合、縦横どちらかでも奇数だとエラーが出るので、scale=2000:-2 などのようにする必要がある。あとで記載する。


### 現在のディレクトリ以下を辿り、jpgファイルを縦横比を維持しつつ2000x1500に収まるよう変更する。拡大はしない。出力ファイル名は末尾に'_small'を付けてみた。

for windows(.bat) - 未確認
```
for /R %i in (*.jpg) do ffmpeg -i "%i" --vf "scale=2000:1500:force_original_aspect_ratio=decrease" "%~dpni_small.jpg"
```

for Linux(.sh)
```
find . -type f -name "*.jpg" -exec sh -c 'ffmpeg -i "$0" -vf "scale=2000:1500:force_original_aspect_ratio=decrease" "${0%.jpg}_small.jpg"' {} \;
```



## メタデータの関係で警告が出るのをなんとかする

映像・音声を再エンコードすることなくffmpegで出力するだけで整理されることがある。
```
ffmpeg -i input.mp4 -c:v copy -c:a copy output.mp4
```



## 指定時刻の動画を切り出し

開始100秒から110秒までを動画として保存。再エンコードしない。
```
ffmpeg -ss 100 -t 10 -i input.mp4 -c copy output.mp4
```

開始100秒（1分40秒）から110秒（1分50秒）までを動画として保存。再エンコードしない。
```
ffmpeg -ss 00:01:40.000 -to 00:01:50.000 -i input.mp4 -c copy output.mp4
```

開始、終了とも、指定時刻はキーフレームではないことがほとんどなので、厳密に時刻を守らせたいときは、-c copyを外すと良い。
```
ffmpeg -ss 100 -t 10 -i input.mp4 output.mp4
```



## 指定時刻の画像を保存

開始100秒のフレームを画像として保存
```
ffmpeg -ss 100 -i input.mp4 -frames:v 1 -q:v 2 -y output.jpg -loglevel error
```

開始100秒（1分40秒）のフレームを画像として保存
```
ffmpeg -ss 00:01:40.000 -i input.mp4 -frames:v 1 -q:v 2 -y output.jpg -loglevel error
```
-iオプションより前で指定することで直近のキーフレームを保存するため、指定時刻が厳守されるわけではない。高速だが。




## 指定時刻を遵守して画像保存

```
〇〇◯
```
