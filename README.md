# BlueSky Web

The blueskyweb package contains a tornado web service, wrapping
[bluesky][https://github.com/pnwairfire/bluesky],
that can be started by simply running ```bsp-web```.

## Non-python Dependencies

See the [bluesky github page](https://github.com/pnwairfire/bluesky)
for information about it's dependencies.

## Development

### Clone Repo

Via ssh:

    git clone git@bitbucket.org:fera/airfire-bluesky-web.git

or http:

    git clone https://jdubowy@bitbucket.org/fera/airfire-bluesky-web.git

### Install Dependencies

First, install pip (with sudo if necessary):

    apt-get install python-pip

Run the following to install python dependencies:

    pip install --no-binary gdal --trusted-host pypi.smoke.airfire.org -r requirements.txt

Run the following to install packages required for development:

    pip install -r requirements-dev.txt

### Notes:

See the [bluesky github page](https://github.com/pnwairfire/bluesky)
for notes on potential pip and gdal issues.

## Running

Use the help (-h) option to see usage and available config options:

    bsp-web -h

### Fabric

For the convenience of those wishing to run the web service on a remote server,
this repo contains a fabfile defining tasks for setting up,
deploying to, and restarting the service on a remote server.

To see what tasks are available, clone the repo, cd into it, and run

    git clone https://github.com/pnwairfire/bluesky.git
    cd bluesky
    BLUESKYWEB_SERVERS=username@hostname fab -l

(The 'BLUESKYWEB_SERVERS=username@hostname' is needed because it's used to set
the role definitions, which are be done at the module scope.)

To see documentation for a specific task, use the '-d' option. E.g.:

    BLUESKYWEB_SERVERS=username@hostname fab -d deploy

An example with two remote servers

    BLUESKYWEB_SERVERS=username-a@hostname-a,username-b@hostname-b fab setup
    BLUESKYWEB_SERVERS=username-a@hostname-a,username-b@hostname-b fab deploy

Note that the deploy task takes care of restarting.

## APIs

### GET /api/v1/domains/

This API returns information about all domains with ARL data

#### Request

 - url: http://bluesky-api-hostname/api/v1/domains/
 - method: GET

#### Response

    {
        "domains": {
            "<domain_id>": {
                "dates": [
                    <date>,
                    ...
                ],
                "boundary": {
                    "center_latitude": <lat>,
                    "center_longitude": <lng>,
                    "width_longitude": <degrees>,
                    "height_latitude": <degrees>
                },
                <other_domain_data?>: <data>,
                ...
            },
            ...
        }
    }

#### Example

    $ curl 'http://bluesky-api-hostname/api/v1/domains/'
    {
        "domains": {
            "CANSAC-6km": {
                "boundary": {
                    "center_latitude": 36.5,
                    "center_longitude": -119.0,
                    "height_latitude": 17.5,
                    "width_longitude": 25.0
                },
                "dates": [
                    "20150922",
                    "20150921",
                    "20150920"
                ]
            },
            "PNW-4km": {
                "boundary": {
                    "center_latitude": 45.0,
                    "center_longitude": -118.3,
                    "height_latitude": 10.0,
                    "width_longitude": 20.0
                },
                "dates": [
                    "20150922",
                    "20150921",
                    "20150920"
                ]
            }
        }
    }

### GET /api/v1/domains/<domain_id>/

This API returns information about a specific domain with ARL data

#### Request

 - url: http://bluesky-api-hostname/api/v1/domains/<domain_id>/
 - method: GET

#### Response

    {
        "<domain_id>": {
            "dates": [
                <date>,
                ...
            ],
            "boundary": {
                "center_latitude": <lat>,
                "center_longitude": <lng>,
                "width_longitude": <degrees>,
                "height_latitude": <degrees>
            },
            <other_domain_data?>: <data>,
            ...
        }
    }

#### Example

    $ curl 'http://bluesky-api-hostname/api/v1/domains/PNW-4km/'
    {
        "PNW-4km": {
            "boundary": {
                "center_latitude": 45.0,
                "center_longitude": -118.3,
                "height_latitude": 10.0,
                "width_longitude": 20.0
            },
            "dates": [
                "20150922",
                "20150921",
                "20150920"
            ]
        }
    }

### GET /api/v1/domains/<domain_id>/available-dates/

This API returns the dates for which a specific d has ARL data

#### Request

 - url: http://bluesky-api-hostname/api/v1/domains/<domain_id>/available-dates
 - method: GET

#### Response

    {
        "dates": [
           <date>,
           ...
        ]
    }


#### Example

    $ curl 'http://bluesky-api-hostname/api/v1/domains/PNW-4km/available-dates

    {
        "dates": [
            "20150922",
            "20150921",
            "20150920"
        ]
    }


### GET /api/v1/domains/available-dates/

This API returns the dates, by domain, for which there exist ARL data

#### Request

 - url: http://bluesky-api-hostname/api/v1/available-dates/
 - method: GET

#### Response

    {
        "dates": [
            "<domain_id>": [
                <date>,
                ...
            ]
           ...
        ]
    }


#### Example

    $ curl 'http://bluesky-api-hostname/api/v1/available-dates

    {
        "dates": {
            "CANSAC-6km": [
                "20150922",
                "20150921",
                "20150920"
            ],
            "PNW-4km": [
                "20150922",
                "20150921",
                "20150920"
            ]
        }
    }


### POST /api/v1/run/

This API requires posted JSON with three top level keys -
'modules', 'fire_information', and 'config'.
The 'fire_information' key lists the one or more fires to process. The
'modules' key is the order specific list of modules through which the fires
should be run. The optional 'config' key specifies configuration data and
other control parameters.

#### Request

 - url: http://bluesky-api-hostname/api/v1/run/
 - method: POST
 - post data:

        {
            "modules": [ ... ],
            "fire_information": [ ... ],
            "config": { ... }
        }

See [BlueSky Pipeline](../../README.md) for more information about required
and optional post data

#### Response

What you get in response depends on which modules you're executing.  If
your run does ***not*** include hysplit, then bluesky will be run in realtime,
and the results will be in the API response.  The response data will be the
modified version of the request data.  It will include the
"fire_information" keys, the "config" key (if specified), a "processing"
key that includes information from the modules that processed the data, and
possibly a "summary" key (depending on whether or not the modules run add
summary data)

    {
        "fire_information": [ ... ],
        "config": { ... },
        "processing": [ ... ],
        "summary": { ... }
    }

If hysplit is run, however, bluesky will be run asynchronously, and the
API reponse will include a guid to identify the run in subsequent
status and output API requests (described below).

    {
        run_id: <guid>
    }

#### Examples

##### Running ```fuelbeds```, ```consumption```, and ```emissions```:

This example requires very little data, since it's starting off with ```fuelbeds```,
one of the earlier modules in the pipeline.

    $ curl 'http://bluesky-api-hostname/api/v1/run/' -H 'Content-Type: application/json' -d '
    {
        "modules": ["fuelbeds", "consumption", "emissions"],
        "fire_information": [
            {
                "id": "SF11C14225236095807750",
                "event_id": "SF11E826544",
                "name": "Natural Fire near Snoqualmie Pass, WA",
                "location": {
                    "perimeter": {
                        "type": "MultiPolygon",
                        "coordinates": [
                            [
                                [
                                    [-121.4522115, 47.4316976],
                                    [-121.3990506, 47.4316976],
                                    [-121.3990506, 47.4099293],
                                    [-121.4522115, 47.4099293],
                                    [-121.4522115, 47.4316976]
                                ]
                            ]
                        ]
                    },
                    "ecoregion": "southern",
                    "utc_offset": "-09:00"
                }
            }
        ]
    }'

Another exmaple, with fire location data specified as lat + lng + size

    $ curl 'http://bluesky-api-hostname/api/v1/run/' -H 'Content-Type: application/json' -d '
    {
        "modules": ["fuelbeds", "consumption", "emissions"],
        "fire_information": [
            {
                "id": "SF11C14225236095807750",
                "event_of": {
                    "id": "SF11E826544",
                    "name": "Natural Fire near Snoqualmie Pass, WA"
                },
                "location": {
                    "latitude": 47.4316976,
                    "longitude": -121.3990506,
                    "area": 200,
                    "utc_offset": "-09:00",
                    "ecoregion": "southern"
                }
            }
        ]
    }'

##### Running ```consumption```, and ```emissions```:

This example starts with fire data that already had fuelbed information and
passes it through consumption and emissions.  Note that is passes in some
custom fuel loadings information.

    $ curl 'http://bluesky-api-hostname/api/v1/run/' -H 'Content-Type: application/json' -d '
    {
        "modules": ["consumption", "emissions"],
        "config": {
            "consumption": {
                "fuel_loadings": {
                    "10046": {
                        "based_on_fccs_id": "46",
                        "w_sound_9_20_loading": 0.42,
                        "w_sound_gt20_loading": 0.43,
                        "w_sound_quarter_1_loading": 0.44,
                        "w_stump_lightered_loading": 0.45,
                        "w_stump_rotten_loading": 0.46,
                        "w_stump_sound_loading": 0.47
                    }
                }
            }
        },
        "fire_information": [
            {
                "id": "SF11C14225236095807750",
                "event_of": {
                    "id": "SF11E826544",
                    "name": "Natural Fire near Snoqualmie Pass, WA"
                },
                "fuelbeds": [
                    {
                        "fccs_id": "10046",
                        "pct": 100.0
                    }
                ],
                "location": {
                    "area": 200,
                    "ecoregion": "southern",
                    "latitude": 47.4316976,
                    "longitude": -121.3990506,
                    "utc_offset": "-09:00"
                }
            }
        ]
    }'

##### Running  ```localmet```, ```timeprofiling```, ```plumerise```, ```dispersion```, ```visualization```, ```export```:

This example assumes you've already run up through emissions.  The consumption data
that would have nested along side the emissions data has been stripped out, since
it's not needed.

    $ curl 'http://bluesky-api-hostname/api/v1/run/' -H 'Content-Type: application/json' -d '
    {
        "modules": ["localmet", "timeprofiling", "plumerise", "dispersion", "visualization", "export"],
        "fire_information": [
            {
                "id": "SF11C14225236095807750",
                "event_id": "SF11E826544",
                "name": "Natural Fire near Snoqualmie Pass, WA",
                "location": {
                    "latitude": 47.4316976,
                    "longitude": -121.3990506,
                    "area": 200,
                    "utc_offset": "-09:00"
                },
                "fuelbeds": [
                    {
                        "fccs_id": "49",
                        "pct": 50.0,
                        "emissions": {
                            'ground fuels': {
                                'basal accumulations': {
                                    'flaming': {
                                        'PM2.5': [
                                            3.3815120047017005e-05
                                        ]
                                    },
                                    'residual': {
                                        'PM2.5': [
                                            4.621500211796271e-01
                                        ]
                                    },
                                    'smoldering': {
                                        'PM2.5': [
                                            6.424985839975172e-06
                                        ]
                                    }
                                }
                            }
                            /* ... (other emissions data) ... */
                        }
                    }
                ],
                "growth": [
                    {
                        "start": "20150120",
                        "end": "20150121",
                        "pct": 100.0
                    }
                ]
            }
        ],
        "config": {
            "localmet": {
                /* ... */
            },
            "timeprofiling": {
                "module": "standard" /* or "feps_rx_timing" or "custom" */
            },
            "plumerise": {
                "module": "sev"
            },
            "dispersion": {
                "module": "hysplit",
                "start": "20150121T000000Z",
                "end": "20150123T000000Z",
                "met_domain": "PNW-4km"
            },
            "visualization": {
                /* ... */
            },
            "export": {
                "module": "playground" /* or "bs_daily" */
            }
        }
    }'

The nested keys in the emissions data are arbitrary.  The timeprofiling
module simply expects a hierarchy of keys.  Generally speaking, the hiearchy
is of the form:

    'emissions' > 'category' > 'subcategory' > 'phase' > 'species'

where the 'category' and 'subcategory' keys correspond to fuel types, but could
be anything.

The fact that the emissions data is in an array is because the consumption
module (more specifically, the underlying 'consume' module) outputs arrays.
The length of each array equals the number of fuelbeds passed into consume.
Since consume is called on each fuelbed separately, the arrays of consumption
and emissions data will all be of length 1.

##### Running ```ingestion```, ```fuelbeds```, ```consumption```, ```emissions```, ```localmet```, ```timeprofiling```, ```plumerise```, ```dispersion```, ```visualization```, ```export```:

    $ curl 'http://bluesky-api-hostname/api/v1/run/' -H 'Content-Type: application/json' -d '
    {
        "modules": ["timeprofiling", "plumerise", "dispersion"],
        "fire_information": [
            {
                "id": "SF11C14225236095807750",
                "event_id": "SF11E826544",
                "name": "Natural Fire near Snoqualmie Pass, WA",
                "location": {
                    "latitude": 47.4316976,
                    "longitude": -121.3990506,
                    "area": 200,
                    "utc_offset": "-09:00"
                }
                "growth": [
                    {
                        "start": "20150120",
                        "end": "20150121",
                        "pct": 100.0
                    }
                ]
            }
        ]
    }

### GET /api/v1/run/<guid>/status

This API returns the status of a specific hysplit run

#### Request

 - url: http://bluesky-api-hostname/api/v1/run/<guid>/status
 - method: GET

#### Response

    {
        "complete": <boolean>,
        "percent": <double>, /* (if available) */
        "failed": <boolean>,
        "message": <string>
    }

#### Example:

    $ curl 'http://bluesky-api-hostname/api/v1/run/abc123/status'

    {
        "complete": false,
        "percent": 62.3, /* (if available) */
        "failed": false,
        "message": "Started HYSPLIT"
    }

### GET /api/v1/run/<guid>/output

This API returns the output location for a specific run

#### Request

 - url: http://bluesky-api-hostname/api/v1/run/<guid>/output
 - method: GET

#### Response


#### Example:

    $ curl 'http://bluesky-api-hostname/api/v1/run/abc123/output'

    {
       "output": {
           "directory": <absolute_path>,
           "images": {
               "hourly": [
                   <filename|relative_path>,
                   ...
               ],
               "daily": {
                   "average": [
                       <filename|relative_path>,
                       ...
                   ],
                   "maximum": [
                       <filename|relative_path>,
                       ...
                   ],
               }
           },
           "netCDF": <filename|relative_path>,
           "kmz": <filename|relative_path>,
           "fireLocations": <filename|relative_path>, (needed?)
           "fireEvents": <filename|relative_path>, (needed?)
           "fireEmissions": <filename|relative_path> (needed?)
       }
    }

## API Aliases (Proposed)

### POST /api/v1/playground/1/

This is just an alias for ```/api/v1/run/```, and running ```ingestion```,
```fuelbeds```, ```consumption```, ```emissions``` - i.e. you don't have
to specify the 'modules' key.  Also

### POST /api/v1/playground/2/

This is just an alias for ```/api/v1/run/```, and running ```localmet```,
```timeprofiling```, ```plumerise```, ```dispersion```, ```visualization```,
```export```.  As with /1/, you don't have to specify either the 'modules'
key or the export type.

Example:

    $ curl 'http://bluesky-api-hostname/api/v1/playground/2/' -H 'Content-Type: application/json' -d '
    {
        "fire_information": [
            {
                "id": "SF11C14225236095807750",
                "event_id": "SF11E826544",
                "name": "Natural Fire near Snoqualmie Pass, WA",
                "location": {
                    "latitude": 47.4316976,
                    "longitude": -121.3990506,
                    "area": 200,
                    "utc_offset": "-09:00"
                },
                "fuelbeds": [
                    {
                        "fccs_id": "49",
                        "pct": 50.0,
                        "emissions": {
                            'ground fuels': {
                                'basal accumulations': {
                                    'flaming': {
                                        'PM2.5': [
                                            3.3815120047017005e-05
                                        ]
                                    },
                                    'residual': {
                                        'PM2.5': [
                                            4.621500211796271e-01
                                        ]
                                    },
                                    'smoldering': {
                                        'PM2.5': [
                                            6.424985839975172e-06
                                        ]
                                    }
                                }
                            }
                            /* ... (other emissions data) ... */
                        }
                    }
                ],
                "growth": [
                    {
                        "start": "20150120",
                        "end": "20150121",
                        "pct": 100.0
                    }
                ]
            }
        ],
        "config": {
            "start": "20150121T000000Z",
            "end": "20150123T000000Z",
            "met_domain": "PNW-4km",
            "timeprofiling": {
                "module": "custom"
            }
        }
    }'
