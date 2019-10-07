# Charon

Fully customizable OAuth server based on Spring security Oauth2.

## Getting Started

Once the project will be hosted on Maven repositories all this will not be mandatory, but in the meanwhile please follow it.

To make the project work you will need to copy the content of several repositories in your workspace.
The repositories required are:
- charon: Parent of the project allowing to manage the dependencies
- charon-authorization-server: The main implementation of the authorization server
- charon-resource-server: The main implementation of the resource server
- charon-core: The core is the parts shared between the authorization and the resource servers

Then you will need projects using the aforementioned libraries to have the servers running.
For example, to have an OAuth server using a Cassandra Database, you wil also need:
- The core implementation embedding all the Cassandra settings. (see charon-core-cassandra)
- The authorization server web application. (see charon-authorization-server-cassandra)
- The resource server web application (see charon-resource-server-cassandra-manager or charon-resource-server-cassandra-users)

Notes:
The resource server design allows to use it as a web services or as a monolithic application.
The "charon-resource-server-cassandra-manager" is an example of interface using Thymeleaf allowing to create its own clients and to manage users.
The "charon-resource-server-cassandra-users" is an example of REST API using scope to give access to the data.

See deployment for notes on how to deploy the project on a live system.

### Prerequisites

For the Cassandra modules, you will have to link it to your own Cassandra database. To do so, edit the properties file (/src/main/resources/application.yml) and change 'localhost' to your server name.
You may also want to change others settings, so have a look.

```
spring:
  data:
    cassandra:
      keyspace-name: oauth
      contact-points: __localhost__
      port: 9042
      schema-action: create_if_not_exists
```

## Deployment

Once everything is ready, just deploy your application. (Using Eclipse, you just have to right click on the main class of you project then select "Run as > Java Application")
The keyspace and the repositories will be automatically created if they don't already exist.
By default, the project "charon-resource-server-cassandra-manager" will create three scope profiles for the tests and an admin user if they don't already exists.

## Testing

To test the deployed servers you will have to create a client first using "charon-resource-server-cassandra-manager" or your own implementation.
Then get your CLIENT_ID and the secret password you used and insert them into a testing client application.
The client should provide a link similar the this one:

```
http://localhost:8090/oauth/authorize?grant_type=authorization_code&response_type=code&client_id=__CLIENT_ID__
```

The link will display a login page, then once the user is authenticated it will ask the approvals of the scopes selected during the client creation.

If the user approves at least one scope, an authorization code will be returned to the URL given during the client registration.

The testing client application should be able to convert the authorization code to an access token. (see a PHP example below)

```
<?php
	define('CLIENT_ID', '__YOUR_CLIENT_ID__');
	define('CLIENT_SECRET', '__YOUR_CLIENT_SECRET__');
	
	echo "Authorization code received: ".$_GET['code']."<br>";
	
	$encodedCredentials = base64_encode ( CLIENT_ID.':'.CLIENT_SECRET );
	$url = "http://localhost:8090/oauth/token"."?"."code=".$_GET['code']."&"."grant_type="."authorization_code";
	
	$crl = curl_init();

	$headr = array();
	$headr[] = 'Authorization: Basic '.$encodedCredentials;

    curl_setopt($crl, CURLOPT_URL, $url);
	curl_setopt($crl, CURLOPT_HTTPHEADER, $headr);
	curl_setopt($crl, CURLOPT_RETURNTRANSFER, true);
	curl_setopt($crl, CURLOPT_SSL_VERIFYPEER, false);
	curl_setopt($crl, CURLOPT_POST,true);

	$result = curl_exec($crl);
	if ($result === false){
		print_r('Curl error: ' . curl_error($crl));
	} else {
		//print_r($rest);
		$json = json_decode($result, true);
		
		echo "The convertion to access token has been done."."<br>";
		echo "<h4>Results</h4>";
		echo "<table>";
		echo "<tr><th>Access token</th><td>".$json["access_token"]."</td></tr>";
		echo "<tr><th>Token type</th><td>".$json["token_type"]."</td></tr>";
		echo "<tr><th>Refresh token</th><td>".$json["refresh_token"]."</td></tr>";
		echo "<tr><th>Expires in</th><td>".$json["expires_in"]."</td></tr>";
		echo "<tr><th>Scope</th><td>".$json["scope"]."</td></tr>";
		echo "</table>";
	}

	curl_close($crl);
```

Once you have the access token use it as a to have access to the data through the resource server. (a Postman example is provided with the project "charon-resource-server-cassandra-users")
The access token should be use as a bearer token. It means you should create an HTTP header similar the one below.

```
	Authorization: Bearer __YOUR_ACCESS_TOKEN__
```

## Todos

The project is still at the beginning. Few functionalities will be added soon:
- display the Client ID when a client is created (also give the possibility to generate automatically the client secret)
- prepare a dashboard implementation of the resource server to monitor the system
- support for JWT
- shared the project on Maven repositories
- develop the accounting service manager to allow the users to see the approvals already given
- remove the tokens when a client is modified or deleted
- implement examples for relational databases
- add JUnit/Mockito support
- include also Swagger/OpenAPI
- conceive a vue.js front (with web sockets support)

If you have a particular request or any other fix/upgrade idea, feel free to send me a message. It will be highly appreciated!

## Authors

* **Alexandre Giraud** - *Initial work* - [alexandregiraud64](https://github.com/alexandregiraud64)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

Special thanks to anyone whose code was used, the list is being prepared and will be published soon.
