---
name: silent-ugc
description: Clone viral TikTok UGC app-ad hooks with your own AI actor and app demo. Use when the user wants to recreate a viral TikTok ad, build a TikTok account around one consistent AI creator, mass-produce UGC ad variations, or swap an actor into an existing hook. Runs on the Arcads MCP (nano-banana-2, gpt-image-2, Seedance 2.5).
---

# Viral UGC Ads — TikTok Hook Cloning & AI Actor Accounts

Turn a viral TikTok app ad into unlimited variations: keep the proven hook, swap in the user's AI actor and app demo, rotate the on-screen text. Runs on the Arcads MCP (nano-banana-2 + gpt-image-2 + Seedance 2.5).

Format being recreated: **[reacting person + on-screen text hook] → hard cut → [phone/laptop screen demo of the app]**. The hook is AI-generated per variation; the demo is real footage reused across ALL variations.

**Status: validated end to end. 10/10 one-shot success rate on the frame-replacement method as of 2026-08-18.**

---

## Two modes — ask which one at the start

**1. Clone mode** — recreate ONE video. User gives a TikTok URL. Run Phases 1-2, then the Pipeline once.

**2. Account mode** — build a TikTok account of N videos (10-20) around one AI actor. User gives ONE TikTok video + actor image + the app + how many videos. Scene #1 clones the source hook. Every other scene is INVENTED by re-skinning the face-swapped anchor frame (same woman, same close-up selfie grammar, new room / outfit / lighting / gesture), then animating each with the actor's Performance Profile. The single source video supplies the FORMAT; the variety comes from the scene library below. Never generate before the user approves the full scene + hook matrix and cost.

---

## Phase 1 — Interview

**The app:** what is it? A store link is fine — fetch it and study category, features, pricing, audience, reviews. This drives the hook angles. Also ask the goal (installs / trials / awareness).

**The two required assets — search the working folder FIRST, ask only if missing:**
1. **App demo video** — screen recording of the app in use (mp4/mov named demo, screen, app, recording). Strip audio. If it has a hook attached, split and keep only the demo.
2. **Actor image** — the face for every video (jpg/png named model, actor, face, reference). Frontal, well-lit, ≥700px. Warn if it's low-res or a stock photo the user may not have rights to; an AI-generated face avoids both problems.

**The source TikTok(s)** — the viral video whose hook and performance are being cloned.

## Phase 2 — Verify Arcads MCP

Call `arcads_list_products`. If it responds, note the productId and continue. If the tools aren't available:

> To connect the Arcads MCP:
> 1. Get your MCP connection details from the Arcads dashboard (Settings → API / Integrations → MCP).
> 2. Claude Code: `claude mcp add arcads <MCP URL>`, then restart the session.
> 3. Claude desktop/web: Settings → Connectors → add the Arcads connector and authorize.
> Tell me when it's added and I'll re-verify.

## Phase 3 — Performance DNA extraction (do this BEFORE any generation)

The actor image gives you a FACE. This gives you a PERSONALITY. Every video on an account must inherit the source creator's performance, not a generic reaction. This step is what separates believable UGC from overacted AI slop.

1. **Extract EVERY frame** of the hook into ordered contact sheets and read them all in sequence:
   `ffmpeg -i hook.mp4 -vf "scale=356:-2,tile=3x4" -q:v 3 sheets/S_%02d.jpg`
   Sampling at 1-2 fps misses the performance — the personality lives in the micro-timing.
2. **Answer these from the frames** and write them down as the account's Performance Profile:
   - **Start state**: does it open mid-gesture or from neutral? Exactly where is the hand at frame 0?
   - **Core gesture**: what is it, when does it land, does it HOLD or transition? Count the frames it stays fixed.
   - **Intensity**: rate peak expression 1-10. Where does emotion live — eyes, mouth, brows? Is the mouth even visible?
   - **Gaze choreography**: how many shifts, how fast, to what targets (lens / off-side / up / down), in what order?
   - **Head movement**: amplitude in degrees, speed. Blinks natural or performed?
   - **Camera**: static, drift, push-in? Does the camera or the subject move more?
   - **Pacing**: any beat changes at all, or one held state?
   - **Forbidden moves**: what the creator NEVER does (laughing, gasping, hand dropping, mouth showing).
3. **Write the profile as a reusable motion paragraph** with caps baked in ("about 30% intensity", "hand never lowers", "2-3 slow gaze shifts", "head almost still"). Profile + scene line + rawness wrapper = the Seedance prompt for every video on that account.
4. Multiple source videos? Extract from the top performer, cross-check the others, keep what's invariant — that IS the formula.

