# Okta Rest Collector IO
----

## About this Pack

This pack is built as a complete SOURCE + DESTINATION solution (identified by the IO prefix). Data collection and delivery happen entirely within the pack's context, eliminating the need to connect it to globally defined Sources and Destinations. This collector based pack is designed to handle JSON data collected from the Okta System Log API endpoint. The JSON is parsed and the timestamp normalized from the proper field within each event. The pack offers two different optional methods of reduction:

1. Drop, Sample, or Suppress based off of eventType. You can target individual eventTypes by modifying the provided okta-event-types.csv lookup under Knowledge > Lookups.
2. Remove nested null value fields.

The pack also currently includes three forms of output formats:

1. Normalized JSON
2. OCSF - Primarily meant for Amazon Security Lake, the pack can normalize the data into the proper OCSF categories based on eventType. The target OCSF category can also be managed on an individual eventType basis through the lookup file **okta-event-types.csv** .
3. Splunk - default index and sourcetype supplied from Knowledge > Variables, but can be overwritten in pipeline

## Deployment

The Okta Rest IO pack ingests events from the Okta System Log API, normalizing the data for use with your configured destinations.


* This pack is configured by default to use the Worker Group's Default Destination.

* To use the Default Destination: No changes are required. The pack will route the data to the destination currently set as the Default on the Worker Group.

* To use a different Destination: You must update the pack's routes to specify your desired Destination.

* For immediate functionality without requiring pack route filter expression modifications, every bundled Source within this pack adds a hidden field: __packsource==’cribl-okta-rest-io.okta-api’. This field allows for seamless routing based on the pack source.

### Source + Destination Configuration
To use this pack as a completely bundled source and destination pack, follow these steps:

1. Configure the Pack
This pack includes several functions that can help reduce events. Please make sure you evaluate the functions before enabling it to ensure vital data is not missed.

Additionally, several output formats are available to be selected. Please only enable one output, as enabling multiple may break the output formatting.

2. Configure the Rest Collector Source
- Enter the Cribl Okta Rest IO pack > Sources > Collectors > REST. 
- Select the OKTA-API source 
- Navigate to the Collect URL field > Replace the REPLACE_OKTA_DOMAIN in the collect URL with your OKTA endpoint's domain
- Navigate to the Collect Headers field > Next to the Authorization, replace the Value of YOUR_OKTA_BEARER_TOKEN with your OKTA Bearer token. Do not remove SSWS as it is required. There should also be a space between SSWS and the bearer token as it is shown in the field. 

3. Configure your Destination/Update Pack Routes
To ensure proper data routing, you must make a choice: retain the current setting to use the Default Destination defined by your Worker Group, or define a new Destination directly inside this pack and adjust the pack's route accordingly.

4. Commit and Deploy
Once everything is configured, Commit & Deploy your end to end telemetry stream. 

### Processing Only Pack Configuration (Only use pack's routes + pipelines)
If you prefer to use the pack's processing logic only (e.g., the collector JSON and event breaker) with external, globally defined Sources and Destinations, follow these steps:

1. Clone the Pack with a new Pack ID.

2. Remove the bundled IO Source and IO Destination objects.

3. Connect the remaining processing pipeline via your Global Routes page.

## Upgrades

Upgrading certain Cribl Packs using the same Pack ID can have unintended consequences. See [Upgrading an Existing Pack](https://docs.cribl.io/stream/packs#upgrading) for details.

## Release Notes

### Version 0.1.0
Initial release

## Contributing to the Pack

To contribute to the Pack, please connect with us on [Cribl Community Slack](https://cribl-community.slack.com/). You can suggest new features or offer to collaborate.

## License
This Pack uses the following license: [Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE).

## Appendix A
### Event Breaker JSON
```
{
  "id": "Okta-API",
  "lib": "custom",
  "description": "Event breakers for Okta logs",
  "rules": [
    {
      "condition": "true",
      "type": "json_array",
      "timestampAnchorRegex": "/published\":\"/",
      "timestamp": {
        "type": "format",
        "length": 150,
        "format": "%Y-%m-%dT%H:%M:%S.%LZ"
      },
      "timestampTimezone": "utc",
      "timestampEarliest": "-420weeks",
      "timestampLatest": "+1week",
      "maxEventBytes": 51200,
      "disabled": false,
      "jsonExtractAll": false,
      "eventBreakerRegex": "/[\\n\\r]+(?!\\s)/",
      "name": "System Logs",
      "jsonTimeField": "published"
    }
  ]
}
```

## Appendix B
### Collector Source JSON
```
{
  "type": "collection",
  "ttl": "4h",
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {},
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "none"
      },
      "collectMethod": "get",
      "pagination": {
        "type": "response_header_link",
        "nextRelationAttribute": "next",
        "maxPages": 0,
        "curRelationAttribute": "self"
      },
      "authentication": "none",
      "timeout": 0,
      "useRoundRobinDns": false,
      "disableTimeFilter": false,
      "safeHeaders": [],
      "collectUrl": "'https://<Domain| This is your Okta Domain, if you need help here is the [Okta Docs](https://developer.okta.com/docs/guides/find-your-domain/main/)>.okta.com/api/v1/logs'",
      "collectRequestParams": [
        {
          "name": "since",
          "value": "`${new Date((earliest * 1000 || Date.now()-(7*24*60*60*1000))).toISOString()}`"
        },
        {
          "name": "until",
          "value": "`${new Date((latest * 1000 || Date.now())).toISOString()}`"
        }
      ],
      "collectRequestHeaders": [
        {
          "name": "Authorization",
          "value": "`SSWS <SSWS Key| This is the Okta API Key, more infor can be found in the [Okta Docs](https://developer.okta.com/docs/guides/create-an-api-token/main/)>`"
        }
      ]
    },
    "destructive": false,
    "type": "rest"
  },
  "input": {
    "type": "collection",
    "staleChannelFlushMs": 10000,
    "sendToRoutes": true,
    "preprocess": {
      "disabled": true
    },
    "throttleRatePerSec": "0",
    "breakerRulesets": [
      "Okta-API"
    ]
  },
  "id": "Okta-API"
}
```
