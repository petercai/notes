# 5 Tasks To Automate With Python
  
![](https://www.kdnuggets.com/wp-content/uploads/5_things_automate_python_01.jpg)
  
Photo by Author  

 

You automate. I automate. We all automate. We automate our finances, our to-do lists, and our social lives. Why then, is there still so much resistance to automating our professional lives? I’ve been a software engineer for over a decade, and I’ve been an automation advocate for just as long. I’ve seen the benefits of automation firsthand and have helped companies adopt it. In this blog post, I’ll share 10 small tasks that you can automate with Python.

Whether you are writing software, writing business logic, or simply taking notes, automation is your friend. The software world has been fighting an “artificial intelligence arms race” with our competitors for a long time now. Even Google is working on autonomous robots. How can we, as developers, compete? By focusing on our own strengths. We can do this by applying the same techniques that we use for product development to software development. We can apply advanced techniques to our problem-solving and then automate collecting information to be used in those solutions. I personally find that the greater the depth of the problem I solve, the easier it is for me to become a master at the solution and to then specialize in the parts of the problem I find most interesting.

   
This is by no means a comprehensive list, nor will it provide the same level of detail for each task. But it should give you a solid starting point. If you’re new to automation, I recommend checking out the [Robot Academy](https://medium.com/robotacademy) archive to learn more.

   
You can turn any file on your Mac into an audiobook with the script below, and listen to it in the background.

First, install the following dependency.

Then create a python file you will be using to execute this task.

```
import sys import mac_say
mac_say.say(["-f", sys.argv[1],  "-v",  "Alex"])
```

Then in the command line just point at a file of your choice, and enjoy

```
python audiobook.py fileofyourchoice.txt
```

   
Checking the weather is usually a quick thing, but there can be a bit of satisfaction, by doing it with a click of a button.

This as well only requires a single dependency.

Once installed just create a file to run with the script below.

After that, you are ready to run or schedule each day the following.

```
python weather.py "Your City"
```

  
![](https://www.kdnuggets.com/wp-content/uploads/5_things_automate_python_02.jpg)
  

   
This one is a bit easier all we need to do is to install the library as below.

```
pip install --user currencyconverter
```

This installation should put `currency_converter` in our `$PATH` so to execute a conversion one just needs to write the following as shown in the example execution.

```
currency_converter 1 USD --to EUR
```

   
In this example, we will just listen for PDFs, images, audio, and video, but this can be expanded quite a bit and should be enough to get you started. I went a little overboard with this one.

```
import sys import os import time from watchdog.observers import  Observer  from watchdog.events import  FileSystemEventHandler folder_to_monitor = sys.argv[1] file_folder_mapping =  {  ".png":  "images",  ".jpg":  "images",  ".jpeg":  "images",  ".gif":  "images",  ".pdf":  "pdfs",  ".mp4":  "videos",  ".mp3":  "audio",  ".zip":  "bundles",  }  class  DownloadedFileHandler(FileSystemEventHandler):  def on_created(self,  event):  if any(event.src_path.endswith(x)  for x in file_folder_mapping): parent = os.path.join( os.path.dirname(os.path.abspath(event.src_path)), file_folder_mapping.get(f".{event.src_path.split('.')[-1]}"),  )  if  not os.path.exists(parent): os.makedirs(parent) os.rename(  event.src_path, os.path.join(parent, os.path.basename(event.src_path))  ) event_handler =  DownloadedFileHandler() observer =  Observer() observer.schedule(event_handler, folder_to_monitor, recursive=True)  print("Monitoring started") observer.start()  try:  while  True: time.sleep(10)  except  KeyboardInterrupt: observer.stop() observer.join()
```

Once you have the file created for this, all you need to do is to run it pointing at your downloads directory to start monitoring it.

```
python downloads-watchdog.py "/your/downloads/folder"
```

   
In the morning usually, you want to do very little until the caffeine hits. This script will get your morning started earlier by opening all of the browser tabs you usually need to open each morning. Save a script file with URLs of your choice as shown in the example below.

```
python -m webbrowser -t "https://www.google.com" python -m webbrowser -t "https://www.dylanroy.com" python -m webbrowser -t "https://www.usesql.com"
```

   
Python is a powerful tool, but the more you learn and practice it, the more efficient and productive you’ll become. It has been my pleasure to share some silly or fun automation tasks with you, and I hope that you found them useful. If you have any questions, feel free to ask.

*   [Mac Text-To-Speech Python Library](https://github.com/andrewp-as-is/mac-say.py)
*   [Python Weather Library](https://github.com/vierofernando/python-weather)
*   [Python Currency Converter](https://github.com/alexprengere/currencyconverter)
*   [Python Watchdog](https://github.com/gorakhargosh/watchdog)
*   [Python Webbrowser](https://docs.python.org/3/library/webbrowser.html)

 

  **[Dylan Roy](https://www.linkedin.com/in/dylan-roy/)** currently works with Dow Jones to deliver innovate products using cutting edge technologies, and entrepreneurial drive. Often leverages big data and cloud technologies to continuously deliver value for customers. Attended the College of Engineering at Iowa State University for his B.S. in Computer Engineering. Subscribe here for even more ([**dylanroy.com**](http://dylanroy.com/))

  [Original](https://medium.com/robotacademy/5-tasks-to-automate-with-python-e7146996f3). Reposted with permission.