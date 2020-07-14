## YAML Basics ##
It’s difficult to escape YAML if you’re doing anything related to many software fields — particularly Kubernetes, SDN, and OpenStack. 
YAML, which stands for Yet Another Markup Language, or YAML Ain’t Markup Language (depending who you ask) is a human-readable text-based format for specifying configuration-type information.

Using YAML for K8s definitions gives you a number of advantages, including:
1. Convenience: You’ll no longer have to add all of your parameters to the command line
2. Maintenance: YAML files can be added to source control, so you can track changes
3. Flexibility: You’ll be able to create much more complex structures using YAML than you can on the command line

YAML is a superset of JSON, which means that any valid JSON file is also a valid YAML file. So on the one hand, if you know JSON and you’re only ever going to write your own YAML (as opposed to reading other people’s) you’re all set.  On the other hand, that’s not very likely, unfortunately.  Even if you’re only trying to find examples on the web, they’re most likely in (non-JSON) YAML, so we might as well get used to it.  Still, there may be situations where the JSON format is more convenient, so it’s good to know that it’s available to you.

Fortunately, there are only two types of structures you need to know about in YAML:
* Lists
* Maps

That’s it. You might have maps of lists and lists of maps, and so on, but if you’ve got those two structures down, you’re all set. That’s not to say there aren’t more complex things you can do, but in general, this is all you need to get started.

### YAML Maps ###
Let’s start by looking at YAML maps.  Maps let you associate name-value pairs, which of course is convenient when you’re trying to set up configuration information.  For example, you might have a config file that starts like this:

```yaml
---
apiVersion: v1
kind: Pod
```

The first line is a separator, and is optional unless you’re trying to define multiple structures in a single file. From there, as you can see, we have two values, v1 and Pod, mapped to two keys, apiVersion and kind.

This kind of thing is pretty simple, of course, and you can think of it in terms of its JSON equivalent:

```json
{
   "apiVersion": "v1",
   "kind": "Pod"
}
```

Notice that in our YAML version, the quotation marks are optional; the processor can tell that you’re looking at a string based on the formatting.

You can also specify more complicated structures by creating a key that maps to another map, rather than a string, as in:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: rss-site
  labels:
    app: web
```

In this case, we have a key, metadata, that has as its value a map with 2 more keys, name and labels. The labels key itself has a map as its value. You can nest these as far as you want to.

The YAML processor knows how all of these pieces relate to each other because we’ve indented the lines. In this example I’ve used 2 spaces for readability, but the number of spaces doesn’t matter — as long as it’s at least 1, and as long as you’re CONSISTENT.  For example, name and labels are at the same indentation level, so the processor knows they’re both part of the same map; it knows that app is a value for labels because it’s indented further.

Quick note: NEVER use tabs in a YAML file.

So if we were to translate this to JSON, it would look like this:
```json
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
               "name": "rss-site",
               "labels": {
                          "app": "web"
                         }
              }
}
```

### YAML lists ###
YAML lists are literally a sequence of objects.  For example:

```yaml
args:
  - sleep
  - "1000"
  - message
  - "Bring back Firefly!"
```

As you can see here, you can have virtually any number of items in a list, which is defined as items that start with a dash (-) indented from the parent.  So in JSON, this would be:
```json
{
   "args": ["sleep", "1000", "message", "Bring back Firefly!"]
}
```

And of course, members of the list can also be maps:
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: rss-site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: rss-reader
      image: nickchase/rss-php-nginx:v1
      ports:
        - containerPort: 88
```

So as you can see here, we have a list of containers “objects”, each of which consists of a name, an image, and a list of ports.  Each list item under ports is itself a map that lists the containerPort and its value.

For completeness, let’s quickly look at the JSON equivalent:

```json
```

As you can see, we’re starting to get pretty complex, and we haven’t even gotten into anything particularly complicated! No wonder YAML is replacing JSON so fast.

So let’s review.  We have:

* maps, which are groups of name-value pairs
* lists, which are individual items
* maps of maps
* maps of lists
* lists of lists
* lists of maps

Basically, whatever structure you want to put together, you can do it with those two structures.
