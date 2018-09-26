# netaddr-ipsets-large
How to work with netaddr ipsets that are super large and not wait 1 million hours

This is a big problem I ran into and took me about a week to debug. Because of this I am writing this up to hopefully help another soul not waste a week banging their head.

## What was I trying to do?
I wanted to load all of the IPv4 publicly routed ipspace into an IPset. So I have a script that downloads this from a routeviews archive then converts it with bgpdump into something human readable. I will provide a small sample set to play with. Intially the script ran very fast about 5 minutes to add all the IPs into the set then check a short list. After inspection of the internet routing table it was found that 5 idiots were advertising 0.0.0.0/0. This is what was allowing the set to be built very fast.

## The BAD
After removing the 0.0.0.0/0 I found the sets working really slow. I was using a loop like this:

`internet_set = netaddr.IPSet([])
for line in handler:
    data = line.strip().split('|')
    prefix = data[5]
    if  """0.0.0.0/0""" == prefix:
        continue
    internet_set.add(prefix)`

This was running slower then a pregnant turtle. After it kept having to re add a prefix. After debugging and debugging I split the disk io from the computation of the building of the prefix and saw a slight performance increse but this is what saved the day.

## The GOOD

`big_list = set()
for line in handler:
    data = line.strip().split('|')
    prefix = data[5]
    if  """0.0.0.0/0""" == prefix:
        continue
    big_list.add(prefix)
    
internet_set = netaddr.IPSet(biglist)`

This brings the code run time down to minutes instead of days. Hope this helps someone. Good luck!
