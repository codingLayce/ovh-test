## Code

```
# echo : writes to the standard output
# exit : stop the execution and return the given code
# IPDifferenciesMetric : metric definied using something like Prometheus

func main(args) {
    if len(args) != 2 { # Check that the args are valid (program name + addresses)
        echo("Invalid program args: Missing addresses")
        exit(1)
    }
    addresses = create_ip_array(args[1])
    
    differences = [] # Array storing queries differencies (even index correspond to https results / odd index correspond to dns results)
    for each (address) in addresses { 
        fqdn_https = ""
        fqdn_dns = ""
        
        # Query for HTTPS
        try: 
            fqdn_https = queryExternalRessource("https", address)
        catch (err):
            echo("Cannot query HTTPS information %s", err)
            continue # Stop execution for this address and continue
            
        # Query for DNS
        try:         
            fqdn_dns = queryExternalRessource("dns", address)
        catch (err):
            echo("Cannot query DNS information ", err)
            continue # Stop execution for this address and continue
        
        if (fqdn_https != fqdn_dns) {
            echo("Responses differs: https(%s) dns(%s)", fqdn_https, fqdn_dns)
            differences.push(fqdn_https, fqdn_dns)
        }
    }
    
    IPDifferenciesMetric.Set(len(differences)/2)) # Set the metric with half of the array's length because it contains both https and dns results for an IP
}

# returns an array of ips. Each IP is on a new line from the input string.
func create_ip_array(addresses) {
    ips = []
    buffer = ""
    for each (ch) in addresses { # Loop through all characters of the string
        if (ch == '\n') { # When reaching the new line character, store the ip in the array and reset the buffer.
            ips.push(buffer)
            buffer = ""
        } else {
            buffer = buffer . ch # Concatenate ch to buffer
        }
    }
    ips.push(buffer) # Push the last IP
    return ips
}
```

## Explanations

The program takes as arguments the program name (default on most of the systems) and the list of IPs, each on a new line.

It starts by checking if the arguments are valid (length of 2).

Then for better readability, I have created a function `create_ip_array` which takes the input args and creates an array with all the IPs. It loops through all characters of the argument, buffer the characters until the new line character `\n` is reached. Then it pushes the buffer (the IP) into the array containing the IPs, reset the buffer and continue collecting the IPs.

In a real world application, depending on the language, I would be using the standard library to do something like a "split" on new line character to create the array.

Once the array of IPs is created, I loop through all the IPs and perform the queries for HTTPS and DNS information. I do it sequentially to simplify the pseudo-code but the two queries could be done in parallel.

If an error occurs during one of the queries, I have decided to log it and ignore the IP (because I don't have much more information on the functional part to do something else).

Then, if the results are different, I log it and I push to the array the HTTPS results first and then the DNS one. So the array always contains an even number of elements, and you can find the HTTPS information of an IP on even indexes and the DNS information on odd indexes. Also, consecutive even and odd indexes (such as 0/1 4/5 66/67) are information of the same IP.

When all the IPs have been checked, I set the metric `IPDifferenciesMetric` with the number of IPs that have differences between their HTTPS and DNS information. The value is half the length of the array because we store both HTTPS and DNS information for each IP that differs.

For the metric, I keep it simple because I don't really know how to do it a better way. So I assume that the program can access a metric created by something like Prometheus's SDK `IPDifferenciesMetric`, the Prometheus SDK then serve an HTTP endpoint where Grafana can query it and use the metrics provided.