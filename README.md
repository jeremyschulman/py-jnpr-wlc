# ABOUT

  Python module for Juniper Wireless LAN Controller (WLC)

**STATUS**: Experimental, under active development.  
The WLC XML API is not public, but can be made available to existing customers.

# OVERVIEW

  The Juniper Wireless LAN Controller products implement a comprehensive XML-RPC API over HTTP/s.  This module provides a *metaprogramming* set of capabilities to fully utilize the API in any easy and consumable manner.  To proficiently use this API, you should be familiar with [XML](http://www.w3schools.com/xml/) and [XPath](http://www.w3schools.com/xpath/) expressions.  
  
This module uses [lxml](http://lxml.de/index.html) for XML processing.  This module uses [Jinja2](http://jinja.pocoo.org/docs) for template processing.

  
This module is developed and tested with Python 2.7.  If you are using another version and it works, please notify the maintainer.  If you are using another version and it does **not** work, please open an issue.

## EXAMPLE

````python
from jnpr.wlc import WirelessLanController as WLC

wlc = WLC( user='jeremy', host='192.168.10.27', password='logmeIn' )

# auth/login to WLC device
wlc.open()

# -----------------------------------------------------
# Get all VLANS
# -----------------------------------------------------
# Retrieve the current VLANs and display them.  The RPC
# invocation here is metaprogramming.  The `rpc` object
# metraprograms whatever comes after.  The response is 
# an lxml Element

vlans = wlc.rpc.get_vlan()

for vlan in vlans.xpath('VLAN'):
  v_num = vlan.attrib['number']
  v_name = vlan.attrib['name']
  print "VLAN(%s) is named: %s" % (v_num, v_name)

# -----------------------------------------------------
# Get only one VLAN
# -----------------------------------------------------

resp = wlc.rpc.get_vlan( name="VoIP" )
vlan = resp.find('VLAN')
print "VoIP vlan number is %s" % vlan.attrib['number']


# cleanup/close
wlc.close()
````
  For more examples, see the [example](https://github.com/jeremyschulman/py-jnprwlc/tree/master/examples) directory.
  
## "FACTS"

  When you make a call to `open()` the method will retrieve bits of data about the WLC and store them
  into a dictionary called `facts`.  You can access this data as property, as illustrated:

````python
>>> pprint( wlc.facts )
{'auto-config': 'NO',
 'backup-config-location': '',
 'booted-config-location': 'configuration',
 'booted-image-location': 'boot0:WC075106.8XX',
 'booted-version': '7.5.1.6.0',
 'bootflags': '',
 'cluster-active-seed': 'NO',
 'console-timeout': '0',
 'domainsecurity': 'NONE',
 'enablepw': 'b5bacf1baa8e028e858c134e3fd2d306cc3b',
 'fips': 'NO',
 'idle-timeout': '3600',
 'ip-addr': '66.129.246.74',
 'macaddress': '78:19:F7:70:96:76',
 'model': 'WLC880R',
 'mx-secret': '',
 'name': 'WLC880R',
 'next-config-location': 'configuration',
 'next-image-location': 'boot0:WC075106.8XX',
 'next-image-version': '7.5.1.6.0',
 'objectid': '1.3.6.1.4.1.14525.3.3.1',
 'platform': '802',
 'serialnumber': 'JJ0211401124',
 'tunnelsecurity': 'NONE',
 'type': 'DP',
 'uptime': '0'}
 

>>> wlc.facts['serialnumber']
'JJ0211401124'

>>> wlc.facts['macaddress']
'78:19:F7:70:96:76'

````

## RPC METAPROGRAMMING

You can issue WLC XML RPCs in a few different ways.  These methods use Python metaprogramming techniques.  
Metapgramming means the this module automatically generates the XML RPC commands on-the-fly without
having to maintain a static set of function bound to a specifc WLC release of code.

For details on issuing [RPCs](docs/wlc_class.md) and [metaprogramming](docs/metaprogramming.md), refer to the embbedded links.  

## "EZ" MICROFRAMEWORK

Each WLC object has an `ez` attribute that can be used to attach helper functions or packages.  The purpose of these functions is provide natural Python language bindings around the WLC XML API so that the results are not XML, but native types, like dictionaries.

There are a few _builtin_ helper functions that are autoinstalled as part of every WLC object.  For details, refer to the [helpers](lib/jnprwlc/helpers) directory.

Here is an example using the builin helper to save the configuration:
````python
wlc.ez.save_config()
````

For more details on using the "EZ" framework, see [here](docs/ez_framework.md).
## LOGGING

Each WLC instance can support transaction logging.  You can use this facility by assigning an open file to the WLC instance, for example:
````python
wlc.logfile = open(r'/var/tmp/'+wlc.hostname+'.xml', "w+")
````
The contents of the log file are the XML commands and assocaited responses.  Each transaction will flush the results to the file.  You are required to perform any file close/cleanup.

## TEMPLATING

This module supports using Jinja2 templates in conjunction with the creation of complex RPCs.  Template files can be located in either the program's current working directory, or template directory for this module.  A configurable search-path option will be added as an enhancement. 

There are a few options for using templates.  These are described in detail [here]().

The following uses a template file [vlan_set_ports.xml](https://github.com/jeremyschulman/py-jnprwlc/blob/master/lib/jnprwlc/templates/vlan_set_ports.xml) that happens to be stored in the module template directory.
````python
vlan_vars = dict(
  number = 100,
  ports = [
    dict(port=2, tag=50),
    dict(port=3)
  ]
)

rpc = wlc.RpcMaker('set', Template='vlan_set_ports', TemplateVars=vlan_vars )

print "Settting ports on VLAN ..."
rsp = rpc()
````


## EXCEPTIONS

  This module provides an `RpcError` exception, which inherits from StandardError.  This exception will be raised if the RPC response is an `ERROR-RESP`.
  The `RpcError` encapsulates both the RPC command and RPC response attributes; both stored as lxml Element.  For 
  example usage, see [this](https://github.com/jeremyschulman/py-jnprwlc/blob/master/examples/try_except.py).
  
  
## DEPENDENCIES

  * [Python 2.7](http://www.python.org/)
  * [lxml](http://lxml.de/index.html)
  * [jinja2](http://jinja.pocoo.org/docs)

## LICENSE
  Apache 2.0

## CONTRIBUTORS

  * Jeremy Schulman, @nwkautomaniac
  * Tim McCarthy
