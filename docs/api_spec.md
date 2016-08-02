
# Common API information

Common information about all exposed API endpoints.

## Date format
Dates are formatted with [RFC 3339](https://www.ietf.org/rfc/rfc3339.txt) standard.

## OPTIONS method support
Each endpoint supports OPTIONS HTTP method.

          Allow: GET,POST,OPTIONS

## Cross-origin resource sharing

CORS and none-CORS requests are supported. Details [wikipedia](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing#Simple_example).

Allows `location` header exposure.

# Group Device

## Authorization

Incoming requests must set `Authorization` header and include device token
obtained from the API. The header shall look like this:

```
Authorization: Bearer <token>
# example
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

```

## List updates for device [GET /api/0.0.1/device/update]
List next update to be installed on the device.

+ Request

    + Headers

            Authorization: Bearer <token>

+ Response 200 (application/json)
    Next update for the device.

    + Schema

            {
                "$schema": "http://json-schema.org/draft-04/schema#",
                "id": "/",
                "type": "object",
                "properties": {
                    "image": {
                        "id": "image",
                        "type": "object",
                        "properties": {
                            "uri": {
                                "id": "uri",
                                "type": "string"
                            },
                            "checksum": {
                                "id": "checksum",
                                "type": "string"
                            },
                            "id": {
                                "id": "id",
                                "type": "string"
                            },
                            "expire": {
                                "id": "expire",
                                "type": "string"
                            },
                            "yocto_id": {
                                "id": "yocto_id",
                                "type": "string"
                            }
                        },
                        "required": [
                            "uri",
                            "id",
                            "yocto_id"
                        ]
                    },
                    "id": {
                        "id": "id",
                        "type": "string"
                    }
                },
                "required": [
                    "image",
                    "id"
                ]
            }

    + Body

            {
                "image": {
                    "uri": "https://aws.my_update_bucket.com/yocto_image123",
                    "checksum": "cc436f982bc60a8255fe1926a450db5f195a19ad",
                    "id": "f81d4fae-7dec-11d0-a765-00a0c91e6bf6",
                    "expire": "2016-03-11T13:03:17.063493443Z",
                    "yocto_id": "core-image-full-cmdline-20160330201408"
                },
                "id": "w81s4fae-7dec-11d0-a765-00a0c91e6bf6"
            }

+ Response 204

+ Response 400 (application/json)
    Invalid request.

    + Attributes (Error)

+ Response 404 (application/json)
    Resource not found.

    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)


## Report deployment status [PUT /api/0.1.0/device/deployments/{deployment_id}/status]

Allows to update the status of deployment on a particular device. Final status
of the deployment is required to be set to indicate end of the installation
process: success or failure. Reporting of intermediate steps such as installing,
downloading, rebooting is optional.

+ Parameters

    + deployment_id: `f81d4fae-7dec-11d0-a765-00a0c91e6bf6` (string,required) - Deplyoment ID

+ Request (application/json)

    + Headers

            Authorization: Bearer <token>

    + Attributes

            - status: installing, finished, pending (enum, required) - Deployment status

+ Response 204

+ Response 400 (application/json)
    Invalid request.

    + Attributes (Error)

+ Response 404 (application/json)
    Resource not found.

    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

# Group Deployment

## Lookup deployments [GET /api/0.0.1/deployments{?search,status}]
Lookup deployments in the system, including active and history.

+ Parameters
    + search: `Jonas fix` (string, optional) - Text to be matched in either
      deployment name or artifact name
    + status: `inprogress` (enum, optional) - Deployment status
        + Members
            + `inprogress` - Deployments in progress
            + `finished` - Finished deployments
            + `pending` - Pending deployments


+ Response 200 (application/json)

    + Attributes (Deployment Lookup Result)


+ Response 400 (application/json)
    Invalid request

    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

## Deploy software [POST /api/0.0.1/deployments]
Deploy software to specified devices. Image is auto assigned to the device from all available images based on artifact name and device type.
NOTE: Because of lack of inventory system, service assumes hardcoded device type for each device: "TestDevice"

+ Request (application/json)
    + Schema

            {
                "$schema": "http://json-schema.org/draft-04/schema#",
                "type": "object",
                "properties": ¸{
                    "name": {
                        "id": "name",
                        "type": "string"
                    },
                    "artifact_name": {
                        "id": "artifact_name",
                        "type": "string"
                    },
                    "devices": {
                        "id": "devices",
                        "type": "array",
                        "items": {
                            "type": "string"
                        }
                    }
                },
                "required": [
                    "name",
                    "artifact_name",
                    "devices"
                ]
            }

    + Body

            {
                "name": "Monthly update: January",
                "artifact_name": "MySecretApp v2",
                "devices": [
                    "00a0c91e6-7dec-11d0-a765-f81d4faebf6",
                    "50b0c91e6-1drc-51d0-a165-g81d4faebry"
                ]
            }

+ Response 201 (application/json)
    + Headers

            Location: /api/0.0.1/deployments/{id}

+ Response 400 (application/json)
    Bad request. The request could not be understood by the server due to malformed syntax.

    + Attributes (Error)

+ Response 404 (application/json)
    Resource not found

    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

## Manage deployment [/api/0.0.1/deployments/{id}]
Manage specific deployment.

### Status [GET]
Check status for specified deployment

+ Parameters
    + id: `f81d4fae-7dec-11d0-a765-00a0c91e6bf6` (string,required) - Deployment identifier

+ Response 200 (application/json)

    + Attributes (Deployment)

+ Response 404 (application/json)

    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

### Cancel [DELETE]
Cancel deployment.
TODO: To be implemented

+ Parameters
    + id: `f81d4fae-7dec-11d0-a765-00a0c91e6bf6` (string,required) - Deployment identifier

+ Response 204

+ Response 404 (application/json)

    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

### Statistics [GET /api/0.0.1/deployments/{deployment_id}/statistics]
Statistics for the deployment.
TODO: To be implemented & statuses may change

+ Parameters
    + deployment_id: `f81d4fae-7dec-11d0-a765-00a0c91e6bf6` (string,required) - Deployment identifier

+ Response 200 (application/json)

    + Attributes (Deployment)

+ Response 404 (application/json)
    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

### List devices [GET /api/0.0.1/deployments/{deployment_id}/devices]
Device statuses for the deployment.

+ Parameters
    + deployment_id: `f81d4fae-7dec-11d0-a765-00a0c91e6bf6` (string,required) - Deployment identifier

+ Response 200 (application/json)
    + Schema

            {
                "$schema": "http://json-schema.org/draft-04/schema#",
                "id": "/",
                "type": "array",
                "items": {
                    "id": "0",
                    "type": "object",
                    "properties": {
                        "id": {
                            "id": "id",
                            "type": "string"
                        },
                        "finished": {
                            "id": "finished",
                            "type": "string"
                        },
                        "status": {
                            "id": "status",
                            "type": "string",
                            "enum": [
                                "inprogress",
                                "pending",
                                "success",
                                "failure",
                                "noimage"
                            ]
                        },
                        "started": {
                            "id": "started",
                            "type": "string"
                        },
                        "device_type": {
                            "id": "device_type",
                            "type": "string"
                        },
                        "artifact_id": {
                            "id": "artifact_id",
                            "type": "string"
                        }
                    },
                    "required": [
                        "id",
                        "status",
                        "device_type"
                    ]
                }
            }

    + Body

            [
                {
                    "id": "00a0c91e6-7dec-11d0-a765-f81d4faebf6",
                    "finished": "2016-03-11T13:03:17.063493443Z",
                    "status": "pending",
                    "started": "2016-02-11T13:03:17.063493443Z",
                    "device_type": "Raspberry Pi 3",
                    "artifact_id": "60a0c91e6-7dec-11d0-a765-f81d4faebf6"
                }
            ]

+ Response 404 (application/json)
    + Schema

            {
                "$schema": "http://json-schema.org/draft-04/schema#",
                "type": "object",
                "properties": {
                    "error": {
                        "id": "error",
                        "type": "string"
                    }
                }
            }

    + Body

            {
                "error": "Detailed error message"
            }

+ Response 400 (application/json)
    Argument error - see response for details.

    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

### Send deployment log [PUT /api/0.1.0/device/deployments/{deployment_id}/log]
Set deployment log. Messages are split by line in the payload.

+ Parameters
    + deployment_id: `f81d4fae-7dec-11d0-a765-00a0c91e6bf6` (string,required) - Deplyoment ID

+ Request (application/json)

    + Headers

            Authorization: Bearer <token>

    + Schema

            {
                "$schema": "http://json-schema.org/draft-04/schema#",
                "type": "object",
                "properties": {
                    "messages": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "timestamp": {
                                    "type": "string"
                                },
                                "level": {
                                    "type": "string",
                                },
                                "message": {
                                    "type": "string"
                                }
                             },
                            "required": [
                                "timestamp",
                                "level",
                                "message"
                             ]
                        }
                    }
                },
                "required": [
                    "messages"
                ]
            }

    + Body

            {
                "messages": [
                    {
                        "timestamp": "2016-02-11T13:03:17.063493443Z",
                        "level": "error",
                        "message": "ASL Sender Statistics"
                    },
                    {
                        "timestamp": "2016-02-11T13:03:18.063493443Z",
                        "level": "notice",
                        "message": "404 not found"
                    }
                ]
            }

