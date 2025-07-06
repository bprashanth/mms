# Mae May Slide (mms)

An app to automate the creation of short "explainer" videos. 

Run via `npm run dev`. 

## Speech to text 

Use [whisper](https://huggingface.co/spaces/Xenova/whisper-web)

## Scaling 

To scale a video by adding black bars to `2:3` aspect ratio

```
$ ffmpeg -i input.mp4 -vf "pad=iw:iw*3/2:(ow-iw)/2:(oh-ih)/2" output_23.mp4
$ ffmpeg -i input.jpg -vf "crop=iw:iw*3/2" output.jpg
```
* add `pad=...,color=white`
