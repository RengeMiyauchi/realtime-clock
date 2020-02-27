<img height="130" src="https://maki.cat/vrchat/realtime_clock_github.png"/>

> 🕒 Show the current time where you live on an analogue clock in VRChat.

## Update 1.04.2019. google app engine fork and new ip geoloc provider

This is a fork of the Time in image shader and web server by Maki and Desunyan.
I've made changes to allow the web application to be run in google app engine.
I've also changed the IP Geolocation service provider to http://ip-api.com/ 

~~If you just want to use it out of the box, i've configured a public server that you can use to get the current time.~~
~~Just replace the url in the vrc_panorama component with~~
~~https://public-maki-time-server.appspot.com/clock~~
no longer providing public end points as its incuring me too much costs

Do note that the ip-api service has a rate limit of 150 requests per min, and as this is a public end point, it is not guaranteed that everyone who visits your vrchat map will get their time depending on traffic and what not. The server is hard coded to serve a maximum of 145 requests per minute. Ip-Api will blacklist your servers ip if you exceed 150 requests per minute so be careful with this.

If you want to set up your own google app engine app.

Sign up for an account at https://console.cloud.google.com/getting-started

Create your own project.

Go to google app engine, start a cloud shell.

```sh
git clone https://github.com/RengeMiyauchi/realtime-clock.git
cd realtime-clock/server
gcloud app deploy app.yaml
```

once done you should see a message
```sh
Deployed service [default] to [https://yourprojectname.appspot.com]
```
where the name "yourprojectname" will be whatever your project name is
add /clock to the end of the url and thats the url you need to use for your vrc_panorama

If you want to see how well this detects your local time in vrchat, I've had this running in the map "Bedroom Theatre" for a number of months now. Do note that the Ip-Api sometimes has service outages on a free account during high traffic periods. From my experience around 96% of requests are successfully served. 

IP Geolocation service was switched to Ip-Api as it was originally just scraping off iplocation.net, which some time ago started adding captcha's to deter automated requests such as this app.

## Update 12.12.2018. IMPORTANT NOTICE!!!

Official time server https://maki.cat/time-in-image is shutting down soon. If you still want to use this clock in your worlds you have to host your own time server. Sorry for inconvinience.

## [Download .UnityPackage here!](https://github.com/Shii2/realtime-clock/releases)

It uses https://maki.cat for finding the time with your IP address, therefore it can sometimes return an inaccurate location, which can cause the time to be offsetted by an hour or two. And fyi, I don't track anything from my HTTP GET requests... That would be mean...

You can change the background and foreground colours in the material and change the numbers texture aswell. I kept the .ai file so you can change fonts easily. The mesh uses vertex colors to index each time handle so keep that in mind if you want to edit it!

## How is time served?

It fetches time from https://maki.cat/time-in-image

This is done by looking up your time from your IP using https://www.iplocation.net, `tz-lookup` and `moment-timezone`, then converts hours (h), minutes (m), seconds (s): 24,60,60 into 255,255,255.

It then generates an 8x8 .png file where it has 4 sections of colour assembled like this: (each section shows r,g,b values)

<table>
	<tr><td>h,m,s</td><td>s,h,m</td></tr>
	<tr><td>m,s,h</td><td>0,0,0</td></tr>
</table>

The shader then looks up each value from the three sections and averages them to get an accurate reading of the current time. The reason why we split them up between red, green and blue is because we were having issues with sRGB and gamma changing them. Though by using `VRC_Panorama`, this might be completely unnecessary.

**Caching** was also an issue which we solved by redirecting `/time-in-image` to `/time-in-image/[random string of letters].png`. Otherwise it would turn back time when you rejoined your world.

My **web server** to generate the image was made using Node.js with the modules: `express`, `request` and `jimp`

## How do I setup a server?

I would 1000% appreciate if you were to host it yourself. I'm like, 99.8% uptime, but I try my best~ You will need `node`, `npm` and `git` to continue.

Before running `app.js`, I recommend you take a look at the file and editing the settings variable at the top. You can easily attach `time-in-image.js` onto any Express server. https://maki.cat uses https and a few other middlewares before it gets to the route. If you need help settings things up, I'm on Discord at `Maki#4845`.

```sh
git clone https://github.com/makitsune/realtime-clock
cd realtime-clock/server
npm install
node app.js
```

Once running, a link will spew out with your local address. Take your public IP, add the port and path to it, and replace the old URL in the `VRC_Panorama` component with your new one. Make sure your IP/hostname is static. I use my [cloudflare-ddns](https://github.com/makitsune/cloudflare-ddns) for that.

If you want it to run indefinitely, I recommend using a process manager like `pm2`.

```sh
npm install pm2 -g
pm2 start app.js --name "Realtime Clock"
```

## Thank you!

Made by [Maki](https://github.com/makitsune) and
[Desunyan](https://github.com/Shii2)
