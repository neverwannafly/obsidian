## Features
- [ ] Docker based deployments should be working
- [ ] Docker compose based deployments should be working
- [ ] Redis, MySQL compliant images need to be made available
- [ ] Better Observability/Platform for observability of consumption and other stuff. 
- [ ] Pricing model
- [ ] Portfolio View / Profile view where users can create their custom portfolios and everything integrates well with API frenzy platform
      This is a pretty ambitious Goal and maybe address this in some later releases
- [ ] Devops agent for Autodeployments
      Essentially I should be able to chat with an API frenzy agent and it should take care of all my deployment needs
- [ ] Function catalogue
      This was the original vision I had with API frenzy with features like function cloning etc. 

## Bugs/Improvements
- [ ] Separation of Function Service
      Have two versions of functions, one should be part of the `platform namespace` (which would be responsible for provisioning namespaces or anything else that needs admin priviledges) while another less priviledged function service should be running on a tenant specific usecase.
- [ ]  Deployment Edit + Redeploy functionality
      Deployments currently are not editable, also public/private visibility thing needs to be fixed. 
- [ ] User Profile Page
- [ ] Namespace/tenant provisioning (currently this is very flawed)
- [ ] Fix soft/hard delete functionality
- [ ] FE inconsistencies
- [ ] Oauth modal doesnt work in mobile
- [ ] Logs not working
- [ ] username namespace management is pretty buggy (i need cap on the max characters there and also find some cool way to have a username - cant rely on emails just)
- [ ] Function support for async executions and callback receiving on rails side