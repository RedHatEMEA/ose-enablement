## OpenShift JBoss EAP Session Replication Demo##

#### This is a very simple application that shows how session replication works on JBoss EAP OpenShift cartridge####

* Start by creating a scalable application
```
$ rhc create-app APPNAME jbosseap -s --from-code https://github.com/amorena/scalable-jbosseap-openshift-demo-app.git
```

* Change to the application directory
```
cd APPNAME
```

* Set the minimum number of instances of JBoss EAP. This would help us see multiple JBoss EAP instance in action
```
$ rhc scale-cartridge --min 3 --cartridge jbosseap
```
##### The command above will take few minutes to finish#####

--

## Session Replication Demo##

1. Open your favorite browser and go to http://APPNAME-{domain-name}.rhcloud.com
2. You would see a simple page that would show session attribute "gear" value null as shown below. Please note your session id would be different
<img src="https://whyjava.files.wordpress.com/2014/08/screen-shot-2014-08-26-at-1-24-43-am.png" height="100" width="300">
3. Visit the URL http://APPNAME-{domain-name}.rhcloud.com/set and the application would set the $OPENSHIFT_GEAR_UUID in the session. Now you would see a valid value in the session as shown below.
<img src="https://whyjava.files.wordpress.com/2014/08/screen-shot-2014-08-26-at-1-30-24-am.png" height="100" width="300">
4. To view all the gear ids, you can run following command
```
$ rhc show-app --gears --app APPNAME
```

--
## Session Replication Demo - Simulate a second user##
* Open a new web browser to simulate a new user. Follow step 2 and 3
   - Now, you have two different users with two different sessions. The '/' would show different values of gear
* Log in to gear one. You can find the ssh details for that gear by typing following command
```
$ rhc show-app --gears |grep 53fb46465973ca21b200016b
```
* SSH into the application gear using the SSH URL you got in previous step
* Stop the gear running the <code>gear stop</code> command
   - This would stop the JBoss EAP container running on that gear. Check the HAProxy status page to view gear status
<img src="https://whyjava.files.wordpress.com/2014/08/screen-shot-2014-08-26-at-1-40-59-am.png" height="100">
* Now, refresh your first browser, this time request would be served by other gear but you would still see the data from the session.
    - This is because data was replicated to the second gear
