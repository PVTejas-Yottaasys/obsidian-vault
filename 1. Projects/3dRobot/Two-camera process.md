## Reason for using two cameras
We have two different tasks:
- 3D position+orientation estimation, suited for a low resolution 3D camera
- ACCEPT/REJECT classification for parts, suited for a high resolution 2D machine-vision camera
## Loop of events
```mermaid
flowchart
  3dcapture[3D camera takes image]
  3dprocess[3d image is processed]
  partpickpresent[Robot picks up a part and presents it to the 3D camera]
  mvcapture[Machine Vision camera takes image]
  mvprocess[Machine Vision image is processed]
  droprgbinhome[Robot drops part into RED/GREEN bin and moves to home position]
  droprgbinhome --> |???| 3dcapture
  3dcapture --> |App on NVIDIA receives image from camera| 3dprocess
  3dprocess --> |201, 205 commands| partpickpresent
  partpickpresent --> |???| mvcapture
  mvcapture --> |App on NVIDIA receives image from camera| mvprocess
  mvprocess --> |201, 205 commands| droprgbinhome
```
### Open Questions
- Can we wait a few seconds for the image to process before we start the 201 and 205 command?
	- Alternatively, can the 205 response be delayed by a few seconds? When does the request timeout?
- How do we command the cameras to take images? Possible options:
	- Some way to tell the NVIDIA device via a TCP/IP command that we are ready to take an image?
- %%%%