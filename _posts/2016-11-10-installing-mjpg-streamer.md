---
layout: post
title: "Installing mjpg-streamer on raspberry pi"
tags: code linux raspberryPi
description: Sample post showing how code samples would look like
---

Setting up video streaming on the raspberry pi is relatively easy on the raspberry pi 3 there's a little change in the installation instructions.

## Installation

The first thing we want to do is install the packages that mjpg-streamer needs.

{% highlight bash %}
{% raw %}
sudo apt-get update
sudo apt-get install libv4l-dev libjpeg8-dev
sudo apt-get install imagemagick
sudo apt-get install subversion
{% endraw %}
{% endhighlight xml %}

Now we need to actually get mjpg-streamer.

{% highlight bash %}
{% raw %}
svn co https://svn.code.sf.net/p/mjpg-streamer/code/mjpg-streamer
{% endraw %}
{% endhighlight xml %}

Unfortunately, we also need to patch mjpg-streamer to make it work with the pi.

{% highlight bash %}
{% raw %}
cd mjpg-streamer
wget https://raw.githubusercontent.com/penguindustin/ARI_2016_Documentation/development/mjpeg-streamer/input_uvc_patch.txt
patch -p0 < input_uvc_patch.txt
{% endraw %}
{% endhighlight xml %}

After it's patched, let's build it!

{% highlight bash %}
{% raw %}
make USE_LIBV4L2=true
{% endraw %}
{% endhighlight xml %}

To use mjpg-streamer simply run:

{% highlight bash %}
{% raw %}
./mjpg_streamer -i "./input_uvc.so -f 15 -r 640x480" -o "./output_http.so -p 8080 -w ./www"
{% endraw %}
{% endhighlight xml %}

In a web browser, go to `127.0.0.1:8080' (replace 127.0.0.1 with whatever address your pi is if you are connected to it over ssh) and you should see your video.

(picture goes here)

And that's all folks! Just kidding, there are some ways we can improve this which is what we'll do next.

## Some Improvements

One of the first things we would want to do is to make mjpg-streamer more easily accessible. We can do this by moving a few files around.

{% highlight bash %}
{% raw %}
sudo cp mjpg_streamer /usr/local/bin
sudo cp output_http.so input_uvc.so /usr/local/lib/
sudo cp -R www /usr/local/www
{% endraw %}
{% endhighlight xml %}

Now we can run mjpg-streamer using following.

{% highlight bash %}
{% raw %}
mjpg_streamer -i "/usr/local/lib/input_uvc.so -f 30 -r 960x720 " -o "/usr/local/lib/output_http.so -p 8080 -w /usr/local/www"
{% endraw %}
{% endhighlight xml %}

For further laziness we can remove the need to put `/usr/local/lib/` for `input_uvc.so` and `output_http.so` by adding the following to the end of your `~/.bashrc` file.

{% highlight bash %}
{% raw %}
export LD_LIBRARY_PATH=/usr/local/lib/
{% endraw %}
{% endhighlight xml %}

Now, quickly apply the changes.

{% highlight bash %}
{% raw %}
source ~/.bashrc
{% endraw %}
{% endhighlight xml %}

This will let us use the same command but with a lot less typing.

{% highlight bash %}
{% raw %}
mjpg_streamer -i "input_uvc.so -f 30 -r 960x720 " -o "output_http.so -p 8080 -w /usr/local/www"
{% endraw %}
{% endhighlight xml %}

## Tips

### Mutliple camera streaming

You can stream using multiple cameras simply by specifying the camera to stream from. The following will list the cameras available for streaming.

{% highlight bash %}
{% raw %}
ls ~\dev | grep video
{% endraw %}
{% endhighlight xml %}

You should see something like this if you have multiple cameras attached.

{% highlight bash %}
{% raw %}
video0
video1
{% endraw %}
{% endhighlight xml %}

Now if we want to stream multiple cameras we simply add the `-d` flag and specify the camera to stream from (also, make sure to have different ports for each stream).

{% highlight bash %}
{% raw %}
mjpg_streamer -i "input_uvc.so -d /dev/video0 -f 30 -r 960x720 " -o "output_http.so -p 8080 -w /usr/local/www" &
mjpg_streamer -i "input_uvc.so -d /dev/video1 -f 30 -r 960x720 " -o "output_http.so -p 8081 -w /usr/local/www" &
{% endraw %}
{% endhighlight xml %}

To end the streams you can use `pkill`.

{% highlight bash %}
{% raw %}
pkill -f mjpg-streamer
{% endraw %}
{% endhighlight xml %}

### Custom webpage

If we stream video from port 8080 we can display that stream in a webpage using the following:

{% highlight html %}
{% raw %}
<html>
  <head>
    <title>MJPG-Streamer - Stream Example</title>
  </head>
  <body>
    <center>
      <img src="http://192.168.112.149:8080/?action=stream" />
    </center>
  </body>
</html>
{% endraw %}
{% endhighlight xml %}
