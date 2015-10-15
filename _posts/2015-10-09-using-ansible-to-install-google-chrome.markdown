---
layout: post
title:  "Using Ansible To Install Headless Google Chrome for Selenium"
date:   2015-10-09 16:25:57
comments: true
categories: ansible
---

This last week I needed to install Google Chrome on a headless Ubuntu 14.04 server for use with rendering web assets in via Selenium's Chrome WebDriver. The configuration took a bit of testing and work to get it all together, so I wanted to share it here. I'll first go over the general method of how everything is installed, then I'll share the actual Ansible scripts used to do it.

# Overview

## Xvfb

[`Xvfb`](http://www.x.org/archive/X11R7.6/doc/man/man1/Xvfb.1.xhtml) is a headless display. It allows us to run Chrome (or any application with a UI) on a device that doesn't have an actual physical display.

Fortunately, it is easily installed via `apt-get install xvfb`.

However, I also wanted `xvfb` to run on boot but it doesn't come with a daemon so I had to create one. I just copied the `xvfb-init.d-template` from below into `/etc/init.d/xvfb`, set it to default runlevels via `update-rc.d xvfb defaults`, then started it with `service xvfb start`.

## Chrome

Because Chrome contains some closed-source components, Ubuntu doesn't include it in any of its default distributed repositories. This means you can't just download it via `apt-get install google-chrome-stable`. [One option](http://askubuntu.com/a/79289/19047) is to add Google's repository to your `apt` sources, but I thought it would be easier to just download and install the `.deb` package that Google provides.

However, `dpkg` can't resolve and install dependencies when installing via `.deb` packages, so you have to follow up the install with `apt-get install --fix-broken`.

    wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
    dpkg -i google-chrome-stable_current_amd64.deb
    sudo apt-get install --fix-broken

## Chrome WebDriver for Selenium

For the Chrome WebDriver, I needed to download the archive, unzip it, and make sure it was owned by the right user so it could be executable when my application ran.

In retrospect, I should have downloaded the Chrome WebDriver in the same way as `xvfb` and `google-chrome-stable`: via ansible. But at the time I thought that the driver needed to exist in the same directory as the Java service, which location didn't exist when Ansible was run. What I did instead was write a rather tedious bash script that was the equivalent of a few lines of ansible. I've included it in the Source section below. It may be of some use to you.

# Source

Here are the actual Ansible and bash scripts that I ended up using. You may need to tweak them for your own use.

## Xvfb Ansible Scripts

    - name: Check if xvfb is installed
      command: dpkg -s {{ xvfb_package_name }}
      register: xvfb_check_deb
      failed_when: xvfb_check_deb.rc > 1
      changed_when: xvfb_check_deb == 1
      tags:
        - xvfb

    - name: Install xvfb package
      apt: name=xvfb state=present
      sudo: true
      when: xvfb_check_deb.rc == 1
      tags:
        - xvfb

    - name: Install xvfb init.d daemon script
      action: template src=xvfb-init.d-template dest=/etc/init.d/xvfb mode=0755
      tags:
        - xvfb

    - name: Set xvfb to run on startup
      shell: update-rc.d xvfb defaults
      sudo: true
      when: xvfb_check_deb.rc == 1
      tags:
        - xvfb

    - name: Start xvfb service
      action: service name=xvfb state=started
      tags:
        - xvfb

## Xvfb Init Script

    # xvfb-init.d-template
    ### BEGIN INIT INFO
    # Provides: Xvfb
    # Required-Start: $local_fs $remote_fs
    # Required-Stop:
    # X-Start-Before:
    # Default-Start: 2 3 4 5
    # Default-Stop: 0 1 6
    # Short-Description: Loads X Virtual Frame Buffer
    ### END INIT INFO

    XVFB=/usr/bin/Xvfb
    XVFBARGS=":1 -screen 0 1920x1080x24 -ac +extension GLX +render -noreset"
    PIDFILE=/var/run/xvfb.pid
    case "$1" in
        status)
            echo "Checking xvfb status"
            if [ -f $PIDFILE ]; then
                PID=`cat $PIDFILE`
                if [ -z "`ps axf | grep ${PID} | grep -v grep`" ]; then
                    printf "%s\n" "Process dead but pidfile exists"
                    exit 1
                else
                    echo "Running"
                fi
            else
                printf "%s\n" "Service not running"
                exit 3
            fi
            ;;
        start)
            echo -n "Starting virtual X frame buffer: Xvfb"
            start-stop-daemon --start --quiet --pidfile $PIDFILE --make-pidfile --background --startas /bin/bash -- -c "exec $XVFB $XVFBARGS > /var/log/xvfb.log 2>&1"
            echo "."
            ;;
        stop)
            echo -n "Stopping virtual X frame buffer: Xvfb"
            start-stop-daemon --stop --quiet --pidfile $PIDFILE
            echo "."
            ;;
        restart)
            $0 stop
            $0 start
            ;;
        *)
            echo "Usage: /etc/init.d/xvfb {start|stop|restart}"
            exit 1
    esac

    exit 0