**Worked example** (a real account's three pinned mega-hits, same creative re-shot): opens mid-reaction with a motion-blurred hand arriving; hand covers mouth+nose the entire clip and never lowers; mouth never visible; the smile exists only as raised cheeks and crinkled eyes at ~30%; the whole performance is 2-3 slow gaze shifts (off-side → drifting up like replaying a memory → brief lens contact → eyes soften / slow blink); head drifts a few degrees; all perceived motion comes from gentle handheld camera drift; zero beat changes. Prompting the opposite (expression transitions, stacked intensity words, laugh beats) produces overacted output that reads as AI immediately.

## Phase 4 — Suggest hooks

Propose 10-20 on-screen texts grouped by angle, strongest first:

1. **Price anchoring vs the human alternative** — "my trainer charges $80 for what this app does free"
2. **Confession / wish-I-knew-sooner** — "i wasted 2 years before finding this"
3. **Insider / gatekeeping / enemy** — "trainers are mad this app exists"
4. **Mistake callout** — "you're doing X wrong and don't even know it"
5. **Anxiety / beginner** — matched to the app's audience
6. **Lazy-person** — "i stopped planning X. the app does it now"
7. **Transformation shock** — "day 1 vs day 60. i'm shook"

Rules: under 12 words, casual lowercase, curiosity gap, pairs with a reaction shot. Tell the user that re-texting an existing generated hook costs 0 credits — the cheapest way to scale.

## Phase 5 — Confirm before running

Show the plan and get an explicit yes:
- Number of videos, which hook text on which scene, which source each scene is based on
- The actor being used
- Cost: **~75 credits per generated hook**; image passes, motion analysis, text overlays and stitching are free

Never start generation without confirmation.

---

## Phase 6 — The pipeline (per video)

### 1. Download and split
`yt-dlp` the TikTok. Its JS challenge is flaky — retry up to 5 times with 3s sleeps. Profile URL instead of a video URL? List with `--flat-playlist` and pick the ad-format video (or ask).

Find the hook/demo boundary: `ffmpeg -i v.mp4 -vf "select='gt(scene,0.3)',metadata=print" -an -f null -` → first big scene score. Verify visually. Keep the hook only, muted, re-encoded for frame accuracy:
`ffmpeg -i v.mp4 -t <cut> -an -c:v libx264 -crf 18 -preset fast hook.mp4`
**Discard the source's demo entirely** — the user's own demo replaces it.

### 2. Extract the anchor frame
`ffmpeg -i hook.mp4 -frames:v 1 -q:v 2 frame0.jpg`

### 3. Image pass 1 — recreate and strip text (`arcads_generate_image_nano_banana`, model `nano-banana-2`, 9:16)
> Recreate the first image exactly: same vertical selfie framing, same camera angle, same pose and hand position, same clothing, same [lighting description], same background, same amateur iPhone front-camera quality with mild sensor noise and soft compression. Remove ALL text overlays so the image is completely clean of text. Keep real skin texture, uneven lighting, no beautification, no retouching.

### 4. Image pass 2 — face swap (`arcads_generate_image_gpt`, model `gpt-image-2`, 9:16)
Inputs: [clean frame, actor photo].
> Face swap. The first image is a raw TikTok video frame. The second image shows the target person. Replace the woman's face in the first image with the face of the woman from the second image so it is unmistakably the SAME PERSON as image two: her exact facial structure, skin tone, eye color, lips, and hair color/length replacing the original's. Match image two's face closely enough that someone comparing them would say it is the same woman. Everything else in the first image stays IDENTICAL: the pose, the expression, the clothing, the background, the framing, and the amateur iPhone quality with sensor noise and compression. Do not beautify, do not smooth the skin, do not change the lighting.

**Use gpt-image-2 here, not nano-banana** — validated 2026-08-18: nano-banana drifts toward a generic blended face, gpt-image-2 tracks the actor reference closely. **Two separate passes are mandatory**; a combined recreate+swap prompt keeps the original face because "recreate exactly" beats the swap instruction.

QC both image passes before continuing. They're free (daily limit), so retries cost nothing.

### 5. Motion description
Default: extract frames at 2fps (`ffmpeg -vf fps=2`) and have a cheap subagent (Sonnet) read them and return ONE paragraph — camera movement plus expressions and gestures moment by moment with timing, motion ONLY, no appearance/background/text. Batch one subagent per hook in parallel for account mode.
Exception: `arcads_analyze_media` (free, sees full frame rate) when the hook has fast choreography 2fps would miss.

### 6. ⚠️ MATCH THE MOTION TO THE ANCHOR FRAME — the rule that prevents weird output
**Before writing the Seedance prompt, look at the swapped frame and identify the exact pose it freezes.** The animation MUST begin from that pose and continue plausibly from it. Seedance animates *forward from the reference image*, so a prompt that contradicts the frame produces a physically incoherent clip — a hand teleporting, a gesture restarting, or a jarring re-pose in the first half second.

- Frame shows hand ALREADY over the mouth → prompt "the hand stays there for the entire video and never lowers". Never "she raises her hand to her mouth".
- Frame shows hand mid-air / off to the side → prompt the completion: "in the first half second her raised hand settles softly over her mouth, and stays there for the rest of the video".
- Frame shows a neutral face, no hand → either prompt a full gesture arc, or re-pick the anchor frame from a moment that already shows the gesture.
- Frame is mid-blink or mid-word → pick a different frame; those animate badly.

The source hook's timing tells you what happens *next* from that pose; the anchor frame tells you where it *starts*. Reconcile the two before spending credits. When they conflict, the frame wins — or regenerate the frame in the pose you want (free).

### 7. Animate (`arcads_generate_video_seedance_25`)
`referenceImages=[swapped frame]`, **NO reference video**, 9:16, 480p, audio off, duration 4s (min).
Prompt structure:
> Animate this exact frame as a raw handheld TikTok selfie video, keeping the woman's identity, the framing, the scene and the amateur iPhone quality of the reference image unchanged throughout. [PERFORMANCE PROFILE paragraph, starting from the frame's exact pose] [scene + lighting line] Natural handheld micro-shake, mild sensor noise, soft compressed phone-footage look, no color grading, no cinematic look, no beautification, no text on screen. Vertical 9:16, looks like a real TikTok filmed [context].

