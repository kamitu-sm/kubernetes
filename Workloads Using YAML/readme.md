# Introduction #

You can easily administer your workloads exclusively on the command line, but there’s an easier and more useful way to do it: creating configuration files using YAML. 
In this article, we’ll look at how YAML works and use it to define various kubernetes objects that you will run into during your work.

## YAML Basics ##
It’s difficult to escape YAML if you’re doing anything related to many software fields — particularly Kubernetes, SDN, and OpenStack. YAML, which stands for Yet Another Markup Language, or YAML Ain’t Markup Language (depending who you ask) is a human-readable text-based format for specifying configuration-type information. For example, in this article, we’ll pick apart the YAML definitions for creating first a Pod, and then a Deployment.

Using YAML for K8s definitions gives you a number of advantages, including:

Convenience: You’ll no longer have to add all of your parameters to the command line
Maintenance: YAML files can be added to source control, so you can track changes
Flexibility: You’ll be able to create much more complex structures using YAML than you can on the command line
YAML is a superset of JSON, which means that any valid JSON file is also a valid YAML file. So on the one hand, if you know JSON and you’re only ever going to write your own YAML (as opposed to reading other people’s) you’re all set.  On the other hand, that’s not very likely, unfortunately.  Even if you’re only trying to find examples on the web, they’re most likely in (non-JSON) YAML, so we might as well get used to it.  Still, there may be situations where the JSON format is more convenient, so it’s good to know that it’s available to you.

Fortunately, there are only two types of structures you need to know about in YAML:

Lists
Maps
That’s it. You might have maps of lists and lists of maps, and so on, but if you’ve got those two structures down, you’re all set. That’s not to say there aren’t more complex things you can do, but in general, this is all you need to get started.
