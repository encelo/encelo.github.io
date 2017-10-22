---
layout: post
title: nCine Dev Update 1
subtitle: Updates from June to July 2017
tags: [nCine]
---

During June and July 2017 I have been working as usual, in my spare time, on the project. :wink:

The first big June addition has been the automatic *screen culling* of sprites, a very important feature needed in order to support games extending on multiple screens. The culling works on sprites of any kind (regular ones, particles, text nodes) and regardless of their scaling or rotation parameters. If those sprites are completely outside of the screen they will just not be rendered, saving draw calls from being issued. 

The next feature has been the complete support for mouse and keyboard on Android, a task which was made more interesting by some hacks I had to perform. For example, I had to implement a combination of bitmak operations just to support the right mouse button, a trick needed becasue most Android devices (but not my Shield TV) map the right mouse button to the special back button, which is handled as a key. :tired_face:
It means that the `AINPUT_SOURCE_MOUSE` can generate `AINPUT_EVENT_TYPE_KEY`, just as if it was a `AINPUT_SOURCE_KEYBOARD`. In this case if the keycode is `AKEYCODE_BACK` I have to simulate a press from `AMOTION_EVENT_BUTTON_SECONDARY`. :sweat_smile:

Those changes together made possible the creation of a new test example where many different sprites wave randomly across the screen while the user can move the view as if controlling a top-down camera. As usual in my nCine tests the user can use multiple input methods like keyboard, mouse, joystick or the touch screen on Android.

The last change is the full support for SDL2 that I added in July. It was required because the old SDL1 back-end was diverging too much in terms of functionalities when compared with the GLFW one. Now both back-ends support events for the connection and disconnection of a joystick, events for the mouse wheel scrolling (which on SDL1 I could only simulate) and, above all, the possibility to pass parameters during the OpenGL context creation. This last feature will be vital for a future support of newer versions of OpenGL and OpenGL ES.

That's all for now, see you soon in a next installment of the nCine Dev Updates! :wink:
