---
layout: post
title: "Uber and the GDPR!"
date: 2020-04-01
author: "guiral Lacotte"
language: english
---

In these times of Covid-19 confinement, I finally took the time to do some projects that I had had in my notebooks for a while. One of them was to make a maximum of requests for access to my personal data via the [GDPR](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation) and to study the data sent to me. This in order to evaluate the implementation of the GDPR by the different companies and platforms. To detail the modalities of access, because I know some of you for whom it is easier to write a request to an [API](https://en.wikipedia.org/wiki/Application_programming_interface) than to write a handwritten letter with a photocopy of an identity document to the post office box of the legal department (accusing look towards the French banking system and telecom operators). But also to better understand the operation and business model of these platforms. And finally, to have real data in order to practice visualization of geolocation data with [Python](https://www.python.org/), [Folium](https://python-visualization.github.io/folium/) and [Leaflet](https://leafletjs.com/).

The goal is to make the work easier for beginners and to encourage users to exercise their access rights (a right you don't use is a right that is dying). There are no plans to engage in complex analyses (reverse engineering of mobile applications or APIs, de-anonymization). Unfortunately, I have neither the time nor the skills to do so.

In the first article, I will look at the Uber platform. It is not the first analysis I have done but it is one of the most interesting and the one that affects the largest population.

## The access request
You can request your data directly from the Uber website via this page: [help.uber.com/riders/...](https://help.uber.com/riders/article/que-contiennent-vos-donn%C3%A9es-t%C3%A9l%C3%A9charg%C3%A9es%C2%A0?nodeId=3d476006-87a4-4404-ac1e-216825414e05). In order to make the request, you must have kept your Uber account and the phone number used when registering the mobile application. Authentication to the website from a new device and for this type of request requires two-factor authentication which is performed by sending a code via SMS. Compared to my other requests which often require to go through email exchanges with the person in charge GDPR of the company, it remains rather simple. My request was processed in about 30 hours, which is also pretty good. I receive an e-mail with a link to a download page. The link is only valid for 7 days and the page requires two factors to authenticate. The data is sent as a [ZIP](https://en.wikipedia.org/wiki/ZIP_(file_format)) archive containing a tree structure and several [CVS](https://en.wikipedia.org/wiki/Comma-separated_values) files. 

As far as access request, transmission and security are concerned, Uber is one of the good guys.

## The data collected and transmitted
The data collected by Uber and transmitted within the framework of the GDPR requests are listed here: [help.uber.com/riders/...](https://help.uber.com/riders/article/que-contiennent-vos-donn%C3%A9es-t%C3%A9l%C3%A9charg%C3%A9es%C2%A0?nodeId=3d476006-87a4-4404-ac1e-216825414e05)
I'm particularly interested in geolocation data. 

> PASSENGER DATA Your passenger data contains the information used to get you to your destination, such as :   
> - The details of the journey such as **request times and locations**, start and end times, and distances travelled;
> the fare and currency used;
> - 30 days of mobile event data, such as the device operating system, device model, device language, application version, and **time and location of data collection**.

The tree structure of the archive is as follows:

    Uber Data/
    ├─ Account and Profile
    │ ├─ payment_methods.csv
    │ └ └─ profile_data.csv
    ├── readme.html
    ├── Regional information
    │ └── For_California_users.txt
    └── Rider
        ├── rider_app_analytics-0.csv
        └─ trips_data.csv

 - **payment_methods.csv** : The list of current and past cards and payment methods.
 - **profile_data.csv** : Corresponds to the user profile, it contains our first geolocation data via Signup Lat and Signup Long which are the coordinates when registering on the application. The field `Referred to Uber?` corresponds to the code of possible sponsor, the field `Referral Code` corresponds to the code of sponsorship. 
 - **readme.html**: A copy of the page listing the collected data.
 - **For_California_users.txt** : A careless user might pass this text file, but this one is actually very interesting. It lists the categories of partners who can potentially access the data collected by Uber and the partners with whom Uber has exchanged your data over the last twelve months. For example in my case:`In the past 12 months, Uber has sold your personal data to the following categories of third parties: -Advertising network providers`. Uber is rather tight-lipped about his business partners. 
 - **rider_app_analytics-0.csv**: Let's get serious, this file contains the "*30 days of mobile event data "*. It's a mine of information and particularly geolocalisation information. The volume of information is particularly astonishing. At the end of March 2020, after two weeks of confinement and almost no use of Uber except for the operations necessary for the access request, the file has more than 2000 lines.
 - **trips_data.csv**: Second particularly interesting file, this one contains all the "*details of the rides such as the times and places of request, start and end, as well as the distances covered*", since the registration on the platform. 

### rider_app_analytics-0.csv
As we have seen, this file contains all mobile event data.
As an example, here is an excerpt of an interaction with the application. During a ride at the other end of Paris, following a problem on one of the subway lines, I launched the application in order to evaluate the opportunity to take a Uber or public transport. I finally took public transport without making any reservations. According to the timestamp data, the operation took 2.966 seconds and generated 332 lines of events. 

 - Event Time (UTC),Session Start Time (UTC),GPS Time (UTC),Horizontal Accuracy,Latitude,Longitude,Speed (GPS),City,Cellular Carrier,Carrier MCC,Carrier MNC,Google Play Services Status,Device ID,IMEI number,IP Address,Device Language,Device Model,Device OS,Device OS Version,Region,Analytics Event Type 
- 2020-03-07 **13:07:53.469** 2020-03-07 13:07:49.979 2020-03-07 13:07:53.763 9.5 48.82764 2.33204 0.0 paris Orange F 208 1 success e80b48a6-08c6-4b4a-b457-XXX 10.35.167.XXX 100.69.178.XXX en SM-A600FN android 9 en_EN custom

- **... 330 lines ...**

- 2020-03-07 **13:07:56.435** 2020-03-07 13:07:49.979 2020-03-07 13:07:53.763 9.5 48.82764 2.33204 0.0 paris Orange F 208 1 success e80b48a6-08c6-4b4a-b457-XXX 10.35.167.XXX 100.69.178.XXX en SM-A600FN android 9 en_EN custom

When we look at the file, it appears that each line corresponds to an interaction with the application or a query of the Uber API. There are three types of "Analytics Event Type", custom, tap and print. Each move of the map, zoom, un-zoom, or click is recorded. 
The `GPS Time (UTC)` field is the date of the last known GPS position.
The `Horizontal Accuracy` field corresponds to the sigma standard deviation of the horizontal position. It is particularly useful to know whether the user is inside or outside a building.
The `Latitude,Longitude` fields are coded in decimal degrees to 5 decimal places.
the `Carrier MCC, Carrier MNC` fields are [Mobile_country_code](https://en.wikipedia.org/wiki/Mobile_country_code) and [Mobile_Network_Codes](https://en.wikipedia.org/wiki/Mobile_Network_Codes_in_ITU_region_2xx_(Europe)#France_-_FR), 208 01 corresponds to Orange France.
the `Device ID` field corresponds to the Google Play Services ID for Android.
the `IMEI number` field is empty. It may be a residue of data that is no longer collected, or conditionally (in the absence of a Device ID, at the time of registration).

In summary, this file allows us to trace all the places and dates of user interaction with the application, even if the user does not make any reservation. It also allows us to know if he was indoors or outdoors, on the move or static. If the user uses several terminals, it also allows us to identify the terminals, the mobile networks used, their models and the version of the operating system.



Unfortunately, I have no data for a real journey (reservation, pick-up, trip, drop-off) on this confinement period. I am not able to evaluate the application's recording time window during a ride. I have not observed any activity in the logs when the application is not in the foreground on the phone.

### trips_data.csv
This is the file that contains the logs of "*details of the rides such as request times and locations, start and end times, and distances traveled*". It has a fairly simple structure. Here is an example of a round trip, in order to preserve my privacy I chose a *slightly* distant route from my work and home places. 

 - City, Product Type, Trip or Order Status, Request Time, Begin Trip Time, Begin Trip Lat, Begin Trip Lng, Begin Trip Address, Dropoff Time, Dropoff Lat, Dropoff Lng Dropoff Address, Distance (miles), Fare Amount, Fare Currency
- Cape Town UberX COMPLETED 2019-02-15 19:42:32 +0000 UTC 2019-02-15 19:44:27 +0000 UTC -33.92129 18.41808 105 Bree St, Cape Town City Centre, Cape Town, 8001, South Africa 2019-02-15 19:51:24 +0000 UTC -33.90604 18.41783 3 Portswood Rd, V & A Waterfront, Cape Town, 8002, South Africa 1.66 31 ZAR
- Cape Town UberX COMPLETED 2019-02-15 17:28:51 +0000 UTC 2019-02-15 17:30:03 +0000 UTC -33.9060588 18.417833 3 Portswood Rd, V & A Waterfront, Cape Town, 8002, South Africa 2019-02-15 17:39:16 +0000 UTC -33.9244897 18.4195529 5 Wale St, Cape Town, 8001, ZA 1.94 40 ZAR

Each line corresponds to a ride, we find the coordinates for picking up `Begin Trip Lat, Begin Trip Lng` and dropping off `Dropoff Lat, Dropoff Lng`. They are encoded in decimal degrees to 5 decimal places.
This file is less verbose than the previous one, but allows us to trace the whole reservation history of the user. We can note that in a will to facilitate us the analysis Uber gives us the postal addresses of the places of pick up and drop off, in French (default langage on the mobil phone) . In this example we can easily identify that I was making a return trip from a hotel to a restaurant.  

## Geolocation data visualization 
The source code to view user activity points and routes is available here:
[https://github.com/berettavexee/Uber_map]

![Screenshot 00](/assets/img/ubermap-00.png)
![Screenshot 01](/assets/img/ubermap-01.png)

It's quite easy to use, just drop your `rider_app_analytics-0.csv` and `trips_data.csv` files into the uber subdirectory and run the script via `python3 uber_map.py` .

## Conclusions

 - It is quite easy to make the GDPR access request.
 - The data provided are easily usable.
 - Uber has a referral graph
 - Uber has a history of all interactions with the app, which includes the last known geolocation point over the last 30 days.
 - Uber keeps technical data that can identify all devices, telephone network and IP network used over the last 30 days.
 - Uber keeps a complete history of all trips made.
 - Some data provided by the user, such as driver or route ratings, have not been transmitted. 
