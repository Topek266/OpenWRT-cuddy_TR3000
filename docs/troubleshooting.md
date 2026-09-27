
# Troubleshotting

1. Lost access to the router after incorrect VLAN configuration

Problem:
I lost acces to the LuCi administration panel and SSH because of an incorrect VLAN configuration. I had to reset the router to factory settings to regain access.

Solution:
I created a separate management bridge witch access to LuCi and SSH. This provides a recovery patch in case make a configuration mistake and lose access through the main VLAN setup. After VLAN 10 (Management) was configured and successfully tested, I removed the temporary managment bridge.