~75 credits. No reference video = the cheap tier AND the only way identity survives on close-ups.

**Prompt rules learned from rejected output:**
- ONE held expression beats a multi-beat transition. Transitions are where Seedance overacts.
- Cap the intensity explicitly ("about 30% intensity", "restrained", "like she's holding it in").
- Give reactions an explicit positive valence ("delighted disbelief", "quietly amazed") — "disbelief" alone renders as distress.
- One intensity word maximum. Never stack "wide eyes" + "widening further" + "stunned".
- Forbidden unless the source actually does it: laughing, giggling, gasping, jaw movement, eyes re-widening.

### 8. QC before spending anything further
Extract 2-3 frames. Check: is it the actor's face (not the original creator's)? Does the scene match? Any AI gloss or beautification? Any text that came back? Does the motion start from the frame's pose? Failed identity → do NOT spend more credits without asking the user.

### 9. Trim, text, stitch
- Trim the generation to the source hook's length (matching pacing matters — these hooks are 2-4s).
- `arcads_add_text_overlay`: `tiktok-none` for classic white TikTok text (most common), `tiktok-dark` for stacked bars. Position copied from the source (usually center or top), `duration: full-video`, fontSize ~17-20. Multi-bar formats = chain two calls. For mass text rotation on one clip, local ffmpeg `drawtext` is free and instant.
- `arcads_stitch_videos`: [hook-with-text, demo]. Deliver the mp4; keep intermediates in `assets/`.

---

## Style DNA (default look — the user's own direction overrides this)

**Shot grammar, every scene:** extreme close-up front-camera selfie, face fills 60-80% of frame, arm's length or closer, constant handheld micro-drift, amateur framing. Deeply unpolished: real skin texture, uneven lighting, sensor noise, compression, motion blur, no grading, no beautification.

**Gesture library** (one per scene): nail-biting stare; hand over mouth shock; finger-to-lips shh; palm-forward talking; eyes-closed then open stare; hand-near-mouth hold; adjusting hair while glancing away; conspiratorial lean-in; eye-roll into smirk.

**Scene/lighting library** (same character, rotate): warm dim lamp bedroom evening; dark bedroom with purple-blue LED at night; bright daylight bedroom in an oversized hoodie; sunny bedroom in a colorful top; outdoor cafe/patio brick wall; kitchen counter morning; parked car; bathroom mirror; couch under a blanket; walking outside in winterwear.

**Wardrobe/props:** everyday variety per scene — hoodies, tanks, casual tops; nail colors, rings, bracelets, a watch. **The face and hair stay IDENTICAL across all scenes** — that's the account's brand.

**Text style:** white TikTok font with black outline, 2-3 lines, centered mid-frame, casual lowercase with an emoji. Hook 2-4s, hard cut to demo.

---

## Arcads gotchas (all learned the hard way)

- The server **cannot read local file paths**. Always `arcads_get_upload_url` → HTTP PUT the bytes with matching Content-Type → use the returned `filePath`.
- Temp S3 uploads are **single-use**. Re-upload references for every call, even the same file. `INVALID_REFERENCE_IMAGES` usually means the upload was already consumed.
- Copy presigned URLs **exactly** — mangling a query param (e.g. `X-Amz-SignedHeaders`) returns HTTP 400.
- Seedance takes 5-7 min; text overlay and stitch ~1 min each and cost 0. Poll `arcads_get_asset` after a background sleep; don't spin.
- `arcads_replace_actor` is **not viable** for downloaded TikToks: burned-in caption text survives the swap and can make the character detector reject the input outright. Only for clean user-supplied footage. Failed runs are refunded.
- Passing a **reference video** to Seedance on a close-up hook makes the output clone the original creator's face regardless of prompt wording — three phrasings failed. That's why the pipeline is image-only.

## Cost model

| Step | Cost |
|---|---|
| Seedance generation (image-only, 4s, 480p) | ~75 credits |
| nano-banana-2 / gpt-image-2 image passes | free (daily limit) |
| Motion analysis (subagent or arcads_analyze_media) | free |
| Text overlay, stitch, trim | free |
| **New text on an existing generation** | **0 credits** |

10-video account ≈ 750 credits. Text variants on top are free.
