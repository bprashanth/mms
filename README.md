# Mae May Slide (mms)

An app to automate the creation of short "explainer" videos. 

Run via `npm run dev` from the `app` directory.  

## Speech to text 

Use [whisper](https://huggingface.co/spaces/Xenova/whisper-web)

## Scaling 

To scale a video by adding black bars to `2:3` aspect ratio

```
$ ffmpeg -i input.mp4 -vf "pad=iw:iw*3/2:(ow-iw)/2:(oh-ih)/2" output_23.mp4
```

No black bars
```
$ ffmpeg -i leopard.mp4 -vf "crop='if(gt(iw/ih,2/3),ih*2/3,iw)':'if(gt(iw/ih,2/3),ih,iw*3/2)',crop='iw:ih:(in_w-iw)/2:(in_h-ih)/2'" leopard_23.mp4
```

## Pauses 

This will insert a 0.8s pause at `119.88`
```json
  {
    "timestamp": [119.88, 119.88],
    "displayText": "what if we had an internet for nature?",
    "fullText": " But what if we had an internet for nature?",
    "asset": null,
    "animation": "fade-in",
    "pauseDuration": 0.8
  },  
```
