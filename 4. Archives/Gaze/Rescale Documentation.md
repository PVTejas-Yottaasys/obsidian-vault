

If the video resolution is not the same as the screen resolution, then screen coordinates do not match video coordinates.  

> "Resolution mismatch is the source of all headaches." - PV Tejas, circa 2022

![Diagram](Freehand%20Drawing.svg "Aspect Ratio Mismatch")

We calculate the aspect ratios of the video and the screen to determine how the screen coordinates map onto the video

```py
def rescale_gaze_coordinates(video_resolution, screen_resolution, coordinates):
  if video_resolution!=screen_resolution:
    offset_w = 0
    offset_h = 0
    vid_w, vid_h = video_resolution
    scr_w, scr_h = screen_resolution
    vid_ar = vid_w/vid_h
    scr_ar = scr_w/scr_h
```

If the video is wider than the screen, then there will be a height offset. And vice versa.  
We calculate the offset and the scaling factor...

```py
    if vid_ar>scr_ar:
      new_w = scr_w
      new_h = scr_w/vid_ar
      scale = vid_w/scr_w
      offset_h = (new_h - scr_h)/2
    else:
      new_h = scr_h
      new_w = new_h*vid_ar
      scale = vid_h/scr_h
      offset_w = (new_w - scr_w)/2
```

... and then run through the point-cloud applying the linear transform.

```py
    for timestamp in coordinates:
      coordinates[timestamp]['x'] = (coordinates[timestamp]['x'] + offset_w)*scale
      coordinates[timestamp]['y'] = (coordinates[timestamp]['y'] + offset_h)*scale
  return coordinates
```

We are, on purpose, not saturating gaze point coordinates to within the video. It is possible that the user is looking at something else.