---
layout: post
title: Database Backup on CentOS Using Dropbox
---

{{ page.title }}
================

<p class="meta">June 5, 2010</p>

One of the most important considerations when running any web server is the ability to backup important data. One of the most important pieces of data on any server is, of course, the database. I currently use Linode as my web host of choice — they’re fast, reliable, and affordable. However, one place where their service is slightly lacking is the inability to perform automated backups (*editor's note: they have since rolled out an automated backup offering*). Most of the files for a given website are already backed up as we create and maintain the site, but one key area where a whole host of data could be lost without a proper backup solution is the database. Having already used Dropbox for some time now to enable our team to collaborate effectively, and knowing that they have a command-line Linux client available, using it for our backup solution is an obvious answer. The process itself wasn’t too terribly difficult, but it did require quite a bit of searching through various partially-complete articles, so I have compiled them all into a quick and easy step-by-step guide that should work for those of you out there using a “headless” VPS (such as Linode) with CentOS installed.

What You’ll Need
----------------

* At least one Dropbox account. I’ll leave it to you to figure out how to sign up for one, but you must have an active free or paid account to complete this process. Note that with a free account you will only be able to maintain up to 2 GB worth of backups. (Note: We actually use several — a free account for the server, which shares a folder with our team’s paid accounts.)
* Command-line and root access to a CentOS web server.
* About 30 minutes.

Step 1: Install Dropbox on the CentOS Server
--------------------------------------------

The first step is to install Dropbox onto the CentOS server itself. This part of the process is largely based off of a handy guide I found by Justin Kelly, so thanks to him for providing that very useful information. SSH into your CentOS server, and in your home directory (or some other temporary place) run the following commands, which will download Dropbox and begin the process of installing it:

	wget -O dropbox.tar.gz http://www.getdropbox.com/download?plat=lnx.x86
	tar zxof dropbox.tar.gz
	wget http://dl.getdropbox.com/u/6995/dbmakefakelib.py
	python dbmakefakelib.py
	
At this point, you may receive an error about not having GCC installed. If that’s the case, you can install it but running:

	sudo yum install gcc
	
And then running the Python command again:

	python dbmakefakelib.py
	
After that command has executed successfully, you should see a pause followed by the message:

	dropboxd ran for 15 seconds without quitting - success?
	
You can simply `Control+C` out of that running script. At this point we have successfully run Dropbox, but it does not know what account to connect to. To get that part working, we will have to manually link the HostID of our CentOS server to our Dropbox account. To do so, we will use SQLite to open a small database.

	cd .dropbox
	sqlite3 dropbox.db

Once at the SQLite prompt enter:

	sqlite> .dump config

A small amount of data will print out. The part we are interested in is:

	INSERT INTO "config" VALUES(3, 'host_id', 'XXXXXXXXXXXXXXXXXXXX
	
Take whatever is in `XXXXXXXX` for you (your `host_id` value), open up this site: Base64 Decode Script, and get the decoded version of your `host_id`. NOTE: You only want the characters after the first captial “V”, on the first line (ignore the “pl” and “.”). Once you have this value, open the following URL in your browser (NOTE: You should either be logged out of Dropbox or logged in as the user you want to link the computer to): `https://www.getdropbox.com/cli_link?host_id=YOUR-DECODED-HOST-ID` After you do that, you should see a message that says “Computer was linked to your account successfully”. Now that we’re linked up, let’s complete the final steps on our server. We need to make a directory for Dropbox to live in. At your server’s command line, enter:

	sqlite> .exit
	cd ~
	mkdir ~/Dropbox

To start Dropbox, enter:

	~/.dropbox-dist/dropboxd &
	
When that’s working, we’ll want to install Dropbox as a server. To do so:

	sudo vi /etc/init.d/dropbox
	
And paste the following into that file:

	# chkconfig: 345 85 15
	# description: Startup script for dropbox daemon
	#
	# processname: dropboxd
	# pidfile: /var/run/dropbox.pid
	#
	# Source function library.

	. /etc/rc.d/init.d/functions

	DROPBOX_USERS="user1 user2"

	prog=dropboxd

	lockfile=${LOCKFILE-/var/lock/subsys/dropbox}

	RETVAL=0

	start() {

	        echo -n $"Starting $prog"
	         for dbuser in $DROPBOX_USERS; do
	            daemon --user $dbuser /bin/sh -c "/home/$dbuser/.dropbox-dist/dropboxd &"
	        done

	        RETVAL=$?

	        echo

	        [ $RETVAL = 0 ] && touch ${lockfile}

	         return $RETVAL

	}

	stop() {

	        echo -n $"Stopping $prog"
	    for dbuser in $DROPBOX_USERS; do
	        killproc /home/$dbuser/.dropbox-dist/dropboxd
	    done
	        RETVAL=$?
	         echo
	        [ $RETVAL = 0 ] && rm -f ${lockfile} ${pidfile}
	}

	# See how we were called.

	case "$1" in
	  start)
	        start
	        ;;
	  stop)
	        stop
	        ;;
	   restart)
	        stop
	        start
	        ;;
	  *)
	        echo $"Usage: $prog {start|stop|restart}"
	        RETVAL=3
	esac

	exit $RETVAL

