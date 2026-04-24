# Seedance 2.0 — Prompting Techniques & Patterns

Source: Theoretically Media (Tim), April 23, 2026 — "WILD Seedance Prompts & Tricks You Need To Know!"
Video: https://www.youtube.com/watch?v=MOkjfFIIb6E
Full copy-paste prompts available in Tim's newsletter: https://theoreticallymedia.beehiiv.com/

---

## Technique 1: Snap Stop-Time Shockwave

**Credit:** Chris First (viral prompt)
**Use case:** ads, product reveals, hero moments

**Pattern:**
- Use a reference image of the character
- Specify camera: Arri Alexa Mini, 35mm lens
- Structure as timed shots
- Key phrase: "at the snap, a subtle spherical shockwave burst from his fingertips"
- Seedance handles spherical shockwaves very well

**What makes it work:**
- The "snap" moment gives the model a clear trigger for the VFX
- Timed shot structure gives Seedance a pacing framework
- The camera/lens spec grounds the visual style in cinematic realism
- Spherical shockwaves are a strength of Seedance's physics simulation

**Modify for your use:** swap the character, swap the trigger action (clap, stomp, point), swap the shockwave effect (frost burst, fire ring, light pulse)

---

## Technique 2: First Frame / Last Frame Storytelling

**Credit:** Framer
**Use case:** narrative bridges, emergent storytelling, surprise reveals

**Pattern:**
- Generate (or provide) two images: a "before" and an "after"
- Use Seedance's first frame / last frame feature (not Omni reference)
- Prompt: "show me what happens in between. Use multiple camera angles."
- That's it. The model fills in the narrative gap.

**What makes it work:**
- Seedance's emergent storytelling is extremely good at inventing the narrative between two states
- Multiple camera angles forces the model to create dynamic coverage, not just a single locked-off morph
- The surprise factor: you don't know what story the model will tell between your two frames

**Examples that worked:**
- Princess in tower + camping scene → knight rescue sequence
- Woman in tavern + pirates → siren scene (captivated pirates)
- Two weapon reference images → action narrative bridge

**Tips:**
- More contrast between first and last frame = more interesting narrative
- Works great for character reveals, transformation sequences, and "what happened?" mysteries
- The model tends to create 3-5 distinct moments in the bridge

---

## Technique 3: Day-In-The-Life Time Loop

**Credit:** Coda
**Use case:** lifestyle content, character pieces, "day in the life" videos

**Pattern:**
- Long, detailed prompt with wardrobe changes throughout
- Image reference for the subject
- Structure as a time-loop: morning → commute → work → evening → bed → loop back to morning
- Include sound effects tied to each shot
- Color logic: "hyperreal pop look, ultrarealistic style"

**What makes it work:**
- Seedance handles wardrobe changes well when prompted explicitly
- The loop structure creates a satisfying narrative arc
- Sound effect cues in the prompt give the model pacing information
- "Hyperreal pop look" is a specific aesthetic trigger Seedance responds to well

**Pro tip: Chinese translation for character limits.**
Coda's prompt exceeds 3,500 characters (Runway's limit on some platforms). Solution: translate the entire prompt to Chinese using Claude/ChatGPT. Chinese is a more character-efficient language, bringing you well under the limit. Seedance was trained on Chinese text and handles it natively.

**Expect re-rolls.** Tim got his keeper on the 5th generation. Don't expect one-shots on complex prompts like this.

---

## Technique 4: "Contact" Mirror Shot (Invisible VFX Recreation)

**Credit:** Plasmo
**Use case:** invisible VFX, trick shots, cinematic homages

**Reference:** the famous mirror shot from Contact (1997, Robert Zemeckis, DP Don Burgess). Young Jodie Foster runs down a hallway, reaches for a medicine cabinet, and the camera reveals the shot was actually the mirror reflection.

**Pattern:**
- Key phrase: "core illusion" — triggers Seedance to understand this is a trick shot
- Speed ramp: frame rate ramps from 24fps to 48fps then into slow motion
- Describe the physical camera trick (hallway run, reach for mirror, reveal)

**What makes it work:**
- "Core illusion" as a concept tag seems to help Seedance understand the shot requires visual trickery
- Frame rate ramp instructions give the model specific pacing to aim for
- Recreating famous shots is a great way to test (and show off) what Seedance can do

**Expect LOTS of re-rolls.** Tim took 8 generations to get a keeper. Many attempts produced "close but not quite right" results. This is a hard shot for any system.

---

## Technique 5: Real Faces Are Getting Through

**Observation, not a technique.**

As of April 2026, Seedance appears to be relaxing face filters. Realistic human faces are appearing much more frequently in generations. Celebrity likenesses are still blocked (don't try to put Tom Cruise in your prompt), but original characters with reference images are passing through reliably.

---

## Technique 6: GPT Image 2 → Seedance Character Pipeline

**Pattern:**
1. Generate or source an illustrated character image
2. Use GPT Image 2 to convert the illustrated character into a photorealistic "live-action" version
3. Have GPT Image 2 create a 9:16 character reference sheet from the photorealistic version
4. Use that reference sheet as Seedance's character input

**Why this works:** Seedance handles photorealistic reference images better than illustrated ones. GPT Image 2's image-to-image is currently the best tool for this conversion step.

---

## General Seedance 2.0 Prompting Tips

From this video and community patterns:

1. **Specify camera and lens.** "Arri Alexa Mini, 35mm lens" or "RED V-Raptor, 50mm anamorphic" grounds the output in cinematic realism.

2. **Use timed shot structures.** Breaking the prompt into timed segments gives Seedance a pacing framework to follow.

3. **Reference images work.** Character consistency via reference images is functional and improving. Face filters are loosening.

4. **Don't expect one-shots.** Complex prompts take 3-8 generations to nail. Budget for re-rolls.

5. **Chinese translation saves character count.** If your prompt is too long, translate to Chinese. Seedance handles it natively.

6. **Name your VFX.** "Spherical shockwave burst," "speed ramp," "frame rate ramp from 24 to 48" — specific VFX terminology gets better results than vague descriptions.

7. **"Core illusion" as a concept tag.** When you want a trick shot or invisible VFX, this phrase seems to help Seedance understand the intent.

8. **First frame / last frame is underused.** Most people think in terms of single prompts. The two-image "show me what happens in between" technique produces some of the most surprising and cinematic results.

---

## Resources

- Full copy-paste prompts: [Theoretically Media Newsletter](https://theoreticallymedia.beehiiv.com/)
- Video: [WILD Seedance Prompts & Tricks](https://www.youtube.com/watch?v=MOkjfFIIb6E)
- Contact mirror shot reference: [Don Burgess interview](https://youtu.be/HQRu9cz5L9E)
- Martini AI Studio (step-into-set, character turnarounds, multiplayer): [martini.film](https://www.martini.film/theoretically-media)
