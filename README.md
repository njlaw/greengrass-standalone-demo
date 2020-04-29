# greengrass-standalone-demo
Demonstrate the connection of IoT devices to a Greengrass Core when the IoT
devices do not have Internet access (i.e., only the Greengrass Core has Internet access).

# Caveat
Obviously you don't want to follow this procedure exactly for a production deployment.  The Greengrass Core and IoT
device private keys should be generated locally, preferably in hardware, on those respective devices.

# Demo Setup
![Topology Diagram](https://github.com/njlaw/greengrass-standalone-demo/raw/master/doc/topology.png "Topology")

# Instructions
Feel free to chose your own group and thing names and topics and just replace the references to them.  I've used Amazon
Linux 2 as the OS for all the devices here (Greengrass Core, IoT device 1, and IoT device 2), but you could easily use
any platform that supports Greengrass Core and the AWS IoT Device SDK, by modifying the install instructions.

1. Create a new Greengrass Group called `standalone-demo` in AWS with the default settings (GUI).
2. Setup the Greengrass Core with the provided config files.
   1. `sudo adduser --system ggc_user`  
      `sudo groupadd --system ggc_group`  
      `curl https://raw.githubusercontent.com/tianon/cgroupfs-mount/951c38ee8d802330454bdede20d85ec1c0f8d312/cgroupfs-mount > cgroupfs-mount.sh`  
      `chmod +x cgroupfs-mount.sh`  
      `sudo bash ./cgroupfs-mount.sh`  
      `sudo yum install java-1.8.0-openjdk`
      `sudo ln -s /usr/bin/java /usr/bin/java8`
      `wget https://github.com/aws-samples/aws-greengrass-samples/raw/master/greengrass-dependency-checker-GGCv1.10.x.zip`
      `unzip greengrass-dependency-checker-GGCv1.10.x.zip`
      `cd greengrass-dependency-checker-GGCv1.10.x`
      `sudo ./check_ggc_dependencies | more`
      `cd`
      `wget https://d1onfpft10uf5o.cloudfront.net/greengrass-core/downloads/1.10.1/greengrass-linux-x86-64-1.10.1.tar.gz`
      `sudo tar xzvf greengrass-linux-x86-64-1.10.1.tar.gz -C /`
   2. Upload the config files from step 1.
   3. Finish Greengrass Core setup:  
      `cd /greengrass`  
      `sudo tar xzvf ~/6893d449c9-setup.tar.gz`  
      `cd certs`  
      `sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem`  
      `sudo /greengrass/ggc/core/greengrassd start`
3. Add two new IoT devices to the group using default settings.  Call one `standalone-demo-iot-device-1` and the other `standalong-demo-iot-device-2`.  Save the generated certificates.
4. Create two subscriptions in the group.
   1. Source: `standalone-demo-iot-device-1`  
      Destination: `standalone-demo-iot-device-1`  
      Topic: `my/topic`
   2. Source: `standalone-demo-iot-device-1`  
      Destination: `standalone-demo-iot-device-2`  
      Topic: `my/topic`
5. Deploy the Greengrass Group.
6. Record the local IP of the Greengrass Core device.  You can find this under the Connectivity screen of the Core in the Console GUI.
7. Save the Greengrass Group CA certificate.
   1. Lookup the Greengrass Group ID in the settings page of the group.
   2. `aws greengrass list-group-certificate-authorities --group-id ab358983-29e8-4873-ba57-6eac7a9e16ba`  
      Use the `GroupCertificateAuthorityId` that is output here.
   3. `aws greengrass get-group-certificate-authority --group-id ab358983-29e8-4873-ba57-6eac7a9e16ba --certificate-authority-id 00b6ae9101d596540e107b554333c4b85133f20a90111a555f7d072e9c02211d | jq -r '.PemEncodedCertificate'`
   4. Save this output as `ggc-root.ca.pem`
8. Prepare two IoT devices with local network connectivity to your Greengrass Core, but not to the Internet.
   Let's call these `device-1` and `device-2`.
9. Install the Python AWS IoT Device SDK on `device-1` and clone this repo there:
   1. `sudo yum install python3 git`
   2. `pip3 install awsiotsdk --user`
   3. `git clone https://github.com/njlaw/greengrass-standalone-demo`
10. Install mosquitto on `device-2`:
   1. `sudo amazon-linux-extras install epel`
   2. `sudo yum install mosquitto`
11. Upload the saved certificates (from step 3) to `device-1` and `device-2` respectively.
12. Run `mosquitto_sub` on `device-2`:
```
[ec2-user@device-2 ~]$mosquitto_sub -h 172.23.4.13 -i standalone-demo-iot-device-2 -p 8883 -t my/topic --cert f47ecc169f.cert.pem --key f47ecc169f.private.key -d --cafile ggc-root.ca.pem
Client standalone-demo-iot-device-2 sending CONNECT
Client standalone-demo-iot-device-2 received CONNACK (0)
Client standalone-demo-iot-device-2 sending SUBSCRIBE (Mid: 1, Topic: my/topic, QoS: 0, Options: 0x00)
Client standalone-demo-iot-device-2 received SUBACK
Subscribed (mid: 1): 0
```
13. Run `standalone_pubsub.py` on `device-1`:
```
[ec2-user@device-1 greengrass-standalone-demo]$ python3 standalone_pubsub.py --endpoint 172.23.4.13 --cert ~/4d80d07ab1.cert.pem --key ~/4d80d07ab1.private.key --root-ca ~/ggc-root.ca.pem --client-id standalone-demo-iot-device-1
Connecting to 172.23.4.13 with client ID 'standalone-demo-iot-device-1'...
Connected!
Subscribing to topic 'my/topic'...
Subscribed with QoS.AT_MOST_ONCE
Sending 10 message(s)
Publishing message to topic 'my/topic': Hello World! [1]
Received message from topic 'my/topic': b'Hello World! [1]'
Publishing message to topic 'my/topic': Hello World! [2]
Received message from topic 'my/topic': b'Hello World! [2]'
Publishing message to topic 'my/topic': Hello World! [3]
Received message from topic 'my/topic': b'Hello World! [3]'
...
```
14. Check the output on `device-2` as well:
```
Client standalone-demo-iot-device-2 received PUBLISH (d0, q0, r1, m0, 'my/topic', ... (16 bytes))
Hello World! [1]
Client standalone-demo-iot-device-2 received PUBLISH (d0, q0, r1, m0, 'my/topic', ... (16 bytes))
Hello World! [2]
Client standalone-demo-iot-device-2 received PUBLISH (d0, q0, r1, m0, 'my/topic', ... (16 bytes))
Hello World! [3]
```
