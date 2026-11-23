# Wwise Notes

My notes from Audiokinetic's Wwise courses.

Focus: understanding how sounds and music are authored in Wwise and integrated with C++ into a custom game engine.

## Progress

- [ ] Wwise Fundamentals (in progress): [Wwise_Fundamentals.md](Wwise_Fundamentals.md)
  - [x] Module 1: First project, SoundBank and Event, debugged with the Profiler (07 Nov 2026)
  - [x] Module 2: Using a single WAV file for multiple applications (11 Nov 2026)
  - [x] Module 3: Creating sonic variety with few resources (11 Nov 2026)
  - [x] Module 4: Using Switches (13 Nov 2026)
  - [x] Module 5: Game Parameters (15 Nov 2026)
  - [x] Module 6: Wwise's States (15 Nov 2026)
  - [x] Module 7: Profiling Game Syncs (15 Nov 2026)
  - [x] Module 8: Using 3D Spatialization (18 Nov 2026)
  - [x] Module 9: 3D Position Automation (20 Nov 2026)
  - [x] Module 10: Speaker Panning (21 Nov 2026)
  - [x] Module 11: Organizing with Actor-Mixers (21 Nov 2026)
  - [x] Module 12: Understanding Master Audio Bug. Lesson Bug 
  - [x] Module 13: Submixing with Additional Audio Busses (22 Nov 2026)
  - [x] Module 14: Working with Effects (22 Nov 2026)
  - [x] Module 15: Using Soundcaster Sessions (22 Nov 2026)
  - [x] Module 16: Configuring a Mixing Desk (23 Nov 2026)
- [ ] Wwise Interactive Music
- [ ] Wwise Performance Optimization

## Highlights

- **Limit Repetition as a queue**: how Random Containers avoid repeats, and why the maximum setting turns randomness into a fixed cycle.
- **Voice stacking**: making a looping sound's start event safe to fire more than once (Stop, then Play).
- **"Posted on" a game object**: the game object is the sound's address, not the code that sent it, confirmed with the Profiler.
