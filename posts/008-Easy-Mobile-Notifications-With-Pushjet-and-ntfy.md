# Easy Mobile Notifications with Pushjet and `ntfy`

I run a lot of long-running Linux commands in my career as a bioinformaticist. On the one hand, I can't be bothered to check on them all the time, but on the other hand, I hate showing up to work in the morning to find that the job I tried to run overnight died after 20 minutes because I forgot to specify something that needed specifying. This presented a problem.

To solve this problem, I decided that I was going to set up a system to send messages to my phone when jobs finish. That way, I can know that I have results to look at, and if the job I was expecting to run overnight messages me suspiciously early, I can surmise that something might be amiss.

## Mobile: Pushjet

I first set out to find something I could use on the phone end, because good mobile software is extremely difficult to find. There are a lot of apps out there that will let you do programmable notifications, but the [big](http://www.notifymyandroid.com/faq.jsp) [obvious](https://pushover.net/faq#overview-fees) [ones](https://play.google.com/store/apps/details?id=com.pushbullet.android&hl=en) all seem to be thoroughly monetized, and I couldn't find anything in F-Droid. Since my patience for software that does not act purely as my agent drops percipitously with the size of the device it's running on, that wouldn't do at all.

I eventually found [Pushjet](https://pushjet.io/), which is a BSD-licensed Android app for customizable push notifications. The Pushjet developers run a public API you can send notifications through, and you can also run your own server instance if you are so inclined. (Since I only need to send the occasional non-secret notification to myself, and not spam things to thousands of users, I figured their server would work fine.) Although the app is only in the Play Store right now, they have a big grayed-out F-Droid download button on their site, so I assume that's where it's headed.

So, having found an app, I also needed to figure out a backend.

## Notifications: `ntfy`

This software search was much simpler. A couple of minutes of Googling turned up [`ntfy`](https://github.com/dschep/ntfy), a pip-installable do-everything command-line notification sending tool which integrates with everything from Slack to Syslog, including Pushjet.

## Plugging it All Together

This is how I made it actually work:

1. **Install `ntfy`** on your computer, which can be accomplished with:

    ```
    pip install --user ntfy
    ```

2. **Create a Pushjet service** by going to [the Pushjet API docs for service creation](http://docs.pushjet.io/docs/creating-a-new-service) and filling out the "Try It Out" form. You don't need an icon, but I used [this one by Nikita Golubev](http://www.flaticon.com/free-icon/programming_363216). You should get some response JSON like this:
    
    ```
    {
        "service": {
            "created": 1492572834,
            "icon": "https://interfect.github.io/images/qr.png",
            "name": "Markdown Notebook",
            "public": "ca25-456026-4ada62ceb03f-ab741-7bee56550",
            "secret": "a5a69e7b9418e8d38878c69e6994dbb2"
        }
    }
    ```

3. **Make a `~/.ntfy.yml`** on your computer. It should look like this, using the `secret` you got in step 2:
    
    ```
    backends:
      - pushjet
    pushjet:
      secret: a5a69e7b9418e8d38878c69e6994dbb2
    ```
    
    Don't forget to secure it with a `chmod 600 ~/.ntfy.yml`.

4. **Install the [Pushjet app](https://pushjet.io/#download)** on your phone or other Android device.

5. **Take you `public` value from the service and paste it in [this QR code generator](https://kazuhikoarase.github.io/qrcode-generator/)**.

6. In Pushjet, hit the menu icon, and then the "+" button, and **scan the QR code** with your device.

7. Since Pushjet doesn't deal well with long titles, and `ntfy` by default includes the working directory in the notification title, **make a wrapper script** called `notify`:
    
    ```
    #!/usr/bin/env bash

    # notify: send ntfy messages with a reasonable-length default title
    set -e

    ntfy -t `hostname` send "${*}"
    ```

8. Whenever you do a long-running job, **tack a `notify 'My job is done!'` onto the end**.

9. **Enjoy** your notifications.
    
    ![Example notification](/images/008/screenshot-small.png)


