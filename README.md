# Roku Volume controller though SmartThings and Google Home

I really didn't want to use a remote to turn the volume up or down on my Roku TV. 

There is a few dependances you need to meet to get this to work:
1. Roku TV (duh.)
2. Local Server (in my case a RaspberryPi but, anything on the same network as the TV and can run python)  
3. Python 
   * Flask 
   * Roku
4. SmartThings  
5. Google Home  


## Setup Python
First, we need the TV's IP address, the Local Server's IP address, and your public IP.  

Next, lets get python ready. On debian / Ubuntu you can:  
  `sudo apt update`  
  `sudo apt install python python-dev python-pip -y`  
  `sudo pip install flask roku`  
  
Next we need to add a little something to roku's core.py  
 `nano +30 /usr/local/lib/python3.4/dist-packages/roku/core.py`
 change this:  
   ```     
    'literal': 'Lit'
    } ``` 
 to:  
  ```     
    'literal': 'Lit',
    'power': 'Power',
    'volup' : 'VolumeUp',
    'voldn' : 'VolumeDown'
} ```  

Finally, for python lets run a detached Flask instance in the background.

`  FLASK_APP=listener.py flask run --host=0.0.0.0 `  


## Setup SmartThings WebCore
 Issac Newton once said, "Something something shoulders of giants". So, lets go stand on some giants!  
[You must complete all of these steps.](https://wiki.webcore.co/#Installing_webCoRE)  
* GitHub [Install](https://wiki.webcore.co/GitHub_Install) or [Manual Install](https://wiki.webcore.co/Manual_Install) of webCoRE source code into the SmartThings Cloud 
* [Enable webCoRE OAuth](https://wiki.webcore.co/Enable_webCoRE_OAuth) in the SmartThings cloud 
* [Install webCoRE](https://wiki.webcore.co/Install_webCoRE) in the SmartThings mobile app 
* [Enabling webCoRE dashboard](https://wiki.webcore.co/Enabling_webCoRE_dashboard) 
* [Enable webCoRE on Another Device](https://wiki.webcore.co/Enable_webCoRE_on_Another_Device) 
  
## Create SmartThings Virtual Devices  
Login it to the SmartThings IDE: [https://graph.api.smartthings.com/](https://graph.api.smartthings.com/)  
1. Click My Devices and click new device.  
2. Fill out the form make sure the type is simulated switch.  Set the name to Roku_Volume_Up  
3. Click My Devices and click new device.  
4. Fill out the form make sure the type is simulated switch.  Set the name to Roku_Volume_Down  
5. Click My Devices and click new device.  
6. Fill out the form make sure the type is simulated switch.  Set the name to Power  


## Setup Our SmartThings SmartApp 
Login to the SmartThings Dashboard: [https://webcore.co/re](https://webcore.co/re)  

You need to create two pistons, one for Volume Up and Volume Down.  
``` 
************************************************************/
/* Volume Up                                                  */
/**************************************************************/

execute
if
Roku_Volume_Up's switch physically changes to on
then
with
Roku_Volume_Up
do
 Make a GET request to http://flask_ip_address:5000/up with type JSON;
end with;
end if;
with
Roku_Volume_Up
do
 Set switch to off;
end with;
end execute;
```  
and  

```
***********************************************************/
/* Volume Down                                                */
/**************************************************************/
 
execute
if
Roku_Volume_Down's switch physically changes to on
then
with
Roku_Volume_Down
do
 Make a GET request to http://flask_ip_address:5000/down with type JSON;
end with;
end if;
with
Roku_Volume_Down
do
 Set switch to off;
end with;
end execute;
``` 

and


```
**************************************************************/
/* Power                                                      */
/**************************************************************/
 
execute
if
Power's switch physically changes to on
then
with
Power
do
 Make a GET request to http://public_ip_address:5000/power with type JSON;
end with;
end if;
with
Power
do
 Set switch to off;
end with;
end execute;
```

## Setup SmartThings App
We need to add the Virtual Switches to the App so that Google Home can then see them and talk to them.  

Open the SmartThings App on your phone.  
Click Add a Thing  
Find and Add the Roku_Volume_Up and Roku_Volume_Down devices.  

## Setup Google Home
Now, we can add the virtual switches to Google Home and Nickname them to 'Roku Up' and 'Roku Down'.  

### Finally!
Hopefully you are still with me here.  
Once this is all done, the flask server is running, the webcore stuff is waiting, we can test!  

Now, say loudly - say proudly... "Hey Google, Turn On Roku Up"  

If your house just sploded, not my fault.  

