
#####HW #4 Deployment 
================

### Setup

* Clone the Deployment repo, and follow the instructions as shown in the workshop to deploy HW3 application (https://github.com/mahasanath/HW3Queue.git) using blue and green strategy.
* Install redis and run on localhost:6379 and localhost:6364. This can be achieved through change in config files and running each server instance.
* Run "node infrastructure.js" , the web application running is ServerInstance1.js.


##### Task 1: Complete git/hook setup     

> -w and --watchDir=deploy/blue-www/ are added to the 'exec' command to facilitate auto-deployment and refresh without having restarted the system. Also, the hooks/post-receive file setup is a git/hookup setup. The snapshot shows the setup:


post-receive file in 'hooks':

````

````

> /set is completed as follows, and the client.expire command is used to set timeout of 10 s for the key.

```
 app.get('/set',function(req,res){
	client.lpush("history",req.url);
	client.set("key", "this message will destruct in 10 sec");
	client.expire("key",10);
	res.send('Success!! - Value added for the key in redis on port 3001');
});
```
> /get is completed as follows: 

```
> app.get('/get',function(req,res){
	client.lpush("history",req.url);
	client.get("key",function(err,value){
	res.send(value)
})
})

res.send(value) sends back the value associated with the key in the request to the client
```      

##### Task 2: Create blue/green infrastructure:    

> HW3 application on deployment is in blue-www and green-www. Both are running on two different redis instances. The blue instance running on port number 5060 is connected to redis instance running on port number 6379 and green instance running on port number 9090 is connected to redis instance running on port number 6364. 

> To enable traffic routing to blue infrastructure by default, change TARGET = GREEN to TARGET = BLUE in 'infrastructure.js' as shown below.



##### Task 3: Demonstrate /switch route     

> upload is completed by running the following CURL command through command promt. Upload is implemented on all three ports 3000,3001 and 3002. If uploaded to ports 3001 or 3002,      

```
curl -F "image=@./img/morning.jpg" localhost:3001/upload  
Visit 'meow' to test the "queue" functionality:   
	http://localhost:3001/meow   
```
> It displays the most recently uploded image and remove that image from the queue. Thus establishing the queue functionality.    
    
    
>  upload is completed as follows by adding lpush to push uploaded images to the queue.

```
	client.lpush('images',img);
```   
##### Task 4: Additional service instance running    

> Additional service is completed by adding another server instance at 'port 3002'. This is run as a child process using main.js
 
> get, set, recent, upload and meow have been implemented in this server as well. Client requests can be made using    
"http://localhost:3002/". For example, http://localhost:3002/get

##### Task 5: Demonstrate proxy   

> Proxy is implemented in two ways. One method in which I have implemented proxy using "http-proxy" node module, proxy behaves as a load balancer. It redirects all requests on port 80 to either 3001 or 3002 using the round robin scheme. Sample code from simple-balancer.js from the http-proxy library has been used. The code is added to the repository as ProxyHtpp.js.   

> Another method is using an other server instance running on port 3000 ( node proxyserver1.js ) which would relay client requests that it receives and redirects it to ports 3001 or 3002. 

> So for example, a upload on port 3000 has been made using the CURL command.  A meow request made on port 3000, http://localhost:3000/meow would be serviced by either http://localhost:3001/meow or http://localhost:3002/meow, thereby displaying the uploaded images and then removed from the queue.   

> proxy server is implemented using  "rpoplpush command" to toggle between ports and http 'redirect' is used to re-route the request.
