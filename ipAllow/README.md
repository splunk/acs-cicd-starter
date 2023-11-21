https://docs.splunk.com/Documentation/SplunkCloud/8.2.2112/Config/ConfigureIPAllowList

ACS manages IP allow lists via additions and subtractions rather than submitting the state we would like.  The ipAllow list files in this folder are the desired state.

When deploying changes to IP allow lists the automation validates what is in git against the current state of the Splunk Cloud stack.  Additions and subtractions are then made to ensure Splunk Cloud resembles the configuration within git.

example payload:

{
  "subnets": [
     ": #.0.0.0/24",
     ": #.0.0.0/24",
     ": #.0.10.6/32"
  ]
}