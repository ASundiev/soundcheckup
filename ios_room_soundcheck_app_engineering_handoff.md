# iOS Room Soundcheck App — Engineering Handoff for Quick Prototype

## 1. Product idea

Build an iOS app that helps a user quickly understand and improve the acoustics of a room.

The app uses the device’s LiDAR/camera to scan the room, captures audio tests from different positions, then identifies acoustic issues such as unwanted reverb, uneven loudness, boomy corners, flutter echo risk, and problematic reflection zones.

The app should then translate the findings into practical recommendations: where to place acoustic panels, rugs, curtains, bass traps, furniture, an acoustic cloud, or where to move the desk, microphone, speakers, instrument, or recording position.

The intended first version is not a professional acoustician replacement. It is a fast consumer/prosumer room soundcheck and treatment planner.

## 2. Core conclusion

This is technically feasible on iOS.

A useful MVP can be built without LLM integration. The core should rely on deterministic room scanning, signal processing, and rule-based recommendations. An LLM can be added later as an optional explanation/coaching layer, but should not be responsible for the actual measurement or diagnosis.

## 3. Product positioning

Recommended positioning:

> Fast room soundcheck and treatment planner for creators, musicians, podcasters, home-office users, and small studio owners.

Avoid positioning:

> Professional acoustic certification tool.

The app can be accurate enough to help people make better decisions, but built-in iPhone microphones are not calibrated measurement microphones. Professional-grade measurements require a more controlled setup, ideally with an external calibrated microphone and a known speaker/test signal.

## 4. Main use cases

### Consumer / creator mode

For people who want to improve:

- home office call quality
- podcast or voiceover recording
- singing or instrument practice
- small music room setup
- YouTube/Reels/TikTok creator recording space
- garden office / small studio room acoustics
- general echo/reverb problems

### Pro / advanced mode, later

For users with:

- external calibrated microphone
- external speaker/monitor
- more controlled test setup
- interest in exported acoustic reports

This could include more formal measurements such as octave-band decay, approximate RT60/EDT, early reflections, and before/after comparison reports.

## 5. Can it work without an LLM?

Yes. The app can work entirely without an LLM.

The non-LLM version should be the core MVP because it is:

- more technically reliable
- easier to validate
- better for privacy
- cheaper to run
- possible to run mostly or fully on-device
- less likely to produce hallucinated acoustic advice

The app’s core intelligence can be built from:

- RoomPlan / ARKit geometry capture
- audio recording and playback APIs
- FFT / spectral analysis
- impulse-response or decay analysis
- room-mode estimation
- rule-based recommendation logic
- visual mapping of acoustic issues onto the scanned room

An LLM can be useful later for natural-language explanation, but it should sit on top of structured acoustic findings, not invent the acoustic diagnosis.

## 6. Suggested app flow for MVP

### Step 1: Select room use case

Ask the user what they care about:

- Video calls / home office
- Podcast / voiceover
- Singing
- Acoustic guitar / piano / instrument practice
- Mixing / speakers
- Drums / loud instruments
- General echo reduction

This matters because the app should prioritize different fixes depending on use case.

Example:

- Podcasting prioritizes voice clarity and short decay.
- Singing may tolerate more liveliness.
- Mixing needs stereo symmetry, reflection control, and bass management.
- Drums need broadband absorption and low-frequency control.

### Step 2: Scan room

Use iPhone/iPad LiDAR + camera via RoomPlan/ARKit to capture:

- room dimensions
- ceiling height if available
- approximate room volume
- walls
- openings
- windows
- doors
- large furniture if recognized
- major hard surfaces
- corners
- potential reflective planes

The scan should produce a simplified acoustic model, not just a visual 3D model.

Useful derived values:

- length, width, height
- room volume
- surface areas
- wall-to-wall parallel pairs
- glass/window area estimate
- likely hard-surface zones
- corner locations
- desk/speaker/mic position if user marks them

### Step 3: Mark important positions

The user should mark or be guided to define:

- listening position
- recording position
- desk position
- speaker positions, optional
- microphone position, optional
- instrument/vocal position
- sofa/bed/large soft furniture, optional if RoomPlan misses it

