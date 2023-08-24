https://docs.splunk.com/Documentation/SplunkCloud/8.2.2112/Config/ConfigureIPAllowList

DD - If on merge there are changes to these files CD pipeline diffs new configuration against configuration running in Splunk Cloud and removes entries that are not in local version and adds entries that are, aim is not to make unnecessary calls.

DD - Targetting Victoria initially so not including IDM UI access and IDM API access

example payload:

{
  "subnets": [
     ": #.0.0.0/24",
     ": #.0.0.0/24",
     ": #.0.10.6/32"
  ]
}