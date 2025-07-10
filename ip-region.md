## IP Region Exercise 1

### Code

```
// Map of regions containing a map of servers containing array of IP's range.
var regions = {
    "roubaix": {
        "dedicated_server": [
            "10.0.0.0/16",
            "100.0.0.0/16",
            "200.0.0.0/16"
        ],
        "virtual_private_server": [
            "50.0.0.0/16",
            "150.0.0.0/16"
        ]
    },
    "london": {
        "dedicated_server": [
            "11.0.0.0/16",
            "110.0.0.0/16",
            "210.0.0.0/16"
        ],
        "virtual_private_server": [
            "51.0.0.0/16",
            "151.0.0.0/16"
        ]
    }
}

func find_region (input_ip) {
    for each (region, servers) in regions { # key: region_name (string) / value: servers (map)
        for each (server, ips) in servers { # key: server_name (string) / value: ips (array)
            for each (ip) in ips {          
                if (ip_in_range(input_ip, ip)) {
                    return region	
                }	
            }	
        }
    }
	return error("No region found")
}
```

### Explanations

I have created a variable `regions` containing the data structure described in the exercise statement. In a real code, this data structure would be a map. A new region can easily be added by creating a new entry in the map.

It's important to pinpoint that for the exercise all regions are stored in memory and hardcoded. In a real world application, I most likely be using a database.

The algorithm asked for the exercise is the function `find_region`. It simply loops through all regions, servers and ranges. For each range it uses the function we have access to `ip_in_range` to check if the ip is in range. If so, returns the region right away (stop the execution) otherwise continue to the next range, server and region. I can return the first region that matches because there is no IP overlap.

I have chosen to return an error if no region is found for the given IP because it will be easier for the user of the function to determine if the lookup succeeded or not.

## IP Region Exercise 2

### Code

```
// Map of regions containing a map of servers containing array of IP's range.
var regions = {
    "roubaix": {
        "dedicated_server": [
            "10.0.0.0/16",
            "100.0.0.0/16",
            "200.0.0.0/16"
        ],
        "virtual_private_server": [
            "50.0.0.0/16",
            "150.0.0.0/16"
        ]
    },
    "london": {
        "dedicated_server": [
            "11.0.0.0/16",
            "110.0.0.0/16",
            "210.0.0.0/16"
        ],
        "virtual_private_server": [
            "51.0.0.0/16",
            "151.0.0.0/16"
        ]
    },
    "madrid": {
        "dedicated_server": [
            "200.0.0.0/19",
        ]
    }
}

func find_region (input_ip) {
    closest_region = ""
    closest_mask   = "0" # Initialize to 0 because we are looking to the biggest one
    
    for each (region, servers) in regions { # key: region_name (string) / value: servers (map)
        for each (server, ips) in servers { # key: server_name (string) / value: ips (array)
            for each (ip) in ips {          
                if (ip_in_range(input_ip, ip)) {
                    try: # Because get_mask can returns an error 
                        mask = get_mask(ip)
                    catch (err):
                        return error("cannot get mask: %s", err)
                        
                    if (mask > closest_mask) { # Bigger mask means closest range so it's the one I keep
                        closest_mask = mask
                        closest_region = region
                    }
                }	
            }	
        }
    }
    
    if (closest_region == "") {
        return error("No region found")
    }
    return closest_region
}

# Returns the mask of the given ip range.
func get_mask (ip) {
    slash_found = false # flag to determine when to collect the mask
    mask        = ""    # actual mask
    for each (ch) in ip { # loop through all characters of the ip
        if (slash_found) { # when reaching the slash I start collecting the mask
            mask = mask . ch # Concatenate ch to mask
        }
        if (ch == '/') { # slash found, update the flag
            slash_found = true
        }
    }
    
    if (!slash_found) {
        return error("Invalid ip range")
    }
    return maskx
}
```

### Explanations

The setup is the same as exercise 1.

But the algorithm evolves because overlap can happen. Because multiple ranges can match an IP, the function `find_region` must returns the closest range containing the given IP.

To determine the closest range I am using the IP mask. The bigger the mask is, the fewer bits are available for the subnet. So, when a range match for the given IP, I keep it only if the mask is bigger than the previous match.

To retrieve the mask I introduce the function `get_mask` which takes an IP range and returns his mask. The function loops through all characters building the IP, looking for the '/'. When the '/' is found, it collects the characters building the mask by concatenating them to the `mask` variable.

In real world application, depending on the language, I would be using the standard library to manipulate the IP's string. I would look for the '/' and extract the characters after it.

Once the mask is found, I check if it's greater than the one set in `closest_mask`, if so I keep it and the associated region, if not I continue.

Once all the ranges have been checked, the function returns the `closest_region` variable.