For an MVP, this can be done by tapping in the room model or by moving the phone to each position and saving the location.

### Step 4: Guided audio test

The app should not rely only on casual speech/singing because that is not repeatable enough for reliable measurement.

Use guided repeatable tests:

#### MVP test options

1. Hand clap test
   - Easiest for users.
   - Good for detecting general decay and flutter/echo problems.
   - Not ideal for bass or precise measurements.

2. Phone-generated test signal
   - Play sine sweep or pink noise from the iPhone speaker.
   - Record response using the phone mic.
   - Limited by phone speaker frequency range, especially low frequencies.

3. External speaker test
   - Better option.
   - App plays a sine sweep or pink noise through a Bluetooth/AirPlay/wired speaker.
   - Phone records response.
   - Better for room response, though still not fully professional unless calibrated.

4. Voice/use-case sample
   - User talks, sings, or plays in the room.
   - Useful as a real-world sample and for UX.
   - Should be treated as supplemental, not the primary measurement signal.

### Step 5: Measure in multiple positions

The app should ask the user to test at 5–9 positions.

Suggested basic positions:

- main use position
- each corner area
- near window/glass wall
- near desk / recording position
- near opposite wall
- center of room

For a small room, 5 positions may be enough. For a larger room, 9+ positions may be useful.

### Step 6: Analysis

Calculate and display:

- noise floor
- relative loudness by position
- spectral balance by position
- obvious boomy frequency bands
- decay/reverb estimate
- early reflection risk zones
- flutter echo risk
- low-frequency room-mode estimates from dimensions
- before/after comparisons if user repeats the test after treatment

### Step 7: Recommendations

Turn findings into prioritized advice.

Example output:

- “Treat the window wall first: likely strong high-frequency reflections.”
- “Add a rug or heavier floor covering if the floor is hard.”
- “Add broadband panels at first reflection points beside the desk.”
- “Add an acoustic cloud above the recording/listening position.”
- “Avoid putting thin foam in corners for bass issues; use thicker corner trapping.”
- “Move the desk 30–60 cm away from the wall and retest.”
- “The right rear corner shows low-frequency buildup; test bass trapping there first.”

The recommendation engine can be rule-based for MVP.

## 7. Core technical components

### 7.1 Room scanning module

Potential Apple stack:

- RoomPlan
- ARKit
- RealityKit, optional for visualization

Responsibilities:

- Start room scan
- Capture dimensions and room layout
- Export simplified room geometry
- Detect walls, windows, doors, large furniture/features where possible
- Convert scan into an internal acoustic map

Data model example:

```swift
struct RoomModel {
    let dimensions: SIMD3<Float> // width, length, height
    let volume: Float
    let surfaces: [RoomSurface]
    let openings: [RoomOpening]
    let objects: [RoomObject]
    let userMarkedPositions: [MeasurementPosition]
}

struct RoomSurface {
    let id: UUID
    let type: SurfaceType // wall, floor, ceiling, window, door, unknown
    let area: Float
    let normal: SIMD3<Float>
    let center: SIMD3<Float>
    let materialHint: MaterialHint?
}
```

### 7.2 Audio capture module

Potential Apple stack:

- AVAudioEngine
- AVAudioSession
- AVAudioPCMBuffer
- Accelerate framework for FFT

Use measurement-style audio session settings where possible to reduce automatic system processing.

Responsibilities:

- Configure audio session
- Record test samples
- Timestamp recordings
- Associate audio sample with room position
- Store raw audio or processed features
- Optionally avoid storing raw voice/audio for privacy

### 7.3 Signal generation module

Generate repeatable test signals:

- sine sweep
- pink noise
- white noise, optional
- short impulse-like signal, if appropriate

For MVP, a sine sweep and hand clap detection may be enough.

### 7.4 Acoustic analysis module

Responsibilities:

- FFT / spectral analysis
- decay curve estimation after impulse/clap/sweep
- approximate reverb estimate
- detect obvious flutter echo patterns
- detect frequency peaks/nulls
- estimate modal frequencies from room dimensions
- compare measurements by position

Potential useful calculations:

#### Noise floor

Measure ambient room level before test.

