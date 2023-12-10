# Manage Private Apps

For detailed information, see official ACS Documentation: [Manage private apps in Splunk Cloud Platform](https://docs.splunk.com/Documentation/SplunkCloud/latest/Config/ManageApps).

Private Apps can be stored as their respective apps here, they are stored uncompressed to make editing easy.

The automation workflows will package the apps and validate them through app inspect before they are deployed.

The automation workflow adds the git commit short ID to the app version before packaging.

App Inspect will fail if a local directory exists within a private app here presently, so the automation workflows won't even attempt app validation if the app contains a local directory.