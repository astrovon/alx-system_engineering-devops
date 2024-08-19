Postmortem
Upon the release of ALX's System Engineering & DevOps project 0x19, approximately 06:00 West African Time (WAT) here in Nigeria, an outage occurred on an isolated Ubuntu 14.04 container running an Apache web server. GET requests on the server led to 500 Internal Server Error's, when the expected response was an HTML file defining a simple Holberton WordPress site.

Debugging Process
Bug debuging Frenzy ("Fake") encountered the issue upon opening the project and being, well, instructed to address iT at around;  21:50 PST proceeded to undergo solving the problem.

STEP 1 : Checked running processes using ps aux. Two apache2 processes - root and www-data - were properly running.

STEP 2 : Looked in the sites-available folder of the /etc/apache2/ directory. Determined that the web server was serving content located in /var/www/html/.

STEP 3 : In one terminal, ran strace on the PID of the root Apache process. In another, curled the SERVER  but  strace gave no useful information.

STEP 4 : Repeated step 3, except on the PID of the www-data process; strace revelead an -1 ENOENT (No such file or directory) error occurring upon an attempt to access the file /var/www/html/wp-includes/class-wp-locale.phpp.

STEP 5 : Looked through files in the /var/www/html/ directory one-by-one, using Vim pattern matching to try and locate the erroneous .phpp file extension. Located it in the wp-settings.php file. (Line 137, require_once( ABSPATH . WPINC . '/class-wp-locale.php' );).

STEP 6: Removed the trailing p from the line.

STEP 7 : Tested another curl on the server. 200 A-ok!

STEP 8 : Wrote a Puppet manifest to automate fixing of the error.

SUMMARY-

A typo. was resposible!? To summarize, the WordPress app was encountering a critical error in wp-settings.php when tyring to load the file class-wp-locale.phpp. The correct file name, located in the wp-content directory of the application folder, was class-wp-locale.php.

Patch involved a simple fix in the typo(removing the trailing p ).

PREVENTION-

This outage was not a web server error, but an application error. To prevent such outages moving forward, please keep the following in mind:

-Testing- Test the application before deploying. This error would have arisen and could have been addressed earlier had the app been tested.

-Status monitoring- Enable some uptime-monitoring service such as UptimeRobot to alert instantly upon outage of the website.

NOTE:
In response to this error, I wrote a Puppet manifest 0-strace_is_your_friend.pp to automate fixing of any such identitical errors should they occur in the future. The manifest replaces any phpp extensions in the file /var/www/html/wp-settings.php with php.

GOOD RIDDANCE! When life gives you errors...(> <)
