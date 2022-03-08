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
* API Details which we send **u-age-restricted** parameter

<pre>
 __u-age-restricted= 0 if Age restriction is false or else 1__
 u-id-map= GPID/IDFa if age restriction is false or else UM5(MD5) and O1(SHA1) 
</pre>

## APIs
<pre>
- https://ads.inmobi.com/sdk
- https://unif-id.ssp.inmobi.com/fetch (Avoid fetching the unifid if age restriction is true)
      - String USER_HAS_AGE_RESTRICTION = "User has age restriction";
- https://unif-id.ssp.inmobi.com/fetch (Avoid sending the unifid if age restriction is true)
</pre>


        
