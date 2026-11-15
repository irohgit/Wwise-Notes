# Wwise Notes

My notes from Audiokinetic's Wwise courses.

Focus: understanding how sounds and music are authored in Wwise and integrated with C++ into a custom game engine.

## Progress

- [ ] Wwise Fundamentals (in progress): [Wwise_Fundamentals.md](Wwise_Fundamentals.md)
- [ ] Wwise Interactive Music
- [ ] Wwise Performance Optimization

## Highlights

- **Limit Repetition as a queue**: how Random Containers avoid repeats, and why the maximum setting turns randomness into a fixed cycle.
- **Voice stacking**: making a looping sound's start event safe to fire more than once (Stop, then Play).
- **"Posted on" a game object**: the game object is the sound's address, not the code that sent it, confirmed with the Profiler.
