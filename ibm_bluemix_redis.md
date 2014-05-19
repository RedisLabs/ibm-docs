[Redis Cloud](http://redislabs.com/redis-cloud) is a fully-managed cloud service for hosting and running your Redis dataset in a highly-available and scalable manner, with predictable and stable top performance. You can quickly and easily get your apps up and running with Redis Cloud through its add-on for Codename: BlueMix, just tell us how much memory you need and get started instantly with your first Redis database.
 
Redis Cloud offers true high-availability with its in-memory dataset replication and instant auto-failover mechanism, combined with its fast storage engine. You can easily import an existing dataset to any of your Redis Cloud databases, cloud storage or from any other Redis server. Daily backups are performed automatically and in addition, you can backup your dataset manually at any given time. The service guarantees high performance, as if you were running the strongest cloud instances.

## Creating A Redis Cloud Service
You can use the Codename: [BlueMix web user interface](https://ace.ng.bluemix.net/) or the [`cf` command line interface](http://www.ng.bluemix.net/docs/BuildingWeb.jsp#install-cf) to create an instance of the Redis Cloud service. The following `cf` command creates the service instance:

	$ cf create-service rediscloud PLAN SERVICE_INSTANCE

where `PLAN` is the desired plan (e.g. the free 25mb plan) and `SERVICE_INSTANCE`is the name of the service instance.

## Binding Your Redis Cloud Service
Use the web user interface or the command line interface to bind a Redis Cloud service instance to an application. When using `cf`, the following command binds the instance to the application:

	$ cf bind-service APP SERVICE_INSTANCE
	
where `APP` is the name of the application and `SERVICE_INSTANCE` is the name of the service instance.

Once your Redis Cloud service is bound to your app, the service credentials will be stored in the `VCAP_SERVICES` environment variable in the following format:
	
    {
       "rediscloud-null": [
          {
             "name": "dev-realtime-agg",
             "label": "rediscloud-null",
             "plan": "25mb",
             "credentials": {
                "port": "6379",
                "hostname": "pub-redis-6379.dal-05.1.sl.redislabs.com",
                "password": "..."
             }
          }
       ]
    }

* [Ruby](#ruby)
* [Rails](#rails)
* [Sinatra](#sinatra)
* [Java](#java)
* [Node.js](#node)

## <a id="ruby"></a>Using Redis from Ruby
The [redis-rb](https://github.com/redis/redis-rb) is a very stable and mature Redis client and is probably the easiest way to access Redis from Ruby. 

Install redis-rb:
	
	sudo gem install redis

### <a id="rails"></a>Configuring Redis from Rails
For Rails 2.3.3 up to Rails 3.0, update the `config/environment.rb` to include the redis gem:
	
	config.gem 'redis' 

For Rails 3.0 and above, update the Gemfile:
	
	gem 'redis'  
	
And then install the gem via Bundler:

	bundle install

Lastly, create a new `redis.rb` initializer in `config/initializers/` and add the following code snippet: 
	
	rediscloud_service = JSON.parse(ENV['VCAP_SERVICES'])["rediscloud-null"]
	credentials = rediscloud_service.first["credentials"]
    	$redis = Redis.new(:host => credentials.hostname, :port => credentials.port, :password => credentials.password)

### <a id="sinatra"></a>Configuring Redis on Sinatra
Add this code snippet to your configure block:

	configure do
        . . .
		require 'redis'
		rediscloud_service = JSON.parse(ENV['VCAP_SERVICES'])["rediscloud-null"]
		credentials = rediscloud_service.first["credentials"]
		$redis = Redis.new(:host => credentials.hostname, :port => credentials.port, :password => credentials.password)
        . . .
	end

### Testing (Ruby)
	$redis.set("foo", "bar")
	# => "OK"
	$redis.get("foo")
	# => "bar"
	
## Using Redis from Java
[Jedis](https://github.com/xetorthio/jedis) is a blazingly small, sane and easy to use Redis java client. You can download the latest build from [github](http://github.com/xetorthio/jedis/downloads) or use it as a maven dependency:

	<dependency>
		<groupId>redis.clients</groupId>
		<artifactId>jedis</artifactId>
		<version>2.0.0</version>
		<type>jar</type>
		<scope>compile</scope>
	</dependency>

Configure the connection to your Redis Cloud service using the `VCAP_SERVICES` environment variable and the following code snippet:

	try {
		String vcap_services = System.getenv("VCAP_SERVICES");
		if (vcap_services != null && vcap_services.length() > 0) {
			// parsing rediscloud credentials
			JsonRootNode root = new JdomParser().parse(vcap_services);
			JsonNode rediscloudNode = root.getNode("rediscloud-null");
			JsonNode credentials = rediscloudNode.getNode(0).getNode("credentials");
			
			JedisPool pool = new JedisPool(new JedisPoolConfig(),
		    		credentials.getStringValue("hostname"),
		    		Integer.parseInt(credentials.getNumberValue("port")),
		    		Protocol.DEFAULT_TIMEOUT,
		    		credentials.getStringValue("password"));				
		}			
	} catch (InvalidSyntaxException ex) {
		// vcap_services could not be parsed.
	} 
	
### Testing (Java)
	jedis jedis = pool.getResource();
	jedis.set("foo", "bar");
	String value = jedis.get("foo");
	// return the instance to the pool when you're done
	pool.returnResource(jedis);

(example taken from Jedis docs).

## <a id="node"></a>Using Redis from Node.js
[node_redis](https://github.com/mranney/node_redis) is a complete Redis client for node.js. 

You can install it with:

	npm install redis

Configure the connection to your Redis Cloud service using `VCAP_SERVICES` environment variable and the following code snippet:
	
	// parsing rediscloud credentials
	var vcap_services = process.env.VCAP_SERVICES;
	var rediscloud_service = JSON.parse(vcap_services)["rediscloud-null"][0]
	var credentials = rediscloud_service.credentials;
	
	var redis = require('redis');
	var client = redis.createClient(credentials.port, credentials.hostname, {no_ready_check: true});
	client.auth(credentials.password);
	
###Testing (Node.js)
	client.set('foo', 'bar');
	client.get('foo', function (err, reply) {
        console.log(reply.toString()); // Will print `bar`
    });
	
## Dashboard
Our dashboard presents all performance and usage metrics of your Redis Cloud service on a single screen, as shown below:

![Dashboard](https://s3.amazonaws.com/paas-docs/redis-cloud/ibm_bluemix_redis_cloud_dashboard.png)

To access your Redis Cloud dashboard, click the service instance in your Code Name: BlueMix console and then the **LAUNCH REDISCLOUD DASHBOARD** button.

When using the Redis Cloud console, you can then find the dashboard using the **MY DATABASES->Dashboard** menu item.

## Support
Any Redis Cloud support issues or product feedbacks are welcome via email at support@redislabs.com or from the service's console under the **SUPPORT->Helpdesk** menu item.

## Additional resources
* [Developers Resources](http://redislabs.com/redis-cloud)
* [Redis Documentation](http://redis.io/documentation)