Output:

- quiet / moderate / noisy
- relative level
- warning if measurement environment is too noisy

#### Frequency spectrum

Use FFT and aggregate into broad bands:

- sub-bass, if measurable
- bass
- low mids
- mids
- upper mids
- highs

For consumer UX, avoid overloading with technical graphs in the default mode.

#### Decay / reverb estimate

Estimate how quickly energy decays after a clap, impulse, or sweep.

For MVP, call this “reverb estimate” or “decay time estimate,” not necessarily formal RT60 unless the measurement is robust enough.

#### Room modes

From room dimensions, estimate axial modal frequencies:

```text
f = c / 2 * n / d
```

Where:

- c = speed of sound, approximately 343 m/s
- d = room dimension in meters
- n = mode order

Use this to flag likely low-frequency problem areas. Treat as advisory, not definitive.

#### Flutter echo risk

Geometry-based heuristic:

- parallel hard surfaces
- little soft furnishing between them
- short sharp repeated reflections in clap response

Output:

- low / medium / high risk
- affected wall pair
- suggested treatment zone

#### Early reflection risk

Geometry-based heuristic:

- desk/listening/mic position near hard side walls
- window wall close to source/listener
- bare ceiling above position

Output:

- likely first reflection points
- suggested panel/cloud placement

### 7.5 Recommendation engine

A rule-based engine is enough for MVP.

Inputs:

- room geometry
- use case
- scanned surfaces/features
- measured decay
- measured spectrum
- noise floor
- user-marked positions
- budget level, optional
- aesthetic preference, optional
- whether user can drill/mount panels

Example rule structure:

```swift
struct AcousticFinding {
    let type: FindingType
    let severity: Severity
    let location: RoomZone?
    let evidence: [String]
}

struct Recommendation {
    let priority: Int
    let title: String
    let explanation: String
    let location: RoomZone?
    let estimatedImpact: ImpactLevel
    let difficulty: DifficultyLevel
    let costBand: CostBand
}
```

Example rules:

- If high decay + many hard surfaces → recommend broadband absorption.
- If high decay + large window area → recommend heavy curtains or panels near glass.
- If strong low-frequency peak + corner position → recommend bass trapping or repositioning.
- If desk/mic close to wall → recommend moving position or treating behind/side areas.
- If flutter risk between parallel walls → recommend absorption/diffusion on one or both surfaces.
- If ceiling is low and recording position has high reflection risk → recommend acoustic cloud.

## 8. Suggested MVP feature set

### Must-have MVP

- Room scan with dimensions
- User selects use case
- User marks main position
- Guided clap test
- Guided voice sample, optional
- Measurement at 3–5 positions
- Basic frequency analysis
- Basic decay/reverb estimate
- Problem summary
- Visual room map with issue zones
- Prioritized treatment recommendations
- Before/after retest comparison

### Strong MVP

- Sine sweep test
- Pink noise test
- External speaker mode
- 5–9 position guided measurement
- Automatic detection of noisy/invalid test
- Export PDF/report
- Save multiple rooms
- Track before/after treatment iterations

### Later / advanced

- External calibrated mic support
- Calibration workflow
- Octave-band / third-octave-band analysis
- More formal RT60 / EDT estimates
- Speaker placement optimizer
- Stereo symmetry analysis
- AR overlay showing where to place panels
- Shopping-list generator
- Budget-aware treatment planner
- Optional LLM explanation layer

## 9. What the app can reliably claim

Safe claims:

- Helps identify likely acoustic problem areas.
- Estimates reverb/decay and loudness differences.
- Maps acoustic issues to room geometry.
- Provides practical treatment suggestions.
- Helps compare before/after changes.
- Helps users prioritize improvements.

Claims to avoid in MVP:

- Professional certified SPL meter.
- Studio-grade acoustic analysis from iPhone mic alone.
- Guaranteed RT60 precision.
- Guaranteed bass accuracy from phone speaker/mic.
- Replacement for an acoustician.

## 10. Key technical limitations

### Built-in iPhone mic limitations

