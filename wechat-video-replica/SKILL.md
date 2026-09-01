---
name: wechat-video-replica
description: 下载并全自动复刻微信公众号内嵌视频（数字人口播科普类）。涵盖：腾讯视频 vid 解析下载、逐帧分析制作手法、OCR+whisper 提取文案、edge-tts 配音、AI 生图素材、PIL 绘制模板卡片、Wav2Lip 对口型、ffmpeg 叠化合成成片。当用户要求"下载公众号视频/分析视频怎么做/复刻一个视频"时使用。
agent_created: true
---

# 公众号视频「下载 → 分析 → 复刻」全流程

## A. 下载微信内嵌视频（腾讯视频）
1. `curl -A <chrome UA>` 抓文章 HTML，搜 `data-src` 找 `v.qq.com/txp/iframe/player.html?vid=XXX`。
2. 调 `https://vv.video.qq.com/getinfo?vid=VID&platform=101001&charge=0&otype=json&defn=sd&sdtfrom=v1010`，JSON 去掉 `QZOutputJson=` 前缀和结尾 `;`。
3. 取 `vl.vi[0]`：`ul.ui[].url`（用最后一条 dispatch host）+ `fn` + **`fvkey`（直接内含，不要再调 getkey，会报 format invalid）**。
4. 完整地址 = base + fn + "?vkey=" + fvkey，带 `Referer: https://mp.weixin.qq.com/` 整段下载。

## B. 逐帧分析制作手法
- `ffmpeg -vf "fps=1/5"` 均匀抽帧 + `select='gt(scene,0.08)'` 场景检测（0 硬切=全溶解转场=模板剪辑）。
- 低码率（<300kbps）= 静态图为主；连续帧只有口部变 = 语音驱动数字人；音频句间真静音(silencedetect -35dB)= TTS 无 BGM。

## C. 文案提取（两条路互补）
1. **字幕 OCR**：抽帧裁底部字幕区 `crop=W:110:0:H-110`，macOS Vision 框架 OCR（venv 装 `pyobjc-framework-Vision pyobjc-framework-Quartz`，zh-Hans + Accurate）。
2. **全量转写**（补无字幕口播）：`pip install mlx-whisper`，`mlx_whisper.transcribe(audio_16k.wav, path_or_hf_repo='mlx-community/whisper-small-mlx', language='zh')`。输出可能是繁体+错字，与 OCR 交叉校验修正。

## D. 配音
`pip install edge-tts`，voice `zh-CN-XiaoxiaoNeural`，rate=-2%，每段一个 mp3，ffprobe 测时长。

## E. 视觉素材
- ImageGen：片头 3D 卡通图、写实人物正面像（Wav2Lip 用，闭嘴正脸）、扁平贴纸页（白底，PIL 阈值转透明抠图）。注意实际出图尺寸可能不是请求的 1024，先 PIL 量尺寸。
- 原片中的实物照片（药盒/二维码）直接从抽帧裁剪复用。
- PIL 绘卡片：方格纸（line 每 26px）+ 粉色双线框 + 贴纸 + 蓝字要点；章节卡（吊牌+虚线框+POP 三层描边字）；字体 `/System/Library/Fonts/Hiragino Sans GB.ttc`（index=1 粗体）。

## F. Wav2Lip 对口型（macOS Apple Silicon）
1. `git clone --depth 1 https://github.com/Rudrabha/Wav2Lip.git`；权重：HF `camenduru/Wav2Lip` 的 `wav2lip_gan.pth`（435MB 完整；`wav2lip.pth` 链接可能只有 166KB 是坏的）；人脸检测 `https://www.adrianbulat.com/downloads/python-fan/s3fd-619a316812.pth` 放 `face_detection/detection/sfd/s3fd.pth`。
2. **兼容性补丁（torch 2.13 + numpy 2.x + 新 librosa 必打）**：
   - `audio.py`: `librosa.filters.mel(sr=..., n_fft=..., ...)` 全关键字
   - `inference.py` 与 `sfd_detector.py`: `torch.load(..., weights_only=False)`
   - `face_detection/utils.py`: `np.int` → `np.int64`
3. 运行：`python inference.py --checkpoint_path checkpoints/wav2lip_gan.pth --face 人物.png --audio seg.wav --outfile out.mp4 --static True --resize_factor 1 --wav2lip_batch_size 32 --pads 0 20 0 0`
   - `--static True` 静态图模式只检测一次人脸；输出比音频短约 0.1-0.7s（后面用 tpad 补）。
   - **不要在 run_in_background 跑**：numba 缓存触发 safe-delete 钩子会被误杀，前台跑。

## G. ffmpeg 合成（关键坑）
本机 homebrew ffmpeg **没有 subtitles/drawtext 滤镜**（无 libass/freetype）：
- 字幕改用 **PIL 生成字幕条 PNG**（864x486 透明底，底部浅青条 + 白字深描边，自动换行），ffmpeg `overlay=0:0:enable='between(t,a,b)'` 链烧录。
分块构建（每块音画等长！）→ xfade 0.6s 叠化 + acrossfade：
- **`tpad stop` 单位是帧不是秒**（stop=2 只补 2 帧），补末帧用 `tpad=stop_mode=clone:stop=150` 再 `-t dur` 截断。
- **xfade 偏移必须用构建后实测的视频流时长**（ffprobe stream=duration）。若某块视频比音频短，offset 错位会导致后续全部塌方（成片视频流只剩几十秒，audio 正常）。
- 片头 Ken Burns：`scale=1296:729,zoompan=z='min(zoom+0.0006,1.10)':d=N:x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)':s=864x486:fps=25`。
- **zoompan 的 d 作用在循环输入的每一帧上**：`-loop 1 -t N` 输入有 N*fps 帧时，zoompan d=帧数 会把总帧数放大 N 倍——同段音频配两张图（concat 两子段）时前段永远播不完。解决：子段输入改 `-loop 1 -t 0.04`（单帧）+ zoompan d=子段帧数，再 concat。
- **PIL 画 UI 图标不要用 unicode 符号**（⚡✓◉ 在 Hiragino Sans GB 下变豆腐块），用 line/polygon/arc 手绘。
- xfade 链 offset 递推：`offset_k = sum(dur_0..k-1) - XF*k`。

## H. 质检
分条命令抽帧（**不要一条 ffmpeg 多 -ss 多输出，会写出同一张图**）；确认：视频流时长≈音频流时长、字幕条位置、口型变化、转场正常。
