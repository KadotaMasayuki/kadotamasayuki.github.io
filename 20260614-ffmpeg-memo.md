# FFmpegでいろいろ（メモ）

調べて使って、その都度追加してゆく・・



## 付帯情報


### コマンド実行後、ffmpegが色んな情報を吐き出すけど、ffmpegのビルド情報などが不要な時はこれ。出力がすっきりする。

```
ffmpeg -hide_banner ....
```


### 現在のディレクトリ以下のすべてのディレクトリを辿って処理する場合のOSごとの処理。出力ファイル名は末尾に'_output'を付けてみた。

for windows(.bat) - バッチファイルにするときは%%iにしなければいけないような気がする。未確認。
```
for /R %i in (*.mp4) do ffmpeg -i "%i"  (何かのフィルタとか)  "%~dpni_output.mp4"
```

for Linux(.sh)
```
find . -type f -name "*.mp4" -exec sh -c 'ffmpeg -i "$0"  (何かのフィルタとか)  "${0%.mp4}_output.mp4"' {} \;
```



## いろんなソフトで開ける動画にする（コンパチビリティ重視）

色々指定しているけれど、気を付けるところは Baseline MP4, AAC かな。

```
ffmpeg   -i input.mp4   -c:v libx264   -crf 23   -preset medium   -pix_fmt yuv420p   -profile:v baseline   -level 3.1   -bf 0   -refs 1   -c:a aac  -ar 44100   output.mp4
```

これに、

- -r 30 --- 映像秒間30フレーム
- -vf "scale=1920x1080:force_original_aspect_ratio=decrease:force_divisible_by=2" --- 映像縦横サイズ
- -b:a 64k --- 音声ビットレート64kbps
- -ac 2 --- 音声2チャンネル

などを付けて好みの動画にする。
たとえば

```
ffmpeg   -hide_banner   -i input.mp4   -r 30   -vf "scale=1920x1080:force_original_aspect_ratio=decrease:force_divisible_by=2"   -c:v libx264   -crf 23   -preset medium   -pix_fmt yuv420p   -profile:v baseline   -level 3.1   -bf 0   -refs 1   -c:a aac  -b:a 64k  -ar 44100   -ac 2   output.mp4
```



## 画面サイズを変更する。縦横比を維持、指定サイズに収まるよう縮小する。拡大はしない。

動画ファイルの場合、mp4では、縦横どちらかでも奇数だとエラーが出るので、vf内でforce_divisible_by=2 で偶数にする。
scale=2000:-2 や、 scale=-2:1500 などのように、縦横どちらかのみ指定してアスペクト比追従させて偶数にする記法もあるようだが、force_divisible_by=2のほうがシンプルだと思う。
```
ffmpeg -i input.mp4 -vf "scale=2000x1500:force_original_aspect_ratio=decrease:force_divisible_by=2" output.mp4
```


画像ファイルも拡大縮小できる。
画像は縦横の画素数が奇数になっても良いのでforce_divisible_by=2は不要。
```
ffmpeg -i input.jpg -vf "scale=2000x1500:force_original_aspect_ratio=decrease" output.jpg
```



### 現在のディレクトリ以下を辿り、jpgファイルを縦横比を維持しつつ2000x1500に収まるよう変更する。拡大はしない。出力ファイル名は末尾に'_small'を付けてみた。

for Linux(.sh)
```
find . -type f -name "*.jpg" -exec sh -c 'ffmpeg -i "$0" -vf "scale=2000:1500:force_original_aspect_ratio=decrease" "${0%.jpg}_small.jpg"' {} \;
```

for windows(.bat) - 未確認
```
for /R %i in (*.jpg) do ffmpeg -i "%i" --vf "scale=2000:1500:force_original_aspect_ratio=decrease" "%~dpni_small.jpg"
```



## ファイルを開いた時にメタデータの関係で警告が出るのをなんとかする

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



## 指定時刻のフレームを画像として保存

開始100秒のフレームを画像として保存
```
ffmpeg -ss 100 -i input.mp4 -frames:v 1 -q:v 2 -y output.jpg -loglevel error
```

