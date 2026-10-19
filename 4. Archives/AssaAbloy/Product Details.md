
# Details of Assa-Abloy Project Components

## Hardware
 - Jetson Nano with a 3-channel-relay HAT to control the stacklights.  
 - Invisible-eye sensor looking at a retro-reflector, which detects when we have an obstruction, and sends that signal (via a relay) to the Jetson.  
 - Cameras connected to a local network, on an onboard network switch.
 - Barcode scanner connected to Jetson
 
Almost everything is housed inside a metal box we've affectionately named "Wilson"^[after the basketball in the movie "Cast Away"].  
The stacklight is attached to Wilson externally.  
The cameras and sensor are mounted on the testing structure, and are wired to external ports provided on Wilson.

[Detailed Wiring Diagram](Panel%20wiring%2020Sep22_1530.pptx)

[How to use barcode scanner inside a background script](https://github.com/vpatron/barcode_scanner_python)


## Onboard Software
There is a service script that calls the following scripts:
 - uploadScheduler.py that uploads the logs once every 3 hours
 - resourceLogger.py that logs resource usage data every 5 minutes
 - main.py that is the main application
	 - This is in a while True loop so that whenever it crashes, it restarts again
	 - This depends on the following modules:
		 - camera.py operates the camera
		 - stacklight.py operates the stacklights and the sensor

```mermaid
stateDiagram-v2

	booting : Booting
	state booting {
	direction LR
		conn_ch : Connection Check
		cam_ch : Camera Check
		harv_i : Initialize Harvester
		log : Initialize Logging
		cam_cr : Create Camera Objects
	}
	scan : Awaiting Barcode Scan
	ready1 : Ready to Capture First Corner
	state ready1 {
		wait1 : Awaiting Trigger
	}
	capturing1 : Capturing First Corner
	state capturing1 {
		ia_cr1 : Create Image Acquirers
		start1 : Start Acquisition
		read1 : Read From Buffer
		stop1 : Stop Acquisition
		ia_de1 : Destroy Image Acquirers

		[*] --> ia_cr1
		ia_cr1 --> start1
		start1 --> read1
		read1 --> stop1
		stop1 --> ia_de1
		ia_de1 --> [*]
	}
	ready2: Ready to Capture Second Corner
	state ready2 {
		wait2 : Awaiting Trigger
	}
	capturing2 : Capturing Second Corner
	state capturing2 {
		ia_cr2 : Create Image Acquirers
		start2 : Start Acquisition
		read2 : Read From Buffer
		stop2 : Stop Acquisition
		ia_de2 : Destroy Image Acquirers

		[*] --> ia_cr2
		ia_cr2 --> start2
		start2 --> read2
		read2 --> stop2
		stop2 --> ia_de2
		ia_de2 --> [*]
	}

	[*] --> booting : System Boot
	booting --> scan
	scan --> ready1
	ready1 --> capturing1 : Corner 1 pushed in
	capturing1 --> ready2
	ready2 --> capturing2 : Corner 2 pushed in
	capturing2 --> waiting : API called
	waiting --> ready1 : Result received
```

## Server-side Software
The server receives images and looks for angle or surface defects, and sends the final result back to the Jetson Nano.

Angle detection works as follows:
1. Template matching to identify the corner
2. Cropping the required area
3. Masking out the background
4. Thresholding pixel luminances to determine which pixels belong to the frame
5. Edge detection on the binary thresholded image
6. Line detection on the edge image
7. Filtering out most prominent horizontal and vertical lines
8. Finding angle between them

Defect detection works with a mask R-CNN.

*[HAT]: Hardware Attached On Top