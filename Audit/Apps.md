Telemetry Audit API - Apps
==========================

Apps Audiding uses HTTP POST against *endpointurl*`/Audit/(apps)`*functionname* for example if your test service is:

    url=https://test5823901.telemetry.com/
    username=x
    password=x

and you wish to perform the [searchApps](#searchApps) function, you perform an HTTP post against:

    https://test5823901.telemetry.com/Audit/(apps)searchApps


## searchApps

Returns details for Apps matching their "bundle" IDs against the provided search. The input consists of a search string (**"search"**), which needs to be 3 or more characters in length. You can specify a specific (**"limit"**) for the number of apps returned, otherwise it defaults to the first 50 matches. There is also an optional Store Identifier (**"store"**) to specify which App Store we are looking in. The possible stores are:

* **iTunes** = Apple's App Store
* **Google Play** = For Android Apps

The returned list (**"apps"**) is sorted by bundleID. Each object in the list contains a (**"stores"**) key, within which is the details for the app, keyed by each store it appears in. This exists because the bundle might be present in multiple stores. The included details for each app are:

* **telemetryID** = our own unique identifier for an app
* **bundleID** = the ID used for this app in the relevant store
* **store** = the App Store where this app can be found (eg, iTunes)
* **os** = the platform for this application (eg, iOS)
* **name** = Title of the app
* **developer** = Name of the developer (person/company)
* **status** = Tells us if we have data for the app, or if it's new. Possible values are: 'NODATA', 'NEW', and 'OK'
* **added_on** = The date at which we entered the application into our App Database

Example Input (all stores):

    {
        "search": "ebo"
    }

Example Input (look only in Apple's store, limit to first 100 apps found):

    {
        "search": "ebo",
        "store": "iTunes",
        "limit": 100
    }
    
Example Output (all stores):

    {
        "success":true,
        "apps":
        [
            {
                "bundleID": "info.smartpocket.ebook590",
                "stores":
                {
                    "iTunes":
                    {
                        "telemetryID": 6,
                        "bundleID": "info.smartpocket.ebook590",
                        "store": "iTunes",
                        "os": "iOS",
                        "name": "Christmas Sunshine",
                        "developer": "DSG",
                        "status": "OK",
                        "added_on": "2015-01-15 12:30:12"
                    },
                    "Google Play":    
                    {
                        "telemetryID": 6215,
                        "bundleID": "info.smartpocket.ebook590",
                        "store": "Google Play",
                        "os": "Android",
                        "name": "Christmas Sunshine ANDROID",
                        "developer": "DSG",
                        "status": "OK",
                        "added_on": "2015-01-15 12:39:44"
                    }
                }
            },
            {
                "bundleID": "at.fms-datenfunk.MobileBookingAmager",
                "stores":
                {
                    "iTunes":
                    {
                        "telemetryID": 806,
                        "bundleID": "at.fms-datenfunk.MobileBookingAmager",
                        "store": "iTunes",
                        "os": "iOS",
                        "name": "Taxi Denmark",
                        "developer": "fms Datenfunk GmbH",
                        "status": "OK",
                        "added_on": "2015-01-15 12:30:12"
                    }
                }
            }
        ]
    }


## getTopApps

Receives information for those Apps which delivered the most within a given time period. This can be called with no arguments, and as such defaults to the top 50 Apps within the last 24 hours. A specific time range (**"start"** and **"end"**) can be provided in SQL format, as can an optional size of return list (**"limit"**). 

The output of this function matches that of [searchApps](#searchApps) - with the addition of delivery information for each App *across all stores* (as we cannot break down the rare cases in multiple stores), including:

* **ad_starts** - Number of ad starts within the app within the time frame
* **ad_requests** - Number of requests for an ad from the app within the time frame

The returned bundles are sorted (in descending order, so the "top" app is first) by **Ad Requests**. This is so we are shown the most requested apps; there may be a delta between ad request and start which we want to observe.

Example input showing overrides:

    {
        "start": "2015-01-01 00:00:00",
        "end": "2015-02-01 00:00:00",
        "limit": 100
    }

Example output:

    {
        "success":true,
        "apps":
        [
            {
                "bundleID": "info.smartpocket.ebook590",
                "ad_starts": 3315351,
                "ad_requests": 3323153,
                "stores":
                {
                    "iTunes":
                    {
                        "telemetryID": 6,
                        "bundleID": "info.smartpocket.ebook590",
                        "store": "iTunes",
                        "os": "iOS",
                        "name": "Christmas Sunshine",
                        "developer": "DSG",
                        "status": "OK",
                        "added_on": "2015-01-15 12:30:12"
                    },
                    "Google Play":    
                    {
                        "telemetryID": 6215,
                        "bundleID": "info.smartpocket.ebook590",
                        "store": "Google Play",
                        "os": "Android",
                        "name": "Christmas Sunshine ANDROID",
                        "developer": "DSG",
                        "status": "OK",
                        "added_on": "2015-01-15 12:39:44"
                    }
                }
            },
            {
                "bundleID": "at.fms-datenfunk.MobileBookingAmager",
                "ad_starts": 125351,
                "ad_requests": 231533,
                "stores":
                {
                    "iTunes":
                    {
                        "telemetryID": 806,
                        "bundleID": "at.fms-datenfunk.MobileBookingAmager",
                        "store": "iTunes",
                        "os": "iOS",
                        "name": "Taxi Denmark",
                        "developer": "fms Datenfunk GmbH",
                        "status": "OK",
                        "added_on": "2015-01-15 12:30:12"
                    }
                }
            }
        ]
    }


## addObservation

Receives information relating to an observation made on a given app. Provide the Bundle ID (**"bundleID"**), App Store (**"store"**) and an object (**"data"**) for the following observation areas:

* **time_spent** - Length of time in seconds the user spent testing the app before submitting this observation
* **visible** = Could we see the ad in the App? (true/false) 
* **audible** = Could we hear the ad in the App? (true/false), 
* **click_to_start** = Did we have to click a "Play" button to start the ad? (true/false) 
* **responsive** = Did the ad look and perform ok, or was it a bad/choppy/unresponsive experience? (true/false)
* **skippable** = Did the app provide a "skip" button at any point to get rid of the ad? And did it work? (true/false)
* **never_saw_ad** = True if we never managed to see an ad in the App. (true/false)

The received output will be confirmation of the full observation data, with the following additions which are recorded after the submission:

* **created_on** - Time/Date when the observation was recorded (SQL format).
* **created_by** - User who recorded the observation (username).

Example:

    {
        "bundleID": "com.stimunationgames.stuntmanbob",
        "store": "Google Play"
        "data": 
        {
            "time_spent": 60, 
            "visible": true, 
            "audible": false, 
            "click_to_start": true, 
            "responsive": true, 
            "skippable": true,
            "never_saw_ad": false
        }
    }

Example output:

    {
        "success":true,
        "observations":
        [
            {
                "created_on": "2015-01-25 11:35:01", 
                "created_by": "jdoe"
                "time_spent": 60, 
                "visible": true, 
                "audible": false, 
                "click_to_start": true, 
                "responsive": true, 
                "skippable": true,
                "never_saw_ad": false
            }
        ]
    }
    
## getEvents

Returns data for observations made against the provided App. The input requires a full and valid bundle ID (**"bundleID"**), App Store (**"store"**), and the return value contains a list (**"observations"**), sorted with most recent first, and containing data for each observation (with a maximum of the last 50 observerations).

Example:

    {
        "bundleID": "com.stimunationgames.stuntmanbob",
        "store": "Google Play"
    }


Each item in the list contains the same data as is received when submitting data using [addObservation](#addObservation)

Example Output:

    {
        "success":true,
        "observations":
        [
            {
                "created_on": "2015-01-25 11:35:01", 
                "created_by": "jdoe"
                "time_spent": 60, 
                "visible": true, 
                "audible": false, 
                "click_to_start": true, 
                "responsive": true, 
                "skippable": true,
                "never_saw_ad": false
            },
            {
                "created_on": "2015-01-25 12:10:50", 
                "created_by": "rhead"
                "time_spent": 60, 
                "visible": true, 
                "audible": true, 
                "click_to_start": false, 
                "responsive": true, 
                "skippable": false,
                "never_saw_ad": false
            }
        ]
    }

## getScores

Summarises all observations made into a set of "scores", for each App provided in the request. Provide a list (**"apps"**) of bundleID + store pairs for each app you want scores for, and the return value will contain the scores for numerous areas relating to the observations, along with the number of observations which have been used for scoring. The return value is sorted by Bundle ID.

Example Input:

    {
        "apps":
        [
            {
                "bundleID": "info.smartpocket.ebook590",
                "store": "iTunes"
            },
            {
                "bundleID": "com.stimunationgames.stuntmanbob",
                "store": "Google Play"
            }
        ]
    }
    
Example output:

    {
        "success": true,
        "scores":
        [
            {
                "bundleID": "info.smartpocket.ebook590",
                "store": "iTunes",
                "numObservations": 17,
                "score_visibility": 8.7,
                "score_audibility": 6,
                "score_responsiveness": 9.3
            },
            {
                "bundleID": "com.stimunationgames.stuntmanbob",
                "store": "Google Play",
                "numObservations": 2,
                "score_visibility": 8.7,
                "score_audibility": 6,
                "score_responsiveness": 9.3
            }
        ]
    }
