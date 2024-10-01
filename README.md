# cleanup_matedata
适用于清理emby等媒体服务器产生的无用nfo,ass,-320-10.bif,-thumb.jpg小脚本

## 清理文件的类型
1. 清理目录内全部找不到母文件的.nfo文件(`name.mp4`/`name.mkv`<---`name.nfo`)
2. 清理目录内全部找不到母文件的.ass文件(`name.mp4`/`name.mkv`<---`name.ass`)
3. 清理目录内全部找不到母文件的-320-10.bif文件(`name.mp4`/`name.mkv`<---`name-320-10.bif`)
4. 清理目录内全部找不到母文件的-thumb.jpg文件(`name.mp4`/`name.mkv`<---`name-thumb.jpg`)

## 配置说明
| 变量        | 说明            | 数据类型       | 实例                        |
| ----------- | -------------- | ------------- | --------------------------- |
| MEDIA_DIR   | 媒体文件夹目录  | string(字符串) | "/tmp/video"                |
| RECYCLE_BIN | 回收站目录      | string(字符串) | "/tmp/RECYCLE"              |
| LOG_FILE    | 日志文件路径    | string(字符串) | "/log/cleanup_matedata.log" |
| ACTION      | 操作(移动/删除) | string(字符串) | "move"/"delete"             |

## 代码
```shell
#!/bin/bash

# 配置
MEDIA_DIR="/volume2/video/TV shows/"    # 媒体文件夹
RECYCLE_BIN="/volume2/video/#recycle/TV shows/" # 回收站文件夹
LOG_FILE="/volume1/scripts/emby/cleanup.log"    # 记录日志文件
ACTION="move"  # "delete" 或 "move"

# 创建日志文件
touch "$LOG_FILE"

# 清理多余的nfo文件
cleanup_nfo() {
    find "$MEDIA_DIR" -type f -name "*.nfo" | while read -r nfo_file; do
        base_name=$(basename "$nfo_file" .nfo)
        dir_name=$(dirname "$nfo_file")
        
        # 排除特定的nfo文件
        if [[ "$base_name" == "tvshow" || "$base_name" == "season" ]]; then
            continue
        fi

        # 检查是否存在同名的mp4/mkv文件
        if [[ ! -e "$dir_name/$base_name.mp4" && ! -e "$dir_name/$base_name.mkv" ]]; then
            if [[ "$ACTION" == "move" ]]; then
                mkdir -p "$RECYCLE_BIN$(dirname "${nfo_file#$MEDIA_DIR}")"
                mv "$nfo_file" "$RECYCLE_BIN$(dirname "${nfo_file#$MEDIA_DIR}")/"
                echo "$(date '+%Y-%m-%d %H:%M:%S') - [nfo]移动文件: $nfo_file 到回收站 ($base_name)" >> "$LOG_FILE"
            elif [[ "$ACTION" == "delete" ]]; then
                rm "$nfo_file"
                echo "$(date '+%Y-%m-%d %H:%M:%S') - [nfo]删除文件: $nfo_file ($base_name)" >> "$LOG_FILE"
            fi
        fi
    done
}

# 清理多余的ass字幕文件
cleanup_ass() {
    find "$MEDIA_DIR" -type f -name "*.ass" | while read -r ass_file; do
        # 提取基础文件名（去掉 .ass 和 .xxxxx.ass 部分）
        base_name=$(basename "$ass_file" | sed 's/\.[^.]*\.ass$//; s/\.ass$//')  # 去掉 .xxxxx.ass 和 .ass
        dir_name=$(dirname "$ass_file")

        # 检查是否存在同名的 mp4/mkv 文件
        if [[ ! -e "$dir_name/$base_name.mp4" && ! -e "$dir_name/$base_name.mkv" ]]; then
            if [[ "$ACTION" == "move" ]]; then
                mkdir -p "$RECYCLE_BIN$(dirname "${ass_file#$MEDIA_DIR}")"
                mv "$ass_file" "$RECYCLE_BIN$(dirname "${ass_file#$MEDIA_DIR}")/"
                echo "$(date '+%Y-%m-%d %H:%M:%S') - [ass]移动文件: $ass_file 到回收站 ($base_name)" >> "$LOG_FILE"
            elif [[ "$ACTION" == "delete" ]]; then
                rm "$ass_file"
                echo "$(date '+%Y-%m-%d %H:%M:%S') - [ass]删除文件: $ass_file ($base_name)" >> "$LOG_FILE"
            fi
        fi
    done
}



# 清理多余的以“-320-10.bif”和“-thumb.jpg”结尾的文件
cleanup_extra_files() {
    find "$MEDIA_DIR" -type f \( -name "*-320-10.bif" -o -name "*-thumb.jpg" \) | while read -r extra_file; do
        base_name=$(basename "$extra_file" | sed -E 's/(-320-10\.bif|-thumb\.jpg)$//')
        dir_name=$(dirname "$extra_file")

        # 检查是否存在同名的mp4/mkv文件
        if [[ ! -e "$dir_name/$base_name.mp4" && ! -e "$dir_name/$base_name.mkv" ]]; then
            if [[ "$ACTION" == "move" ]]; then
                mkdir -p "$RECYCLE_BIN$(dirname "${extra_file#$MEDIA_DIR}")"
                mv "$extra_file" "$RECYCLE_BIN$(dirname "${extra_file#$MEDIA_DIR}")/"
                echo "$(date '+%Y-%m-%d %H:%M:%S') - [bif/jpg]移动文件: $extra_file 到回收站 ($base_name)" >> "$LOG_FILE"
            elif [[ "$ACTION" == "delete" ]]; then
                rm "$extra_file"
                echo "$(date '+%Y-%m-%d %H:%M:%S') - [bif/jpg]删除文件: $extra_file ($base_name)" >> "$LOG_FILE"
            fi
        fi
    done
}

echo "---------------------------------------------------------------------------------------------------- $(date '+%Y-%m-%d %H:%M:%S') - 开始 ----------------------------------------------------------------------------------------------------" >> "$LOG_FILE"
# 执行清理操作
cleanup_nfo
cleanup_ass
cleanup_extra_files
echo "---------------------------------------------------------------------------------------------------- $(date '+%Y-%m-%d %H:%M:%S') - 结束 ----------------------------------------------------------------------------------------------------" >> "$LOG_FILE"

echo "清理完成，日志已记录在 $LOG_FILE"

```
