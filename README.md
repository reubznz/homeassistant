# Home Assistant

The following will enable the configuration for details FlightInfo

### HACS Requirements
Install Event Sensor

### Add the following to your configuration.yaml:
```
homeassistant:
  allowlist_external_dirs:
    - /config
  customize:
    sensor.flightinfo:
      icon: 'mdi:airplane'
```

```
shell_command:
  flightinfo_script: /config/flightinfo.sh {{flightnumber}}
```

### Add the following to your sensors.yaml
```
sensors:
  - platform: opensky
    name: OpenSky
    radius: 10

  - platform: eventsensor
    name: OpenSkyFlights
    event: opensky_entry
    state: callsign

  - platform: file
    name: flightinfo
    file_path: /config/flightinfo.txt
    value_template: '{{ value_json.flightinfo }}'
```

### Create a new file called flightinfo.sh
```
#!/bin/bash

url=https://flightaware.com/live/flight/$1

wget --output-document flightinfo.txt $url

findstring=$(grep "twitter:description" flightinfo.txt)

replacestring1="<meta name\=\"twitter:description\" content\=\""
replacestring2="\" \/>"
replacestring3="Track"
replacestring4=" Int&#39;l"
replacestring5="#"
replacestring6="flight"
replacestring7="("
replacestring8=")"
replacestring9="from"

nothing=""

cleanstring=$(echo "$findstring" | sed "s/$replacestring1/$nothing/")
cleanstring=$(echo "$cleanstring" | sed "s/$replacestring2/$nothing/")
cleanstring=$(echo "$cleanstring" | sed "s/$replacestring3/$nothing/")
cleanstring=$(echo "$cleanstring" | sed "s/$replacestring4/$nothing/")
cleanstring=$(echo "$cleanstring" | sed "s/$replacestring5/$nothing/")
cleanstring=$(echo "$cleanstring" | sed "s/$replacestring6/$nothing/")
cleanstring=$(echo "$cleanstring" | sed "s/$replacestring7/$nothing/")
cleanstring=$(echo "$cleanstring" | sed "s/$replacestring8/$nothing/")
cleanstring=$(echo "$cleanstring" | sed "s/$replacestring9/$nothing/")

cleanstring=$(echo "$cleanstring" | sed "s/$1/$nothing/")

cleanstring=$1 $cleanstring

outputstring="{\"flightinfo\":\"$cleanstring\"}"
echo $outputstring > flightinfo.txt

echo $1 > flightnumber.txt
```

### Using Terminal and SSH
```
cd /config
chmod +x flightinfo.sh
touch flightinfo.txt
```

### Restart homeassistant

### Add a Glance Gard with the following config
```
type: glance
entities:
  - entity: sensor.flightinfo
    name: Recent Flight Info
```
