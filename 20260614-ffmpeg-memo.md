# FFmpegでいろいろ（メモ）

調べて使って、その都度追加してゆく・・

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
