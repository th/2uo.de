# Firewall in Oracle Cloud

It took me some time to find out why outside services like Letsencrypt could not connect to my free Oracle Cloud instance.

Sure, I had created an “allow all traffic from everywhere” rule in Oracle's Virtual Cloud Network admin. And I had checked that Ubuntu's ufw firewall was inactive.

But as it turns out, Oracle's Ubuntu image doesn't use ufw, like every Ubuntu, but the older iptables firewall.

So you need to do this to get rid of those rules:

``` bash
sudo iptables -F
sudo netfilter-persistent save
```
