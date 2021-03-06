# Pulse : a musical example for the Slate
By Pierre Schefler for iskn.

![Pulse screenshot](screenshot.png "Pulse screenshot")

## General information

### Description
In this demo, we use the slate as a 3D sensor to track the pen as a drum stick.  
The user first learns to keep the beat and then is allowed to play drums on top of a song.

### Installation
1. Download openframeworks here: http://openframeworks.cc/download/ (on windows, prefer visual studio to qt creator for OF but both should work fine).
2. Copy the addons you will find in 'Addons' to openframeworks/addons.
3. Copy the 'Pulse' folder inside 'apps/myApps'.
4. Try to start the project corresponding to your IDE and platform (linux : qt creator ; win : visual studio ; os x : xcode).
5. If the project doesn't compile or if the one you need doesn't exist, just create it:
  1. try openframeworks' project generator (don't forget to tick the addons: ofxRing, ofxSplashScreen, ofxVoronoi2D) or create a raw project and include all of OF sources and the project's sources (except iskn API located in Pulse/src/Slate)
  2. add the include folder of the API corresponding to your platform (Pulse/src/Slate/ISKN_API) to the include directories of your project
  3. link to iskn API (Pulse/src/Slate/ISKN_API) and don't forget to copy the shared lib to your PATH or next to the binary (which should be in Pulse/bin)
 
### How to use
Place the Slate in landscape mode with the buttons behind it and run the program.

### Change settings
To change the settings, you can go to 'bin/data/settings.xml' and edit the file.  
However, you can also use the Easy Settings Tool to change the settings with a nice GUI.

### Remarks about changing the song
There are a few things you should take care of when setting up another song:
* It should have a constant tempo (only for level 1)
* You must know the tempo (only for level 1)
* The format should be mp3, aif or wav (refer to the openFrameworks documentation for more information)
* The audio file should start on a "beat", meaning that it shouldn't have a few milliseconds of silence or noise for example, or the score might not be correctly evaluated (only for level 1)
To know if the song is well synchronized, you can open the settings and activate the "tick" option in the debug panel. This will periodically play a sound in level 1 which will allow you to check the phase.

### License
This example is licensed under the Do What The Fuck You Want Public License.

### Troubleshooting
**My project won't compile.**  
Make sure you followed all the steps described in "installation" (downloaded openFrameworks, copied the addons, linked to the api, etc.).  
  
**The example starts but immediately exits.**  
You have to plug the Slate in, or the program will close. If the Slate is indeed connected to your computer, make sure you have the drivers installed (on Windows). Otherwise, check the logs and/or fill in a bug report.  

**I want to try the example but I don't have my Slate.**  
Open the settings and enable the "replace Slate by mouse" option. You will be able to use the software with your mouse replacing the Slate.

**The example starts but it doesn't look like the screenshot and there is no sound.**  
Verify that the 'data' folder located in 'Pulse/bin' is next to your binary. It contains all the resources the program needs to correctly run.

**I don't understand how to navigate inside the example.**  
First make sure the Slate is correctly oriented (landscape mode, buttons on the back). If the problem is the height, you may want to update your Slate with the 3D firmware which allows a wider tracking.

**The program is slow.**  
Maybe your GPU isn't powerful enough. If you're on a laptop, you can try forcing your PC to use the GPU instead of the chipset (on windows: right click -> run with graphics processor).
If it's still too slow, open the settings and disable the background. This will probably fix your problem.

### TODO
* Finish commenting stuff
* Set the OS X project up and make a release version
* Add a tempo detection feature to the settings tool

## How it works

### Slate input and HMI
The user can navigate inside the software using the Slate only (with a pen or any magnet). The magnet's orientation is detected and the program makes a projection of the magnet's position along its orientation 
so that user feels like he has a pointer. Using this pointer, the user can easily interact with any element of the GUI.  
For instance, how do you click a button in 3D ? You can't. Instead, and that's what's being done here, you have to place your pointer on a button and wait long enough for it to be "clicked".  
  
About the Slate, openFrameworks uses an event and listeners system which allows us to send messages back from an object to the class containing it. This was however replaced by a call at each frame to reduce latency
since events usually take quite some time to be delivered.

### Levels and parts
The software is divided into "parts": the menu, level 1, level2.  
Each part is called a "scene", because it always contain the same basic things such as the pointer, buttons, etc. Then, a "level" inherits a "scene", adding a few more things like an intro, an outro, score, etc. 
This simple inheritance structure allows us to easily add or remove a part (especially a level) but also change the currently displayed part.

### Drums
A drum is represented by a rectangular zone sliced by a voronoi algorithm and animated by a shader which makes the drum "bounce" when the pointer hovers it. When the pointer crosses the drum going downwards, a "hit" is fired.
When a hit is fired, the drum plays the sound that is associated with it. In the first level, the hit is used by the level to evaluate the score. In the second level, it is simply ignored because we only want to play the sound.

### Score evaluation in level 1
The length of a "beat" is defined by the tempo written in the settings file. It could be a half note or a quarter note, depending on what you want.  
When the level starts, a timer also starts and keeps track of when are the beats. Hence, when a hit is fired by the "stick", we exactly know the gap between the hit and the previous or next beat. 
The earliest you can hit a beat is half a beat before and the latest is half a beat after. The score function simply is a gaussian centered on the beat and going from -1/2 beat to +1/2 beat.  
Also, a bonus is given if you're (almost) perfectly on the beat and a malus is attributed if you fire multiple hits during one beat (it means you're probably cheating).
