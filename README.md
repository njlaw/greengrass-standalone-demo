# greengrass-standalone-demo
Demonstrate the connection of IoT devices to a Greengrass Core when the IoT
devices do not have Internet access (i.e., only the Greengrass Core has Internet access).

# Instructions
Feel free to chose your own group and thing names and topics and just replace the references to them.  I've used Amazon
Linux 2 as the OS for all the devices here (Greengrass Core, IoT device 1, and IoT device 2), but you could easily use
any platform that supports Greengrass Core and the AWS IoT Device SDK, by modifying the install instructions.

1. Create a new Greengrass Group called `standalone-demo` in AWS with the default settings (GUI).
2. Setup the Greengrass Core with the provided config files.
3. Add two new IoT devices to the group.  Call one `standalone-demo-iot-device-1` and the other `standalong-demo-iot-device-2`
4. Create two subscription in the group.
   1. Source: `standalone-demo-iot-device-1`  
      Destination: `standalone-demo-iot-device-1`  
      Topic: `my/topic`
   2. Source: `standalone-demo-iot-device-1`  
      Destination: `standalone-demo-iot-device-2`  
      Topic: `my/topic`
5. Deploy the Greengrass Group.
6. Prepare two IoT devices with local network connectivity to your Greengrass Core, but not to the Internet.
   Let's call these `device-1` and `device-2`.
7. Install the Python AWS IoT Device SDK on `device-1` and clone this repo there:
   1. `sudo yum install python3`
   2. `pip3 install awsiotsdk --user`
   3. `git clone https://github.com/njlaw/greengrass-standalone-demo`
8. Install mosquitto on `device-2`:
   1. `sudo yum install mosquitto`
