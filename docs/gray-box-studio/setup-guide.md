# Gray Box Studio — Setup Guide

## Physical Stage

### The Gray Surface

- **Color:** 18% neutral gray (same as a photography gray card). Matte finish, zero reflectivity.
- **Options:**
  - Paint walls with flat gray latex paint (cheapest for permanent install)
  - Hang gray muslin backdrop fabric (cheapest for testing, portable)
  - Gray seamless paper rolls (expendable, clean surface, standard in photography)
- **Coverage:** floor to ceiling, wall to wall. the AI needs a consistent neutral background behind the performer from every angle.
- **Why gray, not green:** the AI pipeline doesn't use traditional chroma keying. gray provides a neutral canvas that preserves natural skin tones in the captured footage and works better with modern AI segmentation (SAM2, YOLO) which doesn't need color contrast to separate foreground from background.

### Tracking Markers

- **Purpose:** give the camera tracking software (COLMAP, PFTrack, SynthEyes) fixed reference points to calculate camera position and lens data for every frame.
- **Pattern:** black circles or X's on the gray surface, spaced 1-2 feet apart in a regular grid.
- **Options:**
  - Printed photogrammetry coded targets (COLMAP ArUco markers, ~$20 for a full set printed at a copy shop)
  - Black gaffer tape X's (free if you have tape, slightly less precise)
  - Pre-printed marker sheets from reality capture suppliers
- **Placement:** cover all walls and ideally the floor. markers should be visible from every camera angle. avoid placing markers where performers will stand in front of them (the AI needs to see some markers in every frame).
- **If using the robot arm:** the arm's built-in encoders already know camera position. markers become a backup/validation rather than the primary tracking source.

### Lighting

- **Goal:** flat, bright, even illumination. no shadows, no color casts, no dramatic lighting. you want the cleanest possible capture of the performer; all "real" lighting gets added in post.
- **Setup:** overhead LED panel lights covering the full stage area evenly.
- **Specs:**
  - CRI 95+ (accurate color reproduction of skin tones)
  - 5600K daylight balanced (or whatever your camera white balance is calibrated to)
  - No point sources or directional lights (those create shadows that fight with the AI-generated lighting in post)
  - Aim for 400-800 lux at performer height, measured at multiple points to confirm evenness
- **Our advantage:** if the warehouse already has overhead fluorescents/LEDs, replace with high-CRI panels. budget: ~$1,000-3,000 for good coverage of a 20x20 foot area.

### Proxy Set Pieces

- Physical objects the actors touch, sit on, walk up, lean against.
- Build from cheap materials: apple boxes, platforms, stairs, foam-core flats.
- Paint them gray to match the background (or leave unpainted; they'll be replaced in post regardless).
- Only build what's necessary for the performance. everything else is AI.

### Camera

- Our RED cameras are ideal. highest possible resolution = better segmentation, better tracking, better final quality.
- Shoot RAW for maximum flexibility in post (color, exposure, white balance all adjustable).
- Record at the highest framerate practical for the scene (24fps for cinema, higher for slow-mo).
- The robot arm provides perfect repeatable camera paths and built-in position data.

## Quick Test Setup (before committing to full install)

1. Hang a 12x12 foot gray muslin backdrop in a corner of the warehouse
2. Tape 20-30 ArUco markers on the backdrop in a grid
3. Set up 2-4 overhead LED panels for even lighting
4. Shoot a 30-second test with one performer on the RED
5. Process through the post pipeline (see post-pipeline.md)
6. Evaluate: is the segmentation clean? does the camera track lock? does the AI environment comp look convincing?
7. If yes: scale up to the full warehouse

**Estimated cost for test:** ~$200 (backdrop fabric + printed markers)
**Estimated cost for full warehouse install:** ~$1,500-3,500 (paint, markers, lighting if needed)
