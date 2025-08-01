# Hello World!

As a self-tutorial, will build own "Hello World" image, host it on docker.io. Next, will pull as a deployment and service, then expose it to the host.
First, we will use the temporary port-forwarding to open up the pod to the host.
Next, we will enable a permanent port mapping to reveal the pod
Goal is to have it working right through to using an ingress controller

## Notes

I found it difficult to follow how ports are mapped / opened so will document what I believe to be working here:

* In the deployment, we have `containerPort: 80`

  * containerPort is informational, not operational

* In the service, we have exposed `port: 80` (read: listening on port 80)

* We have also instructed the service to forward the traffic to our pod on port 80 with `targetPort: 80`

* The service also has a `nodePort: 30080` that ensures the cluster is able to forward the traffic to our service

* The cluster is listening for incoming traffic on `hostPort: 8080`

* The cluster will forward the traffic it receives to its `containerPort: 30080` so it matches the `nodePort: 30080` of the service


