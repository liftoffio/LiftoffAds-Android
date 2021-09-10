# LiftoffAds Android SDK

This repository describes the LiftoffAds Android SDK technical integration. For
help configuring and testing ad units or setting up reporting, please contact
your Liftoff POC.

For any other questions, please email sdk@liftoff.io.

## Table of Contents

- [Overview](#overview)
  - [Latest Releases](#latest-releases)
  - [Supported Devices](#supported-devices)
  - [Supported Ad Sizes](#supported-ad-sizes)
  - [Supported Ad Types](#supported-ad-types)
- [Development Requirements](#development-requirements)
- [Integrating the SDK](#integrating-the-sdk)
  - [Downloading the SDK](#downloading-the-sdk)
    - [Maven Central](#maven-central)
    - [Direct Download](#direct-download)
  - [Code Changes](#code-changes)
    - [MoPub Mediation](#mopub-mediation)
    - [Self Mediation](#self-mediation)
    - [GDPR/CCPA and User Consent](#gdprccpa-and-user-consent)
    - [Test Ad Units](#test-ad-units)
  - [Creating a MoPub Custom SDK Network](#creating-a-mopub-custom-sdk-network)
- [COPPA](#coppa)
- [Troubleshooting](#troubleshooting)
- [Reporting](#reporting)

## Overview

### Latest Releases

- [LiftoffAds SDK][latest-display-sdk]
- Mediation Adapter SDKs
  - [Liftoff MoPub Adapter SDK][latest-mopub]

### Supported Devices

- Phone
- Tablet

### Supported Ad Sizes

- 320x480 (phone portrait interstitial)
- 480x320 (phone landscape interstitial)
- 768x1024 (tablet portrait interstitial)
- 1024x768 (tablet landscape interstitial)
- 320x50 (phone banner)
- 728x90 (tablet banner)
- 300x250 (medium rectangle)
- 0x0 (native)

### Supported Ad Types

- HTML and HTML video
- VAST video
- Rewarded
- Native

## Development Requirements

To integrate the latest version of the LiftoffAds display SDK, you will need at
minimum:

- Android API >= 19

## Integrating the SDK

### Downloading the SDK

You can download the SDK through Maven Central or directly.

#### Maven Central

The easiest way to add the LiftoffAds SDK to your project is via Maven Central.

Add Maven Central to the `repositories` block of your project's `build.gradle`:

```
repositories {
  // ...
  mavenCentral()
}
```

Add the `LiftoffAds` SDK as a dependency to your app module's `build.gradle`:

```
dependencies {
  // Only if using MoPub mediation
  implementation "io.liftoff:liftoffads-mopub:1.2.1"
  // Only if using custom mediation
  implementation "io.liftoff:liftoffads:1.3.0"
}
```

#### Direct Download

If you aren't using Maven Central as described above, you can download the SDK
and manually add it to your project:

1. Download and unzip the [LiftoffAds SDK][latest-display-sdk].
2. Move the extracted `liftoffads.aar` to your local repository.

```
repositories {
  // ...
  flatDir {
    dirs "libs"
  }
}

dependencies {
  implementation(name:"liftoffads", ext:"aar")
}
```

Only if you are using MoPub mediation, download the Liftoff MoPub Adapter SDK
and add it to your project:

1. Download and unzip the [Liftoff MoPub Adapter SDK][latest-mopub].
2. Move the extracted `liftoffads-mopub.aar` to your local repository.

```
repositories {
  // ...
  flatDir {
    dirs "libs"
  }
}

dependencies {
  implementation(name:"liftoffads-mopub", ext:"aar")
}
```

### Code Changes

We currently support MoPub mediation and self-mediated setups. If you are using
a self-mediated setup, skip the section below and continue with [Self
Mediation](#self-mediation).

#### MoPub Mediation

LiftoffAds can be added as a MoPub custom SDK network.

Instructions for requesting and displaying ads through MoPub can be found in the
[MoPub documentation](https://developers.mopub.com/publishers/android/integrate/).

The following code samples may be used as reference for initializing the
LiftoffAds Android SDK through MoPub. Adjust the logic as necessary to meet the
requirements of your app.

```kotlin
import com.mopub.common.MoPub
import com.mopub.common.SdkConfiguration
import com.mopub.common.SdkInitializationListener
import com.mopub.common.logging.MoPubLog
import com.mopub.mobileads.LiftoffAdapterConfiguration

class MainActivity : AppCompatActivity(), SdkInitializationListener {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val sdkConfig = SdkConfiguration.Builder("MOPUB_AD_UNIT_ID")

    // BEGIN: Required for Liftoff custom SDK network.
    with(sdkConfig) {
      withLogLevel(MoPubLog.LogLevel.DEBUG)
      withAdditionalNetwork(LiftoffAdapterConfiguration::class.java.name)
      withMediatedNetworkConfiguration(
        LiftoffAdapterConfiguration::class.java.name,
        // Contact your Liftoff POC to retrieve your API key. Define this
        // constant elsewhere or replace with a hardcoded string.
        mapOf("apiKey" to LIFTOFF_API_KEY)
      )
    }
    // END

    MoPub.initializeSdk(this, sdkConfig.build(), this)
  }

  // Native Ads
  fun setUpNativeAds() {
    val moPubNative = MoPubNative(
      /* context= */ this, MOPUB_AD_UNIT_ID, nativeNetworkListener
    )
    moPubNative.registerAdRenderer(
      MoPubStaticNativeAdRenderer(
        ViewBinder.Builder(R.layout.native_ad)
          .titleId(R.id.nativeAdTitle)
          .textId(R.id.nativeAdDescription)
          .callToActionId(R.id.nativeAdCTA)
          .iconImageId(R.id.nativeAdIcon)
          .mainImageId(R.id.nativeAdMainImage)
          .build()
      )
    )
    moPubNative.makeRequest()
  }
}
```

#### Self Mediation

The following code samples may be used as reference for requesting and
displaying ads. Adjust the logic as necessary to meet the requirements of your
app.

```kotlin
import io.liftoff.liftoffads.Liftoff
import io.liftoff.liftoffads.banners.LOBanner
import io.liftoff.liftoffads.common.AdSize
import io.liftoff.liftoffads.interstitials.LOInterstitial

class Activity :
  AppCompactActivity(),
  LOInterstitial.LOInterstitialListener,
  LOBanner.LOBannerListener {
  lateinit var loInterstitial: LOInterstitial
  lateinit var loBanner: LOBanner
  lateinit var loNative: LONative

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    // Supported log levels: NONE, ERROR, INFO, DEBUG.
    Liftoff.logLevel = Liftoff.LogLevel.DEBUG

    // Contact your Liftoff POC to retrieve your API key. Define this constant
    // elsewhere or replace with a hardcoded string.
    Liftoff.initializeSDK(LIFTOFF_API_KEY)

    Log.d("TAG", "Initialized LiftoffAds SDK ${Liftoff.SDK_VERSION}")

    // Contact your Liftoff POC to retrieve your ad unit IDs.
    // NOTE: Liftoff interstitial, banner, and native objects cannot be reused
    // to request multiple ads. You must initialize a new object for each ad
    // request.

    this.loInterstitial = Liftoff.newInterstitial(this, LIFTOFF_INTERSTITIAL_AD_UNIT, this)
    this.loInterstitial.load()

    // The size argument may be one of the following (a 0 indicates a flexible
    // dimension):
    // AdSize.PHONE_BANNER             // width: 320, height:  50
    // AdSize.PHONE_BANNER_FLEX_WIDTH  // width:   0, height:  50
    // AdSize.TABLET_BANNER            // width: 728, height:  90
    // AdSize.TABLET_BANNER_FLEX_WIDTH // width:   0, height:  90
    // AdSize.MRECT                    // width: 300, height: 250
    // AdSize.MRECT_FLEX_WIDTH         // width:   0, height: 250
    // AdSize.FLEX_ALL                 // width:   0, height:   0
    this.loBanner = Liftoff.newBanner(
      this,
      LIFTOFF_BANNER_AD_UNIT,
      AdSize.PHONE_BANNER_FLEX_WIDTH,
      this
    )
    this.loBanner.load()

    this.loNative = Liftoff.newNative(
      activity = this,
      LIFTOFF_NATIVE_AD_UNIT,
      LONative.ViewBinder(
        layout = R.layout.native_ad,
        parent = nativePlaceholderView,
        title = R.id.nativeAdTitle,
        desc = R.id.nativeAdDescription,
        callToAction = R.id.nativeAdCTA,
        icon = R.id.nativeAdIcon,
        mainImage = R.id.nativeAdMainImage
      ),
      listener = this
    )
    this.loNative.load()
  }


  // LOInterstitial.LOInterstitialListener implementation

  // Called when the interstitial ad request is successfully filled.
  override fun onInterstitialLoaded(interstitial: LOInterstitial) {
    // This will display the interstitial immediately after the ad request is
    // filled.
    interstitial.showAd()

    // To instead display the interstitial at an appropriate time as determined
    // by your app UX:
    // if (this.loInterstitial.ready) {
    //   this.loInterstitial.showAd()
    // }
  }

  // Called when the interstitial ad request cannot be filled.
  override fun onInterstitialFailed(interstitial: LOInterstitial, error: String?) {}

  // Called after the interstitial activity is shown.
  override fun onInterstitialShown(interstitial: LOInterstitial) {}

  // Called after the interstitial activity is dismissed.
  override fun onInterstitialDismissed(interstitial: LOInterstitial) {}

  // Called when the interstitial becomes visible to the user.
  override fun onInterstitialImpression(interstitial: LOInterstitial) {}

  // Called when the user will be directed to an external destination.
  override fun onInterstitialClicked(interstitial: LOInterstitial) {}

  // Called when the user has earned a reward by watching a rewarded ad.
  override fun onRewardEarned(interstitial: LOInterstitial) {}


  // LOBanner.LOBannerListener implementation

  // Called when the banner ad request is successfully filled. The banner
  // argument is a View.
  override fun onBannerLoaded(banner: LOBanner) {
    this.someView.addView(banner)
  }

  // Called when the banner ad request cannot be filled.
  override fun onBannerFailed(banner: LOBanner, error: String?) {}

  // Called when the banner becomes visible to the user.
  override fun onBannerImpression(banner: LOBanner) {}

  // Called when the user will be directed to an external destination.
  override fun onBannerClicked(banner: LOBanner) {}

  // LONative.LONativeListener implementation

  // Called when the ad has been loaded successfully.
  override fun onNativeLoaded(native: LONative) {}

  // Called when the ad view has been inflated and the text views are set to
  // the values from the loaded ad.
  override fun onNativeReady(native: LONative, adView: View) {}

  // Called when the icon has been loaded successfully.
  override fun onNativeIconLoaded(native: LONative) {}

  // Called if loading the icon has failed.
  override fun onNativeIconFailed(native: LONative, error: String?) {}

  // Called when the main image has been loaded successfully.
  override fun onNativeMainImageLoaded(native: LONative) {}

  // Called if loading the main image has failed.
  override fun onNativeMainImageFailed(native: LONative, error: String?) {}

  // Called if loading has failed.
  override fun onNativeFailed(native: LONative, error: String?) {}

  // Called when the ad view is actually shown.
  override fun onNativeImpression(native: LONative) {}

  // Called when the CTA element is clicked.
  override fun onNativeClicked(native: LONative) {}
}
```

#### GDPR/CCPA and User Consent

LiftoffAds complies with the EU's General Data Protection Regulation (GDPR) and
the California Consumer Privacy Act (CCPA). However, LiftoffAds does not
currently manage its own consent mechanism, so you will be required to pass user
consent information to our SDK. The following code samples can be used as
reference:

```kotlin
LOPrivacySettings.hasUserConsent = true
```

#### Test Ad Units

Use the following ad unit IDs to display a LiftoffAds test creative and verify
successful integration.

| Ad Unit ID                        | Size                  | Type                   |
| --------------------------------- | --------------------- | ---------------------- |
| `liftoff-banner-mrect-test`       | Banner / MRECT        | VAST video, HTML video |
| `liftoff-interstitial-video-test` | Interstitial          | VAST video             |
| `liftoff-interstitial-html-test`  | Interstitial          | HTML video             |
| `liftoff-rewarded-video-test`     | Rewarded Interstitial | VAST video             |
| `liftoff-native-test`             | Native                | Native                 |

### Creating a MoPub Custom SDK Network

*You may skip this section if you are not using MoPub mediation.*

You will need to create a new custom SDK network in the MoPub web portal for the
Liftoff ad network.

1. Create a new custom SDK network for Liftoff in [MoPub Networks](https://app.mopub.com/networks).
2. Create a new order for Liftoff in [MoPub Orders](https://app.mopub.com/orders).
3. Create line items for your Liftoff ad units. Contact your Liftoff POC to set
   up ad units and retrieve your ad unit IDs.

The screenshots below show example configurations for Liftoff banner/medium
rectangle, standard interstitial, rewarded interstitial, and native line items.

#### Banner / Medium Rectangle (MRECT)

Both banner and mrect ads use the same custom event class.

Custom event class: `com.mopub.mobileads.LiftoffBanner`

Custom event data: `{"adUnitID": "LIFTOFF_BANNER_AD_UNIT_ID"}`

![](https://user-images.githubusercontent.com/573865/116912857-9e8c4300-abfd-11eb-8529-96cd3caa9a4b.png)

#### Interstitial

Custom event class: `com.mopub.mobileads.LiftoffInterstitial`

Custom event data: `{"adUnitID": "LIFTOFF_INTERSTITIAL_AD_UNIT_ID"}`

![](https://user-images.githubusercontent.com/573865/116912861-9f24d980-abfd-11eb-99ad-5b9969bcf71b.png)

#### Rewarded Interstitial

Custom event class: `com.mopub.mobileads.LiftoffRewardedInterstitial`

Custom event data: `{"adUnitID": "LIFTOFF_REWARDED_INTERSTITIAL_AD_UNIT_ID"}`

![](https://user-images.githubusercontent.com/573865/116912863-9fbd7000-abfd-11eb-826f-faa17392aeaa.png)

#### Native

Custom event class: `com.mopub.nativeads.LiftoffNative`

Custom event data: `{"adUnitID": "LIFTOFF_NATIVE_AD_UNIT_ID"}`

![](https://user-images.githubusercontent.com/11006152/128574243-2be1c5d4-613e-4885-a378-ad6c48cf2a1d.png)

## COPPA

LiftoffAds does not serve end users who fall under the restrictions of the
Children’s Online Privacy Protection Act (COPPA), specifically age 12 years and
younger. If you collect information that indicates a user falls under this
category, you must not use the LiftoffAds SDK for the user's sessions.

## Troubleshooting

Set the log level to `DEBUG` before troubleshooting.

Common integration issues:

- `"No API key provided"`: Missing API key. Verify that you've included the
  proper initialization code (see above).
- `"Error authentication: Check API key"`: Incorrect API key. Check for any
  typos in your API key.
- `"Unable to fetch ad unit: <PROVIDED_AD_UNIT_ID>"`: Could not fill ad request.
  Check ad unit ID for typos.

## Reporting

Reporting is available via programmatic API or scheduled emails.

* [Reporting API documentation](https://liftoff.io/support/publisher-reporting-api/)
* To receive scheduled email reports, contact your Liftoff POC with your desired
  recipient email addresses. By default, you will receive daily and monthly
  reports. The columns below are included; for further customization, contact
  your Liftoff POC.
  * Date
  * OS
  * Bundle
  * Ad Unit
  * Ad Unit ID
  * Country
  * Rewarded
  * Size
  * Requests
  * Fills
  * Fill Rate
  * Impressions
  * Clicks
  * CTR
  * Revenue
  * eCPM

[latest-display-sdk]: https://github.com/liftoffio/LiftoffAds-Android/releases/download/v1.3.0/LiftoffAds-v1.3.0.zip
[latest-mopub]: https://github.com/liftoffio/LiftoffAds-Android/releases/download/mopub-v1.2.1/LiftoffMoPubAdapter-v1.2.1.zip
