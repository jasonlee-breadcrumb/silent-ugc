---
name: silent-ugc
description: Clone viral TikTok UGC app-ad hooks with your own AI actor and app demo. Use when the user wants to recreate a viral TikTok ad, build a TikTok account around one consistent AI creator, mass-produce UGC ad variations, or swap an actor into an existing hook. Runs on the Arcads MCP (nano-banana-2, gpt-image-2, Seedance 2.5).
---

# Silent UGC: TikTok hook cloning and AI actor accounts

Take a viral TikTok app ad and turn it into as many variations as you want. Keep the proven hook, put your own AI actor and app demo in it, and rotate the on-screen text. Everything runs on the Arcads MCP (nano-banana-2, gpt-image-2, and Seedance 2.5).

The format you are recreating: a person reacting to something off screen, a line of text over their face, then a hard cut to a screen recording of the app. The hook is generated fresh for each variation. The app demo is one real recording reused across all of them.

## Two modes, ask which one at the start

**Clone mode** recreates one video. The user gives you a TikTok URL. Run the interview, then the pipeline once.

**Account mode** builds a TikTok account of 10 to 20 videos around one AI actor. The user gives you a TikTok video, an actor image, the app, and how many videos they want. Every scene reuses the same face in a new room, outfit, and light, so a scroller reads the whole account as one person. Get the user to approve the full scene and hook list, plus the cost, before you generate anything.

## Phase 1: interview

**The app.** What is it? A store link is enough. Fetch it and read the category, features, pricing, audience, and reviews. This is what drives the hook angles. Ask what the goal is: installs, trials, or awareness.

**The two assets you need.** Search the working folder first, and only ask if they are missing.
1. App demo video. A screen recording of the app in use (look for mp4 or mov files named demo, screen, app, or recording). Strip the audio. If it has a hook attached, split it and keep only the demo.
2. Actor image. The face for every video (jpg or png named model, actor, face, or reference). Frontal, well lit, at least 700px. If it is low resolution or a stock photo the user may not have rights to, say so. A generated face avoids both problems.

**The source TikTok.** The viral video whose hook you are cloning.

## Phase 2: verify the Arcads MCP

Call `arcads_list_products`. If it responds, note the productId and continue. If the tools are not available:

> To connect the Arcads MCP:
> 1. Get your MCP connection details from the Arcads dashboard (Settings, then API / Integrations / MCP).
> 2. In Claude Code: `claude mcp add arcads <MCP URL>`, then restart the session.
> 3. In Claude desktop or web: Settings, Connectors, add the Arcads connector and authorize.
> Tell me when it is added and I will check again.

This skill handles the source video on its own. `yt-dlp` downloads it, `ffmpeg` pulls the frames, and the Read tool renders those frames directly, so you do not need a separate video-watching skill. The hooks are silent, so there is no transcript to fetch either.

## Phase 3: pick the hook and suggest the text

Propose 10 to 20 on-screen texts, grouped by angle, strongest first:

1. Price against the human alternative. "my trainer charges $80 for what this app does free"
2. Confession. "i wasted 2 years before finding this"
3. Gatekeeping. "trainers are mad this app exists"
4. Mistake callout. "you're doing X wrong and don't even know it"
5. Beginner anxiety, matched to the app's audience.
6. Lazy. "i stopped planning X, the app does it now"
7. Transformation. "day 1 vs day 60, i'm shook"

Keep each under 12 words, casual lowercase, with a curiosity gap that pairs with a reaction shot. Re-texting a video you already generated costs nothing, so that is the cheapest way to scale a rotation.

## Phase 4: confirm before you run

Show the plan and get an explicit yes:
- How many videos, which hook goes on which scene, and which source each scene is built from.
- The actor you are using.
- The cost (see the cost model at the end).

Do not start generating until the user confirms.

## The pipeline, per video

### 1. Download and split
TikTok's JS challenge fails now and then, so retry instead of stopping on the first error:

```bash
for i in 1 2 3 4 5; do yt-dlp -o "video.%(ext)s" "<TIKTOK_URL>" && break; sleep 3; done
```

Given a profile URL instead of a single video, list it first and pick the ad, or ask:
`yt-dlp --flat-playlist --print "%(id)s | %(view_count)s views | %(title).70s" --playlist-end 20 "<PROFILE_URL>"`

Find the boundary between the hook and the demo:
`ffmpeg -i v.mp4 -vf "select='gt(scene,0.3)',metadata=print" -an -f null -`
Take the first big scene score, then confirm it by eye. Cut the hook, mute it, and re-encode it for frame accuracy:
`ffmpeg -i v.mp4 -t <cut> -an -c:v libx264 -crf 18 -preset fast hook.mp4`
Keep the hook. The user's own demo replaces the source's demo.

### 2. Build the reference image with nano-banana
This is your actor, in the scene, holding the pose. It becomes the reference image for Seedance.

For a new scene, generate the actor directly in a fresh room with `arcads_generate_image_nano_banana` (model `nano-banana-2`, 9:16). Pass the actor image as the reference and describe the room, outfit, light, and gesture. Keep the same face and hair every time.

