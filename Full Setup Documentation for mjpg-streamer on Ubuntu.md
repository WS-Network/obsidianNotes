
This guide explains how to:

- Verify your camera is recognized by Ubuntu
- Install necessary dependencies
- Clone and build **mjpg-streamer** from source
- Resolve build issues (including missing headers)
- Run and test the streamer
- (Optional) Configure mjpg-streamer to start automatically on boot using systemd

Each section also includes explanations for the fixes we applied.

---

## 1. Verify Your Camera

Before starting, check that your camera is recognized by Ubuntu. In your terminal, run:

bash

Copy code

`ls /dev/video*`

**Expected output (for example):**

bash

Copy code

`/dev/video0  /dev/video1`

We will use **/dev/video0** as our camera device in this guide.

---

## 2. Install Required Dependencies

To compile and run **mjpg-streamer**, several development tools and libraries are required. We encountered missing files during the build (specifically, `libv4l2.h`), which led us to install an extra package. Run the following commands:

bash

Copy code

`sudo apt-get update sudo apt-get install -y git build-essential cmake libjpeg8-dev imagemagick libv4l-dev`

### Why These Packages?

- **git:** Clones the **mjpg-streamer** source code repository.
- **build-essential & cmake:** Provide the compilers and build system needed to compile the code.
- **libjpeg8-dev:** Provides JPEG encoding/decoding support.
- **imagemagick:** Required for image processing tasks within some plugins.
- **libv4l-dev:**  
    **Fix Reason:** The build process failed with an error stating `fatal error: libv4l2.h: No such file or directory`.  
    **Fix Explanation:** The header `libv4l2.h` is provided by the **libv4l-dev** package. Installing it ensures that the V4L2 (Video for Linux 2) development files are available to compile the UVC plugin.

---

## 3. Clone the mjpg-streamer Repository

Clone the experimental version from GitHub:

bash

Copy code

`git clone https://github.com/jacksonliam/mjpg-streamer.git`

This creates a folder named `mjpg-streamer` in your current directory.

---

## 4. Build mjpg-streamer from Source

Change to the experimental directory and compile:

bash

Copy code

`cd mjpg-streamer/mjpg-streamer-experimental make`

### What Happens During the Build?

- **CMake Configuration:**  
    CMake checks for necessary libraries and sets up the build environment. You may see warnings (for example, regarding CMake version compatibility or missing optional packages) that are generally safe to ignore.
- **Compilation:**  
    The source code is compiled into executables and shared libraries.

### Encountered Issue and Fix:

During the build, the following error was encountered:

yaml

Copy code

`fatal error: libv4l2.h: No such file or directory`

**Fix:** Installing **libv4l-dev** (see Step 2) provided this header file, allowing the build to proceed.

Once the error is resolved, the build completes and you see messages like:

python-repl

Copy code

`[ 13%] Built target mjpg_streamer ...`

**Optional Installation:**  
You can install the built binaries system-wide:

bash

Copy code

`sudo make install`

This copies the executable (typically to `/usr/local/bin`) and the `www` folder (web interface files) to `/usr/local/share/mjpg-streamer/www`.

---

## 5. Run and Test mjpg-streamer

Run mjpg-streamer using the following command:

bash

Copy code

`./mjpg_streamer -i "input_uvc.so -d /dev/video0 -r 640x480 -f 30" -o "output_http.so -p 8080 -w ./www"`

### Explanation of Command Options:

- **Input Plugin (`input_uvc.so`):**
    - `-d /dev/video0`: Specifies the camera device.
    - `-r 640x480`: Sets the resolution to 640×480 pixels.
    - `-f 30`: Sets the frame rate to 30 frames per second.
- **Output Plugin (`output_http.so`):**
    - `-p 8080`: Specifies the TCP port for the HTTP stream.
    - `-w ./www`: Points to the `www` directory containing the web interface files.  
        (If installed system-wide, you might use `-w /usr/local/share/mjpg-streamer/www`.)

### Observing Warnings:

You may see several warnings like:

