# Wwise Fundamentals: My Notes

Notes from Audiokinetic's Wwise Fundamentals course, written while learning Wwise as a game programmer.

## Work log

- 07 Nov 2026 : Module 1 complete. Created the project, a SoundBank and an Event, connected to Cube, and debugged a wrong Event name with the Profiler.
- 10 Nov 2026 : Started Module 2.
- 11 Nov 2026 : Modules 2 and 3 complete. Delayed Play actions (one game call, three timed sounds); Random Containers, Limit Repetition, randomised volume and pitch, sequences and silence.
- 13 Nov 2026 : Module 4 complete. Switch Groups and Switch Containers (footsteps by surface).
- 14 Nov 2026 : Modules 5 and 6 Completed. RTPCs and their control points; fixed heartbeat voice stacking with Stop then Play; used the Profiler to find which game object Map_Loaded is posted on.
- 15 Nov 2026 : Module 7 Completed. Introduced to Profiler, Voice Monitor and Voice Profiler as techniques to debug issues. Reviewed and corrected these notes, and added a README.
- 18 Nov 2026 : Module 8 Completed. Introduced to Attenuation. Utilizing distance to affect audio properties, e.g LPF and Volume. Introduced to Cone Attenuation, with angles and cone LPF.
- 20 Nov 2026 : Module 9 Completed. 3D Position Automation Introduction.
  Created a new attentuation object. 
  3D Position Listener with Automation.
  Created randomized pathing for object to follow and randomized the
  selection of those paths.
## Wwise shortcuts

