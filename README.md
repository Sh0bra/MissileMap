# Visualizing Security with Missile Map

![my imgae](/asset/default.png)

## Learning Objectives
- Extract meaningful data from logs using Splunk's field extraction tools and SPL (Search Processing Language).
- Normalize and format data to meet specific requirements.
- Leverage geolocation data using Splunk's iplocation command to map IP addresses to geographic locations.
- Visualize connections on a map using Missile Map in Splunk.

## Introduction
One of the largest sources of data I started capturing from our Student Data Center was the VPN logs. We used a pfSense router and hosted a OpenVPN server on it to allow students access to the SDC resources.

As I kept staring at the VPN logs coming into our Splunk server I wondered if there was a way to visualize the connections like on a map. A quick Google search and I found [Missile Map](https://splunkbase.splunk.com/app/3511) an app created by Luke Monahan made available on SplunkBase.

## Requirements
To get the correct data into Missile Map we need to format the data coming in to match the specification for Missile Map. The documentation for Missile Map conveniently shows us what exactly the app looks for when processing the data.
- start_lat: The starting point latitude (required)
- start_lon: The starting point longitude (required)
- end_lat: The ending point latitude (required)
- end_lon: The ending point latitude (required)
- color: The color of the arc in hex format (optional, default "#FF0000")
- animate: Whether to animate this arc (optional, default "false")
- pulse_at_start: When animated, set to true to cause the pulse to be at the start of the arc instead of the end (optional, default "false")
- weight: The line weight of the arc (optional, default 1).
- start_label: A text label on the start point of the arc
- end_label: A text label on the end point of the arc

From the above we see that start_lat, start_lon, end_lat and end_lon are required fields.

Missile Map also provides a test data set which allows us to see how it wants the data formatted. To access this dataset you can search the test dataset with the following SPL query:
```splunk
| inputlookup missilemap_testdata
```
![my imgae](/asset/defaultdata.png)


## Normalizing our data
Using the Splunk Search app I queried the VPN logs and below you can see a sample what pfSense OpenVPN logs look like.

![my imgae](/asset/data.png)


To feed the data into Missile Map I first needed to filter for all the IP addresses for connecting sessions. In Splunk we can extract a new field from the data using the built-in field extractor function. 
Notice that everytime a connection to the server is made the log "openvpn server 'ovpns1' user 'vpn' address '47.153.255.97' - connected" is generated. Using the field extractor I can extract the IP address and assign it a fieldname of src_ip. Here is the output regex if you think regex is cool :)

```splunk
^(?:[^'\n]*'){5}(?P<src_ip>[^']+)
```

In order to get the latitude and longitude coordinates of the IP addresses I use Splunk's built-in iplocation command. This command takes in an IP address as input and returns the City, Country, lat, lon fields.

## Putting it all together
Now that we had all the data needed we could format the data into how Missile Map wants it using the SPL below.

```splunk
index = pfsense sourcetype = pfsense:openvpn connected
| iplocation src_ip
| eval start_lat = lat
| eval start_lon = lon
| eval end_lat="34.059705"
| eval end_lon="-117.819758"
| eval animate = "true"
| eval pulse_at_start = "true"
| table src_ip, start_lat, start_lon, end_lat, end_lon, animate, pulse_at_start
```
> Note: animate and pulse_at_start don't have to be there and can be removed if it gets too laggy.

![my imgae](/asset/output.gif)

## Limitations
The iplocation function seems to return a lat and lon that isnt the most accurate as the origin lines did not match with our actual location
Another reason for this could be the fact that external IPs are being forwarded from the ISP and not from our homes.

## Learning Outcomes
- Understand how to process logs (e.g., VPN logs) in Splunk to extract relevant fields.
- Use regex for custom field extraction in Splunk.
- Utilize Splunk's iplocation command to enrich data with geolocation information.
- Format data correctly to integrate with a visualization tool (Missile Map) for creating geographic visualizations of connections.
- Gain insight into network activity by visualizing the origin and destination of VPN connections in a clear, interactive format.

## Whats Next?
- Creating alerts based on geolocation
- Adding Missile Map to dashboard
- Implement Missile Map for external connections
- Implement Missile Map for Website visitors
