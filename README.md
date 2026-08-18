# silent-ugc

A Claude Code skill that clones viral TikTok app-ad hooks with your own AI actor.

Silent UGC is the format that quietly prints installs: a person reacting to something off-screen, a line of text over their face, then a hard cut to a screen recording of the app. No dialogue, no voiceover, no production. This skill takes one viral video of that shape and turns it into as many variations as you want, starring a face you own.

## What it does

Give it a TikTok URL, a photo of your actor, and a screen recording of your app. It will:

1. Split the source video, keep the hook, throw away their demo
2. Rebuild the hook's first frame with the original's text stripped out
3. Swap your actor's face into that frame
4. Read every frame of the original to learn how the creator actually performs
5. Animate your actor doing the same performance in the same room
6. Burn on your hook text and stitch it to your app demo

Two modes:

- **Clone mode** — recreate one video
- **Account mode** — one source video becomes 10-20 videos starring the same actor in different rooms, outfits and lighting, so a scroller reads it as one person's account

## Why it works

Most AI UGC fails for two reasons: the face drifts between videos, and the actor overacts. This skill fixes both.

**Identity** is solved at the image layer, not the video layer. Video models conditioned on a reference clip will clone the *original creator's* face on close-up shots no matter how the prompt is worded — three phrasings were tested and all three failed. Swapping the face in a still frame first, then animating that still with no reference video, keeps your actor and costs a quarter as much.

**Performance** is solved by frame-by-frame analysis before any generation. The skill extracts every frame of the source hook and derives a Performance Profile: where the hand starts, whether the gesture holds or transitions, peak intensity on a 1-10 scale, gaze choreography, head amplitude, what the creator never does. That profile drives every generation on the account. The finding that pays for itself: winning hooks are usually ONE frozen gesture plus eye micro-acting at about 30% intensity, not a multi-beat emotional arc. Prompting an arc is what makes AI UGC look like AI UGC.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- The **Arcads MCP** connected ([arcads.ai](https://arcads.ai)) — used for image generation, face swap, video generation, text overlay and stitching
- `ffmpeg` and `yt-dlp` on your PATH

```bash
brew install ffmpeg yt-dlp
```

## Install

```bash
git clone https://github.com/jasonlee-breadcrumb/silent-ugc.git
cp -r silent-ugc/skills/silent-ugc ~/.claude/skills/
```

Restart Claude Code. Then just point it at a video:

```
clone this hook with my actor: https://www.tiktok.com/@someone/video/123456
```

It will interview you for the app, find your demo footage and actor image in the working folder, verify the Arcads connection, propose hook texts, and show you the full plan with a credit estimate before spending anything.

## Cost

| Step | Cost |
|---|---|
| Video generation (4s, 480p, image-only) | ~75 Arcads credits |
| Image passes, motion analysis, text, stitching | free |
| New text on an existing generation | free |

A 10-video account runs about 750 credits. Text variants on top are free, which is the cheapest way to scale a rotation.

## A note on ethics and rights

This skill clones the *format* of a video: the shot grammar, the pacing, the performance style. Do not clone a real creator's face. The pipeline is explicitly designed to replace the original person with an actor you own, and it stops and asks rather than shipping a clip whose face drifted back toward the source. Use a generated or licensed actor image, and make sure you have the rights to the app footage you're promoting.

## License

MIT
