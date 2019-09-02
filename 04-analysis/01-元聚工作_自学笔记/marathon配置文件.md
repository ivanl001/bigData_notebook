https://mesosphere.github.io/marathon/

https://mesosphere.github.io/marathon/docs/marathon-ui.html

https://docs.mesosphere.com/cn/services/Marathon-LB/mlb-auth/











```json
{
  "id": "/aries/etl/collection/advertising",
  "cmd": null,
  "cpus": 2,
  "mem": 4096,
  "disk": 0,
  "instances": 10,
  "constraints": [
    [
      "hostname",
      "UNIQUE"
    ]
  ],
  "acceptedResourceRoles": [
    "*"
  ],
  "container": {
    "type": "DOCKER",
    "docker": {
      "forcePullImage": false,
      "image": "images-center.io/online.dw.aries/online.dw.aries.etl.collection.advertising:v.0.0.13",
      "parameters": [
        {
          "key": "add-host",
          "value": "dwcm.com:10.200.3.2"
        },
        {
          "key": "add-host",
          "value": "dw01.com:10.200.3.3"
        },
        {
          "key": "add-host",
          "value": "dw02.com:10.200.3.4"
        },
        {
          "key": "add-host",
          "value": "dw03.com:10.200.3.5"
        },
        {
          "key": "add-host",
          "value": "dw04.com:10.200.3.6"
        },
        {
          "key": "add-host",
          "value": "dw05.com:10.200.3.7"
        },
        {
          "key": "add-host",
          "value": "dw06.com:10.200.3.8"
        },
        {
          "key": "add-host",
          "value": "dw07.com:10.200.3.9"
        },
        {
          "key": "add-host",
          "value": "dw08.com:10.200.3.10"
        },
        {
          "key": "add-host",
          "value": "dw09.com:10.200.3.11"
        },
        {
          "key": "add-host",
          "value": "dw10.com:10.200.3.12"
        },
        {
          "key": "add-host",
          "value": "dw11.com:10.200.3.13"
        },
        {
          "key": "add-host",
          "value": "dw12.com:10.200.3.14"
        },
        {
          "key": "add-host",
          "value": "dw13.com:10.200.3.15"
        },
        {
          "key": "add-host",
          "value": "dw14.com:10.200.3.16"
        },
        {
          "key": "add-host",
          "value": "dw15.com:10.200.3.17"
        },
        {
          "key": "add-host",
          "value": "dw16.com:10.200.3.18"
        },
        {
          "key": "add-host",
          "value": "dw17.com:10.200.3.19"
        },
        {
          "key": "add-host",
          "value": "dw18.com:10.200.3.20"
        },
        {
          "key": "add-host",
          "value": "dw19.com:10.200.3.21"
        },
        {
          "key": "cpu-quota",
          "value": "200000"
        }
      ],
      "privileged": false
    },
    "volumes": [
      {
        "containerPath": "/var/log/aries",
        "hostPath": "/data/var/log/aries",
        "mode": "RW"
      }
    ],
    "portMappings": [
      {
        "containerPort": 50214,
        "hostPort": 0,
        "labels": {},
        "protocol": "tcp",
        "servicePort": 50214
      }
    ]
  },
  "env": {
    "JAVA_OPTS": "-server -Xmx2176m -Xms2176m -Xmn1280m -Xss512m -XX:+UseParallelGC -XX:ParallelGCThreads=8"
  },
  "healthChecks": [
    {
      "gracePeriodSeconds": 300,
      "ignoreHttp1xx": false,
      "intervalSeconds": 60,
      "maxConsecutiveFailures": 3,
      "path": "/health",
      "portIndex": 0,
      "protocol": "HTTP",
      "ipProtocol": "IPv4",
      "timeoutSeconds": 20,
      "delaySeconds": 15
    }
  ],
  "labels": {
    "HAPROXY_GROUP": "external"
  },
  "networks": [
    {
      "mode": "container/bridge"
    }
  ],
  "portDefinitions": []
}
```