- Not a calibrated measurement microphone.
- Automatic gain/noise processing can affect results if not controlled.
- Low-frequency accuracy may be limited.
- Device model differences matter.
- Case, hand position, and mic orientation affect readings.

### Built-in iPhone speaker limitations

- Poor low-frequency output.
- Not ideal for full-room response measurement.
- Good enough for rough tests, not enough for pro bass diagnosis.

### LiDAR / RoomPlan limitations

- May miss soft furnishings or classify them roughly.
- May not reliably infer material absorption.
- Glass, mirrors, curtains, clutter, and occlusion can affect scan quality.
- Geometry alone cannot tell full acoustic behavior.

### Human voice/instrument limitations

- Not repeatable enough as a primary test signal.
- Useful for real-world sampling and UX.
- Should supplement, not replace, clap/sweep/noise tests.

## 11. Privacy considerations

The app can be designed to keep sensitive data on-device.

Sensitive data includes:

- room layout
- home scan
- voice recordings
- children/family sounds in background
- location-like context from room layout

Recommended approach:

- Process audio on-device where possible.
- Store extracted acoustic features instead of raw audio by default.
- Let users delete room scans and recordings.
- Make cloud upload optional only if needed.
- If LLM is added later, send structured findings rather than raw audio or full room scan whenever possible.

## 12. Optional LLM layer

The LLM is not required for MVP.

Useful later LLM features:

- Explain findings in plain English.
- Ask clarifying questions about the room use case.
- Generate a prioritized plan.
- Suggest cheaper / renter-friendly alternatives.
- Create a shopping list.
- Explain tradeoffs between foam, panels, curtains, rugs, bass traps, and diffusion.
- Convert technical results into a friendly report.

Important constraint:

The LLM should not diagnose directly from vibes. It should receive structured measurements and produce explanation/recommendations based on those findings.

Example structured input to LLM later:

```json
{
  "room": {
    "dimensions_m": [2.15, 3.10, 2.40],
    "volume_m3": 15.99,
    "features": ["two full-height windows", "hard floor", "desk against window wall"]
  },
  "use_case": "voice recording and video calls",
  "findings": [
    {
      "type": "high_reverb_decay",
      "severity": "medium",
      "zone": "window wall / desk position"
    },
    {
      "type": "early_reflection_risk",
      "severity": "high",
      "zone": "ceiling above desk"
    }
  ]
}
```

## 13. Prototype architecture

Suggested modules:

1. `RoomScanService`
   - Handles RoomPlan/ARKit capture.
   - Produces simplified room geometry.

2. `PositionManager`
   - Lets user mark measurement/use positions.
   - Associates each audio test with a physical zone.

3. `AudioSessionManager`
   - Configures iOS audio session.
   - Starts/stops recording.
   - Handles permissions.

4. `SignalGenerator`
   - Generates sine sweep / pink noise.
   - Plays test signal.

5. `AudioAnalyzer`
   - FFT.
   - Decay estimation.
   - noise floor.
   - band analysis.
   - rough impulse/clap detection.

6. `AcousticModelBuilder`
   - Combines room scan + measurements.
   - Estimates modes/reflection risk.

7. `RecommendationEngine`
   - Rule-based findings and suggestions.

8. `ReportRenderer`
   - Creates user-facing summary.
   - Displays heatmap/room map.
   - Exports PDF later.

## 14. MVP prototype sequence for engineer

### Prototype 1: Audio-only proof of concept

Goal: prove that iPhone can capture useful acoustic differences.

Build:

- record clap test
- detect impulse start
- calculate decay curve
- show rough decay estimate
- FFT spectrum snapshot
- compare two positions

No LiDAR yet.

Success criteria:

- App can distinguish a very echoey room from a treated/soft room.
- App can show before/after difference when adding soft furnishings.

### Prototype 2: Room scan proof of concept

Goal: prove that RoomPlan data can be converted into useful acoustic geometry.

Build:

- scan room
- extract dimensions
- identify walls/openings/windows if available
- display simplified 2D plan
- let user mark recording/listening position

Success criteria:

- App can produce a usable simplified room model.
- User can understand where measurements were taken.

### Prototype 3: Combined room + audio

Goal: map measurements to room zones.

Build:

