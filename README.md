# internal-game-hacking
This is a Visual Studio 2022 C++ DLL project for internal game hacking. It is the result of following this [YouTube tutorial](https://www.youtube.com/watch?v=hlioPJ_uB7M) by Guided hacking. Please read [this article](https://guidedhacking.com/threads/how-to-hack-any-game-tutorial-c-trainer-3-first-internal.12142/) before watching the tutorial to get warm. 

## Dependencies

As stated in the article, some of the project dependencies are:

- [Assault Cube 1.2.0.2](https://github.com/assaultcube/AC/releases/tag/v1.2.0.2) - You must use this specific version for all our tutorials
- [Cheat Engine](https://www.cheatengine.org/) - Always use the latest version of Cheat Engine
- [Visual Studio Community](https://visualstudio.microsoft.com/) - Always use the latest version of Visual Studio Community

Cheat Engine will be used as the Injector to inject the DLL into the game.

## Build for 32-bit Platform

When building the project, make sure it is built for 32-bit platform. It will not work if 64-bit distribution is built.

To make sure the project will create a 32-bit DLL, do the following [as mentioned here](https://stackoverflow.com/a/15717999/5426539):

- Check the Build Configuration (Build -> Configuration Manager), and ensure that your DLL project for 32-bit is selected.
- Check the linker settings, and ensure correct path is given.
- After build, check that .DLL is produced at correct path, and is 32-bit (should be!).

