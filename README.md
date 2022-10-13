# Car Rental System

[System Design](v3/README.md)

Implementation of a system consisting of several services interacting with each other.

## Microservices

* Each service has its own storage if it needs it. For educational purposes, you can use one database, but each system works only with its own schema. Requests to another schema are prohibited.
* Use HTTP for inter-service communication (RESTful).
* Gateway Service is a single point of entry and inter-service communication. Horizontal requests between services cannot be made.
* GitHub Actions are used for building and automated testing.
* Deploy on Heroku with GitHub Actions.
* Each service is wrapped in Docker.
* The classroom.yml describes the steps for building, running unit-tests, and deploying

## Fault Tolerance

* On the Gateway Service, for all read operations, implement the Circuit Breaker pattern. Accumulate statistics in memory, and if the system did not respond N times, then immediately give fallback instead of a N+1 request. After a short timeout, make a request to the real system to check its status.
* In case of unavailability of data from a non-critical source (not the main one), a fallback response is returned. Depending on the situation, this may be:
  + empty object or array;
  + an object with a filled field (uid or similar), through which there is a connection with another system;
  + default string (if this does not change the type of the variable).
* If one of the systems participating in the operation that changes the state of several systems is unavailable, perform:
  + rollback of the entire operation;
  + return to the user a response about the successful completion of the operation, and add this request to queue for re-execution on the Gateway Service.

## Deploy to Cloud

* Deployment on Google Kubernetes Engine. 
* Configuration of the Ingress Controller.
* Build and publish docker images to the Docker Registry.
* Describe manifests for deployment in the form of Helm Charts, that should be universal for all services and differ only in a set of launch parameters.
* In a k8s cluster, use one database, but each service must work only with its own schema.
* Add steps to classroom.yml:
  + application assembly;
  + building a docker image;
  + updating images in Docker Hub;
  + deploy each service to a k8s cluster.

## OAuth2 Authorization

* Use OpenID Connect for authorization, use a third-party solution as an Identity Provider (Auth0).
* Implement Authorization flow on Gateway.
* On the Identity Provider, configure the ability to use the Resource Owner Password flow.
* On the Gateway Service, implement the GET /api/v1/authorize method to initiate the OAuth2 Authorization Flow process and GET /api/v1/callback to process the callback from the Identity Provider to exchange code for a token.
* All other /api/** methods (except /api/v1/authorize and /api/v1/callback) on the Gateway are closed with token-based authorization.
* Use JWT as a token, use JWKs to validate the token, you do not need to make a request to the Identity Provider.
* Remove the X-User-Name header and get the user from the JWT token.
* If the authorization is incorrect (token missing, JWT token validation error, token lifetime expired (exp field in payload)), then return a 401 error.
* Implement cross-service authorization by forwarding a JWT token between services. Token validation is also implemented through JWKs.
