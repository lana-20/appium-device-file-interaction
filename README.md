# Work with Files on the Device

Why Interact with Files Directly?
- [ ] Seed the app with user data
- [ ] Verify internally saved app data
- [ ] Get photos/media onto the device quickly and reliably

Mobile devices are little computers, which means they have their own file systems. Users of the devices don't usually interact directly with the file system, but it can sometimes be convenient to do so when writing automated tests. Let's take a look at the API's Appium mix available for doing this. If users can't interact directly with the device file system, why should I need to do this for an automated test? Really, it's to make my life easier. 

First of all, if my app saves data to the device disk storage as a way of managing its state, I could pre-populate the device with some of this kind of data to put the app into a known state before I begin my test. This might help make setup possible, or just less tedious. 

Secondly, I can essentially do the reverse. Interact with the app from the UI, and then use the Appium file API to have a look at the results of what my app did with those interactions on disk. This can verify that the app is saving its data correctly. 

Finally, the **file API** is a good way to **get photos and other media onto the device**. Especially when working with simulators, which is where the file API is relevant. It can be hard to, for example, take photos. Especially if they need to match something that my test expects later on down the line. 

The ***Appium file API*** is very simple and consists of two basic commands. 

1. The first command is called <code>pushFile</code>, and it is what enables me to take a file on my computer local to the Appium server, and send it to the file system on the device. The first parameter is the path I wish to send the file to on the device. The second parameter is a file object representing the file I want to send. Appium takes care of reading that file on my computer when I run this command. 

        // put a file on the device from the computer disk
        driver.pushFile(“/path/on/device”, new File(“/path/on/disk”));

2. The second command is called <code>pullFile</code>. It takes just one argument, which is the path of the file on the device file system that I want to download. The result of running the command is a byte array in the Appium Java client. With this byte array, I could perform some analysis on the file contents within my script or write the file to disk for later review. 

        // download a file from the device to the test script
        byte[] fileData = driver.pullFile(“/path/on/device”);

There are a few **limitations** to this API to keep in mind when I’m thinking of using it that have to do with where I’m allowed to place files or retrieve files from. 
- [ ] On **iOS**, applications are sandboxed on the file system in their own containers, and I’m not able to access core system files. In other words, I’m limited to working within the app container.
- [ ] On **Android**, I do have full root access, but only if I’m on an **emulator** or if the device has been **rooted**. 

Here is a [Java script sample](https://github.com/lana-20/device-file-interaction/blob/main/file_interaction_appium_api.java) for the Appium file API:

<img width="800" src="https://user-images.githubusercontent.com/70295997/222875857-eac8d06e-39ec-4fb9-b33b-8c2789e8128f.png">