## Google Chrome Ansible Scripts

    - name: Check if google-chrome-stable is installed
      command: dpkg -s {{ google_chrome_package_name }}
      register: google_chrome_check_deb
      failed_when: google_chrome_check_deb.rc > 1
      changed_when: google_chrome_check_deb == 1
      tags:
        - chrome

    - name: Download google-chrome-stable
      get_url:
        url="{{ google_chrome_base_url }}/{{ google_chrome_package_filename }}"
        dest="/home/{{ service_user }}/{{ google_chrome_package_filename }}"
      when: google_chrome_check_deb.rc == 1
      tags:
        - chrome

    - name: Install google-chrome-stable
      apt: deb="/home/{{ service_user }}/{{ google_chrome_package_filename }}"
      sudo: true
      when: google_chrome_check_deb.rc == 1
      tags:
        - chrome

    - name: Fix any missing google-chrome-stable dependencies
      shell: apt-get install -f
      sudo: true
      when: google_chrome_check_deb.rc == 1
      tags:
        - chrome

## Ansible Properties

Some of the `xvfb` and `chrome` specific properties I supplied in a vars file:

    # Google chrome package url and name
    google_chrome_package_filename: 'google-chrome-stable_current_amd64.deb'
    google_chrome_base_url: 'https://dl.google.com/linux/direct'
    google_chrome_package_name: 'google-chrome-stable'

    # Xvfb package url and name
    xvfb_package_name: 'xvfb'

## Chrome WebDriver Install Bash Script

    #!/bin/bash

    echo "Begin Chrome Driver install script."

    if [ -z $SERVICE_USER -a -z $SERVICE_GROUP ]; then
        # On maestro deploy, user should be specified with RUN_USER
        # On domo vm deploy, user should be specified by SERVICE_USER
        SERVICE_USER=$RUN_USER
        SERVICE_GROUP=$RUN_USER
    fi

    CHROME_ZIP="chromedriver_linux64.zip"
    if [[ "$OSTYPE" == "linux-gnu" ]]; then
        echo "Detected Linux operating system"
        CHROME_ZIP="chromedriver_linux64.zip"
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        echo "Detected OS X operating system"
        CHROME_ZIP="chromedriver_mac32.zip"
    else
        echo "ERROR: Unsupported operating system for DaVinci's Chrome Driver. Exiting now."
        exit 1
    fi

    CHROME_DRIVER="chromedriver"
    CHROME_DRIVER_VERSION="2.19"
    CHROME_DRIVER_URL="http://chromedriver.storage.googleapis.com/${CHROME_DRIVER_VERSION}/${CHROME_ZIP}"

    echo "Checking for Selenium Chrome driver at ./${CHROME_DRIVER}"
    if [ -f $CHROME_DRIVER ] ; then
        echo "Chrome driver already installed."
    else
        TRIES=0
        while [ $TRIES -lt 5 -a ! -f $CHROME_DRIVER ]; do
            echo "Attempt $TRIES at downloading Selenium Chrome driver from ${CHROME_DRIVER_URL}"
            if curl -O $CHROME_DRIVER_URL ; then
                if file $CHROME_ZIP | grep "Zip archive data"; then
                    echo "Download successful."
                    echo "Unzipping $CHROME_ZIP"
                    if unzip $CHROME_ZIP ; then
                        echo "$CHROME_ZIP unzipped successfully"
                        if [ -f $CHROME_DRIVER ] ; then
                            echo "Chrome driver unzipped and $CHROME_DRIVER located."
                            echo "Setting $CHROME_DRIVER ownership to ${SERVICE_USER}:${SERVICE_GROUP}"
                            chown ${SERVICE_USER}:${SERVICE_GROUP} $CHROME_DRIVER
                        else
                            echo "ERROR: Chrome driver unzipped successfully but unable to locate ${CHROME_DRIVER}."
                        fi
                    else
                        echo "ERROR: Unzip of Chrome driver failed."
                    fi
                else
                    echo "Download unsuccessful. $CHROME_ZIP is not a zip archive."
                fi
            else
                echo "ERROR: Download of Selenium Chrome driver failed. Failing prematurely."
                exit 1
            fi
            if [ ! -f $CHROME_DRIVER ] ; then
                ((TRIES++))
                SLEEP=$(($TRIES * 30))
                echo "Failed to install Chrome driver. Sleeping for $SLEEP seconds then trying again."
                sleep $SLEEP
            fi
        done
        if [ -f $CHROME_ZIP ] ; then
            echo "Cleaning up $CHROME_ZIP"
            rm $CHROME_ZIP
        fi
        if [ -f $CHROME_DRIVER ] ; then
            echo "Chrome driver installed successfully!"
        else
            echo "ERROR: Unable to install Chrome driver, service has failed to start. Exiting now."
            exit 1
        fi
    fi
