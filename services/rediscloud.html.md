---
title: Redis Cloud
---

[Redis Cloud](http://redislabs.com/redis-cloud) is a fully-managed cloud service for hosting and running your Redis dataset in a highly-available and scalable manner, with predictable and stable top performance. You can quickly and easily get your apps up and running with Redis Cloud through its add-on for Cloud Foundry, just tell us how much memory you need and get started instantly with your first Redis database.

Redis Cloud offers true high-availability with its in-memory dataset replication and instant auto-failover mechanism, combined with its fast storage engine. You can easily import an existing dataset to any of your Redis Cloud databases, from your S3 account or from any other Redis server. Daily backups are performed automatically and in addition, you can backup your dataset manually at any given time. The service guarantees high performance, as if you were running the strongest cloud instances.

## <a id="managing-services"></a>Managing Services ##

### <a id="create-service"></a>Creating a Redis Cloud Service Instance ###

Create a Redis Cloud service instance with the following command:

<pre class="terminal">
$ cf create-service rediscloud PLAN_NAME INSTANCE_NAME
</pre>

where `PLAN_NAME` is the desired plan's name, and `INSTANCE_NAME` is a name meaningful to you.

### <a id="bind-service"></a>Binding Your Redis Cloud Service Instance ###

Bind your Redis Cloud service to your app, using the following command:

<pre class="terminal">
$ cf bind-service APP_NAME INSTANCE_NAME
</pre>

Once your Redis Cloud service is bound to your app, the service credentials will be stored in the `VCAP_SERVICES` environment variable in the following format:

	{
	  "rediscloud": [
	    {
	      "name": "rediscloud-42",
	      "label": "rediscloud",
	      "plan": "20mb",
	      "credentials": {
		    "port"": "6379",
			"hostname": "pub-redis-6379.us-east-1-2.3.ec2.redislabs.com",
			"password": "your_redis_password"
	      }
	    }
	  ]
	}
For more information, see [Binding Applications to Service Instances](/devguide/services/application-binding.html#use) and [VCAP_SERVICES Environment Variable](/devguide/deploy-apps/environment-variable.html).

### <a id="ruby"></a>Using Redis from Ruby ###

The [redis-rb](https://github.com/redis/redis-rb) is a very stable and mature Redis client and is probably the easiest way to access Redis from Ruby.

Install redis-rb:

	sudo gem install redis

#### <a id="rails"></a>Configuring Redis from Rails

For Rails 2.3.3 up to Rails 3.0, update the `config/environment.rb` to include the redis gem:

	config.gem 'redis'

For Rails 3.0 and above, update the Gemfile:

	gem 'redis'

And then install the gem via Bundler:

	bundle install

Lastly, create a new `redis.rb` initializer in `config/initializers/` and add the following code snippet:

	rediscloud_service = JSON.parse(ENV['VCAP_SERVICES'])["rediscloud"]
	credentials = rediscloud_service.first["credentials"]
    $redis = Redis.new(:host => credentials["hostname"], :port => credentials["port"], :password => credentials["password"])

#### <a id="sinatra"></a>Configuring Redis on Sinatra

Add this code snippet to your configure block:

	configure do
        . . .
		require 'redis'
		rediscloud_service = JSON.parse(ENV['VCAP_SERVICES'])["rediscloud"]
		credentials = rediscloud_service.first["credentials"]
		$redis = Redis.new(:host => credentials.hostname, :port => credentials.port, :password => credentials.password)
        . . .
	end

#### <a id="unicorn"></a>Using Redis on Unicorn

No special setup is required when using Redis Cloud with a Unicorn server. For Rails apps on Unicorn, follow the instructions in the [Configuring Redis from Rails](#rails) section and for Sinatra apps on Unicorn see [Configuring Redis on Sinatra](#sinatra) section.

#### <a id='testing-ruby'></a>Testing (Ruby)

	$redis.set("foo", "bar")
	# => "OK"
	$redis.get("foo")
	# => "bar"

### <a id='java'></a>Using Redis from Java ###

[Jedis](https://github.com/xetorthio/jedis) is a blazingly small, sane and easy to use Redis java client. You can download the latest build from [Github](http://github.com/xetorthio/jedis/downloads) or use it as a maven dependency:

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
			JsonNode rediscloudNode = root.getNode("rediscloud");
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

#### <a id='testing-java'></a>Testing (Java)

	jedis jedis = pool.getResource();
	jedis.set("foo", "bar");
	String value = jedis.get("foo");
	// return the instance to the pool when you're done
	pool.returnResource(jedis);

(example taken from Jedis docs).

### <a id="python"></a>Using Redis from Python ###

[redis-py](https://github.com/andymccurdy/redis-py) is the most commonly-used client for accessing Redis from Python.

Use pip to install it:

	sudo pip install redis

Configure the connection to your Redis Cloud service using `VCAP_SERVICES` environment variable and the following code snippet:

	import os
	import urlparse
	import redis
	import json

	rediscloud_service = json.loads(os.environ['VCAP_SERVICES'])['rediscloud'][0]
	credentials = rediscloud_service['credentials']
	r = redis.Redis(host=credentials['hostname'], port=credentials['port'], password=credentials['password'])


#### <a id='testing-python'></a>Testing (Python)

	>>> r.set('foo', 'bar')
	True
	>>> r.get('foo')
	'bar'

#### <a id="django"></a>Django

[Django-redis-cache](https://github.com/sebleier/django-redis-cache)

Redis can be used as the back-end cache for Django. To do so, install django-redis-cache:

	pip install django-redis-cache

Next, configure your `CACHES` in the `settings.py` file:

	import os
	import urlparse
	import json

	rediscloud_service = json.loads(os.environ['VCAP_SERVICES'])['rediscloud'][0]
	credentials = rediscloud_service['credentials']
	CACHES = {
		'default': {
		'BACKEND': 'redis_cache.RedisCache',
		'LOCATION': '%s:%s' % (credentials['hostname'], credentials['port']),
		'OPTIONS': {
            'PASSWORD': credentials['password'],
			'DB': 0,
        }
	  }
	}

#### <a id='testing-django'></a>Testing (Django)

	from django.core.cache import cache
	cache.set("foo", "bar")
	print cache.get("foo")

### <a id="php"></a>Using Redis from PHP ###

[Predis](https://github.com/nrk/predis) is a flexible and feature-complete PHP client library for Redis.

Instructions for installing the [Predis](https://github.com/nrk/predis) library can be found [here](https://github.com/nrk/predis#how-to-use-predis).

Loading the library to your project should be straightforward:

	<?php
	// prepend a base path if Predis is not present in your "include_path".
	require 'Predis/Autoloader.php';
	Predis\Autoloader::register();

Configure the connection to your Redis Cloud service using `VCAP_SERVICES` environment variable and the following code snippet:

	// parsing rediscloud credentials
	$vcap_services = getenv("VCAP_SERVICES");
	$rediscloud_service = json_decode($vcap_services, true)["rediscloud"][0]
	$credentials = $rediscloud_service["credentials"]

	$redis = new Predis\Client(array(
		'host' => $credentials["hostname"],
		'port' => $credentials["port"],
		'password' => $credentials["password"],
	));

#### Testing (PHP)

	$redis->set('foo', 'bar');
	$value = $redis->get('foo');

### <a id="node"></a>Using Redis from Node.js ###

[node_redis](https://github.com/mranney/node_redis) is a complete Redis client for node.js.

You can install it with:

	npm install redis

Configure the connection to your Redis Cloud service using `VCAP_SERVICES` environment variable and the following code snippet:

	// parsing rediscloud credentials
	var vcap_services = process.env.VCAP_SERVICES;
	var rediscloud_service = JSON.parse(vcap_services)["rediscloud"][0]
	var credentials = rediscloud_service.credentials;

	var redis = require('redis');
	var client = redis.createClient(credentials.port, credentials.hostname, {no_ready_check: true});
	client.auth(credentials.password);

#### Testing (Node.js)

	client.set('foo', 'bar');
	client.get('foo', function (err, reply) {
        console.log(reply.toString()); // Will print `bar`
    });

## <a id='dashboard'></a>Dashboard ##

Our dashboard presents all performance and usage metrics of your Redis Cloud service on a single screen, as shown below:

![Dashboard](https://s3.amazonaws.com/paas-docs/redis-cloud/CF+-+redis.png)

To access your Redis Cloud dashboard, click the 'Manage' button next to the
Redis Cloud service in Apps Manager.
You can find your dashboard under the `MY DATABASES` menu.

## <a id='support'></a>Support ##

Any Redis Cloud support issues or product feedbacks are welcome via email at support@redislabs.com.
Please make sure you are familiar with the CloudFoundry method of [contacting service providers for support](../contacting-service-providers-for-support.html).

## <a id='additional'></a>Additional Resources ##

* [Developers Resources](http://redislabs.com/redis-cloud)
* [Redis Documentation](http://redis.io/documentation)