+ Response 204

+ Response 400 (application/json)
    Invalid request.

    + Attributes (Error)

+ Response 404 (application/json)
    Resource not found.

    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

### Deployment log [GET /api/0.0.1/deployments/{deployment_id}/devices/{device_id}/log]
Device statuses for the deployment.
TODO: To be implemented

+ Parameters
    + deployment_id: `f81d4fae-7dec-11d0-a765-00a0c91e6bf6` (string,required) - Deployment identifier
    + device_id: `00a0c91e6-7dec-11d0-a765-f81d4faebf6` (string, required) - Device identifier

+ Response 200 (application/text)

    ```
    Feb 23 15:43:38 mrowa BezelServices 255.10[92] <Error>: ASSERTION FAILED: dvcAddrRef != ((void *)0) -[DriverServicesgetDeviceAddress:] Feb 23 15:45:10 mrowa com.apple.WebKit.WebContent[606] <Error>: [15:45:10.933] FigAgglomeratorSetObjectForKey signalled err=-16020 (kFigStringConformerError_ParamErr) (NULL key) at /Library/Caches/com.apple.xbs/Sources/CoreMedia/CoreMedia-1731.15.33/Prototypes/LegibleOutput/FigAgglomerator.c line 92 Feb 23 15:45:18 mrowa com.apple.WebKit.WebContent[606] <Error>: [15:45:18.956] <<<< Boss >>>> figPlaybackBossPrerollCompleted: unexpected preroll-complete notification
    Feb 23 15:45:18 mrowa com.apple.WebKit.WebContent[606] <Error>: [15:45:18.957] <<<< Boss >>>> figPlaybackBossPrerollCompleted: unexpected preroll-complete notification
    Feb 23 15:45:40 mrowa syslogd[44] <Notice>: ASL Sender Statistics

    ```

