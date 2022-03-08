# Family App changes
## Introduction
Family apps (under Designed for Families program) should not “transmit” identifiers such as Advertising ID (AAID) from children and users of unknown age. This means that apps which solely target children should never transmit these identifiers, and apps that target children and older age groups may only transmit these identifiers from users that they know are adults. 
Details [link](https://support.google.com/googleplay/android-developer/answer/11043825 ) 

## Changes
* We need to introduce a new method to check if **setIsAgeRestricted** in the SDK.
* The possible values can be True or False.
* The default value will be False, ie user will be considered above age of consent.
* There is no common definition of age of consent, it should be determined by publisher based on law of the land.
* This flag should be transmitted along with the ad request as well **(u-age-restricted)**. In this phase, the server will not consume this flag, however, later we can use this to handle & log such requests.
* The **UnifId** should also NOT be passed when age restricted flag is True. This is done to take a conservative stance on children privacy. 
* There is no change for bloom filter implementation, as bloom filter is a probabilistic model and does on-device targeting (not transmitting data to any other partners)


<img width="234" alt="familyapp" src="https://user-images.githubusercontent.com/6571244/157170176-4a7e1235-1cb5-43cc-acb3-a113d40388cd.png">

## SDK Changes
* Public API 
<pre>
  public static void setIsAgeRestricted(boolean isAgeRestricted) {
        PublisherProvidedUserInfoDao.setIsAgeRestricted(isAgeRestricted);
        UidHelper.getInstance().setAgeRestricted();
        if (isAgeRestricted) {
            InMobiUnifiedIdService.reset();
        }
    }
  </pre>
* If Age restriction is true then we will avoid sending GPID/IDFA and Unifid in the APIs
* Key which we send in APIs **u-age-restricted** and value will be 0 or 1

## APIs parameter
<pre>
 u-age-restricted = 0 if Age restriction is false or else 1
 u-id-map = GPID/IDFA if age restriction is false or else UM5(MD5) and O1(SHA1) will be sent to API
</pre>

## APIs
<pre>
- https://ads.inmobi.com/sdk
- https://unif-id.ssp.inmobi.com/fetch (Avoid fetching the unifid if age restriction is true)
      - String USER_HAS_AGE_RESTRICTION = "User has age restriction";
- https://unif-id.ssp.inmobi.com/fetch (Avoid sending the unifid if age restriction is true)
</pre>

## Server Side Changes

This compliance is currently solved on the SDK to adhere with April deadline. The extra parameter discussed above will be passed on to the server along with ad request, which will be ignored as an extra value. Thus, no processing is to be done for this at the server for direct SDK requests.

However, in case of **Audience Bidding flow**:
1.	It is expected that the SDK will provide this flag value while getting the token from getToken() API. 
2.	In case of oRTB request coming from mediation partner, if the value of “u-age-restricted” signal is TRUE(1) in the token, and the API payload contains device id, DO NOT store or process the device id(GPID or IDFA). Note that in case of request coming from InMobi SDK, SDK will take care of not transmitting device id, but some mediation partner may still pass it.
3.	Need to support when **"u-id-adt"** is 0 and GPID(Android) is ‘string of zeroes’. This may be possible now when the AD_ID permission is missing. Currently the server drops such requests.

<img width="321" alt="server_side" src="https://user-images.githubusercontent.com/6571244/157184342-5e21fb16-a6de-4409-9fc8-d875eb9ac5cb.png">



