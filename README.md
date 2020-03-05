# funk-sdk-java-spring
fun(k) SDK in Java with Spring

Can use from any directory. Currently requires separate gcloud build (or docker build) before deploy step (until buildpacks integrated).

# demo event funk
```
$ mkdir playground
$ cd playground
$ kn funk init java-spring
$ kn funk function create HelloSpringEvent --t com.mikehelmick.eventutils.user.user
$ gcloud builds submit funks/hellospringevent --tag $KO_DOCKER_REPO/hellospringevent
$ kn funk deploy
```
then create an event to test your funk. This assumes you've created a curl pod per https://knative.dev/docs/eventing/getting-started/
```
$ kubectl --namespace default attach curl -it
[ root@curl:/ ]$ curl -v "http://default-broker.default.svc.cluster.local" \
  -X POST \
  -H "Ce-Id: say-hello" \
  -H "Ce-Specversion: 0.3" \
  -H "Ce-Type: com.mikehelmick.eventutils.user.user" \
  -H "Ce-Source: not-sendoff" \
  -H "Content-Type: application/json" \
  -d '{"first_name":"Paul", "last_name":"Blart"}'
```

And from here, among other things, there will be a generated YourFunkNameFunction.java function handler in ./funks/your-funk-name/src/main/java/com/example/funks that accepts the concrete event type, and has a TODO to add your function code. E.g.

```java
package com.example.funks;

import java.util.Map;
import java.util.logging.Logger;

import com.mikehelmick.eventutils.user.User;

import org.springframework.stereotype.Component;

@Component
public class HelloSpringEventFunction {
    private static final Logger logger = Logger.getLogger(HelloSpringEventFunction.class.getName());

    public void onEvent(User event, Map<String, String> cloudEventHeaders) {
        logger.info("Received User event: " + event);

        // TODO(you): process the event!
    }
}
```

There is also a YourFunkNameReceiver.java that need not be modified (which handles the unmarshalling and invokes the Function). The concrete data type is generated as well. 

# demo http funk
(the first 3 steps aren't necessary to do if you've previously created a funk)
```
$ mkdir playground
$ cd playground
$ kn funk init nodejs
$ kn funk function create HelloSpring --http=true
$ gcloud builds submit funks/hellospring --tag $KO_DOCKER_REPO/hellospring
$ kn funk deploy
```
Then curl your http funk:
```
$ curl -v -H "Host: funk-hellospring.default.example.com" <your IP>
```