+ Response 404 (application/json)
    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

# Group YOCTO images
Manage YOCTO images.

## Lookup images [GET /api/0.0.1/images]
List all YOCTO images.

+ Response 200 (application/json)

    + Attributes (array[Image Details])

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

## Create image [POST /api/0.0.1/images]
Create YOCTO image. Afterwards upload link can be generated to upload image file.

Notice:
Every image must link to a unique name and device_type.

+ Request (application/json)

    + Attributes (Image)

+ Response 201
    + Headers

            Location: /api/0.0.1/images/{id}

+ Response 404 (application/json)
    Resource not found

    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

## Manage images [/api/0.0.1/images/{id}]
Manage selected image

### Image details [GET]
Image datails.

+ Parameters
   + id: `f81d4fae-7dec-11d0-a765-00a0c91e6bf6` (string,required) - Image ID

+ Response 200 (application/json)

    + Attributes (Image Details)

+ Response 404 (application/json)

    + Attributes (Error)

+ Response 500 (application/json)
   Internal server error. Please retry in a while.

    + Attributes (Error)

### Edit image [PUT]
Edit image information.
Image is not allowed to be edited if it was used in any deployment.

+ Parameters
   + id: `f81d4fae-7dec-11d0-a765-00a0c91e6bf6` (string,required) - Image ID

+ Request (application/json)

    + Attributes (Image)

+ Response 204

+ Response 400 (application/json)
    Invalid request

    + Attributes (Error)

+ Response 404 (application/json)
    + Attributes (Error)