開始100秒（1分40秒）のフレームを画像として保存
```
ffmpeg -ss 00:01:40.000 -i input.mp4 -frames:v 1 -q:v 2 -y output.jpg -loglevel error
```
-iオプションより前で指定することで直近のキーフレームを保存するため、指定時刻が厳守されるわけではない。高速だが。



## 動画をスマホで観るためにファイル容量や画面サイズを調整

### Windowsバッチファイル

```
@echo off
setlocal EnableDelayedExpansion

REM ==================================================
REM User Settings
REM ==================================================

REM Maximum long side
REM 1920 = Full HD class (recommended)
REM 1280 = HD class

set MAX_LONG_SIDE=1920

REM Maximum short side
REM 1080 = Full HD class (recommended)
REM 720  = HD class

set MAX_SHORT_SIDE=1080

REM x264 preset
REM ultrafast superfast veryfast faster fast
REM medium (recommended)
REM slow slower veryslow
REM Slower = smaller files, longer encode time
set PRESET=medium

REM CRF
REM 18 : visually near-lossless
REM 20 : high quality
REM 22 : recommended
REM 23 : slightly smaller files
REM 24 : capacity priority
REM Smaller CRF:
REM   Better quality
REM   Larger files
REM   Slightly slower
set CRF=22

REM H.264 profile
REM baseline / main / high
set PROFILE=high

REM GOP length
REM 30  = ~1 sec @30fps
REM 60  = recommended
REM 120 = smaller files, slower seek
set GOP=60

set AUDIO_BITRATE=128k

REM ==================================================

if "%~1"=="" (
    echo Usage:
    echo   mp4_convert.bat video1.mkv video2.mp4 ...
    pause
    exit /b 1
)

for /f %%A in ('powershell -NoProfile -Command "(Get-Date).ToString(\"yyyyMMdd_HHmmss\")"') do set LOGSTAMP=%%A

if not exist logs mkdir logs

set LOGFILE=logs\mp4_convert_%LOGSTAMP%.log

for /f %%A in ('powershell -NoProfile -Command "[DateTimeOffset]::Now.ToUnixTimeSeconds()"') do set SCRIPT_START=%%A

(
echo ==================================================
echo Started : %DATE% %TIME%
echo LogFile : %LOGFILE%
echo.
ffmpeg -version | findstr /b "ffmpeg version"
echo.
echo Settings:
echo MAX_LONG_SIDE=%MAX_LONG_SIDE%
echo MAX_SHORT_SIDE=%MAX_SHORT_SIDE%
echo PRESET=%PRESET%
echo CRF=%CRF%
echo PROFILE=%PROFILE%
echo GOP=%GOP%
echo AUDIO_BITRATE=%AUDIO_BITRATE%
echo ==================================================
echo.
) > "%LOGFILE%"

set SUCCESS=0
set FAILURE=0
set FAILED_LIST=

:LOOP

if "%~1"=="" goto SUMMARY

set "INPUT=%~1"
set "BASE=%~dpn1"
set "OUTPUT=%BASE%_.mp4"

:CHECK

if exist "!OUTPUT!" (
    set "OUTPUT=!OUTPUT:~0,-4!_.mp4"
    goto CHECK
)

for /f %%A in ('powershell -NoProfile -Command "[DateTimeOffset]::Now.ToUnixTimeSeconds()"') do set FILE_START=%%A

echo.
echo ==================================================
echo Input : %INPUT%
echo Output: !OUTPUT!
echo ==================================================

>> "%LOGFILE%" echo --------------------------------------------------
>> "%LOGFILE%" echo Input  : %INPUT%
>> "%LOGFILE%" echo Output : !OUTPUT!

ffmpeg -n ^
-i "%INPUT%" ^
-vf "scale=w='if(gte(iw,ih),min(%MAX_LONG_SIDE%,iw),min(%MAX_SHORT_SIDE%,iw))':h='if(gte(iw,ih),min(%MAX_SHORT_SIDE%,ih),min(%MAX_LONG_SIDE%,ih))':force_original_aspect_ratio=decrease,pad=ceil(iw/2)*2:ceil(ih/2)*2" ^
-c:v libx264 ^
-preset %PRESET% ^
-crf %CRF% ^
-profile:v %PROFILE% ^
-pix_fmt yuv420p ^
-g %GOP% ^
-c:a aac ^
-b:a %AUDIO_BITRATE% ^
-movflags +faststart ^
"!OUTPUT!"

set ERR=%ERRORLEVEL%

for /f %%A in ('powershell -NoProfile -Command "[DateTimeOffset]::Now.ToUnixTimeSeconds()"') do set FILE_END=%%A

set /a FILE_SEC=%FILE_END%-%FILE_START%

for /f %%A in ('powershell -NoProfile -Command "$ts=[TimeSpan]::FromSeconds(%FILE_SEC%);('{0:00}:{1:00}:{2:00}' -f $ts.Hours,$ts.Minutes,$ts.Seconds)"') do set ELAPSED=%%A

if %ERR% EQU 0 (
    echo *** SUCCESS ***
    set /a SUCCESS+=1
    >> "%LOGFILE%" echo Result  : SUCCESS
) else (
    echo *** FAILED ***
    set /a FAILURE+=1
    set FAILED_LIST=!FAILED_LIST!%INPUT%;;
    >> "%LOGFILE%" echo Result  : FAILED
)

echo Elapsed : %ELAPSED% ^(%FILE_SEC% sec^)
>> "%LOGFILE%" echo Elapsed : %ELAPSED% ^(%FILE_SEC% sec^)
>> "%LOGFILE%" echo.

shift
goto LOOP

:SUMMARY

for /f %%A in ('powershell -NoProfile -Command "[DateTimeOffset]::Now.ToUnixTimeSeconds()"') do set SCRIPT_END=%%A

set /a TOTAL_SEC=%SCRIPT_END%-%SCRIPT_START%

for /f %%A in ('powershell -NoProfile -Command "$ts=[TimeSpan]::FromSeconds(%TOTAL_SEC%);('{0:00}:{1:00}:{2:00}' -f ([int]$ts.TotalHours),$ts.Minutes,$ts.Seconds)"') do set TOTAL_HMS=%%A

echo.
echo ==================================================
echo Success : %SUCCESS%
echo Failure : %FAILURE%
echo Total   : %TOTAL_HMS% ^(%TOTAL_SEC% sec^)
echo ==================================================

(
echo ==================================================
echo Success : %SUCCESS%
echo Failure : %FAILURE%
echo Total   : %TOTAL_HMS% ^(%TOTAL_SEC% sec^)
echo.
echo Finished: %DATE% %TIME%
echo ==================================================
) >> "%LOGFILE%"

if %FAILURE% GTR 0 (

    echo.
    echo Failed Files:
    echo -------------------------

    >> "%LOGFILE%" echo.
    >> "%LOGFILE%" echo Failed Files:
    >> "%LOGFILE%" echo -------------------------

    for %%F in (!FAILED_LIST:;;= !) do (
        echo %%F
        >> "%LOGFILE%" echo %%F
    )
)

echo.
echo Log:
echo %LOGFILE%

pause
```