To match a specific source scene closely, recreate the source frame first, then put your actor in it:
- Recreate and strip the text (`arcads_generate_image_nano_banana`, `nano-banana-2`, 9:16):
> Recreate the first image exactly: same vertical selfie framing, same camera angle, same pose and hand position, same clothing, same [lighting], same background, same amateur iPhone front-camera quality with mild sensor noise and soft compression. Remove all text overlays so the image is completely clean of text. Keep real skin texture, uneven lighting, no beautification, no retouching.
- Swap in your actor (`arcads_generate_image_gpt`, model `gpt-image-2`, 9:16), inputs [clean frame, actor photo]:
> Face swap. The first image is a raw TikTok video frame. The second image shows the target person. Replace the woman's face in the first image with the face of the woman from the second image so it is unmistakably the same person as image two: her exact facial structure, skin tone, eye colour, lips, and hair colour and length. Everything else stays identical: the pose, the expression, the clothing, the background, the framing, and the amateur iPhone quality with sensor noise and compression. Do not beautify, do not smooth the skin, do not change the lighting.

Image passes are free, so check the result and regenerate until the face is clearly your actor and the frame is clean.

### 3. Motion transfer with Seedance 2.5
This is the core step, and the definitive method. Give Seedance two inputs: your actor scene image as the reference image, and the original TikTok hook as the reference video. Seedance keeps your actor's face and scene and performs them through the exact motion, expression, and camera move from the hook.

`arcads_generate_video_seedance_25`, `referenceImages=[your actor scene image]`, `referenceVideos=[the hook]`, 9:16, 480p, audio off. Set the duration to the hook's length.

Keep the prompt short and let the video carry the performance:
> The woman in the output is the woman from the reference image: keep her exact face, skin tone, eyes, eyebrows, and hair, and keep her room, outfit, and lighting. Everything else comes from the reference video. Reproduce that performance exactly: the same eye movement and gaze, the same hand movement and position, the same head movement, the same expression and timing, and the same handheld camera move. Keep the raw amateur TikTok front-camera look. Vertical 9:16, no text on screen.

### 4. Trim, text, stitch
- Trim the result to the hook's length. These hooks run two to four seconds, and the pacing matters.
- Add the on-screen text with `arcads_add_text_overlay`: use `tiktok-none` for classic white TikTok text, or `tiktok-dark` for stacked bars. Match the source's position (usually centre or top), `duration: full-video`, fontSize around 17 to 20. For a stacked format, chain two calls. To rotate many texts over one clip, local ffmpeg `drawtext` is free and instant.
- Stitch with `arcads_stitch_videos`: [hook with text, demo]. Deliver the mp4 and keep the intermediates in `assets/`.

## Scaling an account

One source hook gives you the format. The variety comes from the scene, not the motion. Build each scene by generating your actor in a new room (step 2 above), then run the same motion transfer with the same hook as the reference video. Re-texting a finished clip is free, so once a scene exists you can rotate every hook angle over it at no cost.

**Gesture library** (one per scene): hand over mouth; finger to lips; palm-forward talking; eyes closed then open; hand near mouth held; adjusting hair while glancing away; lean-in; eye-roll into a smirk.

**Scene and light library** (same face, rotate): warm dim lamp bedroom in the evening; dark bedroom with a purple-blue LED at night; bright daylight bedroom in an oversized hoodie; sunny bedroom in a colourful top; outdoor cafe or patio; kitchen counter in the morning; parked car; bathroom mirror; couch under a blanket; walking outside in winter clothes.

**Wardrobe and props:** everyday variety per scene, hoodies, tanks, casual tops, different nail colours, rings, a watch. The face and hair stay identical across every scene. That is the account's brand.

## Style

Every scene is an extreme close-up front-camera selfie. The face fills most of the frame, shot at arm's length or closer, with constant handheld drift and amateur framing. Keep it unpolished: real skin texture, uneven light, sensor noise, compression, no grading, no beautification.

For the on-screen text, use the white TikTok font with a black outline, two or three lines, centred mid-frame, casual lowercase. The hook runs two to four seconds, then a hard cut to the demo.

## Working with Arcads media

- Arcads reads uploaded files, not local paths. Call `arcads_get_upload_url`, PUT the bytes with a matching Content-Type, and use the returned `filePath`.
- Upload URLs are single use. Re-upload a reference for every call, even the same file.
- Copy presigned URLs exactly, including every query parameter.
- Seedance takes a few minutes. Text overlay and stitching take about a minute each and are free. Poll `arcads_get_asset` after a short wait rather than looping.

## Cost model

| Step | Cost |
|---|---|
| Seedance motion transfer, per video | ~250 credits |
| nano-banana-2 and gpt-image-2 image passes | free (daily limit) |
| Text overlay, stitch, trim | free |
| New text on an existing video | free |

A 10-video account runs about 2,500 credits. Text variants on top are free.
