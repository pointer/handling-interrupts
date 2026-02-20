# handling-interrupts
### Steps to Install and Launch MCU8051IDE in Docker on macOS:

1. Install XQuartz:
- Download and install XQuartz from [XQuartz's official page](http://www.xquartz.org/).

2. Configure XQuartz:
- Open XQuartz and go to preferences to allow connections from network clients.

3. Start XQuartz:
- Make sure XQuartz is running.

4. Run the Docker Container:
- Use the following command to launch your Docker container with display forwarding:
   docker run -e DISPLAY=host.docker.internal:0 -it 
               
5. Install MCU8051IDE Inside the Container:
- In the terminal of the Docker container, run:
   apt update
   apt install -y mcu8051ide

6. Launch MCU8051IDE:
- After installation, start the MCU8051IDE by executing:
   mcu8051ide
  
### Using SSH with X11 Forwarding
Alternatively, if you are using SSH to connect to the Docker container, you can enable X11 forwarding:
1. SSH into Docker Container:
- Make sure to do:
   docker exec -it  /bin/bash

2. Run with X11 Forwarding:
- When you SSH, add the `-X` option to enable X11 forwarding.