- Jump into the Project Explorer tree:
  1. `Ctrl + F` (focus the Project Explorer)
  2. `Shift + Tab` (cycle back into the selected tab's object tree)
- Audio Tab
  - Random   Container: Ctrl + Shift + Alt + R
    - Looks Like: 3x3 Grid
  - Sequence Container: Ctrl + Shift + Alt + Q
    - Looks Like: Hamburger icon
  - Switch   Container: Ctrl + Shift + Alt + W
    - Looks like Orders of Growth Piecewise Notation {             

## Wwise Tabs
- Audio     : Sounds
- Events    : What the game can call via an agreed event string
- GameSyncs : Switches, States, Game Parameters, Triggers
- Soundbanks: What gets packaged for the game
- ShareSets
  Sessions  : Shares settings (i.e Object_Attentuation),
              Soundcaster and Mixing Desk sessions


# Module 2: Using a single WAV file for multiple applications

- Copying a Sound object creates a new object that points at the same WAV in Originals.
  - The object is independent; only the audio file is shared.
  - Like two pointers to one buffer: a *shallow copy* of the file, a deep copy of the settings.
- Each Sound object has its own Source Editor.
  - Thus trimming or fading in the Source Editor only changes that object.
  - Edits are non-destructive: the original WAV on disk is never modified.
- One game call via an Event can have multiple sounds.
  - Why: an Event is a list of Actions (Play, Stop, Set State...), so one call from the game can trigger several sounds at once.

# Module 3: Creating sonic variety with few resources

- Random Container: a parent object that picks one of its children at random each time it is played.
- Random Containers can remember the last N sounds they played and won't pick any of them next time (Limit Repetition).
  - Example: a cannon shot has 3 primary sounds: A, B and C.
    - Limit Repetition 1: the last sound is blocked, so A, B, A is allowed. The brain notices this pattern quickly.
    - Limit Repetition 2: the last two sounds are blocked.
    - Limit Repetition 3: not possible with 3 sounds. The maximum is (number of sounds - 1) = 2, or nothing would be left to pick.
  - Insight: at the maximum (2 of 3), only one sound is ever eligible, so after the first picks it becomes a fixed cycle (e.g. A, C, B, A, C, B). Maximum repetition-avoidance quietly destroys randomness. More source sounds is the real fix.
- In algorithmic terms, Limit Repetition keeps a queue of the last N sounds the Random Container played.
  - Each pick is random, but only from the sounds not in the queue.
  - After each pick, that sound joins the queue, and the oldest one leaves it.
- TODO: Implement a toy simulation of this in C++.

- Sequence Containers: play their children in a set order (the playlist).
  - Step mode: each time the event fires, play the NEXT item only (e.g. footstep 1, then 2, then 3).
  - Continuous mode: one event plays the whole playlist through, item after item.
  - The items in the playlist are called playlist objects.
  - An item can be a nested group of other sound objects.
    - This can include a Random Container inside a Sequence Container (or the reverse). You can get creative.
- Silence can be used (it's a built-in source plugin).
  - Uses: a pause between items in a Sequence, or a child in a Random Container so that sometimes nothing plays (great for sparse ambience like birds).

# Module 4: Using Switches

- Game Syncs: information the game sends Wwise that changes how sounds behave.
  - Events say WHAT to play; Game Syncs shape HOW it plays.
  - The four kinds:
    - **Switch**: a per-game-object choice (this character's floor surface: Grass, Wood, Metal).
    - **State**: a global mode for the whole game (Exploring, Combat, Paused).
    - **Game Parameter**: a number that varies continuously (Speed, Health).
    - **Trigger**: a one-off cue, used by interactive music (e.g. play a stinger).
- Switch Container: picks which child plays based on the current Switch value on the game object that posted the event.
  - Each footstep checks its own character's surface.
- Associate Switches with Switch Containers.

# Module 5: Game Parameters

- Real Time Parameter Control (RTPC): a curve that maps a Game Parameter (a number from the game) onto a sound property such as volume, pitch or filter.
  - Game Parameter = the input; RTPC = the mapping.
- Outcome: use Wwise's RTPC to control game audio during game runtime.
- What I will learn:
  - Importing .WAV files, looping sounds, changing sounds based on parameter values, auditioning RTPCs in graph view, and Transport Control.
- Wwise definition: Game Parameters communicate a number within a defined range. This numerical range could be any range of numbers.
  - They are floats. In the SDK, `SetRTPCValue` takes an `AkRtpcValue`, which is a float.
  - You define the min and max; values outside the range are clamped.

- Voice Low-pass Filter: removes high frequencies so the sound gets darker and more muffled.
  - The 0 to 100 value is the amount of filtering: 0 = untouched, 100 = heavily muffled.
  - Classic uses: sound behind a wall, underwater, or a low-health "tunnel" effect.

# Module 6: Wwise's States

- States: Wwise's States are generally used to make "sweeping" changes to the game's audio in response to a global change in the game.
- Topics:
  - Creating State Groups
  - Auditioning State Groups in the Transport Control
  - Triggering sounds when levels load
  - Transition Times

## Important: loops started by repeatable events

- Make these loops idempotent. If a looping sound starts silent and only becomes audible through a Game Parameter (an RTPC), its start event MUST be safe to fire more than once.
- Example: Map_Loaded (our game event) starts the heartbeat, which is audible only when player health is low.
  - Potential issue: Map_Loaded fires on every level load, so Play alone would stack heartbeat instances (voice stacking): two, then three heartbeats on top of each other.
  - Fix: clear, then start. The Map_Loaded event first runs Stop on the heartbeat to remove any existing instance, then Play, so exactly one instance is ever playing.

## Important: Wwise technical

- An Event's Actions run in the order listed.
- Thus: list Stop (heartbeat) first and Play (heartbeat) second.
- Insight: if the Stop has a fade-out time, the old and new heartbeat overlap for that fade.

## Very important: C++ vs Wwise objects

- **Object** (e.g. a Sound SFX or Random Container) = the blueprint you build in Wwise. Like a C++ class.
- **Instance** (a voice) = one copy of that object actually playing at runtime. Like an object created from the class.
- **Game object** = an ID the game registers with Wwise to say where sounds come from (an emitter). Sounds are posted ON a game object.

## Stop action scope

- **Game object** (default): stops instances on the game object that posted the event.
- **Global**: stops instances on every game object.
- Use Global if the same event can be posted from different game objects.
- Why not Stop All?
  - Stop All ends every sound in the game (music, ambience, UI).
  - Use the narrowest action that fixes the problem.

## "Posted on" a game object = the address, not the sender

- `PostEvent` names two things: **which event** and **which game object**.
  - `AK::SoundEngine::PostEvent("Map_Loaded", LocalPlayerGameObjectID);`
- **Sender**: whichever code calls `PostEvent` (e.g. the level-loading code). It can live in any class.
- **Address**: the game object passed in. It decides where the sound plays and which values apply.
- So "posted on Local Player" does **not** mean the map loader lives in the player class.
- **Game object**: a registered ID plus name that the game gives Wwise; the sound's audio identity in the world.
  - `AK::SoundEngine::RegisterGameObj(LocalPlayerGameObjectID, "Local Player");`
  - Not the player class itself; usually stored as a member of it.
- **Why post the heartbeat on the player**
  - **Position**: sounds follow their game object in 3D, and a heartbeat should come from the player.
  - **RTPC values can be per game object**: a sound reads an RTPC from its own game object (or the global value). `Player_Health` set on Local Player only reaches sounds on Local Player.
- **How I found this (black-box investigation)**
  - Profiler (`F6`), then Remote: connect to the game, then Start Capture, then reload the map.
  - Capture Log: **Object Name** = which event (the blueprint); **Game Object** = where it was posted.
  - Cube result: `Map_Loaded` is posted on **Local Player**, the same game object every load, so Stop with the default (game object) scope is enough.
  - To check RTPC values per game object: the **Game Object Profiler**.

# Module 7: Profiling Game Syncs

- Outcome: Learn several techniques to debug issues.
- What I will learn: 
  - Rearranging Wwise Layout
  - Using Voice Profiler to observe property values
  - Using Profiler layout to monitor game calls during gameplay
- Reset Factory Layouts in Layouts top tab
- Voice Profiler : F11 
  - Voice Profiler Layout does not include Transport Control
- Gear Button: Alt+G
- Start Capture Alt+C
- Voice monitor allows a graphical representation of dB over time.
- Voice Inspector
  - Left Side: Showcases the object, its ID with transport/soundcaster and what bus it routes to.
  - Right Side: Showcases the Drivers which affect the objects filter values.
- Layouts Designer - F5
- Layouts Profiler - F6
- Connect to Remote Platform - Shift + H
  - Start Capture on Connect: Check when you want to profile at start of runtime.

# Module 8: Using 3D Spatialization

- Outcome: Use 3D spatialization to change volume and filtering based on distance and direction.
- What I will learn:
  - Attenuation Curves
  - Cone attenuation allowing sounds to vary based on orientation with respect to the player as the listener.
- Emitter: Anything that makes a sound
- Listener: Usually associated with the main character in a first-person perspective game.
- User Preferences: Shift + U
- Emitters and Listeners are connected to game objects
  - These game objects have Transform Coordinates: X,Y,Z
  - It also has rotation information
- The Transform and Rotation information is used by 3D Spatialization to create attenuation curves which compute things like volume and pan.
- The combination of information and attenuation curves create the illusion of 3D space.
- Attenuation curve ShareSet: A Resuable curve that can be applied to objects.
- Speaker Panning: Default Value = 100 -> 3D Spatialized only. (In-Depth Module 10)
- New Attenuation: Positioning > Attenuation > New 
- Low Pass Filter: Applied to Distance
- Cone attenuation to simulate perception of listener when the direct sound rotated away.
  - Acoustics
    - Models directivity: How a source's sound changes with the direction it faces relative to the listener.
    - Front (inside Inner Angle, on-axis): No change
    - Sides (between Inner and Outer Angle, off-axis): Gradual blend toward the back settings.
    - Back (beyond Outer Angle): Full Max Attenuation plus the cone's filters.
    - Low Pass Filter (Cuts high frequencies): High Frequencies move forward. Low frequencies diffract around the source.
      - Thus, understanding acoustics allows understanding of the following:
        - Listener is behind a source, the lower frequencies (bass) will still reach the listener. The treble (higher frequencies) beams forward and does not diffract (bend) around the source, so it does not reach the listener.
          - Therefore, knowing the physics of sound, we apply:
            - The Low Pass Filter. To cut the higher frequencies if a listener is behind the teleporter.
            - Sound Outcome of Teleporter: Hear it all when facing it. Hear mostly the bass when behind it, and partly at its sides.
  - Link to direct sound and reflections: facing away weakens the DIRECT sound, but REFLECTIONS still
    arrive from the room, so the direct-to-reverberant ratio drops and the source sounds duller and further away.
  - Distance attenuation = how FAR; cone attenuation = which way it FACES;

# Module 9: Using 3D Position Automation
- Outcome: Use 3D Position Automation to place a sound that has
           no game object in 3D space, by moving it along
           designer-drawn paths relative to the listener,
           with a random path chosen each time it plays.
- What I will learn:
  - Technique: 3D Position Automatio
  - Creating predefined paths a sound can move along relative to the listener.
  - Configure a sound for 3D spatialization
  - Create a path
  - Create more paths
- Listener Relative Routing
  - Enabled, 3D Spatialization : Position + Orientation
- Attenuation 
  - Set Attenuation: Object_Attenuation
- 3D Position
  - 3D Position: Listener with Automation
    - 3D Automation - 3D Position Editor
      - Useful for: Birds flying overhead, old building creaks and groans
    - Allows us to define paths the sounds can move along in 3D space
- Additional Control Points
- Configure Timeline: Length Property

