# This is a way of adding a country-block to an EdgeOS Device. It works by pulling a CIDR table from http://www.iwik.org based on the list of countries you can set

Download the country-load.sh to your EdgeOS
```
curl -o /config/scripts/post-config.d/country-load.sh https://raw.githubusercontent.com/gabrielpc1190/EdgeOS-CountryBlock/master/country_load.sh
```

Set the country two digits code ALL-IN-CAPS at the beggining of the country-load.sh script:
```
vi /config/scripts/post-config.d/country-load.sh
```
Find the line and add or remove countries you want to allow:
```
#countryList="CR US ES"
```

+ Enter EdgeOS configuration mode to add the firewall rules needed using the network-group countries_allowed generated by the script:
```
configure

set firewall group network-group countries_allowed description 'Allowed countries'
set firewall group network-group countries_allowed network 10.0.0.0/8

set port-forward rule 4 description 'Home Assistant'
set port-forward rule 4 forward-to address 192.168.1.10
set port-forward rule 4 forward-to port 8123
set port-forward rule 4 original-port 8123
set port-forward rule 4 protocol tcp

set firewall name WAN_IN rule 40 action accept
set firewall name WAN_IN rule 40 description 'Home Assistant'
set firewall name WAN_IN rule 40 destination port 8123
set firewall name WAN_IN rule 40 protocol tcp
set firewall name WAN_IN rule 40 source group network-group countries_allowed

set system task-scheduler task country_load interval 1d
set system task-scheduler task country_load executable path /config/scripts/post-config.d/country-load.sh

commit; save
```

# Make sure you created the script file “/config/scripts/post-config.d/country-load.sh” (chmod 755) (or download from here)
https://raw.githubusercontent.com/gabrielpc1190/EdgeOS-Blacklist/master/country_load.sh
```
chmod 755 /config/scripts/post-config.d/country-load.sh
```

# This script will run when the Edgerouter boots. It will:
Traverse the list of countries defined in the top of the script
Download a list of subnets in each country
Add it to the ipset table (thats what the Edgerouter uses for network-groups)

# Testing
After rebooting the edgerouter or manually running the script, you can check that we #actually got some subnets in our network-group:

sudo ipset -L countries_allowed

Dont be fooled by looking in the GUI – it will know nothing about all this happening behind #the scenes!
Be careful!
If you do any change to the network group “countries_allowed” from the GUI, the Edgerouter #will empty the list generated from the script! Don’t do that 🙂


The original idea was found on this website. Thanks for the ideas!
http://www.cron.dk/firewalling-by-country-on-edgerouter/
