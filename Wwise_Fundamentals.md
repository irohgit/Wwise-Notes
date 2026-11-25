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
- 21 Nov 2026 : Module 10 Completed. Speaker Panning: Grouped ...
  Gem_Recharge_Woosh and Gem_Recharge_Magic under a Staff Recharge ...
  Actor-Mixer, set Speaker Panning to Balance-Fade, and placed it to the
  right of the listener (X: 60, Y: 0) so the staff recharge is heard from
  the right. Note: This was a fixed, single-position exercise. Next, I want to
  learn how to move a sound from L to R based on a changing value, such as
  the staff's actual position in game.
  Module 11 Completed. Organizing with Actor-Mixers.- 22 Nov 2026 : Module 12 Error. Lesson Bug. No teleporter in Voice Explorer.
  Module 13 Completed. Created Environment, SFX and Music Bus. 
  Nested SFX. Overrode Parent (Heartbeat Object). Used schematic view.
  Module 14 Completed. Add Delay Effect to teleport. Create Auxiliary Bus.
  Add RoomVerb Effect to create reverb within Auxiliary Bus for Main Character.
  Went into Secondary editor to edit RoomVerb values. Looked at env ID.
  Module 15 Completed: Create Soundcaster Sessions. Group Objects.
  Simultaneous play.
- 23 Nov 2026 : Module 16 Completed. Created mixing desk. Ducking music
  when signals are received by another bus. Create different mixes using states.
  Module 17: Cannot Complete until MIDI is borrowed.
  Module 18 Completed. Created SoundBank for specific Level. Generated checked.
  Fixed Pathing.
- 24 Nov 2026 : Module 19 Completed. Brought both SoundBanks within memory
  budget. Utilizing 3 techniques, Conversion, Exclusion and Streaming.
- 25 Nov 2026 : 
  Fundamentals course Completed.
  Module 20 Completed: Utilizing multiple techniques to ensure project
  stays within memory and processing budget.
  Virtual Voice Culling: Based on Volume Threshold and Max Voice Instances
  Prioritization based on OffsetPriority, Attenuation.


## Wwise shortcuts

