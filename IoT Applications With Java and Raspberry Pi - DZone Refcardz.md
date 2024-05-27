# IoT Applications With Java and Raspberry Pi - DZone Refcardz
                              IoT Applications With Java and Raspberry Pi - DZone Refcardz  

 [![](https://dz2cdn1.dzone.com/themes/dz20/images/dz_logo_2021_cropped.png)](/) 

Thanks for visiting DZone today,

  

[](#)[Edit Profile](#)

*   [Manage Email Subscriptions](#)
*   [How to Post to DZone](/articles/how-to-submit-a-post-to-dzone?utm_source=DZone&utm_medium=user_dropdown&utm_campaign=how_to_post)
*   [Article Submission Guidelines](/articles/dzones-article-submission-guidelines)

[Sign Out](/users/logout.html) [View Profile](#)

Post

*   ![](https://dzone.com/themes/dz20/images/dz-postarticle.svg)
     [Post an Article](/content/article/post.html)
*   [Manage My Drafts](#)

Over 2 million developers have joined DZone.

[Log In](/users/login.html) / [Join](/static/registration.html)

[](/users/login.html)

 Search 

Please enter at least three characters to search

No search results

0

[View All Results](/search)

[Refcards](/refcardz) [Trend Reports](/trendreports)

[Events](/events) [Video Library](/events/video-library)

[Refcards](/refcardz)

[Trend Reports](/trendreports)

Events

[View Events](/events) [Video Library](/events/video-library)

Zones

[Culture and Methodologies](/culture-and-methodologies) [Agile](/agile) [Career Development](/career-development) [Methodologies](/methodologies) [Team Management](/team-management)

[Data Engineering](/data-engineering) [AI/ML](/ai-ml) [Big Data](/big-data) [Data](/data) [Databases](/databases) [IoT](/iot)

[Software Design and Architecture](/software-design-and-architecture) [Cloud Architecture](/cloud-architecture) [Containers](/containers) [Integration](/integration) [Microservices](/microservices) [Performance](/performance) [Security](/security)

[Coding](/coding) [Frameworks](/frameworks) [Java](/java) [JavaScript](/javascript) [Languages](/languages) [Tools](/tools)

[Testing, Deployment, and Maintenance](/testing-deployment-and-maintenance) [Deployment](/deployment) [DevOps and CI/CD](/devops-and-cicd) [Maintenance](/maintenance) [Monitoring and Observability](/monitoring-and-observability) [Testing, Tools, and Frameworks](/testing-tools-and-frameworks)

[Culture and Methodologies](/culture-and-methodologies)

[Agile](/agile) [Career Development](/career-development) [Methodologies](/methodologies) [Team Management](/team-management)

[Data Engineering](/data-engineering)

[AI/ML](/ai-ml) [Big Data](/big-data) [Data](/data) [Databases](/databases) [IoT](/iot)

[Software Design and Architecture](/software-design-and-architecture)

[Cloud Architecture](/cloud-architecture) [Containers](/containers) [Integration](/integration) [Microservices](/microservices) [Performance](/performance) [Security](/security)

[Coding](/coding)

[Frameworks](/frameworks) [Java](/java) [JavaScript](/javascript) [Languages](/languages) [Tools](/tools)

[Testing, Deployment, and Maintenance](/testing-deployment-and-maintenance)

[Deployment](/deployment) [DevOps and CI/CD](/devops-and-cicd) [Maintenance](/maintenance) [Monitoring and Observability](/monitoring-and-observability) [Testing, Tools, and Frameworks](/testing-tools-and-frameworks)

**Spark + Kafka for Real-Time Machine Learning:** Join us May 29 to learn more about real-time ML and how to solve challenges data teams face.

[Register today!](https://events.dzone.com/dzone/Apache-Spark-and-Apache-Kafka-for-Real-Time-Machine-Learning?utm_bmcr_source=announcementbar)

**Data Engineering:** Work with DBs? Build data pipelines? Or maybe you're exploring AI-driven data capabilities? We want to hear _your_ insights.

[Join Our Research](https://survey.alchemer.com/s3/7847723/dzann)

**Modern API Management**: Dive into APIs’ growing influence across domains, prevalent paradigms, microservices, the role AI plays, and more.

[Download the NEW Trend Report](https://dzone.com/link/2024-tr-api-announcement)

**PostgreSQL:** Learn about the open-source RDBMS' advanced capabilities, core components, common commands and functions, and general DBA tasks.

[Read the Refcard](https://dzone.com/link/2024-postgresql-rc-announcement)

1.  [DZone](https://dzone.com)
2.  [Refcards](https://dzone.com/refcardz)
3.  IoT Applications With Java and Raspberry Pi

![](https://dz2cdn1.dzone.com/storage/rc-covers/1385378-refcard-cover229.png)

Refcard #229

IoT Applications With Java and Raspberry Pi
===========================================

Bring Your Internet of Things Vision to Life
--------------------------------------------

Covers hardware setup, Raspbian installation, and general-purpose I/O (GPIO) via two Java libraries. So where will your next project take you – a self-driving car, a portable gaming console, or...?

Download Refcard

Free PDF for Easy Reference

 ![](https://dz2cdn1.dzone.com/storage/rc-covers/1385378-refcard-cover229.png) 

Written By

 [![](https://secure.gravatar.com/avatar/07661a920fe6ebb1a5ee26d951bc3c86?d=identicon&r=PG)](/users/334759/steveonjava.html)  [Stephen Chin](/users/334759/steveonjava.html) 

Lead Java Community Manager, Oracle

Table of Contents

[► Introduction](#section-1) [► Setting Up Your Raspberry Pi](#section-2) [► Accessing GPIO From Java](#section-3) [► Where to Go From Here](#section-4)

Section 1

Introduction
------------

The Raspberry Pi is a powerful and inexpensive embedded computing platform that has great community support. It is powerful enough to run a full Linux operating system, comes with Java SE pre-installed, and has digital input/output ports that you can use to interact with LEDs, buttons, sensors, and motors.

You can use Java on any of the Raspberry Pi models from the original Model B through the latest Model 3. Here is a list of all the different Raspberry Pi models and their capabilities:

| 

Model

 | 

Processor

 | 

Memory

 | 

GPIO

 | 

USB

 |
| :-: | :-- | :-- | :-- | :-- |
| **Raspberry Pi B** | 700Mhz 1 Core | 256/512MB | 28 | 2 |
| **Raspberry Pi A** | 700Mhz 1 Core | 256MB | 28 | 1 |
| **Raspberry Pi B+** | 700Mhz 1 Core | 512MB | 40 | 4 |
| **Raspberry Pi A+** | 700Mhz 1 Core | 256MB | 40 | 1 |
| **Raspberry Pi 2** | 900Mhz 4 Core | 1GB | 40 | 4 |
| **Raspberry Pi Zero** | 700Mhz 1 Core | 512MB | 40 | 1 |
| **Raspberry Pi 3** | 1.2Ghz 4 Core | 1GB | 40 | 4 |

![](https://dzone.com/storage/temp/1387070-figure-1.png)

The Raspberry Pi 2 features a faster processor with exactly the same price point as the original Raspberry Pi B at 35 USD. One of the very latest models, the Raspberry Pi Zero, has an even more compact design than the Model A while keeping all the core features including a host USB port, HDMI display output, and 40 GPIO pins. And all of these boards support running Java right out of the box from the Raspbian distribution that the Raspberry Pi Foundation supports.

Section 2

Setting Up Your Raspberry Pi
----------------------------

To setup your Raspberry Pi for the first time you will need the following hardware:

*   A Raspberry Pi of your choice (The Raspberry Pi 2 and Zero are both great choices)
*   microSD Card – Recommended to get the official Raspberry Pi SD Card that includes the NOOBS installer
*   Power Supply – A Micro USB power supply capable of at least 700mA, but preferably 2A
*   Display or TV – Any HDMI compatible display or TV will work (You can also use composite on all the models except the Zero)
*   Keyboard and Mouse – A USB keyboard and mouse you can use to configure the Pi. It is possible to get through setup without a mouse, which is handy on the A, A+, and Zero models that only have one USB port.
*   HDMI Cable – To connect to your TV/Display. If you are using the Zero make sure you have a cable with a mini jack end or the appropriate adapter to full size HDMI.

![](https://dzone.com/storage/temp/1387217-figure-2.png)

Insert the SD Card with NOOBS on it, hook your Raspberry Pi up to the keyboard/mouse and Display, and then power it on with the Micro USB power supply. Upon doing this you should see a rainbow boot logo followed by the NOOBS installer.

Choose a standard Raspbian distribution, which is a Debian variant optimized for the Raspberry Pi. This also includes Oracle Java as part of the distribution allowing you to start coding with Java right out of the box. Installation takes around 20 minutes, so this is a good time to grab some liquid Java.

After install is complete, press “Enter” to reboot and the Raspbian operating system will load. On first boot you will be automatically logged in and sent to the Raspberry Pi Configuration Tool (raspi-config).

![](https://dzone.com/storage/temp/1387218-figure-3.png)

In the config tool you can change various options, but I would recommend specifically tweaking the following:

*   Password (2) – Default password is “raspberry” for the “pi” user. Make sure to change this if you use the Raspberry Pi on an unprotected network.
*   Internationalization Options (4) – The defaults assume you use a UK keyboard layout. If you are using a US keyboard it is highly recommended to change this or you will have trouble typing special symbols like the # and @ characters.
*   Advanced Options (8) – And in this submenu:
*   Overscan – If you have a modern Display you can turn this off and reclaim your borders
*   Memory Split – If you plan to do any Java UI work (e.g. JavaFX), set this to 128mb or higher
*   SPI – Recommended to enable this so you can use the SPI bus for high speed communication with devices
*   I2C – Recommended to enable this so you can use the I2C bus, which is what most sensors require
*   Serial – Recommended to disable this to free up the serial pins for device communication (this option lets you connect via serial port in to the Raspberry Pi for headless setup, which is no longer required if you have made it this far)

Feel free to browse around and look through the other options, but the ones highlighted above are what I recommend tweaking on your first pass. You can always bring up the tool again by typing “raspi-config” on the command line.

The last bit of setup I recommend is networking your Raspberry Pi so you can access it from your computer via SSH. The easiest way to do this is to plug it in via Ethernet in which case it will automatically grab an IP address via DHCP. The IP address will print on startup, or can be queried using the “ifconfig” command. You can also connect to a wireless network using a USB wifi dongle.

To confirm you have everything working, reboot your Raspberry Pi and login (or better yet connect via SSH over the network). The default username and password are “pi” and “raspberry”. To confirm you have Java working, type “java –version” on the command prompt and it should tell you the exact Java SE Runtime version that you have installed.

Section 3

Accessing GPIO From Java
------------------------

Two libraries are commonly used for accessing the GPIO pins on the Raspberry Pi: Device I/O and Pi4J. The Device I/O library, which is part of the OpenJDK project, provides a standard library for accessing I/O pins across any embedded platform, including the Raspberry Pi. The Pi4J library is an open-source library that is exclusive to the Raspberry Pi. Here is a quick comparison of the two libraries:

|   
 | 

Device I/O

 | 

Pi4J

 |
| :-: | :-- | :-- |
| **Open Source** | Yes | Yes |
| **Portable to Java ME** | Yes | No |
| **API Optimized for Java 8** | Yes | No |
| **Support for New Raspberry Pi Boards** | Lagging | Excellent |
| **Advanced Features** | None | Button Debouncing, Interrupt-based Listening |
| **Extensible Providers for Add-On Boards** | No | Yes |
| **Low-Level GPIO Access** | No | Yes |

### Installing the Device I/O Library

Currently the Device I/O library is not released with the version of Java on the Raspberry Pi. Also, there are no binary builds available, so you have to build the library from source. Fortunately, this is a fairly easy process and can be accomplished directly on the Raspberry Pi.

To start, download the source code from the OpenJDK Mercurial repository by executing the following command from an SSH prompt or the console on your Raspberry Pi:

1

```
sudo apt-get install mercurial
```

Next, clone the Device I/O library source code to your Raspberry Pi. The following command creates a folder called dio in your current working directory with the latest source code:

1

1

```
hg clone http://hg.openjdk.java.net/dio/dev dio
```

To build the source code, you need to run the make command on the newly downloaded code. The build script also requires a few variables to be set up so it knows where to access the Raspberry Pi native libraries and the Java 8 JDK:

4

1

```
cd dio
```

2

```
export PI_TOOLS=/usr
```

3

```
export JAVA_HOME=/usr/lib/jvm/jdk-8-oracle-arm-vfp-hflt
```

4

```
make
```

Next, install it into the local Java Runtime Environment (JRE):

1

1

```
sudo cp -r build/deviceio/lib/\* $JAVA_HOME/jre/lib
```

You will also need the libraries locally for your project. To do this you can use any visual Secure Copy (SCP) or Secure FTP (SFTP) client, such as Cyberduck. However, real geeks will do it from the command line (this works out of the box in OS X or Linux). Unlike the other commands, this one must be run from your desktop or laptop that is connected to your Raspberry Pi:

1

1

```
scp pi@timerpi.local:~/dio/build/deviceio/lib/ext/dio.jar dio.jar
```

Note: Substitute your Pi IP address or hostname for timerpi.local. Also, this command assumes that the dio folder was created in your user home directory on the Raspberry Pi. If not, adjust the path as appropriate.

To use the Device I/O library in your project, create a new Java project called DioLed in NetBeans and add dio.jar as a compile-time library. To access the library settings, choose File | Project Properties (DioLed), click Libraries in the Categories pane, and make sure the Compile tab is selected. Then click Add JAR/Folder and choose the dio.jar file you copied over from the Raspberry Pi.

![](https://dzone.com/storage/temp/1387219-figure-4.png)

### Device I/O Pin Assignments

The Java Device I/O library uses the standard GPIO port assignments as specified by the Motorola Broadcom chipset. The following diagram shows a full list of all the ports, their numbers, and what their functions are for the Raspberry Pi B+, A+, 2, and Zero.

![](https://dzone.com/storage/temp/1387220-figure-5.png)

Even though there are 40 pins total, several of them are not usable for GPIO since they supply power (3.3v/5v) or ground. Most of the remaining pins can be used for GPIO, but may require some configuration before they are accessible. Specifically:

*   Pins 3 and 5 are for ICBy default, the IC bus is disabled and these pins can be used for GPIO. However, these have hard-wired internal 1.8kΩ pull-up resistors to 3.3V.
*   Pins 8 and 10 are for Serial UARTThese pins are reserved for console serial communication by default, which needs to be disabled if you want to use them for GPIO or to connect to a serial device.
*   Pins 19, 21, 23, 24, and 26 are for SPIThis is another communication bus for devices that is disabled by default, so these pins can be used for GPIO.
*   Pins 27 and 28 are for EEPROMThese pins are designed for identifying add-on boards stacked on top of the Raspberry Pi and can’t be used for GPIO.

### Device I/O Library LED Test

To demonstrate how to use the Device I/O library for output, we are going to blink a light-emitting diode (LED). This is a great way to test out your GPIO pins since it is a visible device that can be powered by the current from the pins.

Connecting an LED to the Raspberry Pi GPIO pins just requires a few jumper cables. To prevent the LED from being overloaded, we also need a resistor that we can hook up in serial as shown in the following wiring diagram.

![](https://dzone.com/storage/temp/1387224-figure-6.png)

This is a simple circuit with a 68Ω resistor, an LED, and some wires. Note that LEDs have a positive side and a negative side, usually indicated by the positive lead (the anode) being slightly longer than the negative lead (the cathode). If you hook up the LED backward, it won’t damage it, but it won’t light up either, so make sure that you hook up the longer end to the GPIO pin.

To turn this on and off with the Device I/O library, we need to create three files:

*   java.policy The policy file that grants permissions for us to use GPIO
*   dio.properties Defines the pin setup for our application
*   DioLed.java Our main Java code that will blink the LED

The policy file is really a legacy from the Java ME Device I/O API and is intended for highly restricted environments where you want to limit the access to peripherals, such as cell phones. For applications running on small, embedded systems like the Raspberry Pi, you will most often have control of the entire embedded device and not be competing with other applications installed by end users, so a permissive policy file is fine. The following code is a great boilerplate permissions file to use for all your projects.

9

1

```
// grant all permissions for the DIO framework
```

2

```
grant {
```

3

```
 permission jdk.dio.DeviceMgmtPermission "*:*", "open";
```

4

```
 permission jdk.dio.gpio.GPIOPinPermission "*:*", "open,setdirection";
```

5

```
 permission jdk.dio.gpio.GPIOPortPermission "*:*";
```

6

```
 permission jdk.dio.i2cbus.I2CPermission "*:*";
```

7

```
 permission jdk.dio.spibus.SPIPermission "*:*";
```

8

```
 permission jdk.dio.uart.UARTPermission "*:*";
```

9

```
};
```

The pin configuration file requires a little bit more customization since you will often want to change which pins are used for input/output and possibly add or remove features like serial, I²C, and SPI. You can often take an existing pin configuration file and customize it or pare it down to the features you need. For the simplest cases, it is easy enough to create one from scratch as well. The following code opens a single pin for output.

2

1

```
gpio.GPIOPin = initValue:0, deviceNumber:0, direction:1, mode:4, trigger:3, predefined:true
```

2

```
1 = deviceType: gpio.GPIOPin, pinNumber:18
```

The first line defines the defaults for the gpio.GPIOPin device type.

Possible constants for direction (dir), mode, and trigger in Device I/O property files are as follows:

15

1

```
DIR\_BOTH\_INIT_INPUT = 2;
```

2

```
DIR\_BOTH\_INIT_OUTPUT = 3;
```

3

```
DIR\_INPUT\_ONLY = 0;
```

4

```
DIR\_OUTPUT\_ONLY = 1;
```

5

```
MODE\_INPUT\_PULL_DOWN = 2;
```

6

```
MODE\_INPUT\_PULL_UP = 1;
```

7

```
MODE\_OUTPUT\_OPEN_DRAIN = 8;
```

8

```
MODE\_OUTPUT\_PUSH_PULL = 4;
```

9

```
TRIGGER\_BOTH\_EDGES = 3;
```

10

```
TRIGGER\_BOTH\_LEVELS = 6;
```

11

```
TRIGGER\_FALLING\_EDGE = 1;
```

12

```
TRIGGER\_HIGH\_LEVEL = 4;
```

13

```
TRIGGER\_LOW\_LEVEL = 5;
```

14

```
TRIGGER_NONE = 0;
```

15

```
TRIGGER\_RISING\_EDGE = 2;
```

There is also a bit of ceremony around how you need to configure your NetBeans project to properly deploy the Device I/O configuration files and refer to them from the run command. To make sure your property and permission files are copied to the output directory, you need to add an extra build post-compile step to your NetBeans build.xml file.

5

1

```
<target name="-post-jar">
```

2

```
 <copy todir="${dist.jar.dir}">
```

3

```
 <fileset dir="config" includes="**"/>
```

4

```
 </copy>
```

5

```
</target>
```

The ant target assumes that both your java.policy and dio.properties files are in a folder called config under the project root.

Once these files are configured to copy over to the build folder after compilation, you also need to modify the java run command to make use of them. To do this, open the Project Properties dialog, click Run in the Categories pane, and enter the following VM Options:

1

1

```
-Djdk.dio.registry=dist/dio.properties -Djava.security.policy=dist/java.policy
```

And finally the Java code to turn on and off the LED in a file called DioLed.java.

10

1

```
public class DioLed {
```

2

```
 public static void main(String\[\] args) throws IOException, InterruptedException {
```

3

```
 try (GPIOPin led = DeviceManager.open(1);) {
```

4

```
 for (int i = 0; i < 10; i++) {
```

5

```
 led.setValue(i % 2 == 0);
```

6

```
 Thread.sleep(500);
```

7

```
 }
```

8

```
 }
```

9

```
 }
```

10

```
}
```

Notice that this code is fairly clean and succinct. It uses a try-with-resources block to open the GPIOPin as well as automatically close it at the end. The function for setting the LED value is very straightforward as well. One important thing to note is that the GPIO number referenced in the open method is the ID in our dio.properties file, not the Broadcom GPIO number!

#### Using Pi4J

The Pi4J library is written by Robert Savage and based on the very widely used WiringPi C library. Compared to the Device I/O library, Pi4J has lots of features and is very easy to get started with.

To use Pi4J you just require one JAR file (pi4j-core.jar)—no permissions, no property files, no custom JVM arguments. However, you do require root access for the java process. To run with root from NetBeans, you need to:

*   Choose Tools | Java Platforms.
*   Click your Raspberry Pi platform under the Remote Java SE tree.
*   Click the Exec Prefix field and enter a value of sudo.

#### Pi4J Pin Assignments

By default, Pi4J uses the WiringPi pin assignments. The advantage of this is that they are sequential and in the same location from revision to revision of the Raspberry Pi boards. In practice this only matters between revisions 1 and 2 of the Raspberry Pi B, because the Raspberry Pi Foundation has standardized on the main header pin layout for all subsequent boards, opting to add more pins to the header instead. Here is a diagram showing the pin assignments to use with Pi4J.

![](https://dzone.com/storage/temp/1387225-figure-7.png)

Pi4J also has the option to initialize pin assignments to the Broadcom GPIO pin numbering. However, I don’t recommend changing this, since it will confuse others who are familiar with the Pi4J library and assume certain pin assignments.

#### Pi4J LED Test

To demonstrate the usage of the Pi4J library, we are going to recode exactly the same LED test using the Pi4J library instead. However, the setup of Pi4J is much simpler and the number of changes is minimal, so this will be a breeze!

To start, create a new project called Pi4JLed as a Java project type. Open the Project Properties dialog of your new project and go to the Libraries category. Click Add JAR/Folder, and select the pi4j-core.jar file. If you don’t already have this library, you can download it from the Pi4J Project site here: pi4j.com.

![](https://dzone.com/storage/temp/1387227-figure-8.png)

Now you can start writing the code for triggering the LED. Here is the same example with an LED blinking five times, coded using Pi4J libraries:

11

1

```
public class Pi4JLed {
```

2

```
 public static void main(String\[\] args) {
```

3

```
 GpioController gpio = GpioFactory.getInstance();
```

4

```
 GpioPinDigitalOutput led = gpio.provisionDigitalOutputPin(RaspiPin.GPIO_01);
```

5

```
 for (int i = 0; i < 10; i++) {
```

6

```
 led.setState(i % 2 == 0);
```

7

```
 Gpio.delay(500);
```

8

```
 }
```

9

```
 gpio.shutdown();
```

10

```
 }
```

11

```
}
```

Here are some of the differences that you should take note of:

*   There is no try block, because Pi4J doesn’t support auto-closable for use with try-with-resources. However, it does register shutdown hooks for you, so you can be lazy and let it do the work for you.
*   Pi4J requires initialization before the use of the first pin. Shutdown is optional, but recommended if you don’t want the system to wait for threads to time out, which can take up to 30 seconds.
*   There are no exceptions in the Pi4J version! Pi4J doesn’t throw IOExceptions that we have no idea how to recover from (thankfully!) Also, rather than using Thread.sleep, there are some nice convenience methods for this on the Pi4J GPIO class.
*   The GPIO port is the same. Well, this is not a difference, but it is a very important similarity by coincidence. We are referring to Broadcom GPIO port 18 in both cases. This maps to WiringPi port 1 (per the diagram in the previous section), and we also happen to have slotted it into configuration 1 in the dio.properties file. However, don’t rely on ports being the same in general.

And there you have it. You were able to replicate the same LED blinking code using the Pi4J library and have completed your first Raspberry Pi project using two different GPIO libraries.

Section 4

Where to Go From Here
---------------------

Now that you have learned to code Java on the Raspberry Pi and access the GPIO pins, a wealth of different project possibilities become possible.

Here are just a few different ideas:

### Self Driving Car

Take advantage of an infrared sensor to follow a line, and distance sensor to stop when faced with an obstacle.

### Autonomous Drone

Hook up a quadcopter using a Raspberry Pi as the controller to enable autonomous flight algorithms.

### “Magic” RFID Hat

Read RFID tagged playing cards with an RFID/NFC reader hooked up to the Raspberry Pi.

### Portable Gaming Console

Create a 100% Java-based gaming system using a Raspberry Pi, touchscreen, GPIO buttons, and a 3D printed case.

### And More!

These examples and more can be found in the “Raspberry Pi with Java” title published by McGraw Hill.

### Like This Refcard? Read More From DZone

 [![](https://dz2cdn1.dzone.com/thumbnail?fid=15729283&w=120)](/articles/java-on-raspberry-pi?fromrel=true) [

DZone Article

Java on Raspberry Pi

](/articles/java-on-raspberry-pi?fromrel=true)

 [![](https://dz2cdn1.dzone.com/thumbnail?fid=3617609&w=120)](/articles/installing-ravendb-40-on-your-raspberry-pi-3?fromrel=true) [

DZone Article

Installing RavenDB 4.0 on Your Raspberry Pi 3

](/articles/installing-ravendb-40-on-your-raspberry-pi-3?fromrel=true)

 [![](https://dz2cdn1.dzone.com/thumbnail?fid=17699926&w=120)](/articles/leveraging-ai-and-vector-search-in-azure-cosmos-db?fromrel=true) [

DZone Article

Leveraging AI and Vector Search in Azure Cosmos DB for MongoDB vCore

](/articles/leveraging-ai-and-vector-search-in-azure-cosmos-db?fromrel=true)

 [![](https://dz2cdn1.dzone.com/thumbnail?fid=17698151&w=120)](/articles/apache-doris-for-log-and-time-series-data-analysis?fromrel=true) [

DZone Article

Apache Doris for Log and Time Series Data Analysis

](/articles/apache-doris-for-log-and-time-series-data-analysis?fromrel=true)

 [![](https://dz2cdn1.dzone.com/thumbnail?fid=16179758&w=120)](/refcardz/getting-started-with-mqtt?fromrel=true) [

Free DZone Refcard

MQTT Essentials

](/refcardz/getting-started-with-mqtt?fromrel=true)

 [![](https://dz2cdn1.dzone.com/thumbnail?fid=16062228&w=120)](/refcardz/messaging-infrastructure-for-iot-at-scale?fromrel=true) [

Free DZone Refcard

Messaging and Data Infrastructure for IoT

](/refcardz/messaging-infrastructure-for-iot-at-scale?fromrel=true)

 [![](https://dz2cdn1.dzone.com/thumbnail?fid=15394954&w=120)](/refcardz/data-management-for-industrial-iot?fromrel=true) [

Free DZone Refcard

Data Management for Industrial IoT

](/refcardz/data-management-for-industrial-iot?fromrel=true)

 [![](https://dz2cdn1.dzone.com/thumbnail?fid=12179277&w=120)](/refcardz/edge-computing-2?fromrel=true) [

Free DZone Refcard

Edge Computing

](/refcardz/edge-computing-2?fromrel=true)

ABOUT US

*   [About DZone](/pages/about)
*   [Send feedback](mailto:support@dzone.com)
*   [Community research](/pages/2023-community-research-report)
*   [Sitemap](/sitemap)

ADVERTISE

*   [Advertise with DZone](https://advertise.dzone.com)

CONTRIBUTE ON DZONE

*   [Article Submission Guidelines](/articles/dzones-article-submission-guidelines)
*   [Become a Contributor](/pages/contribute)
*   [Core Program](/pages/core)
*   [Visit the Writers' Zone](/writers-zone)

LEGAL

*   [Terms of Service](https://technologyadvice.com/terms-conditions/)
*   [Privacy Policy](https://technologyadvice.com/privacy-policy/)

CONTACT US

*   3343 Perimeter Hill Drive
*   Suite 100
*   Nashville, TN 37211
*   [support@dzone.com](mailto:support@dzone.com)

Let's be friends: