Telemetry Creative Upload
=========================

Creative-upload is awkward, since it requires a number of services.

Your application will need to:

* [Log in](../LoginService/README.md) to the test system, saving the cookies the login service generates
* [Generate Tickets](#generatetickets) from the login cookie
* Look up the SRV record for `_smelter._tcp.telemetry.com`
* *Optionally* ping each smelter by pulling https://*smelterhost*/blank.gif and chosing the closest host
* Check [the status](#check-status) of a file using its SHA-1 hash
* If the file hasn't been received, [upload the file](#upload-file)
* Continue to [check the status](#check-status) until the file is `DONE` or an `ERROR` occurs.

##GenerateTickets

Tickets are granted via the SOAP-based interface, but a full SOAP implementation is not required. You need to have obtained:

* [`ADVERTISER_GUID`](../Platform/CampaignService.md#getadvertisers)
* [`BRAND_GUID`](../Platform/CampaignService.md#getbrands)
* [`REGION_GUID`](../Platform/CampaignService.md#getregions)
* `FILE_SHA1_HASH` This can be one of two things
    * If the edition id is know the SHA1-hash of the raw [file](#formats) being uploaded. For interactives, this should be the sha1 of the main file XORed with the sha1 of the video.
    * Otherwise this needs to be some unique ID that will be used to refer to the file later. This needs to be a value that will be unique to the edition that gets created

Send an HTTP request that looks like the following to *endpointurl*`/soap/apiv03/`, e.g. `https://test5823901.telemetry.com/soap/apiv03/`: 

    Accept: text/xml
    Accept: multipart/*
    Accept: application/soap
    Content-Length: 634
    Content-Type: text/xml; charset=utf-8
    SOAPAction: "http://telemetry.com/dataapi/#GenerateTickets"

    <?xml version="1.0" encoding="UTF-8"?>
    <soap:Envelope soap:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"
                   xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
                   xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/"
                   xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <soap:Body>
        <GenerateTickets xmlns="http://telemetry.com/dataapi/">
          <Application>cut</Application>
          <Advertiser>ADVERTISER_GUID</Advertiser>
          <Brand>BRAND_GUID</Brand>
          <Region>REGION_GUID</Region>
          <Extra>{"tgxID":"FILE_SHA1_HASH","advertiser":"ADVERTISER_GUID"}</Extra>
        </GenerateTickets>
      </soap:Body>
    </soap:Envelope>


The response will be:

    <SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
                       xmlns:ns1="http://telemetry.com/dataapi/">
      <SOAP-ENV:Body>
        <ns1:GenerateTicketsReplyMsg>
          <Tickets>TICKET_STRING</Tickets>
        </ns1:GenerateTicketsReplyMsg>
      </SOAP-ENV:Body>
    </SOAP-ENV:Envelope>

Save the `TICKET_STRING` for [checking the status](#check-status) and [uploading the files](#upload-file).

##Upload File

This can be done in two ways: with or without an edition id. If an edition id id provided the tgxId used when you create the ticket must be sha1 of the file. Otherwise the tgxId in the ticket must be a value that will be unique to the edition that gets created.

You will need to know the:

* [`TICKET_STRING`](#generatetickets)
* [`CREATIVE_ID`](../Platform/CampaignService.md#getcreativeseditions) (optional)

You must have a file matching [the upload specs](#formats). If you are uploading video, you will also need a thumbnail for some vendors.

Connect to https://*SmelterEndpoint*`/process.php` if an edition id is pbeing provided, otherwise https://*SmelterEndpoint*`/processandcreate.php`, by looking up the SRV record `_smelter._tcp.telemetry.com`. Send an HTTP POST
of the content-type `multipart/form-data` with the following fields:

* `token` - the [`TICKET_STRING`](#generatetickets) above
* `edition_id` - the [`CREATIVE_ID`](../Platform/CampaignService.md#getcreativeseditions) (optional)
* `main_file` - the [raw file](#formats)
* `thumbnail` - if the `main_file` is a video, include a JPEG `512x288` pixels.
* `video_file` - if the `main_file` is an SWF (interactive), then this must be supplied. It is treated as a [raw file](#formats)
* creative_params - any query string parameters that need to be added to the end of the creative path as a string. Must include leading '?' (e.g. '?key1=value1&key2=value2')

The following fields are optional:

* `bypass_validation` - set to 1 to not validate the uploaded file meets the published Telemetry creative specification. This should only be used if trafficking of a not-to-spec video has been agreed with all parties. If this field is not set, or set to 0, then normal creative validation will be peformed by the smelter.

When the file is done uploading, you will get a JSON response:

    { "error_code": 0, "status": "PROCESSING" }
    
For unverified uplods this will also contain the edition id when it is available like this. 

    { "error_code": 0, "status": "PROCESSING", "edtiion" : "50004"}

You can receive the following status:

* `PROCESSING`: The file is being transcoded/converted
* `STAGING`: The file exists on the staging area (and has been accepted)
* `DONE`: The file is on Telemetry's content servers and is ready for serving.
* `BAD_HASH`: The hash in the ticket doesn't match the hash of the uploaded file.
* `ERROR`: Something is wrong. Record the response JSON.

If the response status is "ERROR", then an "error_msg" field will also be in the response JSON, providing a description of the error encountered.

##Check Status

You will need to know the:

* [`TICKET_STRING`](#generatetickets)
* [`CREATIVE_ID`](../Platform/CampaignService.md#getcreativeseditions) (only if you used one in the original request)

Connect to https://*SmelterEndpoint*`/process.php` or https://*SmelterEndpoint*`/processandcreate.php` (depenging on what you used earlier) by looking up the SRV record `_smelter._tcp.telemetry.com` and send the GET request:

    ?token=TICKET_STRING&edition_id=CREATIVE_ID

omit the edition id section if you did not provide one with the original upload

You will get a JSON response like this:

    { "error_code": 0, "status": "UNKNOWN_FILE" }

If the file has finished processing, it will look like this 

    { "error_code": 0, "status": "DONE", "edition"="50001", creative="40001", "paths" =["/tv2n/path/loc1.swf", "/tv2n/path/loc1_h264.mp4"] }
    
Note that from the 'process.php' url you will not set the edition and the creative ids.

You can receive the following status:

* `UNKNOWN_FILE`: The file must be uploaded. [Upload the files](#upload-file).
* `PROCESSING`: The file is being transcoded/converted
* `STAGING`: The file exists on the staging area (and has been accepted)
* `DONE`: The file is on Telemetry's content servers and is ready for serving.
* `ERROR`: Something is wrong. Record the response JSON.

##Formats

Refer to current published specifications:
http://techref.telemetryverification.net/creative/Materials%20Required%20for%20Creative%20Upload.pdf
