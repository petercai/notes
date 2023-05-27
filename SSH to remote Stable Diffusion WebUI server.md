# SSH to remote Stable Diffusion WebUI server
Save From : [Does anybody know how to access automatic 1111 webui on a remote (outside the LAN) Linux instance? I can setup and start the UI, but cannot access via the local host up/port provided…. I am accessing the instance via CLI & SSH… : r/StableDiffusion](https://www.reddit.com/r/StableDiffusion/comments/10brogh/does_anybody_know_how_to_access_automatic_1111/) 

## Content
I just got this working. I came across this thread, but it didn't help me. Now, having gotten it to work, I'm coming back to this realizing that it has all the right info, but you have to do _everything_ that people have suggested to access automatic1111 running on a workstation via ssh from a different computer, presumably your laptop.

The two main components of the solution are ssh'ing in with the -L setting, and starting automatic1111 with the --listen command line arg. The first step lets you access the webserver that's running on your workstation from your preferred computer, and --listen tells automatic1111 to listen to the port it's running on.

Steps:

FIRST: From your non-workstation computer (probably a laptop), run the command like the one suggested by OldAnimator815: `ssh -L localhost:some_unused_port:workstation_ip:automatic1111_port username@workstation_ip:ssh_port`.

To find what `automatic1111_port` should be, ssh in to your workstation normally and spin her up and see where she's running. Mine was set to 7860 by default. If your laptop-or-whatever isn't using the same port number, make things easy on yourself and set `some_unused_port` to that number, too. Not required at all but why not. To get the `username@workstation_ip:ssh_port` piece, you might find it easiest to set up "properly" (this is a good guide: [https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)) so you can use the host alias in place of that entire usern..._port arg. When ssh is set up properly, you'll have a .ssh folder in your root dir that contains a config file. If only one host is specified in that file, it'll look very similar to this:

Host \[alias you want to use to specify this ssh config, ex: myworkstation\]
  HostName \[workstation ip address\]
  Port 22
  IdentityFile /Users/\[laptop user\]/.ssh/id_rsa
  User \[workstation user\]

If you put all of that together, you'll get something like the command I used to ssh in: `localhost:7860:[workstation ip address]:7860 myworkstation`.

SECOND: Having ssh'ed into your workstation with the above command, navigate to your automatic1111 install and start the program with the `--listen` arg.

This GitHub issue in the automatic1111 repo shows you how to add the `--listen` arg to your config file so you don't have to type it every time: [https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/2628#issuecomment-1279311075](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/2628#issuecomment-1279311075).

THIRD: You've done all the hard work, so now all you have to do is open up your browser on your laptop and type in localhost:7860
## Note