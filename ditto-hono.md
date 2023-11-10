# Example Eclipse Ditto and Hono Configuration

After deployment of example helm chart use `kubectl forward` command to forward http port of pod `demo-hono-service-device-registry-0` and call following API to create new tenant with trusted CA of CoAP client:

```
curl --location 'http://localhost:8080/v1/tenants/default' \
--header 'Content-Type: application/json' \
--data '{
  "alias": "default",
  "trusted-ca": [
    {
      "id": "esp32",
      "auto-provisioning-enabled": true,
      "cert": "MIICDzCCAbWgAwIBAgIESZYC0jAKBggqhkjOPQQDAjBcMQswCQYDVQQGEwJaWTESMBAGA1UECAwJWW91clN0YXRlMRAwDgYDVQQKDAdZb3VyT3JnMRQwEgYDVQQLDAtZb3VyT3JnVW5pdDERMA8GA1UEAwwIVmVuZG9yQ0EwIBcNMTgwNzEzMTE1NjA5WhgPMjI5MjA0MjYxMTU2MDlaMFwxCzAJBgNVBAYTAlpZMRIwEAYDVQQIDAlZb3VyU3RhdGUxEDAOBgNVBAoMB1lvdXJPcmcxFDASBgNVBAsMC1lvdXJPcmdVbml0MREwDwYDVQQDDAhWZW5kb3JDQTBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABGAAuYcBIgP0fMC1Bd+1nAH5S1goR0TaDAIadK4hULQr5LwziuDk9XTQaOTwmWB9iR1eiHC6RY8WwyrGBbnEbzujYzBhMB0GA1UdDgQWBBQ+yCpIszhzbmXe2At1GofREjnBxjAfBgNVHSMEGDAWgBQ+yCpIszhzbmXe2At1GofREjnBxjAPBgNVHRMBAf8EBTADAQH/MA4GA1UdDwEB/wQEAwIBhjAKBggqhkjOPQQDAgNIADBFAiBW60XgdSRD24rbTgdneS+VSHVix8LuXunPYW50LmxbrwIhAOw4gMroRIOS26y0TcND03FnyO3wBNF9MjM0hWKQJXk3",
      "auto-provisioning-device-id-template": "default:{{subject-cn}}"
    }
  ]
}'
```

Now forward http port of pod `demo-ditto-gateway-...` and call following API to create default thing policy:

```
curl --location --request PUT 'https://example.domain.com/api/2/policies/default:default' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZGl0dG86ZGl0dG8=' \
--data '{
  "entries": {
    "INIT": {
      "subjects": {
        "nginx:ditto": {
          "type": "Ditto user pre-authenticated on service"
        }
      },
      "resources": {
        "thing:/": {
          "grant": [
            "READ",
            "WRITE"
          ],
          "revoke": []
        },
        "policy:/": {
          "grant": [
            "READ",
            "WRITE"
          ],
          "revoke": []
        },
        "message:/": {
          "grant": [
            "READ",
            "WRITE"
          ],
          "revoke": []
        }
      }
    },
    "HONO": {
      "subjects": {
        "pre-authenticated:hono-connection": {
          "type": "Connetion to Eclipse Hono"
        }
      },
      "resources": {
        "thing:/": {
          "grant": [
            "READ",
            "WRITE"
          ],
          "revoke": []
        },
        "message:/": {
          "grant": [
            "READ",
            "WRITE"
          ],
          "revoke": []
        },
        "policy:/": {
          "grant": [
            "READ"
          ],
          "revoke": []
        }
      }
    }
  }
}'
```