+ Response 500 (application/json)
   Internal server error. Please retry in a while.

    + Attributes (Error)

### Remove image [DELETE]
Remove YOCTO image.
Image is not allowed to be deleted if it is a part of pending or active deployment.

+ Parameters
   + id: `f81d4fae-7dec-11d0-a765-00a0c91e6bf6` (string,required) - Image ID

+ Response 204

+ Response 404 (application/json)
    + Attributes (Error)

+ Response 500 (application/json)
   Internal server error. Please retry in a while.

    + Attributes (Error)

### Generate upload link [GET /api/0.0.1/images/{id}/upload{?expire}]
Generate signed URL for uploading image file.

URI can be used only with PUT HTTP method.
It is valid for specified period if time.

In case link is used multiple times to upload file, file will be overwritten.

+ Parameters
    + id: `0C13A0E6-6B63-475D-8260-EE42A590E8FF` (string, required) - Image ID
    + expire: 60 (number, optional) - Link validity length in minutes. Min 1 minute, max 10080 (1 week)
        + Default: 60

+ Response 200 (application/json)

    + Attributes (Image Link)

+ Response 400 (application/json)
    Invalid request

    + Attributes (Error)

+ Response 404 (application/json)

    + Attributes (Error)

+ Response 500 (application/json)
   Internal server error. Please retry in a while.

    + Attributes (Error)

### Generate download link [GET /api/0.0.1/images/{id}/download{?expire}]
Generate signed URL for downloading image file.

URI can be used only with GET HTTP method.
Link supports such HTTP headers: `Range`, `If-Modified-Since`, `If-Unmodified-Since`
It is valid for specified period if time.

To be able to recieve download link, image file have to be uploaded first.

+ Parameters
    + id: `0C13A0E6-6B63-475D-8260-EE42A590E8FF` (string, required) - Image ID
    + expire: 60 (number, optional) - Link validity length in minutes. Min 1
      minute, max 10080 (1 week)
        + Default: 60

+ Response 200 (application/json)

    + Attributes (Image Link)

+ Response 400 (application/json)
    Invalid request

    + Attributes (Error)

+ Response 404 (application/json)

    Resource not found. Could mean for not having access, image does not exist
    or file have not been uploaded.

    + Attributes (Error)

+ Response 500 (application/json)
    Internal server error. Please retry in a while.

    + Attributes (Error)

# Data Structures

## Error (object)

+ error: `Detailed error message` (string, required)

## Deployment Base (object)

+ created: `2016-02-11T13:03:17.063493443Z` (string, required)
+ name: `production` (string, required)
+ artifact_name: `Application 0.0.1` (string, required)
+ id: `00a0c91e6-7dec-11d0-a765-f81d4faebf6` (string, required)
+ finished: `2016-03-11T13:03:17.063493443Z` (string, optional)

## Deployment Lookup Result (Deployment Base)

+ status: pending, inprogress, finished (enum, required) - Deployment status

## Deployment (Deployment Base)

+ stats (Deployment Statistics)

## Deployment Statistics (object)

+ success: 3 (number, required) - Number of successful deployments
+ pending: 2 (number, required) - Number of pending deployments
+ downloading: 1 (number, required) - Number of deployments being downloaded
+ rebooting: 4 (number, required) - Number of deployments devices are rebooting into
+ installing: 0 (number, required) - Number of deployments being installed
+ failure: 2 (number, required) - Number of failed deployments
+ noimage: 0 (number, required) - Do not have apropriate image for the device model

## Image Link (object)

+ uri: `https://exmple.com/file123` (string, required) - Download URL
+ expire: `2016-03-11T13:03:17.063493443Z` (string) - Expiration date/time

## Image (object)

+ name: `Application 1.1` (string) - Image name, unique across all images
+ description: `Monday build for production` (string)
+ checksum: `cc436f982bc60a8255fe1926a450db5f195a19ad` (string)
+ device_type: `Beagle Bone` (string)
+ yocto_id: `core-image-full-cmdline-20160330201408` (string)

## Image Details (Image)

+ id: `0C13A0E6-6B63-475D-8260-EE42A590E8FF` (string)
+ verified: false (boolean)
+ modified: `2016-03-11T13:03:17.063493443Z` (string) - Creation/last modification date/time