- Jump into the Project Explorer tree:
  1. `Ctrl + F` (focus the Project Explorer)
  2. `Shift + Tab` (cycle back into the selected tab's object tree)
- Project
  - Platform Manager : Shift + Alt + P 
- Audio Tab
  - Random   Container: Ctrl + Shift + Alt + R  Looks Like: 3x3 Grid
  - Sequence Container: Ctrl + Shift + Alt + Q  Looks Like: Hamburger icon
  - Switch   Container: Ctrl + Shift + Alt + W  Looks like {             
  - Auxiliary Bus: Ctrl + Shift + Alt + L
- New Parent
  - Actor-Mixer : Ctrl + Alt + A 
  - Master-Mixer :
    - Audio Bus: Ctrl + Shift + Alt + B
- Layouts:
  - Designer       : F5
  - SoundBank      : F7
  - Mixer          : F8
  - Schematic      : F9
  - Voice Profiler : F11
- View Settings: Ctrl + Alt + H
- Views
  - Soundcaster : Shift + S

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

# Module 10: Using Speaker Panning
- Outcome: Use panning techniques to make wizard recharge action, be heard on the left and right speakers.
- What I will learn:
  - Technique: Speaker Panning
  - Positioning sounds at a fixed point relative to the listener.
- Position: Speaker Panning -> Balance-Fade 
  - Direct Assignment: Source channels of sound played through the front LR.
  - Balance-Fade: Volume is balanced.
    - Sound heard through Front & Rear speakers via Surround Sound Playback ENV.
- Note: I thought this would be a RTPC so that it would fade from L to R
  -TODO: If future tutorials do not teach this. Find out how.

# Module 11: Organizing with Actor-Mixers
- Outcome: Organizing several thousand sounds within a Wwise Project.
- What I Will Learn:
  - Group Objects in Actor-Mixers, Colour-Coding Objects
- Color Coded Actor-Mixers, and Grouped Them

# Module 12: Understanding the Master Audio Bus
- Outcome: Utilize Master-Mixer Hierarchy 
- What will I Learn:
  - Explore the Master Audio Bus
  - Adjust voice volumes
  - Checking which bus an object is routed to
  - Using Voice Profiler to determine the final volume of sounds
- Bus: Where multiple audio signals are mixed together to form a single audio
       signal.
  -Only exist in Master-Mixer Hierarchy
- Master Bus
- Nested objects have their numeric properties, like voice volume, offset by the corresponding of it's parented objects.
  - Voice Volume(-5) will play at a volume (-7) if its a child of an Actor-Mixer that has a Voice Volume of (-2).
- Lesson Bug - Cannot find Teleporter in Voice Explorer

# Module 13: Submixing with Additional Audio Busses
- Outcome: Create several busses, nested and unnested for different game audio.
- What Will I Learn:
  - Nesting Audio Busses
  - Create audio busses: Environment, Music, SFX
  - Assign objects to busses
  - Use the Schematic View
  - Routing objects to busses
  - Using the Multi Editor
  - Overriding parent output bus assignments
  - Using the Schematic View 

# Module 14: Working with Effects
- Outcome: Add a Delay Effect and RoomVerb Effect.
- What Will I Learn
 - Insert an Effect on a Sound SFX
 - Insert an Effect on a bus
 - Creating Auxiliary Busses.
 - Configure Auxiliary Sends
 - Overriding a parent Effect.
- By Default: Sound Objects inherit the Effects configured for its parent.
- Thus to override the default, check the `Override Parent` box.
- Check `Render` <-- Why? Not explained [?]
- Configure Auxiliary Sends
 -  Method One: User-Defind Method
 -  Method Two: Game sends messages to Wwise indicating which auxiliary bus a
    sound should be sent to and at which level. (Ensure no other send conflict)
    More Options: None
    - Game-Defined Auxiliary Sends
      - ID number of enviornment is also sent to Wwise
- Via Master-Mixer Hierarchy: Click secondary Editor's Effect `RoomVerb`
  to edit settings

# Module 15: Using Soundcaster
- Outcome: Use Soundcaster to arrange multiple instances of Transport control.
  These instances will target specific objects so I can play many different
  objects at the same time.
- What will I learn:
 - Create a Soundcaster Session
 - Add objects to a Soundcaster Session
 - Create another Soundcaster Session
 - Adjust some object properties without leaving Soundcaster view
   - Volume, Pitch , LPF
- First : Decide a preset layout of objects I want to have quick access to.
- This preset is the Soundcaster Session.
- Soundcaster sessions are not limited.
- Limits to Soundcaster Session:
  You cannot add bus objects to a Soundcaster Session. They cannot be played
  in the Transport Control.
- Events Tab: SearchField -> player
- Default Behaviour of Soundcaster: Only displays Game Syncs that correspond
  with the objects you've loaded. You can force Game Syncs to appear by 
  selecting the Show All buttons next to each Game Sync header.
- Event Objects: Don't contain properties for Volume, Pitch, and LPF
- Audio Objects: Do have properties thus Vol, Pitch and LPF can be edited.

# Module 16: Configuring a Mixing Desk
- Outcome: Use one view, the Mixing Desk, to group several audio properties
  and fine-tune them. 
- What I Will Learn: Use Mixing Desk alongside the Soundcaster view.
  - Creating Mixing Desk sessions
  - Adding objects to Mixing Desk sessions
  - Ducking Busses
  - Working with States in the Mixing Desk
- Mixing Desk Sessions: Act as presets for objects you want to have mixing
  channels for.
- Drag and Drop Objects into Mixing Desk to reveal vertical mixer strips.
- Ducking: Select Bus > Auto-ducking > Add Bus to Ducking List > Select Bus OK
- Fade In & Fade Out: Determines lenght of time that passes between the moment
  a signal is detected on the bus, and the act of modifying the target bus.
- Target Property Column: Where is the change being applied.
- Testing Mixing Changes
  - Create new Mixing Desk - Bus Masters
  - Put in all busses so that you can see the bus Meter's
  - Play a sound
  - You can see the changes in the desired bus
    - In our example: Music was ducked by making its volume -9
      whenever a signal went through the Environmental bus
    - You can see a literal orange duck
- Work with States
  - Example 1: Wizard Defeated State -> Environmental sounds quieter
    Music unaffected
- Clicking on a header of a bus in the mixing desk
  - Opens itself as a floating window
- Editing States Area: Will display States once you 
  - Double click the Bus you want and go into the `STATES` tab and choose
    the state you want to edit. Then go back into mixer to edit.
- Push States button will reflect values changed in the mixing desk
  to the Soundcaster and Transport Control

# Module 17: Using Control Surfaces
- Note: TODO - Borrow MIDI 
- Outcome: Use an external MIDI device to control adjust multiple 
  parameters.
- What I Will Learn:

# Module 18: Working with Multiple SoundBanks
- Outcome: Make the content of the project a permanent part of the game.
  By creating, populating, and generating SoundBanks.
- What I Will Learn:
  - Create a level-specific SoundBank
  - Add Events to SoundBanks
  - Generate SoundBanks
- Connecting Wwise to the game via Remote Connection will allow changes to be
  reflected immediately.
- Now we will use/generate SoundBanks.
- Manage Memory Efficiently:
  - Sounds that are used in particular levels can be grouped in SoundBanks
    accordingly.
  - An Example:
    - Wwise - Module 18.proj has the Actor-Mixer object `Map Voices`
      The objects within are only heard in the level `dcp_the_core`.
      Thus, the sounds will not load until the level runs and uses the 
      related SoundBank.
- Ensure the correct operating system is selected
  - Above Projected Explorer Left Drop Down: 
- Data Size Column in User-Defined SoundBanks is important to know space cost.
- Event Viewer: Default Layout -> Bottom Left
  Shows all the Events
- SoundBankEditor is in the Secondary Window
- Adding an Event to the Same SounBank will not duplicate the Event.
- `IMPORTANT`!!
  - Generating Checked objects to make a SoundBank will create a
    Generation Log. This provides warnings, messages, parameters etc.
    Use this to debug.
  - Ensure the build path is in the `GAME'S` project folder.
  - SHIFT + K to access project settings and then click SoundBanks Tab
    To change Windows Path to Desired

# Module 19: Managing SoundBank size
- Outcome: Reducing size SoundBanks to ensure Audio meet design spec.
- What I Will Learn:
  - Considering Memory and also SoundBanks
  - Generating Soundbanks
  - Configuring the maximum size of SoundBanks
  - Creating Conversion Settings ShareSets  
  - Applying Conversion Settings
  - Playing converted audio
  - Excluding objects from your SoundBanks
  - Streaming Audio
- SoundDesign:
  - Memory Budget: Expressed in bytes
- SoundBank Generation:
  - Using SoundBank Settings from: Project Settings
  - Using External Sources output paths from: Project Settings
  - Using SoundBank paths from: Project Settings
  - Using Source Control for Generates Files: False
  - External Sources output path: C:${MY_WWISE_19_PROJECTPATH}
  - SoundBanks output path: C:${MY_GAME_PROJECTPATH}
  - Root output path: C:${GAME_PROJECT_SRC_OUTPUT_PATH} 
- SoundBank Manager : Assign a maximum size.
- Data Size is Red = Exceed memory budget.
- Free Space: Negative means how much the Data needs to reduce by. 
- Conversion is an umbrella term for turning original imported audio
  into the version the game acutally ships with.
  Original files are untouched.
  Wwise produces a converted copy for each platform.
- Transport Control: Play Originals vs Play Converted
- Change: VORBIS MP3        ; Permanent Dataloss     ; Lossy Compression
- Change: PCM               ; Non-Permanent Dataloss ; No, uncompressed
- Change: Lower Sample Rate ; Permanent Dataloss     ; Downsampling
- Change: Stereo to Mono    ; Permanent Dataloss     ; Downmixing
- Technique Convert Audio
  - Reduce: Sample Rate, Channels - Downmix to mono, filetype conversion.
  - Usually results in a reduciton in sound quality. 
    - An engineering compromise.
  - `!!!__IMPORTANT__!!!` 
    - In Wwise, conversion does not need to occur during import process.
      It may be delayed until design nears completion, and then doing conversions if necessary and to the extent necessary.
  - Conversion Settings: ShareSet
    Propert Editor > 
                   Conversion >
                              Conversion Settings More Options >
                                                               New
  - Conversion Center
    Upper Pane: Conversion parameters applied to all objects assigned to 
                this shareset.
    Lower Pane: Displays list of currently assigned objects to shareset.
                Details about the audio files e.g channels, sample rate, 
                    original file size.
  - Converting Stereo Channels: To Mono or Lowering Sample Rate ...
        Can reduce file size but will create obvious changes to the audio sound.
- Default Wwise Setting: `!!!___IMPORTANT___!!!` 
      Audio imported into Wwise is typically a PCM file.
      Thus no data compression has been applied.
- Game Engine: Audio System
  - Files that are compressed means the game system must decompress to play it.
  - This adds additional load on system's processor.
  - Cost Benefit is Reduction of Cost of Processer Usage vs Benefit of File
    Size Reduction
- Default Wwise Functionality: 
  - Wwise does not provide MP3 comrpession.
  - Vorbis compression is provided: It is similar
        The industry standard in game audio integration.
        This compression is said to sound better by many.
- In ShareSets Conversion Settings:
     The SFX share set is selected.
     In the Upper Pane, the Format is changed to vorbis.
     Click the correct Plaform and press Convert.
- Technique 2 : Exclude Audio - Don't delete, just exclude from SoundBank.
  - Go into the Actor-Mixer Hierarchy 
    Uncheck the Sound Objects
    They wont be generated.
- Technique 2 : Streaming Audio
  - Load part of audio file into game system's RAM at one time.
  - Direct way to play audio directly from stored memory, SSD, NVME, HDD.
  - Effective for large audio files, such as music that is several minutes long.
  - Cons: Latency as system needs to retrieve from storage.
          Ok for BGM, maybe not for things like projectiles.
          Limits to streaming bandwidth:
              [?]: What is this at at the hardware level? 
                   What lane is the audio streaming competing with
                   The tutorial says video could compete
  - Property Editor >
                    Sound SFX >
                              Stream_Group : Check Enable

# Module 20: Optimizing your sound design
- Outcome: Understand the processing power required for the project's sound
           design. Ensuring platform compliance. Familiarity with demand.
           Adding Constraints. Assigning priorities.
- What I Will Learn:
  - Profiling games
  - Filtering data in Capture Log.
  - Defining a volume threshold per platform.
  - Defining the maximum number of voices per platform
  - Limiting sound instances
  - Defining playback priorities
- Sound and Computer System Architecture
  - Every time a sound is played in game, processing power is required from
    the processor.
  - Every sound played in a game requires something called a voice through which
    the sound is played. `not to be confused with Sound Voice Object` for
    dialogue.
 - Using Profiler:
   -Panes
    - Performance Monitor
      - Description Name & Value
      - CPU - Total: In %
      - Number of Voices (Physical)
      - Number of Voices (Total)
      - Output Peak: Decibels
    - Advanced Profiler: Multiple Tabs
      - Some interesting ones: Busses, Sends, Emitter Listener,
            Memory, Memory Areans, CUP, Obs/Occ(occlusion?)
- Technique 1: Virtal Voice - Distance culling / Similar to light and graphics
  - Idea: Define a volume threshold that sounds must reach before it is able
          to be rendered through a voice.
  - Sounds that become silent due
    - A higher Volume Threshold or 
    - Max Voice Instance reduction
    Is referred to as a virtual voice.
- Platform manager allows independent configuration for game platforms.
    - Default Wwise Setting Volume Threshold: -80db. 
    - Raising this may cut off subtle ambience.
    - Max Voice Instances: Maximum number of `voices` that are allowed
    to play at one time. Lowering this value will mean less processing power.
    - An important consideration is that, having max voice instances may
      cut off voices. Some of these voices may be cut off similar to
      flapjack where my own cannon got cut off.
- Virtual Voicing: Poison Gem Example
  - Actor-Mixer > Default Work Unit > PoisonGemMagic-Mixer
  - Property Editer: Playback Limit Group
    - Limit Sound Instances - Checked
    - Limitation Scope: Globally
      A global setting will mean only 4 can ever occur at once.
      `This was a potential solution to hearing 1000 pirate enemy projectiles`
- `PRIORITIZING SOUNDS`
  - Sounds can be given a priority depending on their importance
    in relation to the player's experience.
  - The sound engine will only cull non-essential sounds.
  - And leave essential sounds like player dialogue or music alone.
  - `!!!__IMPORTANT__!!!` 
    Playback Priority: Priority Parameter MAX value is 100.
  - Distance Based Attenuation and Prioritization:
    - Advanced: Playback Priority Group > Use Offset Priority
    - USE OFFSET PRIORITY : Specifies a value by which the priority 
      of an object is offset when it reaches the `Max distance` value 
      specified in the Attenuation editor.
      - `VERY HELPFUL WHEN`: Footsteps are high priority when close to the
        listener and low priority when they are far away.


  
