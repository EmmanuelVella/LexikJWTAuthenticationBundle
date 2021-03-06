Data customization and validation
=================================

#### Events::JWT_CREATED - add data to the JWT payload

By default the JWT payload will contain the username and the token TTL,
but you can add your own data.

``` yaml
# services.yml
services:
    acme_api.event.jwt_created_listener:
        class: Acme\Bundle\ApiBundle\EventListener\JWTCreatedListener
        tags:
            - { name: kernel.event_listener, event: lexik_jwt_authentication.on_jwt_created, method: onJWTCreated }
```

Example : add client ip to the encoded payload

``` php
// Acme\Bundle\ApiBundle\EventListener\JWTCreatedListener.php
class JWTCreatedListener
{
    /**
     * @param JWTCreatedEvent $event
     *
     * @return void
     */
    public function onJWTCreated(JWTCreatedEvent $event)
    {
        if (!($request = $event->getRequest())) {
            return;
        }

        $payload       = $event->getData();
        $payload['ip'] = $request->getClientIp();

        $event->setData($payload);
    }
}
```

#### Events::JWT_DECODED - validate data in the JWT payload

You can access the jwt payload once it has been decoded to perform you own additional validation. 

``` yaml
# services.yml
services:
    acme_api.event.jwt_decoded_listener:
        class: Acme\Bundle\ApiBundle\EventListener\JWTDecodedListener
        tags:
            - { name: kernel.event_listener, event: lexik_jwt_authentication.on_jwt_decoded, method: onJWTDecoded }
```

Example : check client ip the decoded payload

``` php
// Acme\Bundle\ApiBundle\EventListener\JWTDecodedListener.php
class JWTDecodedListener
{
    /**
     * @param JWTDecodedEvent $event
     *
     * @return void
     */
    public function onJWTDecoded(JWTDecodedEvent $event)
    {
        if (!($request = $event->getRequest())) {
            return;
        }

        $payload = $event->getPayload();
        $request = $event->getRequest();

        if (!isset($payload['ip']) || $payload['ip'] !== $request->getClientIp()) {
            $event->markAsInvalid();
        }
    }
}
```

#### Events::AUTHENTICATION_SUCCESS - add public data to the JWT response

By default, the authentication response is just a json containing the JWT but you can add your own public data to it.

``` yaml
# services.yml
services:
    acme_api.event.authentication_success_listener:
        class: Acme\Bundle\ApiBundle\EventListener\AuthenticationSuccessListener
        tags:
            - { name: kernel.event_listener, event: lexik_jwt_authentication.on_authentication_success, method: onAuthenticationSuccessResponse }
```

Example : add user roles to the response

``` php
// Acme\Bundle\ApiBundle\EventListener\AuthenticationSuccessListener.php
/**
 * @param AuthenticationSuccessEvent $event
 */
public function onAuthenticationSuccessResponse(AuthenticationSuccessEvent $event)
{
    $data = $event->getData();
    $user = $event->getUser();

    if (!$user instanceof UserInterface) {
        return;
    }

    // $data['token'] contains the JWT

    $data['data'] = array(
        'roles' => $user->getRoles(),
    );

    $event->setData($data);
}
```
