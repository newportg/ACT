# 3, System Scope and Context

## Business Context

A internal user will have the ability to pass property and occupier information, to this service and receive a property report at scheduled times. 
A internal user will have the ability to 

![image](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/newportg/ACT/master/plantuml/BusinessContext.puml)

## Technical Context

![image](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/newportg/ACT/master/plantuml/TechnicalContext.puml)


* The API runs as a microservice on our cloud platform, and will be accessible via a API Management interface.
* The connection to the external Address provider is over HTTPS, secured by OAuth2.
* The data passed in this application is not sensitive and doesn't need any encryption protocols.
* Any data retuned by the external provider will be stored, by our document management and scanned for viruses.
* A service bus interface will be available, for asynchronous queueing of jobs.
* A http interface will be available, for ad-hoc querying of information.

| Business Interface         | Channel          |
| -------------------------- | ---------------- |
| API for business functions | internet (https) |
| API for admin / audit      | internet (https) |
| Bulk Updates               | Service Bus      |
