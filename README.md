# <img width=40 src="https://user-images.githubusercontent.com/70295997/222876027-f80bbd52-08fc-4da2-aa8e-95ee3562f81f.png"> Work with Files on the Device

Why Interact with Files Directly?
- [ ] Seed the app with user data
- [ ] Verify internally saved app data
- [ ] Get photos/media onto the device quickly and reliably

| File Management Use Cases |
| --------  |
| Mobile devices are tiny computers and have their own filesystems. Depending on the device, users don't have direct access to these filesystems. |
| It can be useful to automation to read or write files from the device filesystem. E.g., populate the device camera roll before a test. |
| Verify files written by your device are written correctly. |

Mobile devices are little computers, which means they have their own file systems. Users of the devices don't usually interact directly with the file system, but it can sometimes be convenient to do so when writing automated tests. Let's take a look at the API's Appium mix available for doing this. If users can't interact directly with the device file system, why should I need to do this for an automated test? Really, it's to make my life easier. 

- [ ] First of all, if my app saves data to the device disk storage as a way of managing its state, I could pre-populate the device with some of this kind of data to put the app into a known state before I begin my test. This might help make setup possible, or just less tedious. 

- [ ] Secondly, I can essentially do the reverse. Interact with the app from the UI, and then use the Appium file API to have a look at the results of what my app did with those interactions on disk. This can verify that the app is saving its data correctly. 

- [ ] Finally, the **file API** is a good way to **get photos and other media onto the device**. Especially when working with simulators, which is where the file API is relevant. It can be hard to, for example, take photos. Especially if they need to match something that my test expects later on down the line. 

It'd be nice to manage files on a device from code rather than by tapping buttons. For example, let's say I want to start off my test session with some specific photos in the camera roll, because I want my app to have access to pictures that I know about in advance. Or, maybe the app writes some files to the device filesystem in the course of doing its business, and I want to verify that they are written correctly, or read them for some other purpose. These are the kinds of things I can do using Appium's file APIs. 

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

Here is a ♨ [Java code sample](https://github.com/lana-20/device-file-interaction/blob/main/file_interaction_appium_api.java) for the Appium file API:

<img width="800" src="https://user-images.githubusercontent.com/70295997/222876163-115ce966-ac00-4f9e-9889-877db251eb7e.png">

In 🐍 Python, an analogy would be as follows:


| File Management Commands |  |
| --------  | --------  |
| driver.push_file(path, b64_data) | Send a file defined with b64_data to an Android device or iOS simulator at path |
| b64_data = driver.pull_file(path) | Pull a file from an Android device or iOS simulator |
| b64_zip = driver.pull_folder(path) | Pull a whole folder (zipped) |

**File Permissions**
- On Android, need permissions to write to the path
- On iOS, can only write inside the app *container*

**Base64 Data**
- Binary data cannot be sent as part of a WebDriver HTTP request
- Data must be base64-encoded so that it can be sent as plain text

❶ First, I have the ability to create a file on the device. The command for this is <code>driver.push_file</code>, and it takes 2 parameters. 

- The first is the path on the device where the file should be created. On Android, this should be an absolute path on the device filesystem. Note that a tester needs to have permissions to write to this path. So I can't write to some areas of a physical device, unless it's been rooted. But I can write to the entire emulator filesystem. And on iOS, the security restrictions are even more restrictive. Basically, each app is limited to writing files into its own sandbox, called an app container. So when I specify paths for this command on iOS, all my paths are going to be relative to the app container for my app under test. 

- The second parameter is the file data itself that I want to write. Because files might contain binary data, and because binary data cannot be sent to the Appium server as part of an HTTP request, I need to always be sure to encode the file data as a Base-64 string. I don't need to know anything more about Base-64 encoding other than that it's a way of taking arbitrary binary data and representing it as text which can safely be sent within an HTTP request. There are built-in Python functions that can help me turn any kind of data into a Base-64 encoded string. Of course, given that I need to be sending the file data here, if what I'm really trying to do is move a file from my local filesystem to the device filesystem, I'll first need to read the local file and get its data before encoding it and sending it on to Appium. So that's <code>push_file</code>, used for getting files onto the device.

❷ Now, the reverse would be <code>pull_file</code>, which allows me to retrieve files from a device. It takes a single parameter, namely the path on the device of the file I want to retrieve. Obviously, this file needs to exist and it needs to be accessible. All the things mentioned earlier about paths for <code>push_file</code> apply here as well. The response for this command is actually a Base-64 encoded string. Presumably I will know what kind of file I'm retrieving here, so I will know whether I need to re-encode this as binary data or not. Once I have this data, I can write it to a local file using Python's filesystem operations, or I can work with the raw file data in my Python script, whichever makes more sense. 

- Lastly, let's take a look at the <code>pull_folder</code> command. This command is useful if I know that my app has written a whole directory full of files, and I want to retrieve all of them in one go. In this case, just specify the path of the directory itself instead of a specific file. What I will get back is a zipfile containing all the files inside the directory. Since a zipfile is a binary file, to do anything with it, I will of course need to decode the response from Base-64 encoding to binary, and then write it to disk so that I can unzip it.


