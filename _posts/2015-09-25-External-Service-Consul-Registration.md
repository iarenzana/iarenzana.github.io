---
title:  "Consul Registration in Go"
date:   2015-09-25 19:40:00
description: "Register third party services with Consul using its Go API "
categoy: Tech
tags: golang development consul microservices programming go
---

Consul Registration in Go
==

Looking around the web, I haven't been able to find good practical examples of how to register external services with Consul using Go. Sure, I could always use the REST interface, but I wanted to use the native API to interact with it. That's why I decided to write a little article on how to do external service registrations using Consul's native API.

What is Consul?
-

[Consul](http://consul.io) is a service discovery application that runs as a service. What is service discovery? Well, in a microservices-based platform, processes need to be able to locate processes offering certain services. Consul provides a solid solution to this problem. For a service to be available through consul, it needs to be able to register itself or have some other process register it. This last scenario is less desirable, but sometimes necessary when trying to register third party services. Below is an image of what Consul's UI looks like.

![IMAGE 1](https://raw.githubusercontent.com/iarenzana/iarenzana.github.io/master/assets/images/2015/09/20150925-consul-ui.png)

Our Goal
-

I'm working on integrating [Sendgrid](https://sendgrid.com) and [Twilio](https://twilio.com) with Consul to be able to create a solid notification microservice in Go.

How to register an external service in Go
--

As I pointed out before, you can register services using Consul's REST API, but in this case we will use Consul's native Go API.
Sendgrid's API hostname is `api.sendgrid.com` and Twilio's is `api.twilio.com`. While it's not really necessary to create a new service for these two APIs (we don't expect either one of them to change), it's good practice to add this abstraction layer to avoid having to re-release a service urgently because something changed.

First we need to import the Consul API into our Go program.

```go
package main

import (
	"math/rand"
	"time"
	"fmt"
	"github.com/hashicorp/consul/api"
)
```

I'm also importing `math/rand`, `fmt` and `time` because we'll use them later as well. Next step is to create a client instance. To do this we will create a Default config indicating where Consul is. 8500 is the default port for Consul and I'm running it locally.

```go
config := api.DefaultConfig()
config.Address = "localhost:8500"
client, err := api.NewClient(config)

if err != nil {
	fmt.Printf("Encountered error connecting to consul on %s => %s\n", hostname, err)
	return
}
```

If you had an error connecting to Consul, make sure it's up and listening on the right port using the following command (*NIX only):

```bash
telnet localhost 8500
```

or:

```bash
nestat -a |grep 8500
```

No errors? Great. Now that we have a client, it's time to create an agent instance using the client we've already created. 
```go
srvRegistrator := client.Agent()
```

Now off to the actual meat. I created a function to do the registration and I would only have to pass the required parameters.

```go
func RegisterService(ag *api.Agent, service, hostname, node, protocol string, port int) error {

	rand.Seed(time.Now().UnixNano())
	sid := rand.Intn(65534)

	serviceID := service + "-" + strconv.Itoa(sid)

	consulService := api.AgentServiceRegistration{
		ID:   serviceID,
		Name: service,
		Tags: []string{time.Now().Format("Jan 02 15:04:05.000 MST")},
		Port: port,
		Check: &api.AgentServiceCheck{
			Script:   "curl --connect-timeout=5 " + protocol + "://" + hostname + ":" + strconv.Itoa(port),
			Interval: "10s",
			Timeout:  "8s",
			TTL:      "",
			HTTP:     protocol + "://" + hostname + ":" + strconv.Itoa(port),
			Status:   "passing",
		},
		Checks: api.AgentServiceChecks{},
	}

	err := ag.ServiceRegister(&consulService)
	if err != nil {
		return err
	}

	return err
}
```

Let me explain this step by step.

```go
func RegisterService(ag *api.Agent, service, hostname, node, protocol string, port int) error {

	rand.Seed(time.Now().UnixNano())
	sid := rand.Intn(65534)

	serviceID := service + "-" + strconv.Itoa(sid)
```

We create the function which will take all the required parameters and return an error.
We will generate a pseudo-unique id, this is not technically required, but it almost guarantees that we won't have 2 identical services on the same node. Next, we create a service name using the id and the service name.

```go
consulService := api.AgentServiceRegistration{
	ID:   serviceID,
	Name: service,
	Tags: []string{time.Now().Format("Jan 02 15:04:05.000 MST")},
	Port: port,
	Check: &api.AgentServiceCheck{
		Script:   "curl --connect-timeout=5 " + protocol + "://" + hostname + ":" + strconv.Itoa(port),
		Interval: "10s",
		Timeout:  "8s",
		TTL:      "",
		HTTP:     protocol + "://" + hostname + ":" + strconv.Itoa(port),
		Status:   "passing",
	},
	Checks: api.AgentServiceChecks{},
}
```

Now we're creating a service registration instance. This will contain all the service registration information that we will send to Consul. We pass the ID, the service name, we're tagging the service with the time it was created, the port, and a check.
The check will be a very basic curl that will run every 10 seconds. If curl returns something other than 0 there's an error and will put the service into a failing state and therefore, considered unavailable.

```go
	err := ag.ServiceRegister(&consulService)
	if err != nil {
		return err
	}
	
	return nil
}
```

Finally, we trigger the registration and return the error if any, otherwise, return nil.

How do we call this function to register Sendgrid and Twilio? Very simple:

```go
if err = RegisterService(srv, "sendgrid", "api.sendgrid.com", "sendgrid", "https", 443); err != nil {
	fmt.Printf("Encountered error registering a service with consul -> %s\n", err)
}

if err = RegisterService(srv, "twilio", "api.twilio.com", "twilio", "https", 443); err != nil {
	fmt.Printf("Encountered error registering a service with consul -> %s\n", err)
}
```

We will call our function using `sendgrid` as the service name (don't worry, our function will generate a unique id, remember?) the API host indicated by the services documentation, in this case `api.sendgrid.com`, the node is where the service runs; because we don't really know that, we can just pass `sendgrid`, think of it as the machine hosting the service. Protocol is `https` and the port is `443`.

Now it's time to make sure our program does what it's supposed to do and it's able to register external services. To do this, you can just run:

```bash
curl -s localhost:8500/v1/catalog/services |python -m json.tool
```

This should return something like this:

```json
{
    "consul": [],
    "sendgrid": [
        "Sep 25 18:11:28.087 CEST"
    ],
    "twilio": [
        "Sep 25 18:11:28.090 CEST"
    ]
}
```
For more details on a service in particular, run this:

```bash
curl -s localhost:8500/v1/catalog/service/sendgrid |python -m json.tool
```

And it will return something similar to this:

```json
[
    {
        "Address": "192.168.1.36",
        "Node": "exodo.local",
        "ServiceAddress": "",
        "ServiceID": "sendgrid-29982",
        "ServiceName": "sendgrid",
        "ServicePort": 443,
        "ServiceTags": [
            "Sep 25 18:11:28.087 CEST"
        ]
    }
]
```

Summary
-

Consul and Go can work very well together, with little effort you can register new services and be able to reach other services registered as well.