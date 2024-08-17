[![downloads](https://static.pepy.tech/badge/pyvidplayer2)](http://pepy.tech/project/pyvidplayer2)

# pyvidplayer2 (please report all bugs!)
Languages: English | [中文](https://github.com/anrayliu/pyvidplayer2/blob/main/README.cn.md)


Introducing pyvidplayer2, the successor to pyvidplayer. It's better in
pretty much every way, and finally allows an easy and reliable way to play videos in Python.

All the features from the original library have been ported over, with the exception of ```alt_resize()```. Since pyvidplayer2 has a completely revamped foundation, the unreliability of ```set_size()``` has been quashed, and a fallback function is now redundant.

# Features (see examples folder)
- Easy to implement (4 lines of code)
- Fast and reliable
- Stream videos from Youtube
- Adjust playback speed
- Reverse playback
- No audio/video sync issues
- Subtitle support (.srt, .ass, etc)
- Play multiple videos in parallel
- Built in GUI
- Support for Pygame, Pyglet, Tkinter, and PyQT6
- Can play all ffmpeg supported video formats
- Post process effects
- Webcam feed

# Installation
```
pip install pyvidplayer2
```
Note: FFMPEG (just the essentials is fine) must be installed and accessible via the system PATH. Here's an online article on how to do this (windows):
https://phoenixnap.com/kb/ffmpeg-windows.

## Linux
Before running `pip install pyvidplayer2`, you must first install the required development packages.
- Ubuntu/Debian example: `sudo apt-get install build-essential python3-dev portaudio19-dev`
  - The Python and PortAudio development packages prevent missing Python.h and missing portaudio.h errors, respectively.

# Quickstart

Refer to the examples folder for more basic guides, and documentation.md contains more detailed information.

```
import pygame
from pyvidplayer2 import Video


# create video object

vid = Video("video.mp4")

win = pygame.display.set_mode(vid.current_size)
pygame.display.set_caption(vid.name)


while vid.active:
    key = None
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            vid.stop()
        elif event.type == pygame.KEYDOWN:
            key = pygame.key.name(event.key)
    
    if key == "r":
        vid.restart()           #rewind video to beginning
    elif key == "p":
        vid.toggle_pause()      #pause/plays video
    elif key == "m":
        vid.toggle_mute()       #mutes/unmutes video
    elif key == "right":
        vid.seek(15)            #skip 15 seconds in video
    elif key == "left":
        vid.seek(-15)           #rewind 15 seconds in video
    elif key == "up":
        vid.set_volume(1.0)     #max volume
    elif key == "down":
        vid.set_volume(0.0)     #min volume

    # only draw new frames, and only update the screen if something is drawn
    
    if vid.draw(win, (0, 0), force_draw=False):
        pygame.display.update()

    pygame.time.wait(16) # around 60 fps


# close video when done

vid.close()
pygame.quit()

```

# Documentation 

Documentation also available in repository as documentation.md.

# Video(path, chunk_size=10, max_threads=1, max_chunks=1, subs=None, post_process=PostProcessing.none, interp=cv2.INTER_LINEAR, use_pygame_audio=False, reverse=False, no_audio=False, speed=1, youtube=False, max_res=1080)

Main object used to play videos. Videos can be read from disk or streamed from Youtube. The object uses FFMPEG to extract chunks of audio from videos and then feeds it into a Pyaudio stream. Finally, it uses OpenCV to display the appropriate video frames. Videos can only be played simultaneously if they're using Pyaudio (see use_pygame_audio below). YTDLP is required to stream videos from Youtube. This object uses Pygame for graphics. See bottom for other supported libraries.

## Arguments
 - ```path``` - Path to video file. Supports most file types such as mkv, mp4, mov, avi, 3gp, etc.
 - ```chunk_size``` - How much audio is extracted at a time, in seconds. Increasing this value can mean less total extracts, but slower extracts.
 - ```max_threads``` - Maximum number of chunks that can be extracted at any given time. Increasing this value can speed up extract at the expense of cpu usage.
 - ```max_chunks``` - Maximum number of chunks allowed to be extracted and reserved. Increasing this value can help with buffering, but will use more memory.
 - ```subs``` - Pass a Subtitle class here for the video to display subtitles.
 - ```post_process``` - Post processing function that is applied whenever a frame is rendered. This is PostProcessing.none by default, which means no alterations are taking place.
 - ```interp``` - Interpolation technique used when resizing frames. In general, the three main ones are cv2.INTER_LINEAR, which is fast, cv2.INTER_CUBIC, which is slower but produces better results, and cv2.INTER_AREA, which is better for downscaling. There is also cv2.INTER_NEAREST for maximum performance and cv2.INTER_LANCZOS4 for maximum quality. For convenience, entering the interpolation technique as a string will also work. For example, "cubic" will set the interpolation to cv2.INTER_CUBIC automatically.
 - ```use_pygame_audio``` - Specifies whether to use Pyaudio or Pygame to play audio. Pyaudio is almost always the best option, so this is mainly only for those with problems installing Pyaudio. Using pygame audio will not allow videos to be played in parallel. 
 - ```reverse``` - Specifies whether to play the video in reverse. Warning: Doing so will load every video frame into memory, so videos longer than a few minutes can temporarily brick your computer. Subtitles are unaffected by reverse playback.
 - ```no_audio``` - Set this to true if the given video has no audio track. Setting this to true can also be used to disable existing audio tracks.
 - ```speed``` - Float from 0.5 to 10.0 that multiplies the playback speed. Note that speed if for example, speed=2, the video will play twice as fast. However, every video frame will still be processed. Therefore, the frame rate of your program must be at least twice that of the video's frame rate to prevent dropped frames. So for example, for a 24 fps video, the draw method will have to be called at least, but ideally more than 48 times a second to achieve true x2 speed.
 - ```youtube``` - Set this to true to stream a youtube video. Path must be a valid youtube video url. Note that reverse playback and playback speed are not supported for youtube videos. Also note that for longer videos (15+ minutes), it is recommended to increase chunk_size to 300+. Leave max_threads and max_chunks at 1 to prevent stuttering. The python package yt_dlp is required for this feature. It can be installed through pip. Cannot play active livestreams.
 - ```max_res``` - Only used when streaming youtube videos. Sets the highest possible resolution when choosing video quality. 4320p is the highest youtube supports.

## Attributes
 - ```path``` - Same as given argument.
 - ```name``` - Name of file without the directory and extension.
 - ```ext``` - Type of video (mp4, mkv, mov, etc).
 - ```frame``` - Current frame index. Starts from 0.
 - ```frame_rate``` - How many frames are in one second.
 - ```frame_count``` - How many total frames there are.
 - ```frame_delay``` - Time between frames in order to maintain frame rate (in fractions of a second).
 - ```duration``` - Length of video in seconds.
 - ```original_size```
 - ```current_size```
 - ```aspect_ratio``` - Width divided by height of original size.
 - ```chunk_size``` - Same as given argument.
 - ```max_chunks``` - Same as given argument.
 - ```max_threads``` - Same as given argument.
 - ```frame_data``` - Current video frame as a NumPy ndarray.
 - ```frame_surf``` - Current video frame as a Pygame Surface.
 - ```active``` - Whether the video is currently playing. This is unaffected by pausing and resuming.
 - ```buffering``` - Whether the video is waiting for audio to extract.
 - ```paused```
 - ```muted```
 - ```speed``` - Same as given argument.
 - ```subs``` - Same as given argument.
 - ```post_func``` - Same as given argument.
 - ```interp``` - Same as given argument.
 - ```use_pygame_audio``` - Same as given argument.
 - ```reverse``` - Same as given argument.
 - ```no_audio``` - Same as given argument.
 - ```youtube``` - Same as given argument.
 - ```max_res``` - Same as given argument.


## Methods
 - ```play()```
 - ```stop()```
 - ```resize(size)```
 - ```change_resolution(height)``` - Given a height, the video will scale its width while maintaining aspect ratio.
 - ```close()``` - Releases resources. Always recommended to call when done.
 - ```restart() ```
 - ```set_speed(speed)``` - Depreciated. Use the argument speed= when initializing video objects.
 - ```get_speed()```
 - ```set_volume(volume)``` - Adjusts the volume of the video, from 0.0 (min) to 1.0 (max).
 - ```get_volume()```
 - ```get_paused()```
 - ```toggle_pause()``` - Pauses if the video is playing, and resumes if the video is paused.
 - ```pause()```
 - ```resume()```
 - ```toggle_mute()```
 - ```mute()```
 - ```unmute()```
 - ```set_interp(interp)``` - Changes the interpolation technique. Works the same as interp= during class instantiation. 
 - ```set_post_func(func)``` - Changes the post processing function. Also works the same post_func= during class instantiation. 
 - ```get_pos()``` - Returns the current position in seconds.
 - ```seek(time, relative=True)``` - Changes the current position in the video. If relative is true, the given time will be added or subtracted to the current time. Otherwise, the current position will be set to the given time exactly. Time must be given in seconds, and seeking will be accurate to one hundredth of a second. Note that 
 frames and audio within the video will not be updated yet after calling seek.
 - ```seek_frame(index, relative=False)``` - Seeks to a particular frame instead of a timestamp.
 - ```draw(surf, pos, force_draw=True)``` - Draws the current video frame onto the given surface, at the given position. If force_draw is true, a surface will be drawn every time this is called. Otherwise, only new frames will be drawn. This reduces cpu usage, but will cause flickering if anything is drawn under or above the video. This method also returns whether a frame was drawn.
 - ```preview(show_fps=False, max_fps=60)``` - Opens a window and plays the video. This method will hang until the video closes. Max_fps enforces how many times a second the video is updated. If show_fps is True, a counter will be displayed showing the actual number of new frames being rendered every second.

# VideoPlayer(video, rect, interactable=False, loop=False, preview_thumbnails=0)

VideoPlayers are GUI containers for videos. This seeks to mimic standard video players, so clicking it will play/pause playback, and the GUI will only show when the mouse is hovering over it. Only supported for Pygame.

## Arguments
 - ```video``` - Video object to play.
 - ```rect``` - An x, y, width, and height of the VideoPlayer. The topleft corner will be the x, y coordinate.
 - ```interactable``` - Enables the GUI.
 - ```loop``` - Whether the contained video will restart after it finishes. If the queue is not empty, the entire queue will loop, not just the current video.
 - ```preview_thumbnails``` - Number of preview thumbnails loaded and saved in memory. When seeking, a preview window will show the closest loaded frame. The higher this number is, the more frames are loaded, increasing the preview accuracy, but also increasing initial load time and memory usage. Because of this, this value is defaulted to 0, which turns seek previewing off.

## Attributes 
 - ```video``` - Same as given argument.
 - ```frame_rect``` - Same as given argument.
 - ```vid_rect``` - This is the video fitted into the frame_rect while maintaining aspect ratio. Black bars will appear in any unused space.
 - ```interactable``` - Same as given argument.
 - ```loop``` - Same as given argument.
 - ```queue_``` - Videos to play after the current one finishes.
 - ```preview_thumbnails``` - Same as given argument.

## Methods
 - ```zoom_to_fill()``` - Zooms in the video so that the entire frame_rect is filled in, while maintaining aspect ratio.
 - ```zoom_out()``` - Reverts zoom_to_fill()
 - ```toggle_zoom()``` - Switches between zoomed in and zoomed out
 - ```queue(input)``` - Accepts a path to a video or a Video object and adds it to the queue. Passing a path will not load the video until it becomes the active video. Passing a Video object will cause it to silently load its first audio chunk, so changing videos will be as seamless as possible.
 - ```get_queue()``` - Returns list of queued video objects.
 - ```resize(size)```
 - ```move(pos, relative)``` - Moves the VideoPlayer. If relative is true, the given coordinates will be added onto the current coordinates. Otherwise, the current coordinates will be set to the given coordinates.
 - ```update(events, show_ui=None)``` - Allows the VideoPlayer to make calculations. It must be given the returns of pygame.event.get(). The GUI automatically shows up when your mouse hovers over the video player, so show_ui can be used to override that. This method also returns show_ui.
 - ```draw(surface)``` - Draws the VideoPlayer onto the given Surface.
 - ```close()``` - Releases resources. Always recommended to call when done.
 - ```skip()``` - Moves onto the next video in the queue.
 - ```get_video()``` - Returns currently playing video.

# Subtitles(path, colour="white", highlight=(0, 0, 0, 128), font=pygame.font.SysFont("arial", 30), encoding="utf-8-sig", offset=50)

Object used for handling subtitles. Only supported for Pygame. 

## Arguments
 - ```path``` - Path to subtitle file. This can be any file pysubs2 can read, including .srt, .ass, .vtt, and others.
 - ```colour``` - Colour of text.
 - ```highlight``` - Background colour of text. Accepts RGBA, so it can be made completely transparent.
 - ```font``` - Pygame Font or SysFont object used to render Surfaces. This includes the size of the text.
 - ```encoding``` - Encoding used to open the srt file.
 - ```offset``` - The higher this number is, the close the subtitle is to the top of the screen.

## Attributes
 - ```path``` - Same as given argument.
 - ```encoding``` - Same as given argument.
 - ```start``` - Starting timestamp of current subtitle.
 - ```end``` - Ending timestamp of current subtitle.
 - ```text``` - Current subtitle text.
 - ```surf``` - Current text in a Pygame Surface.
 - ```colour``` - Same as given argument.
 - ```highlight``` - Same as given argument.
 - ```font``` - Same as given argument.
 - ```offset``` - Same as given argument.

## Methods 
 - ```set_font(font)```
 - ```get_font()```

# Webcam(post_process=PostProcessing.none, interp=cv2.INTER_LINEAR, fps=30, cam_id=0)

Object used for displaying a webcam feed. Only supported for Pygame.

## Arguments
 - ```post_process``` - Post processing function that is applied whenever a frame is rendered. This is PostProcessing.none by default, which means no alterations are taking place.
 - ```interp``` - Interpolation technique used when resizing frames. In general, the three main ones are cv2.INTER_LINEAR, which is balanced, cv2.INTER_CUBIC, which is slower but produces better results, and cv2.INTER_AREA, which is better for downscaling. There is also cv2.INTER_NEAREST for maximum performance and cv2.INTER_LANCZOS4 for maximum quality.
 - ```fps``` - Maximum number of frames captured from the webcam per second.
 - ```cam_id``` - Specifies which webcam to use if there are more than one. 0 means the first, 1 means the second, etc.

## Attributes
 - ```post_process``` - Same as given argument.
 - ```interp``` - Same as given argument.
 - ```fps``` - Same as given argument.
 - ```original_size```
 - ```current_size```
 - ```aspect_ratio``` - Width divided by height.
 - ```active``` - Whether the webcam is currently playing.
 - ```frame_data``` - Current video frame as a NumPy ndarray.
 - ```frame_surf``` - Current video frame as a Pygame Surface.
 - ```cam_id``` - Same as given argument.

## Methods
 - ```play()```
 - ```stop()```
 - ```resize(size)```
 - ```change_resolution(height)``` - Given a height, the video will scale its width while maintaining aspect ratio.
 - ```close()``` - Releases resources. Always recommended to call when done.
 - ```get_pos()``` - Returns how long the webcam has been active. Is not reset if webcam is stopped.
 - ```draw(surf, pos, force_draw=True)``` - Draws the current video frame onto the given surface, at the given position. If force_draw is true, a surface will be drawn every time this is called. Otherwise, only new frames will be drawn. This reduces cpu usage, but will cause flickering if anything is drawn under or above the video. This method also returns whether a frame was drawn.
 - ```preview()``` - Opens a window and plays the webcam. This method will hang until the window is closed. Videos are played at whatever fps the webcam object is set to.

# PostProcessing
Used to apply various filters to video playback. Mostly for fun. Works across all graphics libraries.
 - ```none``` - Default. Nothing happens.
 - ```blur``` - Slightly blurs frames.
 - ```sharpen``` - An okay-looking sharpen. Looks pretty bad for small resolutions.
 - ```greyscale``` - Removes colour from frame.
 - ```noise``` - Adds a static-like filter. Very resource intensive.
 - ```letterbox``` - Adds black bars above and below the frame to look more cinematic.
 - ```cel_shading``` - Thickens borders for a comic book style filter.

# Supported Graphics Libraries
 - Pygame (```Video```) <- default and best supported
 - Tkinter (```VideoTkinter```)
 - Pyglet (```VideoPyglet```)
 - PyQT6 (```VideoPyQT```)

To use other libraries instead of Pygame, use their respective video object. Each preview method will use their respective graphics API to create a window and draw frames. See the examples folder for details. Note that Subtitles, Webcam, and VideoPlayer only work with Pygame installed.

# Misc

```
print(pyvidplayer2.get_version_info())
```

Returns a dictionary with the version of pyvidplayer2, FFMPEG, and Pygame. Version can also be accessed directly
with ```pyvidplayer2.VERSION```.

When there are no suitable exceptions, ```pyvidplayer2.Pyvidplayer2Error``` may be raised.