Make sure to replace `user1` and `user2` with the users that you want Dropbox to run as (we just created a user called ‘backup’ that runs all the backup scripts on the server.) And finally:

	sudo chmod +x /etc/init.d/dropbox
	sudo /sbin/chkconfig --add dropbox
	sudo chmod 755 /etc/init.d/dropbox
	
So, at this point, you should have a working Dropbox installation on your CentOS server. Test it out by creating a couple of text files in the folder — they should show up in the web interface. You can also create a Shared Folder as your backup folder and any files created during backup will be copied to other Dropbox users who are sharing that folder.

Step 2: Create a MySQL Backup Script
------------------------------------

Now that we have Dropbox running and syncing files, all we really need to do is copy the files that we want backed up into the Dropbox folder (or Shared Folder, depending on your setup). To do so, we’ll create a basic CRON script that automatically dumps our MySQL database every day, and puts that dump file into the Dropbox folder. Go to your user’s home directory, and create a new file:

	vi daily.backup.cron

And paste the following code into the newly-created file:

	#!/bin/bash

	mysqldump --opt --host=localhost --user=DBUSER --password=DBPASSWORD --all-databases > /path/to/Dropbox/dailyBackup.sql
	
Be sure to replace `DBUSER` and `DBPASSWORD` with the username and password of a database user with access to all of your databases, and replace `/path/to/` with the path to your Dropbox folder (the one we created earlier). Set the script as executable, and test it out:

	chmod +x daily.backup.cron
	./daily.backup.cron
	
You should get a database dump saved to your Dropbox folder, which is then synced to Dropbox’s servers and any other folks you are sharing the folder with! Finally, create a crontab entry so that this script will be executed automatically ever day at 2:00 AM:

	crontab -e

	0 2 * * * ./path/to/daily.backup.cron

`ESC`, `:wq`, `ENTER` to save your changes and install the new crontab. And that’s it! Automated daily MySQL backups using Dropbox on CentOS. Whew! A couple of things to note before you head off into the sunset:

* If you have a large number of databases, doing `-—all-databases` in your mysqldump command may not be the best idea. Consider doing a series of separate crontabs for each database.
* Be sure to run this script at an off-peak time of day for your site — it will take a considerable amount of your system’s resources to dump your databases, and then Dropbox will use quite a few sending it off to their servers.
* DON’T link the Dropbox account for the server to one that is constantly receiving files (especially large ones), otherwise you’ll be using a lot of inbound bandwidth getting files that the server doesn’t really need — you want this to be mostly a one-way connection.
* Keep in mind that when Dropbox syncs the backup, it will use your server’s bandwidth to do so. If you’re syncing a 300-MB file every day, that can get expensive!

Hope that it works out well for you!