- scan room
- ask user to test 3–5 positions
- capture clap/sweep measurements
- show issue markers on room plan
- generate basic recommendations

Success criteria:

- App can tell the user where problems are most likely located.
- Recommendations feel practical and not generic.

### Prototype 4: Treatment planner

Goal: make the app useful enough for real users.

Build:

- rule-based prioritization
- before/after retest flow
- simple report
- treatment checklist

Success criteria:

- User can make one or two room changes and see measurable improvement.

## 15. Example rule-based findings

### Finding: Room is too reflective

Evidence:

- long decay after clap
- hard floor/walls
- sparse furniture
- large window/glass area

Recommendation:

- Add broadband absorption near the main recording/listening area.
- Add rug/curtains if floor/window reflections dominate.
- Prioritize panels at first reflection points.

### Finding: Window wall reflection risk

Evidence:

- large glass area near source or mic
- high-frequency decay remains elevated
- user’s desk/mic faces or backs onto glass

Recommendation:

- Add heavy curtains, acoustic curtain, or panels near the window wall.
- Move mic or desk position if possible.

### Finding: Boomy corner / bass buildup

Evidence:

- low-frequency peaks near corners
- room-mode estimate matches measured peak
- user position close to wall/corner

Recommendation:

- Avoid thin foam as primary fix.
- Try thicker bass trapping in the affected corner.
- Move source/listener position away from exact wall/corner.

### Finding: Flutter echo risk

Evidence:

- parallel hard walls
- sparse side-wall furnishing
- clap response has repeated sharp reflections

Recommendation:

- Add absorption/diffusion to one or both parallel surfaces.
- Break up one surface with shelving, curtains, or panels.

### Finding: Ceiling reflection risk

Evidence:

- low ceiling
- desk/recording position below bare hard ceiling
- strong early reflection estimate

Recommendation:

- Add an acoustic cloud above the desk/recording position.

## 16. Suggested UX language

Use accessible language by default.

Instead of:

- RT60
- modal ringing
- early reflection comb filtering
- third-octave decay

Say:

- The room holds onto sound too long.
- This corner is likely making the sound boomy.
- Your voice is bouncing off this wall/window.
- Treat this area first.
- This fix should make speech clearer.

Advanced users can open detailed metrics.

## 17. Potential screens

1. Welcome / use-case selection
2. Room scan
3. Confirm room model
4. Mark key position
5. Guided sound test
6. Position-by-position measurement
7. Results map
8. Priority fixes
9. Before/after retest
10. Export/share report

## 18. Key risks

### Risk: Measurements are too noisy/inconsistent

Mitigation:

- Guided test process.
- Repeat measurements.
- Warn user when background noise is too high.
- Use relative scoring rather than overclaiming precision.

### Risk: User expects professional-grade results

Mitigation:

- Clear product language.
- Use “estimate” and “likely” where appropriate.
- Offer external mic/speaker mode for advanced results.

### Risk: Room scan misses important material properties

Mitigation:

- Ask simple follow-up questions:
  - Is the floor carpet, wood, tile, or concrete?
  - Are there curtains?
  - Are walls mostly bare?
  - Is there a sofa/bed/bookshelf?

### Risk: Recommendations feel generic

Mitigation:

- Tie every recommendation to a room zone and evidence.
- Show “why this matters.”
- Prioritize fixes by impact, cost, and ease.

## 19. Quick prototype brief

Build an iOS prototype that:

1. Records a clap test.
2. Estimates decay/reverb roughly.
3. Runs FFT and shows broad frequency bands.
4. Lets the user test multiple positions.
5. Scans the room using RoomPlan.
6. Places measurements on a simple room map.
7. Produces a rule-based acoustic diagnosis.
8. Recommends 3–5 prioritized fixes.

Do not build LLM integration first.

Do not aim for professional precision first.

The first goal is to prove that a normal user can walk into a room, scan it, perform a guided sound test, and receive advice that is more useful than generic “add foam panels” guidance.

## 20. One-sentence engineering goal

Create an on-device iOS prototype that combines RoomPlan room geometry with guided acoustic measurements to produce a visual, position-aware room soundcheck and practical treatment plan.