Now open Ditto UI (https://<your-domain>/ui) and create Ditto connection:

```
{
  "name": "hono-kafka",
  "connectionType": "kafka",
  "connectionStatus": "open",
  "uri": "tcp://demo-kafka:9092",
  "failoverEnabled": true,
  "specificConfig": {
    "bootstrapServers": "demo-kafka:9092",
    "saslMechanism": "scram-sha-512"
  },
  "sources": [
    {
      "addresses": [
        "hono.telemetry.default",
        "hono.event.default"
      ],
      "authorizationContext": [
        "pre-authenticated:hono-connection"
      ],
      "consumerCount": 1,
      "enforcement": {
        "input": "{{ header:device_id }}",
        "filters": [
          "{{ entity:id }}"
        ]
      },
      "headerMapping": {
        "hono-device-id": "{{ header:device_id }}",
        "content-type": "{{ header:content-type }}"
      },
      "payloadMapping": [
        "Ditto",
        "status",
        "implicitThingCreation"
      ],
      "replyTarget": {
        "enabled": true,
        "address": "hono.command.default/{{ thing:id }}",
        "headerMapping": {
          "device_id": "{{ header:hono-device-id }}",
          "subject": "{{ header:subject | fn:default(topic:action-subject) | fn:default(topic:criterion) }}-response",
          "correlation-id": "{{ header:correlation-id }}",
          "content-type": "{{ header:content-type | fn:default('application/vnd.eclipse.ditto+json') }}"
        },
        "expectedResponseTypes": [
          "response",
          "error"
        ]
      },
      "acknowledgementRequests": {
        "includes": [],
        "filter": "fn:filter(header:qos,'eq','1')"
      }
    },
    {
      "addresses": [
        "hono.command_response.default"
      ],
      "authorizationContext": [
        "pre-authenticated:hono-connection"
      ],
      "headerMapping": {
        "content-type": "{{ header:content-type }}",
        "correlation-id": "{{ header:correlation-id }}",
        "status": "{{ header:status }}"
      },
      "replyTarget": {
        "enabled": false,
        "expectedResponseTypes": [
          "response",
          "error"
        ]
      }
    }
  ],
  "targets": [
    {
      "address": "hono.command.default/{{ thing:id }}",
      "authorizationContext": [
        "pre-authenticated:hono-connection"
      ],
      "topics": [
        "_/_/things/live/commands",
        "_/_/things/live/messages"
      ],
      "headerMapping": {
        "device_id": "{{ thing:id }}",
        "subject": "{{ header:subject | fn:default(topic:action-subject) }}",
        "content-type": "{{ header:content-type | fn:default('application/vnd.eclipse.ditto+json') }}",
        "response-required": "true",
        "correlation-id": "{{ header:correlation-id }}",
        "reply-to": "{{ fn:default('hono.command_response.default') | fn:filter(header:response-required,'ne','false') }}"
      }
    },
    {
      "address": "hono.command.default/{{ thing:id }}",
      "authorizationContext": [
        "pre-authenticated:hono-connection"
      ],
      "topics": [
        "_/_/things/twin/events",
        "_/_/things/live/events"
      ],
      "headerMapping": {
        "device_id": "{{ thing:id }}",
        "subject": "{{ header:subject | fn:default(topic:action-subject) }}",
        "content-type": "{{ header:content-type | fn:default('application/vnd.eclipse.ditto+json') }}",
        "correlation-id": "{{ header:correlation-id }}"
      }
    }
  ],
  "mappingDefinitions": {
    "status": {
      "mappingEngine": "ConnectionStatus",
      "options": {
        "thingId": "{{ header:device_id }}"
      }
    },
    "implicitThingCreation": {
      "mappingEngine": "ImplicitThingCreation",
      "options": {
        "thing": {
		  "thingId": "{{ header:device_id }}",
		  "policyId": "default:default",
		  "features": {
		  }
		},
        "commandHeaders": {
          "reply-to": "hono.command_response.default"
        }
      },
      "incomingConditions": {
        "honoRegistration": "fn:filter(header:hono_registration_status, 'eq', 'NEW')",
        "notBehindGateway": "fn:filter(header:gateway_id, 'exists', 'false')"
      },
      "allowPolicyLockout": false
    }
  }
}
```