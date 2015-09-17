# Change the notification sound for the Google Voice extension in Chrome

These instructions apply to *Linux* but can be applied to any operating system.

1.  Install apache2 and get it running `sudo apt-get install apache2`
2.  Copy your ringtone to the web folder `cp ~/Downloads/my-ringtone.mp3 /var/www/`
3.  Test it out by pointing your browser to the ringtone `http://localhost/my-ringtone.mp3`
4.  Find the real path of the Google Voice extension 
    1.  Open the extensions tab in Chrome
    2.  Copy the id from the Google Voice extension (mine is `kcnhkahnjcbndmmehfkdnkjomaanaooo`)
    3.  Use this ID to select the correct folder in the directory below
5.  Edit the Google Voice extension itself to replace the default ringtone path with your own `vim ~/.config/google-chrome/Default/Extensions/kcnhkahnjcbndmmehfkdnkjomaanaooo/2.3.6.8_0/background.html` 
    1.  Copy and paste the line labelled `audio id`
    2.  Comment out one of the copies in case these steps go horribly wrong
    3.  Replace the value of the src variable with the path to your ringtone `http://localhost/files/my-ringtone.mp3`
6.  Go to the extension page and disable then enable the Google Voice extension to force a reload of the extension file you just edited Ba-da-boom ba-da-bing, you now can change the ringtone to whatever you want!
