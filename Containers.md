# Intro to Containers
- Businesses run many applications and they used to buy actual servers to run those apps on them.
- first improvement was introduced through _Hypervisors_ (like VMware)
  - it lets you run different apps on different VMs that all run on the same server
  - so let's say you have 4 VMs and each use 25% of the server resources
  - each can have a different OS
  - managing each of these then requires many admin time, still not very easy.
- in container model, instead we install one OS on our server. On top of that we add one container per app which are faster and we don't have heavy processes of managing different VMs and OSes (!)
  - we have more space for more apps (instead of wasting on VMs and OS)
  - apps are much faster and take seconds to run  


  <img src="/resources/containerVsHypervisor.png" width="600" style="margin-left: 100px"/>
- Docker to containers is like VMware to hypervisors!
- Docker is written in Go Lang. It is open source.
- Docker hub and other container registries are becoming play store/apple store of enterprise apps
  - container registry = image registry 
