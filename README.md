# E-commerce system design

![E-commerce system design](/PESI_Q2_Design.png)


## Choice of database 
* Product information from different vendors can have varying degrees of information. The product service will want to show all information (key value pairs like ISBN10: xyz, ISBN12: abc) on the website.  As such, it makes sense to use a no sql database for storing product details.
* We should use a relational database for information, where schema is known, such as user data and order data.

## App service layer services
* Product catalogue service – Returns all products and products by id
* Search service - The search service is going to azure search (elastic search) to provide fast responses to customer’s search queries.
* Payment service – This service will offer both credit card and PayPal option. Upon payment completion, service will send a message to service bus.
* Service bus queue – This bus will call the receipt service and also the inventory updating service.
* Receipt service – This service will generate email receipt by pulling information from the database using orderId. It will then send an email to the user and also update the database.
* Redis cache – Service requests should be cached to respond to duplicate requests quicker.


## Nonfunctional requirements
* Scalability – As per requirements, traffic boost happens at known times (sales initiated by the ecommerce biz). The app service allows automatic scaling. We could also use manual scaling if we don’t want to risk Azure making decisions. On the database layer, azure offers read replicas for database that can be used during high loads (higher load during sales will have more impact on reads than writes)
* Performance – The indexing capability of elastic search offer low response time for search queries.
* Affordability - 
   * The auto scaling capabilities of Azure makes costs go up only during sales. 
   * We could also explore azure functions and other serverless architectural resources that are now claimed to have reasonable cold latency. 
   * Overall, we cannot build a scalable, highly available, and low latency user experience, without incurring costs. This cost argument should be discussed with management and agreement should be reached on what level of customer experience is ultimately acceptable.
   
## Risk Mitigation
* Email processor can run in parallel. Considering the nature of this process, there will not be any concurrency issues.
* Health monitoring
   * Azure offers out of the box resources for monitoring health of services and will spin out new resources if needed.
   * If the above approach is not an option, we could implement our own health monitoring. The message bus architecture allows creating a new component that should receive a heart-beat from each service. The heart beat can be a response to dummy request at frequent intervals. This approach slightly increases the load on the system but allows full telemetry on the state of the system.
   
## Additional Thoughts
* Azure DNS – Design suggest using azure dns for fast DNS look up
* Sticky sessions – The load balancers should be configured such that all requests from a session are sent to the same server.
* Geographic clustering – We could implement google analytics and learn about geography demographics and create multiple clusters (example us-east, us-west and Midwest). The traffic can then be directed to the nearest datacenter using Azure traffic manager.
* Regional backup – To offer high availability we need to create an active or a passive backup. Azure recommends doing this in the same region (They try to revive a datacenter in each region during broad outages)