java

Copy code

`UVCIOC_CTRL_ADD - Error at Pan (relative): Inappropriate ioctl for device (25) UVCIOC_CTRL_MAP - Error at Focus (absolute): Inappropriate ioctl for device (25)`

**Explanation:**  
These messages indicate that the streamer is attempting to configure camera controls (such as pan, tilt, or focus) that your specific camera does not support. These warnings are typical and harmless if your video stream appears and functions correctly.

### Verify the Stream:

1. Open a web browser on another device on the same network.
    
2. Navigate to:
    
    cpp
    
    Copy code
    
    `http://<your_ubuntu_machine_IP>:8080`
    
3. You should see the live video feed.
    

#### SSH Port Forwarding (Optional)

If you are accessing the machine via SSH, you can forward the port locally:

bash

Copy code

`ssh -L 8080:localhost:8080 toor@<ubuntu_machine_IP>`

Then open your browser at `http://localhost:8080`.

---

## 6. (Optional) Configure mjpg-streamer to Start on Boot with systemd

If you wish the streamer to run automatically at boot, create a systemd service.

### Step 6.1: Create the Service File

Open a new service file:

bash

Copy code

`sudo nano /etc/systemd/system/mjpg-streamer.service`

Paste the following configuration:

ini

Copy code

`[Unit] Description=MJPG-Streamer Service for Webcam Streaming After=network.target  [Service] ExecStart=/usr/local/bin/mjpg_streamer -i "input_uvc.so -d /dev/video0 -r 640x480 -f 30" -o "output_http.so -p 8080 -w /usr/local/share/mjpg-streamer/www" Restart=always User=toor Group=toor Environment=PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin  [Install] WantedBy=multi-user.target`

### Why These Settings?

- **ExecStart:**  
    Specifies the full command to launch mjpg-streamer. Adjust the path if you did not install system-wide.
- **User/Group:**  
    Runs the service as the user `toor`. Change this if necessary.
- **Restart=always:**  
    Ensures that the service is restarted if it crashes.
- **After=network.target:**  
    Makes sure the service starts after the network is up.

### Step 6.2: Enable and Start the Service

Reload systemd and enable the service:

bash

Copy code

`sudo systemctl daemon-reload sudo systemctl enable mjpg-streamer.service sudo systemctl start mjpg-streamer.service`

Check the service status:

bash

Copy code

`sudo systemctl status mjpg-streamer.service`

The status should indicate that the service is active and running.

---

## 7. Troubleshooting and Fix Summary

- **Missing `libv4l2.h`:**  
    **Issue:** The build failed with a missing header file.  
    **Fix:** Installed **libv4l-dev** to provide the required header.
    
- **Camera Control Warnings:**  
    **Issue:** Warnings about unsupported camera controls (e.g., pan, tilt, focus) appeared during runtime.  
    **Explanation:** These warnings occur because the camera does not support the requested controls. They can be safely ignored if the video stream functions properly.
    
- **Path and Plugin Considerations:**  
    **Issue:** Depending on your installation method, the path to the `www` directory or the `mjpg_streamer` binary might differ.  
    **Fix:** Adjust the command-line options or systemd service file accordingly.
    
- **SSH Port Forwarding:**  
    **Issue:** Accessing the stream remotely when logged in via SSH.  
    **Fix:** Use SSH port forwarding to map the remote port 8080 to your local machine.
    

---

# Summary

4. **Camera Verification:**  
    Confirmed that the camera is detected (e.g., `/dev/video0`).
    
5. **Dependency Installation:**  
    Installed essential packages along with **libv4l-dev** to fix missing header issues.
    
6. **Source Cloning and Building:**  
    Cloned the **mjpg-streamer** repository and built the project. Fixed build errors by ensuring all required libraries and headers were present.
    
7. **Testing the Stream:**  
    Ran mjpg-streamer and noted harmless warnings regarding unsupported controls while confirming the stream worked.
    
8. **Automating Startup with systemd:**  
    Created a systemd service file to run the streamer on boot.