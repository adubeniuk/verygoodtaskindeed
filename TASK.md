# System Engineering Task

* Create a Kubernetes cluster
* Use Weave to setup networking
* Add three hosts
    * Logger
    * API
    * DB
* Use Weave to setup the following network rules
    * API → Logger UDP port 514
    * API → DB TCP port 5432
    * DB → Logger UDP port 514
* Visualize this in a network diagram
* Write a script to prove that each machine can *only* connect to the machine(s) specified in the network rules above