### Bashスクリプト

```
#!/bin/bash

# ==================================================
# User Settings
# ==================================================

# Maximum long side
# 1920 = Full HD class (recommended)
# 1280 = HD class

MAX_LONG_SIDE=1920

# Maximum short side
# 1080 = Full HD class (recommended)
# 720  = HD class

MAX_SHORT_SIDE=1080

# x264 preset
# ultrafast superfast veryfast faster fast
# medium (recommended)
# slow slower veryslow
# Slower = smaller files, longer encode time

PRESET=medium

# CRF
# 18 near-lossless
# 20 high quality
# 22 recommended
# 23 slightly smaller
# 24 size priority
CRF=22

# H.264 profile
# baseline / main / high

PROFILE=high

# GOP length
# 30  = ~1 sec @30fps
# 60  = recommended
# 120 = smaller files, slower seek

GOP=60

AUDIO_BITRATE=128k

# ==================================================

SUCCESS=0
FAILURE=0

declare -a FAILED_FILES

mkdir -p logs

LOGSTAMP=$(date +"%Y%m%d_%H%M%S")
LOGFILE="logs/mp4_convert_${LOGSTAMP}.log"

SCRIPT_START=$(date +%s)

{
echo "=================================================="
echo "Started : $(date '+%F %T')"
echo "LogFile : $LOGFILE"
echo
ffmpeg -version | head -n 1
echo
echo "Settings:"
echo "MAX_LONG_SIDE=$MAX_LONG_SIDE"
echo "MAX_SHORT_SIDE=$MAX_SHORT_SIDE"
echo "PRESET=$PRESET"
echo "CRF=$CRF"
echo "PROFILE=$PROFILE"
echo "GOP=$GOP"
echo "AUDIO_BITRATE=$AUDIO_BITRATE"
echo "=================================================="
echo
} > "$LOGFILE"

if [ "$#" -eq 0 ]; then
    echo "Usage:"
    echo "./mp4_convert.sh video1 video2 ..."
    exit 1
fi

for INPUT in "$@"
do

    BASE="${INPUT%.*}"

    OUTPUT="${BASE}_.mp4"

    while [ -e "$OUTPUT" ]
    do
        OUTPUT="${OUTPUT%.mp4}_.mp4"
    done

    FILE_START=$(date +%s)

    echo
    echo "=================================================="
    echo "Input : $INPUT"
    echo "Output: $OUTPUT"
    echo "=================================================="

    {
        echo "--------------------------------------------------"
        echo "Input  : $INPUT"
        echo "Output : $OUTPUT"
    } >> "$LOGFILE"

    ffmpeg -n \
        -i "$INPUT" \
		-vf "scale=w='if(gte(iw,ih),min(${MAX_LONG_SIDE},iw),min(${MAX_SHORT_SIDE},iw))':h='if(gte(iw,ih),min(${MAX_SHORT_SIDE},ih),min(${MAX_LONG_SIDE},ih))':force_original_aspect_ratio=decrease,pad=ceil(iw/2)*2:ceil(ih/2)*2" \
        -c:v libx264 \
        -preset "$PRESET" \
        -crf "$CRF" \
        -profile:v "$PROFILE" \
        -pix_fmt yuv420p \
        -g "$GOP" \
        -c:a aac \
        -b:a "$AUDIO_BITRATE" \
        -movflags +faststart \
        "$OUTPUT"

    RESULT=$?

    FILE_END=$(date +%s)
    FILE_SEC=$((FILE_END - FILE_START))

    ELAPSED=$(printf '%02d:%02d:%02d' \
        $((FILE_SEC/3600)) \
        $(((FILE_SEC%3600)/60)) \
        $((FILE_SEC%60)))

    if [ $RESULT -eq 0 ]; then
        echo "*** SUCCESS ***"
        echo "Result  : SUCCESS" >> "$LOGFILE"
        SUCCESS=$((SUCCESS+1))
    else
        echo "*** FAILED ***"
        echo "Result  : FAILED" >> "$LOGFILE"
        FAILURE=$((FAILURE+1))
        FAILED_FILES+=("$INPUT")
    fi

    echo "Elapsed : $ELAPSED ($FILE_SEC sec)"

    {
        echo "Elapsed : $ELAPSED ($FILE_SEC sec)"
        echo
    } >> "$LOGFILE"

done

SCRIPT_END=$(date +%s)
TOTAL_SEC=$((SCRIPT_END - SCRIPT_START))

TOTAL_HMS=$(printf '%02d:%02d:%02d' \
    $((TOTAL_SEC/3600)) \
    $(((TOTAL_SEC%3600)/60)) \
    $((TOTAL_SEC%60)))

echo
echo "=================================================="
echo "Success : $SUCCESS"
echo "Failure : $FAILURE"
echo "Total   : $TOTAL_HMS ($TOTAL_SEC sec)"
echo "=================================================="

{
echo "=================================================="
echo "Success : $SUCCESS"
echo "Failure : $FAILURE"
echo "Total   : $TOTAL_HMS ($TOTAL_SEC sec)"
echo
echo "Finished: $(date '+%F %T')"
echo "=================================================="
} >> "$LOGFILE"

if [ "$FAILURE" -gt 0 ]; then

    echo
    echo "Failed Files:"
    echo "-------------------------"

    {
    echo
    echo "Failed Files:"
    echo "-------------------------"
    } >> "$LOGFILE"

    for FILE in "${FAILED_FILES[@]}"
    do
        echo "$FILE"
        echo "$FILE" >> "$LOGFILE"
    done
fi

echo
echo "Log:"
echo "$LOGFILE"
```



## 指定時刻を遵守して画像保存

```
〇〇◯
```
